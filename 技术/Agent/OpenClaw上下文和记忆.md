
OpenClaw 是一个“文件优先（file‑first）”的本地 AI 代理框架。它不是靠一条长 prompt 把一切都塞进模型，而是：
- 把会话历史、工具、技能、项目配置等动态拼成**本轮 Context**
- 把“真正要长期记住的东西”写到工作区的 Markdown 文件里（Memory），并做向量索引，以便后续检索和注入 Context

因此：
- **Context 是“当前一轮调用时送给模型的大 Prompt”**，随每轮对话动态变化
	- Context = 当前这轮送进模型的一切内容（受 token 上限约束）
- **Memory 是“写在磁盘上的长期记忆”**，跨会话、跨渠道持久存在，通过检索选择性注入到 Context 中
	- Memory = 写在磁盘上的 Markdown 文件，是长期事实的唯一真实来源

## 一、Context(上下文)

OpenClaw将动态组装的系统提示词，连同对话历史和当前用户消息一起做为上下文发送给大模型

%% 这种可组合的方法意味着你可以通过在工作区编辑文件来改变代理的行为、风格和任务能力，而无需触及源代码，同时执行、权限和沙箱功能由运行时强制执行 %%


### ⚙ System Prompt —— 系统提示词

1. Base System: 
	1. - Pi Agent Core — 来自agent runtime library的基础指令
2. Workspace Files: 
	1. AGENTS.md： 核心Agent指令（默认捆绑）。Agent的行为基准：Agent允许做什么、全局约束以及适用于所有会话的不可协商规则
	2. SOUL.md：Agent的性格设定与语气指导（可选）。语音和互动风格：指Agent如何说话和构建回答，但不包括工具行为或安全边界
	3. USER.md：个性化
	4. TOOLS.md：Tool使用指南。用户专用工具约定（可选）。你个人关于工具在你环境中如何使用的笔记，而不是工具登记册。OpenClaw 会自动为模型提供工具定义。
3. Dynamic Context:
	1. Skills：多个SKILL.md文件的元数据（name和describe）。`skills/<skill>/SKILL.md`文件包含结构化指南，用于使用现有工具完成特定任务，类似操作手册或标准操作经验
	2. Memory Serach: 语义检索结果（BM25和向量混合检索）。语义相似的过去对话，提供有用的背景
4. Tool Definitions: 
	1. Built-in tools: 内置工具，如bash、文件操作、浏览器、画布及核心功能
	2. Plugin tools: 插件工具，通过扩展系统添加的自定义工具（通过`api.registerTool(toolName, toolDefinition)`注册）

### 📃 Conversation Histroy —— 当前会话的历史消息


### 💬 User Message —— 当前用户消息





## 二、Memory (记忆)

核心定义：文件是记忆的唯一真相。OpenClaw 文档的核心原话是：

> Memory is plain Markdown in the agent workspace. The files are the source of truth; the model only “remembers” what gets written to disk.

解释成三句话：
1. **记忆 = 工作区里的 Markdown 文件**​（主要是 MEMORY.md 和 memory/YYYY-MM-DD.md）
2. 模型本身不会“魔法记忆”，只有写入磁盘的内容可以被索引和检索
3. 检索到的记忆片段，会按需注入到下次 Context 中，从而“表现出”记住了

![openclaw记忆架构](https://snowan.gitbook.io/study-notes/~gitbook/image?url=https%3A%2F%2F388701358-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-LmDY11BLFD0Iupj9U9t%252Fuploads%252Fgit-blob-09d2f21bf3aee37969856024c8ec6b740e04c29e%252Fopenclaw-memory.png%3Falt%3Dmedia&width=768&dpr=3&quality=100&sign=4419ffec&sv=2)

### 2.1 记忆的层次结构

Memory 大体分三层：

1. **会话记忆**
	- 路径：`sessions/YYYY-MM-DD-<slug>.md`
2. **日记式记忆（Ephemeral / Daily Memory）**—— 日常笔记、运行上下文
    - 路径：`memory/YYYY-MM-DD.md` （每日日志是仅附加的文件，每天创建一个新文件）
    - 作用：
        - 记录当天发生的对话和事实（类似“对话日志 + 笔记”）；
        - 内容可以比较嘈杂和长，作为语料库供检索用。
    - 加载策略：
        - 会话启动时，会优先读取**今天和昨天**两天的日志，作为背景
    - 例如：
	    - 今天讨论了如何部署到 AWS EC2
	    - 遇到一个 bug，目前尚未修复，日志见某路径
3. **长期记忆（Long-term / Durable Memory）**—— 决策、偏好、持久事实
    - 文件：`MEMORY.md`（可选，但强烈推荐使用）
    - 用途：
        - 保存“经过整理和筛选的长期事实和偏好”，比如：
            - 用户名、角色、公司信息
            - 用户固定偏好（回答风格、使用语言、常用工具等）
            - 项目重要长期决策（如架构选择、安全策略等）
    - 加载策略：
        - 默认只在 **主私有会话(main/private)** 中加载；
        - 群组会话不会加载 MEMORY.md，以避免隐私泄露。
    - 例如：
	    - 用户喜欢简短回答，代码要可直接复制执行
	    - 项目数据库统一选用 PostgreSQL，不再讨论 MySQL
	    - 所有部署默认走 staging 分支

此外，还有两个与“记忆性信息”高度相关的重要文件：

- `USER.md`：用户偏好与工作风格（更偏向“系统设定+人物设定”）
- `SOUL.md`：代理性格与行为规则（类似个性 prompt）

它们虽然在严格意义下属于“Project Context 文件”，但在很多实践里，会作为“软记忆文件”长期保存偏好和设定。



### 2.2 记忆写入机制

1. 用户说“记住这个”等类似的语义
	- 应当写入 Memory（其中一个文件），而不是只回复一句“好的我记住了”
	- 例如：
		- 用户：以后你回答技术问题的时候，尽量给出简短的 Bash 命令示例，少讲废话，也别加解释
		- 龙虾：在 USER.md 或 MEMORY.md 中写入类似 `偏好：回答技术问题时，只给简短 Bash 命令，尽量不要长篇解释`；然后向用户确认”已将该偏好写入 USER.md / MEMORY.md，以后会按此执行“
2. 自动Memory Flush（在上下文触发自动压缩前，防止压缩时丢关键事实）
	- 当对话越来越长、Context 接近模型上限时，如果直接做 compaction，可能会导致**一些还没写入 Memory 的关键信息被摘要化乃至遗失**。为此，OpenClaw 设置了一个“预压缩记忆刷新（pre‑compaction memory flush）”机制
	- 当估计的 token 数接近 `contextWindow - reserveTokensFloor - softThresholdTokens` 时：
		- 触发一个“静默回合”（通常不显示给用户）；
		- 系统 prompt 会指示代理：“当前会话即将压缩，请把现在还没写到 Memory、但未来可能需要的关键信息写入 memory/YYYY-MM-DD.md”
	- 每个压缩周期只触发一次 flush（在 sessions.json 中有标记）


#### ① memoryFlush的典型配置
```yaml
agents.defaults.compaction.memoryFlush:
  enabled: true
  softThresholdTokens: 4000
  reserveTokensFloor: 20000
  systemPrompt: "Session nearing compaction. Store durable memories now."
  prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store."
```


#### ② 记忆写入工具

在openclaw中，没有专门为记忆写入配置专门的工具，而是使用通用的 `WriteFile` 和 `EditFile` 工具


### 2.3 记忆读取机制

#### 2.3.1 配置型"软"记忆

像`USER.md`等配置文件，每次问答构建上下文时，都会被读取加入到system_prompt中

#### 2.3.2 历史记忆

结合关键词匹配BM25算法和向量化索引做混合搜索(hybrid search)，融合公式 = 关键词30% + 向量检索70%
- 上下文组装阶段
	- 对于 MEMORY.md（长期记忆/精选记忆），系统仅在**主/私有会话**（Main/Private Session）中加载，群组上下文（Group Contexts）通常不加载此文件以保护隐私
	- **自动注入 (默认行为)**：OpenClaw 会将`最近的对话历史` 和 `今天+昨天的每日日志（即 memory/YYYY-MM-DD.md）`注入到上下文窗口中
	- **插件干预 (Auto-Recall)**：如果你使用了如 memory-core 插件或 mem0、cognee 等第三方插件，系统会在此阶段执行**向量/语义搜索**。它会根据当前用户的问题，通过 memory_search 逻辑在后台检索 MEMORY.md 和 memory/*.md 中的相关片段，并将其作为“背景知识”自动注入
- 工具执行阶段
	- **智能决策**：在默认配置下，OpenClaw 的 Agent 拥有 memory_search 工具。当模型（LLM）发现当前上下文不足以回答问题时，它会在推理过程中**主动发起**工具调用请求
	- **搜索范围**：此时 memory_search 会对 MEMORY.md 和所有 memory/**/*.md 文件进行语义检索（Semantic Recall）
	- **返回结果**：工具会返回最相关的 Markdown 文本片段（通常是约 400 token 的块，带有 80 token 的重叠），模型随后利用这些新信息生成回答

##### ① 索引源

被纳入记忆索引的主要是：
- `MEMORY.md`
- `memory/**/*.md`（所有每日记忆文件）
变动检测：
- Monitor 文件变更
- 有更新时标记索引为“脏”（dirty）
- 1.5 秒防抖之后重建该部分索引（1.5秒内有新的变更，则重新计时）

##### ② 向量索引

- 索引数据存储在 `~/.openclaw/memory/<agentId>.sqlite` 中
- 嵌入模型的选择顺序通常是：
	- 有本地嵌入模型（local.modelPath） → 用本地
	- 否则有 OpenAI Key → 用 OpenAI Embedding
	- 否则有 Gemini Key → 用 Gemini
	- 否则关闭向量记忆（只靠简单关键字）
- 有的部署(新版本)会用 QMD（自研向量引擎）作为 backend，配置 `memory.backend = "qmd"` 即可切换；失败时回退到内置 SQLite 管理器

##### ③ 记忆读取工具

OpenClaw 给代理提供了一些“内部工具”来管理 memory：
- `memory_search`：根据当前需求，在索引里做语义检索，找到相关片段
- `memory_get`：根据文件路径和行范围，获取某一段原文（如果文件不存在，则返回空文本，不抛错）
Agent代理会通过这些工具，在构建 Context 时（？？？）决定要不要把某段记忆插入进来




## 参考
- https://dr.miromind.ai/share/b4ec1731-be28-4ebe-bc0e-e3d3e3f09ab9
- [OpenClaw 架构解析：其工作原理](https://ppaolo.substack.com/p/openclaw-system-architecture-overview)


