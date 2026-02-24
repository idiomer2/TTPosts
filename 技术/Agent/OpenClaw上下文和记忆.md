
OpenClaw 是一个“文件优先（file‑first）”的本地 AI 代理框架。它不是靠一条长 prompt 把一切都塞进模型，而是：
- 把会话历史、工具、技能、项目配置等动态拼成**本轮 Context**；
- 把“真正要长期记住的东西”写到工作区的 Markdown 文件里（Memory），并做向量索引，以便后续检索和注入 Context。

因此：
- **Context 是“当前一轮调用时送给模型的大 Prompt”**，随每轮对话动态变化
	- Context = 当前这轮送进模型的一切内容（受 token 上限约束）
- **Memory 是“写在磁盘上的长期记忆”**，跨会话、跨渠道持久存在，通过检索选择性注入到 Context 中
	- Memory = 写在磁盘上的 Markdown 文件，是长期事实的唯一真实来源

## Context(上下文)




## Memory (记忆)

核心定义：文件是记忆的唯一真相。OpenClaw 文档的核心原话是：

> Memory is plain Markdown in the agent workspace. The files are the source of truth; the model only “remembers” what gets written to disk.

解释成三句话：
1. **记忆 = 工作区里的 Markdown 文件**​（主要是 MEMORY.md 和 memory/YYYY-MM-DD.md）。
2. 模型本身不会“魔法记忆”，只有写入磁盘的内容可以被索引和检索。
3. 检索到的记忆片段，会按需注入到下次 Context 中，从而“表现出”记住了。


### 记忆的层次结构

Memory 大体分两层：

1. **日记式记忆（Ephemeral / Daily Memory）**
    - 路径：`memory/YYYY-MM-DD.md`。
    - 作用：
        - 记录当天发生的对话和事实（类似“对话日志 + 笔记”）；
        - 内容可以比较嘈杂和长，作为语料库供检索用。
    - 加载策略：
        - 会话启动时，会优先读取**今天和昨天**两天的日志，作为背景。
2. **长期记忆（Long-term / Durable Memory）**
    - 文件：`MEMORY.md`（可选，但强烈推荐使用）；
    - 用途：
        - 保存“经过整理和筛选的长期事实和偏好”，比如：
            - 用户名、角色、公司信息；
            - 用户固定偏好（回答风格、使用语言、常用工具等）；
            - 项目重要长期决策（如架构选择、安全策略等）。
    - 加载策略：
        - 默认只在**主私有会话（main/private）**中加载；
        - 群组会话不会加载 MEMORY.md，以避免隐私泄露。

此外，还有两个与“记忆性信息”高度相关的重要文件：

- `USER.md`：用户偏好与工作风格（更偏向“系统设定+人物设定”）。
- `SOUL.md`：代理性格与行为规则（类似个性 prompt）。

它们虽然在严格意义下属于“Project Context 文件”，但在很多实践里，会作为“软记忆文件”长期保存偏好和设定。







## 参考
- https://dr.miromind.ai/share/b4ec1731-be28-4ebe-bc0e-e3d3e3f09ab9


