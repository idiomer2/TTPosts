
## 镜像站

```json
{
  "registry-mirrors": [
    "https://xget.xi-xu.me/cr/ghcr",
    "https://docker.1panel.live",
    "https://docker.m.daocloud.io",
    "https://docker.1panelproxy.com"
  ]
}
```


## 账号登录和登出
```bash
# 登录
docker login -u xy-algo --password-stdin new-harbor.evkland.cn
docker login -u xy-algo -p Xy25! new-harbor.evkland.cn

# 查看已登录信息
cat ~/.docker/config.json

# 登出
docker logout new-harbor.evkland.cn
```






