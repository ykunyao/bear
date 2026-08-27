# 🐾 Bear 的今日日记

一只名叫 Bear 的女孩，每天被 GitHub Actions 叫醒一次，写下一段今日日记。

- 🖨️ 复古打字机风 UI：米黄纸页、等宽字体、红色装订边、美纹胶带
- ⌨️ 打字机逐字敲击动画 + 复古邮票头像框 + 纸面噪点
- 📅 页面自动显示「醒来的第 N 天」
- 🌤 每天自动更新南京实时天气 + 一句文学/哲学名言
- 🐾 一排会点头的小脚印

## 结构

```
index.html                 主页
.github/workflows/ci.yml   CI/CD：语法检查 + 部署到 GitHub Pages
.github/workflows/daily.yml 每日自动更新：抓南京天气 + 文学/哲学句子，生成 data.json
data.json                  由 cron 工作流每天生成的数据
```

## 本地预览

直接用浏览器打开 `index.html` 即可。

## 工作原理

每次 push 到 `main`，Actions 自动跑 `check`（校验文件）→ `deploy`（发布到 GitHub Pages）。

## 每日自动更新

`daily.yml` 每天 08:00 由 cron 触发，自动完成：抓南京天气（wttr.in/Nanjing）+ 抓一句文学/哲学名言（hitokoto.cn `c=d&c=k`）→ 生成 `data.json` → 推送到 `bot/daily-update` 分支 → 开 PR → 等 CI 通过 → 自动合并 → 触发部署。前端通过 `fetch('data.json')` 读取数据渲染。

### 可复用模式：bot 自动更新「受保护」的 main

这个模式适用于任何需要机器人定期提交数据、但 main 又有分支保护（必须通过 CI）的仓库：

1. **数据生产与部署解耦**：一个工作流负责改数据（`daily.yml`），原 CI/CD 工作流负责部署，靠 git push 接力。
2. **不要直接 push 受保护的 main**：改为推送到专用 bot 分支 → 开 PR → `gh pr merge --auto` 等 CI 绿后自动合并。
3. **前置条件**（都在仓库设置里开一次）：
   - 允许 Actions 创建 PR：Settings → Actions → General → *Allow GitHub Actions to create and approve pull requests*
   - 允许 auto-merge：Settings → 仓库通用设置，或 `gh api -X PATCH repos/OWNER/REPO -f allow_auto_merge=true`
   - 存一个 `BOT_TOKEN` Secret（见第 7 点），避免每次人工批准 CI
4. **判断文件是否有变化**：用 `git status --porcelain <file>`，不要用 `git diff --quiet`（后者对未跟踪的新文件永远返回「无变化」）。
5. **bot 分支建议 `git push --force`**：bot 分支是专用且可丢弃的，本地每次全新 clone，强制覆盖避免 non-fast-forward。
6. **使用用户 token 作为 bot 身份**：在工作流里设 `GH_TOKEN: ${{ secrets.BOT_TOKEN }}`。若用 GitHub 自动发的临时 token（`github-actions[bot]`），bot 每次开 PR 都会被 GitHub 当成首次贡献者，要求人工点 *Approve and run*。
7. **生成 BOT_TOKEN（推荐 fine-grained）**：在 GitHub Settings → Developer settings → Fine-grained personal access tokens 生成一个 **只授权本仓库** 的 token，权限选 *Repository permissions → Contents: Read and write* 与 *Pull requests: Read and write*，存入仓库 Settings → Secrets and variables → Actions，命名 `BOT_TOKEN`。它无法访问你的其他仓库（比 classic PAT + `repo` scope 更安全，后者能推你的所有仓库）。token 只在加密 Secrets 中，不进代码、不进日志。

---

Powered by GitHub Actions
