# 🐾 Bear 的今日日记

一只名叫 Bear 的女孩，每天被 GitHub Actions 叫醒一次，写下一段今日日记。

- 🧡 形象：emoji 主角 + 每天更新的文字
- 📅 页面自动显示「醒来的第 N 天」
- ☁️ 根据当天天气切换心情（后续第三关接入）
- 🐾 一排会点头的小脚印

## 结构

```
index.html                 主页
.github/workflows/ci.yml   CI/CD：语法检查 + 部署到 GitHub Pages
data.json                  （第三关）每天由 cron 工作流生成的数据
```

## 本地预览

直接用浏览器打开 `index.html` 即可。

## 工作原理

每次 push 到 `main`，Actions 自动跑 `check`（校验文件）→ `deploy`（发布到 GitHub Pages）。

---

Powered by GitHub Actions
