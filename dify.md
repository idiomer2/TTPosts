

```bash
git clone -b 1.11.1  https://github.com/langgenius/dify.git

cd dify/docker
cp .env.example .env
docker compose up -d

# 如果打开web界面遇到502，就等所有服务都正常后，重启nginx即可
docker compose restart nginx
```




## 参考资料
- [dify 部署 install界面转圈圈 502错误 - TIFOSI_Z - 博客园](https://www.cnblogs.com/aidaminiu/p/18739908)