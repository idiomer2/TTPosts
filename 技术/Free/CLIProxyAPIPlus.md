
```bash
# 下载配置文件
mkdir ~/cliproxyapiplus && cd ~/cliproxyapiplus
curl -o config.yaml https://raw.githubusercontent.com/router-for-me/CLIProxyAPIPlus/main/config.example.yaml
#下载完成后修改远程访问配置 vi config.yaml
#allow-remote: true
#secret-key: '你的访问密码'

# 拉取并运行镜像
docker pull eceasy/cli-proxy-api-plus:v6.8.49-0 && docker tag eceasy/cli-proxy-api-plus:v6.8.49-0 eceasy/cli-proxy-api-plus:latest  # docker pull eceasy/cli-proxy-api-plus:latest

docker run -d --name cli-proxy-api-plus --restart unless-stopped -p 8317:8317 -v "$(pwd)/config.yaml:/CLIProxyAPI/config.yaml" -v "$(pwd)/auths:/root/.cli-proxy-api" -v "$(pwd)/logs:/CLIProxyAPI/logs" eceasy/cli-proxy-api-plus:latest

```


## 参考资料

- [快速开始 | CLIProxyAPI](https://help.router-for.me/cn/introduction/quick-start.html)
- [eceasy/cli-proxy-api-plus - Docker Image](https://hub.docker.com/r/eceasy/cli-proxy-api-plus/tags)
- [最新OpenAI账号批量注册教程：轻松实现Codex自由~_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV14MPyz8EwJ/)
- [最新OpenAI账号批量注册教程：手把手教你实现Codex自由 | Z-TTS专业文本转语音工具 | Text to Speech](https://www.z-tts.cn/openai-batch-registration-codex-guide/)
