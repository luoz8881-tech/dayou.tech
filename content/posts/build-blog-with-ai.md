---
title: "零基础用 AI 搭建个人网站，我只花了一个下午"
date: 2026-03-09T15:00:00+08:00
description: "不会写代码的普通人，如何用 AI + 免费工具从零搭建专业技术博客的完整记录"
tags: ["AI", "Hugo", "GitHub Pages", "建站", "Lobechat", "Perplexity"]
categories: ["实践记录"]
draft: false
---

## 起因

今天下午突然想：我能不能拥有一个自己的网站？

不是公众号，不是知乎专栏，而是一个**真正属于自己的、有独立域名的技术博客**。问题是——我完全不会写代码。

## 我用了什么工具

| 工具 | 角色 | 说明 |
|------|------|------|
| **Lobechat** | 执行者 | 连接我的云服务器，提供终端操作能力 |
| **Perplexity AI** | 技术顾问 | 告诉我每一步该执行什么命令，分析报错 |
| **Hugo** | 网站框架 | 开源静态网站生成器，速度极快 |
| **PaperMod** | 主题 | 简洁优雅的 Hugo 主题，暗色模式好看 |
| **GitHub Actions** | 自动构建 | push 代码后自动编译部署，完全免费 |
| **GitHub Pages** | 托管 | 免费托管静态网站，自带 CDN |

整个过程在**手机上完成**。Lobechat 连服务器当终端，Perplexity 当顾问，我当决策者。

## 真实流程

1. 在服务器上安装 Hugo（apt 装的版本太旧，得手动下载最新版）
2. 创建 Hugo 站点，安装 PaperMod 主题
3. 配置 hugo.yaml，设置网站名称、菜单、暗色模式
4. 在 GitHub 创建仓库，配置 GitHub Actions 自动构建
5. 绑定自定义域名 dayou.tech
6. 逐步优化首页设计：布局、卡片、渐变效果、CTA 区域

整个过程大约 3 小时，其中一半时间在踩坑。

## 踩过的坑

**Git 认证** — GitHub 2021 年就不支持密码登录了，必须用 Personal Access Token。第一次 push 被拒绝，查了才知道。

**Hugo 版本** — Ubuntu apt 安装的 Hugo 是 v0.68，而 PaperMod 需要 v0.112+。得去 GitHub Releases 手动下载最新的 extended 版本。

**主题安装方式** — 不能直接 git clone，必须用 git submodule add，否则 GitHub Actions 构建时拉不到主题文件。

**YAML 缩进** — 配置文件缩进错一个空格，整个网站就挂。Hugo 的报错信息又不太直观，靠 AI 帮忙分析才定位。

## 费用

| 项目 | 费用 |
|------|------|
| Hugo + PaperMod | 免费 |
| GitHub + Actions + Pages | 免费 |
| Lobechat | 免费（开源） |
| 域名 dayou.tech | 约 ¥60/年 |
| **总计** | **约 ¥60/年** |

## 最大的感受

一行 git push，两分钟后网站就自动更新了。

这不是 AI 替我做事，而是 AI 让我**能做到原本做不到的事**。我依然是做决策的人，AI 帮我跨过了技术门槛。
