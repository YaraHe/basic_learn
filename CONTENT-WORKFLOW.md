# 学习笔记内容维护

本目录连接原有 `YaraHe/basic_learn` 仓库，是个人主页学习笔记唯一正式维护目录。`articles/` 当前包含 RAG 16 篇、Agent AI 12 篇、Voice Agent 3 篇，共 31 篇 Markdown 和 186 张文章图片。已确认这些文章与课程截图可进入公开发布准备；提交前仍需处理任何明确发现的第三方许可或隐私问题。

网站从本目录的 `articles/` 读取 Markdown、Front Matter 和相对路径图片，生成目录卡片与文章阅读页，不手工维护第二份正文。文章阅读地址由类别目录与文件名决定。增改文章时保留现有 Front Matter 结构及图片相对路径，并在兄弟目录 `../YaraHe.github.io` 运行：

```powershell
npm run build
npm run check
```

在网站本地预览中复查对应目录、阅读页及图片。首次 GitHub Pages 发布计划由网站仓库的 GitHub Actions 检出本仓库并构建 `dist/`；工作流尚未配置或部署。未来笔记更新后，暂定手动触发网站重建，无需在两个目录之间复制 Markdown。

`articles/Voice-Agent-Portfolio-Content/02-Final-Articles.zip` 与已展开的 Voice Agent 文章及图片重复，作为本地材料保留，已由 `.gitignore` 排除。其他审查与导出过程材料不属于正式 `articles/` 内容源，也不应作为网站资源发布。
