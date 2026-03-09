---
title: "Hugo + GitHub Pages：零成本搭建你的独立博客"
date: 2026-03-09
description: "这个博客本身就是用 Hugo + GitHub Pages 搭的。从零到上线，我把整个过程记录下来了。"
tags: [Hugo, GitHub Pages, 建站, 独立博客, 教程]
categories: [AI工具]
draft: false
---

这个博客，就是用 Hugo + GitHub Pages 搭的。

我不写代码，从决定建博客到第一篇文章上线，花了不到一天时间。这篇文章把整个过程记录下来，你可以照着做。

**费用：零。** GitHub Pages 免费，Hugo 开源免费，域名（可选）大约 ¥50-80/年。

---

## 为什么选 Hugo + GitHub Pages

建博客的方案很多：WordPress、Hexo、Ghost、Notion 转博客……我最终选 Hugo + GitHub Pages，原因有三：

**1. 速度极快**
Hugo 生成的是纯静态 HTML 文件，没有数据库，没有服务器端渲染。页面加载速度比 WordPress 快 10 倍不止，对 SEO 很重要。

**2. 完全免费托管**
GitHub Pages 免费托管静态网站，每次你提交代码，网站自动更新。不需要买服务器。

**3. 写作体验好**
文章用 Markdown 写，用你熟悉的任何编辑器，文件保存在本地，不依赖任何在线平台。

---

## 你需要准备什么

- 一个 GitHub 账号（免费）
- 一台电脑（Mac/Windows/Linux 都可以）
- 大约 1-2 小时时间

可选：
- 一个自定义域名（推荐，显得更专业）

---

## 第一步：安装 Hugo

**Mac：**
```bash
brew install hugo
```

**Windows：**
去 [Hugo Releases](https://github.com/gohugoio/hugo/releases) 下载最新版本的 `.zip` 文件，解压后把 `hugo.exe` 放到任意目录，添加到系统 PATH。

**验证安装：**
```bash
hugo version
# 应显示：hugo v0.xxx.x ...
```

---

## 第二步：创建新博客

```bash
# 在你喜欢的目录下创建博客
hugo new site my-blog

# 进入博客目录
cd my-blog

# 初始化 Git 仓库
git init
```

---

## 第三步：安装主题

这个博客用的是 **PaperMod** 主题，简洁、加载快、SEO 友好。

```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

然后编辑 `hugo.yaml`，添加主题配置：

```yaml
baseURL: "https://你的用户名.github.io/"
languageCode: zh-cn
title: 你的博客名称
theme: PaperMod

params:
  defaultTheme: dark
  ShowReadingTime: true
  ShowPostNavLinks: true
  ShowToc: true
```

---

## 第四步：写第一篇文章

```bash
hugo new posts/my-first-post.md
```

这会在 `content/posts/` 目录下创建一个文件，用任何文本编辑器打开：

```markdown
---
title: "我的第一篇文章"
date: 2026-03-09
draft: false
---

正文内容写在这里。

Markdown 格式，支持 **粗体**、*斜体*、`代码`、表格等。
```

把 `draft: false` 设置好（false = 发布，true = 草稿）。

---

## 第五步：本地预览

```bash
hugo server -D
```

浏览器打开 `http://localhost:1313`，实时预览效果。

---

## 第六步：部署到 GitHub Pages

### 创建 GitHub 仓库

去 GitHub 创建一个新仓库，名称格式：`你的用户名.github.io`（这是 GitHub Pages 的固定格式）。

### 配置自动部署

在博客根目录创建文件 `.github/workflows/hugo.yaml`：

```yaml
name: Deploy Hugo Site

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: true

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

### 推送代码

```bash
git add -A
git commit -m "初始化博客"
git branch -M main
git remote add origin https://github.com/你的用户名/你的用户名.github.io.git
git push -u origin main
```

推送后，GitHub Actions 会自动构建并发布。大约 2-3 分钟后，访问 `https://你的用户名.github.io`，博客就上线了。

---

## 第七步（可选）：绑定自定义域名

如果你有自己的域名（比如 `dayou.tech`），可以绑定到 GitHub Pages。

**在域名 DNS 里添加记录：**

| 类型 | 名称 | 值 |
|------|------|-----|
| A | @ | 185.199.108.153 |
| A | @ | 185.199.109.153 |
| A | @ | 185.199.110.153 |
| A | @ | 185.199.111.153 |
| CNAME | www | 你的用户名.github.io |

**在博客根目录创建 CNAME 文件：**
```
dayou.tech
```

**在 GitHub 仓库设置里：**
Settings → Pages → Custom domain，填入你的域名，勾选 Enforce HTTPS。

DNS 生效通常需要 1-24 小时。

---

## 日常发文流程

博客搭好之后，日常发文只需要三步：

```bash
# 1. 新建文章
hugo new posts/文章标题.md

# 2. 编辑文章内容（用任何编辑器）

# 3. 提交推送
git add -A
git commit -m "发布：文章标题"
git push
```

推送后 2-3 分钟，文章自动上线。

---

## 常见问题

**Q：不会 Git 怎么办？**
A：用 GitHub Desktop（图形界面），拖拖拽拽就能完成提交和推送，不需要命令行。

**Q：文章里能插图片吗？**
A：可以。把图片放在 `static/images/` 目录下，文章里用 `![描述](/images/图片名.jpg)` 引用。

**Q：能用中文域名吗？**
A：技术上可以，但不建议，搜索引擎收录不友好。

**Q：Hugo 和 WordPress 哪个更适合我？**
A：如果你需要评论系统、会员功能、电商功能，用 WordPress。如果只是写文章，Hugo 更快更省钱。

---

## 小结

建独立博客这件事，门槛其实没有很多人想象的高。Hugo + GitHub Pages 的组合，让"不写代码的人"也能拥有一个专业的博客。

这个博客就是最好的证明——我从零开始，一天之内完成了部署，现在每篇文章发布只需要三行命令。

**下一步你可以做的事：**
1. 按照本文步骤，今天就把博客搭起来
2. 写一篇关于你自己的"关于页面"
3. 发布第一篇文章，哪怕只有 200 字

**相关阅读：**
- [OpenClaw 完全上手指南：普通人的 AI 员工部署实录](/posts/2026-03-09-openclaw-deploy-guide/)
- [n8n 入门：用自动化工作流消灭重复劳动](/posts/2026-03-09-n8n-automation-guide/)

---
*有问题？在下方留言，或发邮件至 hi@dayou.tech*
