
## 通过docker基础安装

1. github下载zip包并解压到D盘
2. 在docker desktop中的终端，cd到解压后的目录，并执行`docker build -t openclaw:local -f Dockerfile .`构建镜像
3. 执行`docker compose run --rm openclaw-cli onboard`完成快速基础设置
	1. 如有必要 (WSL)，提前建立需要用的目录`mkdir -p ~/.openclaw/workspace && chmod -R 777 ~/.openclaw`；
	2. 如有必要 (dd终端)，在非C盘提前建立`.openclaw/workspace`目录，然后执行run之前，设置好环境变量OPENCLAW_CONFIG_DIR和OPENCLAW_WORKSPACE_DIR分别指向`D:\.openclaw`和`D:.openclaw\workspace`
4. 快速基础设置详情
	- （yes） I understand this is powerful and inherently risky. Continue?
	- （QuickStart）Onboarding mode
	- （OpenRouter）Model/auth provider
	- （OpenRouter API key）OpenRouter auth method
	- （sk-or-v1-xxx）Enter OpenRouter API key
	- （openrouter/stepfun/step-3.5-flash:free）Default model
	- （Skip for now）Select channel (QuickStart)
	- （No）Configure skills now?
	- （No）Enable zsh shell completion for openclaw?
5. 执行`docker compose up -d openclaw-gateway` 启动gateway服务
6. 打开 http://127.0.0.1:18789 应该会显示如下报错
	1. disconnected (1008): unauthorized: gateway token mismatch (open the dashboard URL and paste the token in Control UI settings)
		- 解决方法1：手动打开`.openclaw/openclaw.json`，逐层找到`gateway.auth.token`，将token复制粘贴到网页的`Overview -> Gateway Access -> Gateway Token`中，点击`Connect`按钮
		- 解决方法2：`docker compose run --rm openclaw-cli config get gateway.auth.token`获取token
	2. disconnected (1008): pairing required
		- 解决方法1：手动打开`.openclaw/openclaw.json`，找到`gateway`，新增内容`"controlUi": { "dangerouslyDisableDeviceAuth": true },`也就是设置`gateway.controlUi.dangerouslyDisableDeviceAuth`为`true`
		- 解决方法2：`docker compose run --rm openclaw-cli config set gateway.controlUi.dangerouslyDisableDeviceAuth true`


## 常用命令

### / 命令

| 命令                      | 说明           |
| ----------------------- | ------------ |
| /help                   | 查看帮助         |
| /commands               | 查看所有/命令      |
|                         |              |
| /models                 | 查看已配置的提供商和模型 |
| /model status           | 查看当前模型状态     |
| /model <provider/model> | 切换模型         |
|                         |              |

### cli命令



## 配置第三方provider
- 参考官方文档：[模型提供商 - OpenClaw](https://docs.openclaw.ai/zh-CN/concepts/model-providers#%E6%9C%AC%E5%9C%B0%E4%BB%A3%E7%90%86%EF%BC%88lm-studio%E3%80%81vllm%E3%80%81litellm-%E7%AD%89%EF%BC%89)
- 如下为配置示例，正确配置后立刻生效，不用重启gateway
``` json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "fucaixie/kimi-k2"
      },
      "models": {
        "openrouter/auto": {"alias": "OpenRouter"},
        "openrouter/stepfun/step-3.5-flash:free": {},
        "fucaixie/kimi-k2": {"alias": "kimi-k2(opencode)"},
        "fucaixie/kimi-k2-0905": {"alias": "kimi-k2-0905(iflowcn)"},
        "fucaixie/deepseek-v3": {"alias": "deepseek-v3(iflowcn)"},
        "fucaixie/deepseek-v3.2": {"alias": "deepseek-v3.2(venice)"},
        "cliproxyapi/qwen3-coder-plus": {"alias": "qwen3-coder-plus(oauth)"},
        "cliproxyapi/qwen3-coder-flash": {"alias": "qwen3-coder-flash(oauth)"}
      }
  },
  "models": {
    "providers": {
      "fucaixie": {
        "baseUrl": "https://fucaixie.xyz/v1",
        "apiKey": "sk-xxxxxxxxxxxxxxxxxxxxxxxxxx",
        "api": "openai-completions",
        "models": [
          {"id": "kimi-k2", "name": "kimi-k2(opencode)", "contextWindow": 256000}, 
          {"id": "kimi-k2-0905", "name": "kimi-k2-0905(iflowcn)", "contextWindow": 256000}, 
          {"id": "deepseek-v3", "name":"deepseek-v3(iflowcn)", "contextWindow": 128000}, 
          {"id": "deepseek-v3.2", "name":"deepseek-v3.2(venice)", "contextWindow": 128000}
        ]
      },
      "cliproxyapi": {
        "baseUrl": "http://192.168.70.144:8317/v1",
        "apiKey": "sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "api": "openai-completions",
        "models": [
          {"id": "qwen3-coder-plus", "name": "qwen3-coder-plus(oauth)", "contextWindow": 128000}, 
          {"id": "qwen3-coder-flash", "name": "qwen3-coder-flash(oauth)", "contextWindow": 128000}
        ]
      }
    }
  },

}
```






## 参考资料
- [Docker - OpenClaw](https://docs.openclaw.ai/zh-CN/install/docker)
- [[Bug]: Docker install/onboarding stuck: CLI can’t connect to gateway (1006/1008), token mismatch due to OPENCLAW_GATEWAY_TOKEN override + confusing pairing flow/docs · Issue #9028 · openclaw/openclaw](https://github.com/openclaw/openclaw/issues/9028)
- [解决 Web UI 配对要求问题 | OpenClaw 中文社区 - 开源免费 AI 助手 | WhatsApp/Telegram/微信自动化](https://clawd.org.cn/gateway/pairing-required-troubleshooting)
- [[Bug]: Onboarding wizard does not install Systemd service on Ubuntu 22.04 · Issue #1818 · openclaw/openclaw](https://github.com/openclaw/openclaw/issues/1818)
- [OpenClaw 配置使用 API 教程](https://docs.n1n.ai/openclaw)
- [别再硬磕官方 API 了：OpenClaw 接入第三方中转 (Claude 4.5) 避坑指南 - 知乎](https://zhuanlan.zhihu.com/p/2003568032631513843)
- [模型提供商 - OpenClaw](https://docs.openclaw.ai/zh-CN/concepts/model-providers#%E6%9C%AC%E5%9C%B0%E4%BB%A3%E7%90%86%EF%BC%88lm-studio%E3%80%81vllm%E3%80%81litellm-%E7%AD%89%EF%BC%89)