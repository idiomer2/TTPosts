
## 安装

从GitHub release下载安装：[Releases · obsidianmd/obsidian-releases](https://github.com/obsidianmd/obsidian-releases/releases)

## 初步使用

1. 在github上创建一个仓库
2. 本地clone这个仓库
3. 新增.gitignore文件，忽略如下文件
4. 在obsidian打开本地仓库即可。后续新建的md文章都会在本地仓库生成对应md文件
5. git add + git commit + git push  一连套同步打包带走

```gitignore
.obsidian/workspace.json
.obsidian/workspace-mobile.json

```

**.obsidian/目录简介**
- `workspace.json`: 记录了当前工作区的状态。如打开了哪些笔记，当前的页面布局
	- 由于会被频繁修改，如果上传到github上，很容易造成冲突
- `app.json`: 记录了当前的app设置，比如app的版本，app的配置等
- `appearance.json`: 记录了当前的外观设置，比如主题，字体，颜色等
- `core-plugins.json`: 记录了当前的插件设置，比如插件的版本，插件的配置等
- `graph.json`: 记录了当前的图谱设置，比如图谱的类型，图谱的配置等



## 插件
pass

