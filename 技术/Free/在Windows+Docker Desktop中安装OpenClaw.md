

1. github下载zip包并解压到D盘
2. 在docker desktop中的终端，cd到解压后的目录，并执行`docker build -t openclaw:local -f Dockerfile .`构建镜像
3. 执行`docker compose run --rm openclaw-cli onboard`完成快速基础设置
	1. 如有必要 (WSL)，提前建立需要用的目录`mkdir -p ~/.openclaw/workspace && chmod -R 777 ~/.openclaw`；
	2. 如有必要 (dd终端)，在非C盘提前建立`.openclaw/workspace`目录，然后执行run之前，设置好环境变量OPENCLAW_CONFIG_DIR和OPENCLAW_WORKSPACE_DIR分别指向`D:\.openclaw`和`D:.openclaw\workspace`
4. 快速基础设置详情
	1. 
5. 执行`docker compose up -d openclaw-gateway` 启动gateway服务
6. 打开 http://127.0.0.1:18789 应该会显示如下报错
	1. disconnected (1008): unauthorized: gateway token mismatch (open the dashboard URL and paste the token in Control UI settings)
		- 解决方法：手动打开`.openclaw/openclaw.json`，逐层找到`gateway.auth.token`，将token复制粘贴到网页的`Overview -> Gateway Access -> Gateway Token`中，点击`Connect`按钮
	2. disconnected (1008): pairing required
		- 解决方法：手动打开`.openclaw/openclaw.json`，找到`gateway`，新增内容`"controlUi": { "dangerouslyDisableDeviceAuth": true },`也就是设置`gateway.controlUi.dangerouslyDisableDeviceAuth`为`true`
