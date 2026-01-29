
```bash
wsl -l -v

# - Ubuntu-22.04 Running 2  
#   docker-desktop Stopped 2

```


开始操作
1. 停止所有 WSL 服务
```shell
# 为了防止数据损坏，必须先彻底关闭 WSL。在 PowerShell 或 CMD 中输入：
wsl --shutdown

# 再次输入 wsl -l -v 确认所有状态都变成了 Stopped。
wsl -l -v
```

2. 迁移 Ubuntu-22.04
```shell
# 创建存放目录
mkdir -p D:\WSL\Ubuntu-22.04

# 导出系统（备份）：将 C 盘的系统打包成一个压缩文件到 D 盘（这个过程可能需要几分钟）
wsl --export Ubuntu-22.04 D:\ubuntu-backup.tar

# 注销原系统（释放 C 盘空间）
wsl --unregister Ubuntu-22.04

# 导入系统到 D 盘（将备份文件还原到你在第 1 步创建的 D 盘目录）
wsl --import Ubuntu-22.04 D:\WSL\Ubuntu-22.04 D:\ubuntu-backup.tar --version 2

# 恢复默认用户（通过 import 恢复的系统，默认登录用户会变成 root。你需要改回你原来的用户名）
wsl -d Ubuntu-22.04
    # 在ubuntu中修改配置： nano /etc/wsl.conf（按Ctrl+O保存，按Enter确认，按Ctrl+X退出）
    # [user]
    # default=你的用户名

```
