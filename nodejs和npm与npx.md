
## 一、安装

- 通用安装：[Node.js — 下载 Node.js®](https://nodejs.org/zh-cn/download)
	- windows记得将bin目录添加到环境变量
- Linux命令安装：`apt install nodejs npm` 或 `yum install nodejs npm`

安装好后，执行 `node -v`查看版本验证安装成功

### ① Windows 默认路径
```bash
# 全局安装包位置
C:\Users\用户名\AppData\Roaming\npm\node_modules

# npm/npx 可执行文件
C:\Users\用户名\AppData\Roaming\npm\npm.cmd
C:\Users\用户名\AppData\Roaming\npm\npx.cmd

# npm 缓存
C:\Users\用户名\AppData\Roaming\npm-cache

# Node.js 安装目录（如果通过安装包安装）
C:\Program Files\nodejs
```

### ② MacOS/Linux 默认路径
``` shell
# 全局安装包位置（使用系统Node.js）
/usr/local/lib/node_modules

# 使用nvm安装时的路径
~/.nvm/versions/node/[版本号]/lib/node_modules

# npm/npx 可执行文件
/usr/local/bin/npm
/usr/local/bin/npx

# 使用nvm时的可执行文件
~/.nvm/versions/node/[版本号]/bin/npm
~/.nvm/versions/node/[版本号]/bin/npx

# npm 缓存
~/.npm
```

## 二、基础使用

### ① 查看当前配置

```bash
# 查看npm配置
npm config list

# 查看所有配置（包括默认值）
npm config ls -l

# 查看全局安装路径
npm root -g

# 查看npm和node位置
which npm   # Linux/Mac
where npm   # Windows
which node
where node
```

### ② 修改默认路径

```bash
# 查看全局包安装位置
npm root -g

# 修改全局安装路径
npm config set prefix "新路径"

# 修改缓存路径
npm config set cache "新缓存路径"

# 示例：在项目中使用特定路径
npm config set prefix ./node_modules/.global
```

### 其它命令

``` bash
# 清理缓存
npm cache clean --force

# 查看已安装的全局包
npm list -g --depth=0
```

## 三、npm安装/npx执行

```bash
# 使用npm安装包
npm install lodash           # 本地安装
npm install -g typescript    # 全局安装

# 使用npx临时执行
npx create-react-app my-app  # 无需全局安装create-react-app
npx eslint index.js         # 临时使用eslint检查
npx cowsay "Hello!"         # 运行未安装的包

# 指定版本运行
npx node@14 myscript.js
```

