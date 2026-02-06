
# 安装配置
## 安装
- 从 [Releases · router-for-me/CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI/releases) 下载 [CLIProxyAPI_6.7.53_windows_amd64.zip](https://github.com/router-for-me/CLIProxyAPI/releases/download/v6.7.53/CLIProxyAPI_6.7.53_windows_amd64.zip)
- 解压到任一目录即可（如：Z:\CLIProxyAPI）

## 基础配置

复制config.example.yaml文件成config.yaml，确认并修改如下配置
```yaml
port: 8317

auth-dir: "~/.cli-proxy-api"

request-retry: 3  
  
quota-exceeded:  
  switch-project: true  
  switch-preview-model: true

api-keys:
  - "sk-xxx1"
  - "sk-xxx2"

```


## OAuth授权
### 转换 Qwen Code 为 API Key
1. 在安装目录下，启动cmd终端，输入`cli-proxy-api --qwen-login`后回车。程序会自动打开浏览器，请在浏览器中登录你的 Qwen 账户并完成授权。
![qwen-code授权页面](https://img.072899.xyz/2025/09/ec2e867f5a8de1d24969cb5225cdaa9e.png)
2. 完成授权后，回到终端，程序会尝试获取认证信息。成功后，会要求输入邮箱或昵称（如图中红色箭头所示）。这只是一个用于标识账户的别名，可以随意填写。假设这里填的是 `qwen-example`。回车后，可以看到认证文件已成功生成，并保存到了配置文件 `auth-dir` 指定的位置。
![](https://img.072899.xyz/2025/09/8d37d893884910db77bd4498373a4922.png)

3. 如果系统没有自动弹出浏览器，请不必担心。手动复制终端中红框标出的网址，粘贴到浏览器中打开即可完成授权。
4. 启动代理服务：以上步骤完成了账户认证。现在，我们来正式启动代理服务。直接双击可执行文件 (cli-proxy-api.exe)。启动cmd窗口后保持不要关闭即可


# Cherry Studio中使用
- 设置  模型服务  添加   







## 参考资料
1. [手把手带你用上AI神器 - CLIProxyAPI（壹：项目介绍+Qwen实战） - 失落之地](https://lostip.de/blog/3452777210/)
2. [提供商 (Providers) - OpenCode 中文文档](https://www.opencodecn.com/docs/providers)
3. [OpenCode 自定义服务商（中转站）接入指南](https://linux.do/t/topic/1329050)
4. [CLIProxyAPI/README_CN.md at main · router-for-me/CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI/blob/main/README_CN.md)