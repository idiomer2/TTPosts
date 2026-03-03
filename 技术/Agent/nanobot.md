## 安装

```bash
conda create -n py312nanobot python=3.12
conda activate py312nanobot
pip install nanobot-ai

nanobot onboard  # 生成配置文件 ~/.nanobot/config.json
```


## 基础配置

- `vi ~/.nanobot/config.json`

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

- 测试：`nanobot agent -m "你好"`

## 将搜索改为Tavily

- `vi ~/.conda/envs/py312nanobot/lib/python3.12/site-packages/nanobot/agent/tools/web.py`

```python
class WebSearchTool(Tool):
    """Tavily 联网搜索工具，支持轮询多个api_key"""
    name = "web_search"
    description = "使用 Tavily 搜索引擎进行联网搜索，获取最新的网络信息"
    parameters = {
        "type": "object",
        "properties": {
            "query": {"type": "string", "description": "Search query"},
            "count": {"type": "integer", "description": "Results (1-10, default 5)", "default": 5, "minimum": 1, "maximum": 10}
        },
        "required": ["query"]
    }
    _api_key_iterator = None
    def __init__(self, api_key: str | None = None, max_results: int = 5):
        self.max_results = max_results
        self._init_api_key = api_key
    @property
    def api_key(self):
        # 支持英文逗号分割，轮询返回下一个 key
        if self._api_key_iterator is None:
            import random, itertools
            api_keys = self._init_api_key or os.getenv("TAVILY_API_KEY", "")
            keys = [k.strip() for k in api_keys.split(",") if k.strip()]; random.shuffle(keys)
            if not keys:
                return None
            self._api_key_iterator = itertools.cycle(keys)
        return next(self._api_key_iterator)
    async def execute(self, query: str, count: int = 5) -> str:
        api_key = self.api_key
        if not api_key:
            return "Error: 请设置 TAVILY_API_KEY 环境变量"        try:
            async with httpx.AsyncClient(timeout=10) as client:
                resp = await client.post(
                    "https://api.tavily.com/search",
                    json={
                        "api_key": api_key,
                        "query": query,
                        "max_results": count or self.max_results or 5,
                        "include_answer": False,
                        "include_raw_content": False
                    }
                )
                resp.raise_for_status()
                data = resp.json()
            # 格式化搜索结果
            results = []
            # 如果有 AI 生成的答案，先展示
            if data.get("answer"):
                results.append(f"📝 AI 摘要: {data['answer']}\n")
            results.append("🔍 搜索结果:")
            for i, item in enumerate(data.get("results", []), 1):
                title = item.get("title", "无标题")
                url = item.get("url", "")
                content = item.get("content", "")[:3000]  # 截断内容
                results.append(f"\n{i}. {title}\n   URL: {url}\n   {content}")
            return "\n".join(results) if results else "未找到相关结果"
        except httpx.HTTPStatusError as e:
            return f"Error: API 请求失败 - {e.response.status_code}"
        except Exception as e:
            return f"Error: {e}"
```

- `vi ~/.nanobot/config.json`
```json
{
  "tools": {
    "web": {
      "search": {
        "apiKey": "tvly-dev-xxxxxxxxxxx",
        "maxResults": 5
      }
    }
  }
}
```

- 测试`nanobot agent -m "查询下当前的btc价格"`


## 接入飞书
1. 修改配置文件`~/.nanobot/config.json`。其中allowFrom是你的飞书open_id，或改为"\*"；appid和appsecret去[飞书开放平台](https://open.feishu.cn/app/)创建
	1. 创建一个应用，开启Bot能力
	2. 添加权限**Permissions**：添加`im:message` (发消息) 和 `im:message.p2p_msg:readonly` (接收消息)
	3. 添加事件**Events**：添加 `im.message.receive_v1` (接收消息)。并选择**Long Connection** 模式 (requires running nanobot first to establish connection)
	4. Get **App ID** and **App Secret** from "Credentials & Basic Info"
	5. 发布app
```
    "feishu": {
      "enabled": true,
      "appId": "cli_xxx",
      "appSecret": "xxxx",
      "encryptKey": "",
      "verificationToken": "",
      "allowFrom": ["ou_xxxx"],
      "reactEmoji": "THUMBSUP"
    }
```

2. `nanobot gateway` 启动网关服务
3. 飞书上发消息给机器人即可 （gateway日志中有ou_xxxx）