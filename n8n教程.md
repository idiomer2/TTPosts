
## 通过docker安装
```bash
docker pull n8nio/n8n  # docker.n8n.io/n8nio/n8n

docker volume create n8n_data

docker run -it --rm --name n8n -p 5678:5678 -e GENERIC_TIMEZONE="Asia/Shanghai" -e TZ="Asia/Shanghai" -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true -e N8N_RUNNERS_ENABLED=true -v n8n_data:/home/node/.n8n n8nio/n8n
```

**参数说明**
- **N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS**：在 n8n 部署中，如果配置文件 /home/node/.n8n/config 权限过宽（如 0644），系统会提示未来版本将自动修正权限。通过设置 N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true，可以在启动时强制修正为安全权限，避免潜在的安全风险。
- **N8N_RUNNERS_ENABLED**：是否启用任务运行器




## 汉化

1. 下载汉化包：[Releases · other-blowsnow/n8n-i18n-chinese](https://github.com/other-blowsnow/n8n-i18n-chinese/releases)
2. 解压后，将解压后的dist目录映射到容器内 `-e N8N_DEFAULT_LOCALE=zh-CN -e N8N_SECURE_COOKIE=false -v 解压后的地址:/usr/local/lib/node_modules/n8n/node_modules/n8n-editor-ui/dist`
3. 如果是Linux系统，记得`chmod 777 解压后的地址`

- **完整启动命令**
``` bash
docker run -it --rm --name n8n -p 5678:5678 -e GENERIC_TIMEZONE="Asia/Shanghai" -e TZ="Asia/Shanghai" -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true -e N8N_RUNNERS_ENABLED=true -e N8N_DEFAULT_LOCALE=zh-CN -e N8N_SECURE_COOKIE=false -v D:\Volumes\n8n\editor-ui\dist:/usr/local/lib/node_modules/n8n/node_modules/n8n-editor-ui/dist -v n8n_data:/home/node/.n8n n8nio/n8n
```






## 参考
- [Docker |n8n 文档](https://docs.n8n.io/hosting/installation/docker/#starting-n8n)
- [5分钟搞定！n8n 本地部署 + 汉化教程，一次到位 - 知乎](https://zhuanlan.zhihu.com/p/1937586953865372387)
- 