
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
5. 管理页面：http://127.0.0.1:8317/management.html


# 客户端中使用
## Cherry Studio中使用
- 设置 --> 模型服务  --> 添加   
- 模型提供商类型可以选择除 Azure 之外的任意类型。如OpenAI
- API密钥：填写我们在 config.yaml 中自己设置的 Key，本例中为 sk-xxx1
- API地址：填写我们本地服务的地址和端口。还记得配置文件中的端口号 8317 吗？这里我们填入 http://127.0.0.1:8317
- 点击“管理模型”，你就可以看到通过代理加载的 Qwen Code 模型了。
- - 添加模型后，就可以对话测试一下

## OpenCode中使用

1. 初始化自定义服务商：`opencode auth login`
	1. 请填写一个唯一标识名，如CLIProxyAPI
	2. 接着输入 API Key
2. 配置中转站 API 地址
	1. 打开 OpenCode 配置目录 ~/.config/opencode/
	2. 在该目录下创建或编辑配置文件`opencode.json`，内容如下
```json
{
    "$schema": "https://opencode.ai/config.json",
    "provider": {
        "cliproxyapi": {  // 这里必须和上一步的提供商 ID 完全一致！
            "npm": "@ai-sdk/openai-compatible",
            "name": "CLIProxyAPI",  // 在 UI 中显示的名称，可自定义
            "options": {
	            "baseURL": "http://127.0.0.1:8317/v1",  // 你的中转站 API 地址（必须以 /v1 结尾或符合 OpenAI 格式）
	            "apiKey": "sk-xxx1"  // 可选："{cred:myproxy}"自动引用上一步存储的密钥（推荐，不用明文写 key）
	            // 如果中转站需要自定义 headers，可添加：
	            // "headers": {
	            //   "X-Custom-Header": "your-value"
	            // }
	            },
	            "models": {
	            "qwen3-coder-plus": {  // 中转站支持的模型 ID，例如 gpt-4o、claude-3-5-sonnet 
	                "name": "qwen3-coder-plus"
	            },
	            "qwen3-coder-flash": {
	                "name": "qwen3-coder-flash"
	            },
	            "vision-model": {
	                "name": "vision-model"
	            }
	            // 添加更多模型...
            }

        }

    }

}
```








## 参考资料
1. [手把手带你用上AI神器 - CLIProxyAPI（壹：项目介绍+Qwen实战） - 失落之地](https://lostip.de/blog/3452777210/)
2. [提供商 (Providers) - OpenCode 中文文档](https://www.opencodecn.com/docs/providers)
3. [OpenCode 自定义服务商（中转站）接入指南](https://linux.do/t/topic/1329050)
4. [CLIProxyAPI/README_CN.md at main · router-for-me/CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI/blob/main/README_CN.md)