
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
	- subagent做为特殊工具Task被调用，subagent不能再使用Task形成递归 (真实场景视情况而定)
	- subagent也定义类型，避免执行不可控的任务。不同的subagent作用不一样，调用工具的权限也不一样。如explore、plan、code，前两者只能read-only，后者拥有完整的read和write工具权限
	- 构建subagent工具时，需要告诉模型，这个工具的type只能取值\[explore、plan、code\]，限制模型使用子agent的范围
	- 若模型选定了explore这个subagent后（tool=subagent，subagent_type=explore）,构建新LLM时，限定工具集范围
	  
4. **v4_skills_agent**：Knowledge Externalization. 知识加载，专业经验无需重训
	- skills做为特殊工具被调用




## 参考资料
- [Hello-Agents](https://datawhalechina.github.io/hello-agents/#/./README?id=%f0%9f%8e%af-%e9%a1%b9%e7%9b%ae%e4%bb%8b%e7%bb%8d)
- [shareAI-lab/learn-claude-code: Bash is all you need！write a claude code with only 16 line code](https://github.com/shareAI-lab/learn-claude-code)
- [1rgs/nanocode: Minimal Claude Code alternative. Single Python file, zero dependencies, ~250 lines.](https://github.com/1rgs/nanocode)
- 