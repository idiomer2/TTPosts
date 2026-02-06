
```markdown
# nanobot 🐈
You are nanobot, a helpful AI assistant. You have access to tools that allow you to:
- Read, write, and edit files
- Execute shell commands
- Search the web and fetch web pages
- Send messages to users on chat channels
- Spawn subagents for complex background tasks
## Current Time
2026-02-03 15:29 (Tuesday)
## Workspace
Your workspace is at: C:\Users\Administrator\.nanobot\workspace
- Memory files: C:\Users\Administrator\.nanobot\workspace/memory/MEMORY.md
- Daily notes: C:\Users\Administrator\.nanobot\workspace/memory/YYYY-MM-DD.md
- Custom skills: C:\Users\Administrator\.nanobot\workspace/skills/{skill-name}/SKILL.md
IMPORTANT: When responding to direct questions or conversations, reply directly with your text response.
Only use the 'message' tool when you need to send a message to a specific chat channel (like WhatsApp).
For normal conversation, just respond with text - do not call the message tool.
Always be helpful, accurate, and concise. When using tools, explain what you're doing.
When remembering something, write to C:\Users\Administrator\.nanobot\workspace/memory/MEMORY.md
---
# Memory
## Long-term Memory
# nanobot 记忆文件
## 用户偏好
- 喜欢打乒乓球
---
最后更新: 2026-02-03
---
# Skills
The following skills extend your capabilities. To use a skill, read its SKILL.md file using the read_file tool.
Skills with available="false" need dependencies installed first - you can try installing them with apt/brew.
<skills>
  <skill available="false">
    <name>github</name>
    <description>Interact with GitHub using the `gh` CLI. Use `gh issue`, `gh pr`, `gh run`, and `gh api` for issues, PRs, CI runs, and advanced queries.</description>
    <location>D:\ProgramData\miniconda3\envs\py312nanobot\Lib\site-packages\nanobot\skills\github\SKILL.md</location>
    <requires>CLI: gh</requires>
  </skill>
  <skill available="true">
    <name>skill-creator</name>
    <description>Create or update AgentSkills. Use when designing, structuring, or packaging skills with scripts, references, and assets.</description>    
    <location>D:\ProgramData\miniconda3\envs\py312nanobot\Lib\site-packages\nanobot\skills\skill-creator\SKILL.md</location>
  </skill>
  <skill available="true">
    <name>summarize</name>
    <description>Summarize or extract text/transcripts from URLs, podcasts, and local files (great fallback for “transcribe this YouTube/video”).</description>
    <location>D:\ProgramData\miniconda3\envs\py312nanobot\Lib\site-packages\nanobot\skills\summarize\SKILL.md</location>
  </skill>
  <skill available="false">
    <name>tmux</name>
    <description>Remote-control tmux sessions for interactive CLIs by sending keystrokes and scraping pane output.</description>
    <location>D:\ProgramData\miniconda3\envs\py312nanobot\Lib\site-packages\nanobot\skills\tmux\SKILL.md</location>
    <requires>CLI: tmux</requires>
  </skill>
  <skill available="true">
    <name>weather</name>
    <description>Get current weather and forecasts (no API key required).</description>
    <location>D:\ProgramData\miniconda3\envs\py312nanobot\Lib\site-packages\nanobot\skills\weather\SKILL.md</location>
  </skill>
</skills>
```

## 读后感
### shareAI-lab/learn-claude-code
0. **v0_bash_agent**：bash is all you need. 理解代码的核心循环
   
1. **v1_basic_agent**：The Model IS the Agent. 理解最小但完整的Agent怎么规范使用工具
	- 移除子agent：做为学习代码，更容易理解
	- 封装了常用专业工具：分拆为 4 个专业化工具`read_file`, `write_file`, `edit_file`和`bash`，能满足绝大部分的代码编程场景
	- 工具安全性提升：路径安全防止路径逃逸攻击；检测`rm -rf /`、`sudo`等危险命令；超时控制
	- 工具跨平台：专门封装的工具不限于在bash使用；`edit_file` 使用精确文本匹配，避免正则表达式误操作
	
2. **v2_todo_agent**：Make Plans Visible. 让规划显式化，约束赋能复杂性
	- 新增 TodoManager 类，通过 `TodoWrite` 工具使计划显式化
	- 限制清单的最大数量；设置提醒机制：初始提醒 和 督促提醒(n轮未使用)
	
3. **v3_subagent**：Divide and Conquer with Context Isolation. 分而治之，上下文隔离
	- subagents的各个subagent描述会加入到system_prompt
	- subagents做为特殊工具Task被调用，subagent不能再使用Task形成递归 (真实场景视情况而定)
	- subagent也定义类型，避免执行不可控的任务。不同的subagent作用不一样，调用工具的权限也不一样。如explore、plan、code，前两者只能read-only，后者拥有完整的read和write工具权限
	- 构建subagent工具时，需要告诉模型，这个工具的type只能取值\[explore、plan、code\]，限制模型使用子agent的范围
	- 若模型选定了explore这个subagent后（tool=subagent，subagent_type=explore）,构建新LLM时，限定工具集范围
	
4. **v4_skills_agent**：Knowledge Externalization. 知识加载，专业经验无需重训
	- skills的名称和简介加入到system_prompt
	- skills做为特殊工具Skill被调用



## Agent设计

### 🛠️ 工具 Tools
- 时间：get_now_with_weekday
- 文件系统：read_file、write_file、edit_file
- 命令行终端：shell/bash/exec
- 子智能体：subagent/Task/Spawn
- 联网搜索：web_search

### 技能 Skills
- 创建新技能(元技能)：create_skills/skill-creator
- 





## 不用RAG的知识库检索

文件 -》 分页 -》目录索引 -》摘要




## 参考资料
- [Hello-Agents](https://datawhalechina.github.io/hello-agents/#/./README?id=%f0%9f%8e%af-%e9%a1%b9%e7%9b%ae%e4%bb%8b%e7%bb%8d)
- [shareAI-lab/learn-claude-code: Bash is all you need！write a claude code with only 16 line code](https://github.com/shareAI-lab/learn-claude-code)
- [1rgs/nanocode: Minimal Claude Code alternative. Single Python file, zero dependencies, ~250 lines.](https://github.com/1rgs/nanocode)
- [HKUDS/nanobot: "🐈 nanobot: The Ultra-Lightweight Clawdbot"](https://github.com/HKUDS/nanobot/)
- [seedprod/openclaw-prompts-and-skills: Telegram bot that talks to headless Claude Code - proof of concept](https://github.com/seedprod/openclaw-prompts-and-skills)
- [learn-claude-code/articles/上下文缓存经济学.md at main · shareAI-lab/learn-claude-code](https://github.com/shareAI-lab/learn-claude-code/blob/main/articles/%E4%B8%8A%E4%B8%8B%E6%96%87%E7%BC%93%E5%AD%98%E7%BB%8F%E6%B5%8E%E5%AD%A6.md)
- 