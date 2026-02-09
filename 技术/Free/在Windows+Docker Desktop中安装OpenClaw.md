

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



## 参考资料
- [[Bug]: Docker install/onboarding stuck: CLI can’t connect to gateway (1006/1008), token mismatch due to OPENCLAW_GATEWAY_TOKEN override + confusing pairing flow/docs · Issue #9028 · openclaw/openclaw](https://github.com/openclaw/openclaw/issues/9028)
- [解决 Web UI 配对要求问题 | OpenClaw 中文社区 - 开源免费 AI 助手 | WhatsApp/Telegram/微信自动化](https://clawd.org.cn/gateway/pairing-required-troubleshooting)
- [[Bug]: Onboarding wizard does not install Systemd service on Ubuntu 22.04 · Issue #1818 · openclaw/openclaw](https://github.com/openclaw/openclaw/issues/1818)