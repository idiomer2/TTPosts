## 安装

```bash
conda create -n py312nanobot python=3.12
conda activate py312nanobot
pip install nanobot-ai

nanobot onboard  # 生成配置文件 ~/.nanobot/config.json
```


## 基础配置

`vi ~/.nanobot/config.json`

``` json
{
  "agents": {
    "defaults": {
      "workspace": "~/.nanobot/workspace",
      "model": "deepseek-chat",
      "provider": "deepseek"
    }
  },
  ...
  {
    "providers": {
      "deepseek": {
        "apiKey": "sk-xxx",
      "apiBase": null,
      "extraHeaders": null
    }
  },
  ...
}

```


## 将搜索改为Tavily

