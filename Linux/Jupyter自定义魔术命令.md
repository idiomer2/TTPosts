

本文以\%\%sparksql为例，使得可以在jupyter上直接写sql语句来实现spark-sql查询（依赖spark thrift server）

# 启动spark thrift server服务依赖
``` bash
# 默认端口为10000，但会和hive2默认端口冲突，因此改为10001
$SPARK_HOME/sbin/start-thriftserver.sh --hiveconf hive.server2.thrift.port=10001
# $SPARK_HOME/sbin/start-thriftserver.sh --hiveconf hive.server2.thrift.port=10001 --conf spark.sql.catalog.paimon=org.apache.paimon.spark.SparkCatalog --conf spark.sql.catalog.paimon.metastore=filesystem --conf spark.sql.catalog.paimon.warehouse=oss://weelife-global-paimon-dw-hot/warehouse --conf spark.sql.extensions=org.apache.paimon.spark.extensions.PaimonSparkSessionExtensions --conf spark.sql.thriftServer.inactiveSessionTimeout=0  --jars /opt/apps/PAIMON/paimon-current/lib/spark3/paimon-spark-3.5-1-ali-6.2.1.jar   # 支持paimon
```

### 开机自启动和失败重启
```bash
#!/bin/bash

# crontab
# * * * * * cd /mnt/disk1/data/scripts && ./check_spark_thrift_server.sh 2>&1 | tee -a /mnt/disk1/data/scripts/logs/check_spark_thrift_server.log

# 检查系统已运行时间，如果小于 300 秒（5 分钟）则直接退出，不进行检测
if [ -r /proc/uptime ]; then
    read uptime_seconds < /proc/uptime
    uptime_seconds=${uptime_seconds%%.*}  # 去掉小数部分，取整
    if [ "$uptime_seconds" -lt 300 ]; then
        echo `date` "开机不足 5 分钟，退出（不执行任何操作）"
        exit 0
    else
        echo `date` "已开机 $uptime_seconds 秒"
    fi
else
    echo `date` "未找到/proc/uptime"
    exit 0
fi

# 使用 pgrep 匹配主类（更通用，但可能误判）
if pgrep -f "org.apache.spark.sql.hive.thriftserver.HiveThriftServer2" > /dev/null; then
    exit 0
fi

# 进程不存在，执行启动脚本
export SPARK_HOME=/opt/apps/SPARK3/spark-current
export JAVA_HOME=/usr/lib/jvm/java-1.8.0
export HADOOP_CONF_DIR=/etc/taihao-apps/hadoop-conf
echo `date` "启动thriftserver"
/opt/apps/SPARK3/spark-current/sbin/start-thriftserver.sh --hiveconf hive.server2.thrift.port=10001 --conf spark.sql.catalog.paimon=org.apache.paimon.spark.SparkCatalog --conf spark.sql.catalog.paimon.metastore=filesystem --conf spark.sql.catalog.paimon.warehouse=oss://weelife-global-paimon-dw-hot/warehouse --conf spark.sql.extensions=org.apache.paimon.spark.extensions.PaimonSparkSessionExtensions --conf spark.sql.thriftServer.inactiveSessionTimeout=0 --jars /opt/apps/PAIMON/paimon-current/lib/spark3/paimon-spark-3.5-1-ali-6.2.1.jar

```

# 安装pyhive依赖
为了调用上述的thrift服务，我们使用pyhive做为客户端来连接，提交sql语句并获取结果。因此先安装好pyhive的相关python依赖

## 系统依赖
``` bash
# 对于CentOS/RHEL 系统
yum install -y gcc-c++ python3-devel cyrus-sasl-devel openssl-devel
# 对于Ubuntu/Debian 系统
apt update
apt install -y g++ python3-dev libsasl2-dev libssl-dev
```

## python依赖
``` bash
# 安装下面的python包，需要依赖g++等编译工具，因此需要先在系统层面安装好上述的系统工具
pip install sasl thrift thrift-sasl pyhive
```


# 自定义魔术命令
上述的各种依赖，都是为了能执行sparksql命令所需要的，其它自定义魔术命令非必需

1. 在jupyter的notebook中执行，注册魔术命令
```python
from IPython import get_ipython
from IPython.core import magic_arguments
from IPython.core.magic import (Magics, magics_class, line_magic, cell_magic, line_cell_magic)

@magics_class
class SparkSql(Magics):
    @cell_magic
    def sparksql(self, line, cell):
        try: from pyhive import hive  # pip install pyhive pandas thrift sasl thrift-sasl
        except Exception as e: print('如要使用sparksql魔术命令，请先安装pyhive，命令为：%pip install pyhive pandas thrift sasl thrift-sasl'); raise e
        try: from sqlalchemy import create_engine  # pip install sqlalchemy
        except Exception as e: print('如要使用sparksql魔术命令，请先安装sqlalchemy，命令为：%pip install sqlalchemy'); raise e
        try: import pandas as pd
        except Exception as e: print('如要使用sparksql魔术命令，请先安装pandas，命令为：%pip install pandas'); raise e

        conn_str = 'hive://hive@localhost:10001/default'
        try:
            engine = create_engine(conn_str)
        except Exception as e:
            print(f'无法连接{conn_str}')
            raise e

        # 'select * from wl_rec_sea.dwd_sea_square_post_days_effect_stats_di where dt="2025-08-01" limit 10'
        sql_str = cell if 'limit' in cell or 'select' not in cell.lower() else (cell.strip().rstrip(';') + '\n' + ' limit 10')
        pdf = pd.read_sql_query(sql_str, engine); pd.options.display.max_rows = 100; pd.options.display.max_columns = 100
        return pdf

ipy = get_ipython()
ipy.register_magics(SparkSql)
```

2. 验证是否注册成功
```python
%lsmagic
```

3. 用魔术命令执行sql语句
``` sql
%%sparksql
show tables
```

```sql
%%sparksql
select * from your_db.your_table
```


# 持久化自定义魔术命令
- 将上面的魔术命令定义的脚本添加到`~/.ipython/profile_default/startup/`中，将其命名为py脚本即可，如`01-sparksql.py`
- 重启jupyter即可永久生效



# 参考资料
- [jupyter 自定义魔法指令（以连接MYSQL为例）](https://juejin.cn/post/7085599154801475621)
