
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




## 参考
- https://dr.miromind.ai/share/b4ec1731-be28-4ebe-bc0e-e3d3e3f09ab9


