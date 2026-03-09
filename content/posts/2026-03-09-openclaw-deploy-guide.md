---
title: "OpenClaw 完全上手指南：普通人的 AI 员工部署实录"
date: 2026-03-09
description: "我是怎么在谷歌云上部署 OpenClaw，让 AI 替我 7×24 小时工作的——不写代码也能做到。"
tags: [OpenClaw, AI工具, 教程, 自动化, 龙虾]
categories: [AI工具]
draft: false
---

2026年初，我在谷歌云上部署了一只"龙虾"。

从那天起，我的手机每天早上9点会收到一条自动推送的时政日报，下午会有加密市场简报，晚上有前沿科技摘要。这些内容，我一个字都没有手动整理过。

这只"龙虾"，就是 **OpenClaw**——一个开源的 AI 智能体框架，圈内人因为它的 Logo 叫它龙虾。

---

## 什么是 OpenClaw？

OpenClaw 不是聊天机器人。

ChatGPT、Claude 这些 AI，你问一句它答一句，本质上是一个对话框。而 OpenClaw 是一个能**接管电脑权限、自主执行任务**的 AI 代理。

你对它说"帮我搜集今天的加密市场新闻，整理成报告，发到我的企业微信"——它不会回复你"好的，以下是步骤"，它会**直接去做**：打开浏览器搜索、整理内容、调用 API 发送消息，全程不需要你盯着。

这就是从"对话"到"执行"的本质跨越。

```
传统 AI：你 → 问 → AI → 答 → 你执行
OpenClaw：你 → 下指令 → OpenClaw 自主执行 → 结果
```

截至2026年3月，OpenClaw 在 GitHub 上的 Star 数突破 **25万**，成为史上增速最快的开源项目，超越了 Linux 和 React。

---

## 我为什么选择部署在云服务器

OpenClaw 可以本地安装，但我选择了云服务器，原因很简单：

| 方式 | 优点 | 缺点 |
|------|------|------|
| 本地安装 | 免费，配置简单 | 电脑关机就停了，无法 7×24 运行 |
| 云服务器 | 全天候运行，稳定 | 需要付费（约 ¥10-20/月） |

对我来说，一个每天能自动干活的 AI 员工，每月十几块钱的服务器费用完全值得。

---

## 部署环境

我用的是 **Google Cloud Platform（GCP）**，Ubuntu 22.04 系统。

其实用哪家云服务商都可以，国内用户可以选：
- **阿里云轻量服务器**：2核2G，¥118/年，新用户优惠 → [查看活动](https://aliyun.com)
- **腾讯云轻量服务器**：2核4G，¥38起/年 → [查看活动](https://cloud.tencent.com)

**最低配置要求：** 2核 CPU，2GB 内存，20GB 硬盘

---

## 部署步骤

### 第一步：连接服务器

买好云服务器后，用 SSH 连接：

```bash
ssh root@你的服务器IP
```

Windows 用户可以用 PowerShell 或者下载 [FinalShell](https://www.hostbuf.com/)（更友好的 SSH 客户端）。

### 第二步：安装 Node.js

OpenClaw 需要 Node.js 22 以上版本：

```bash
# 安装 nvm（Node 版本管理器）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash

# 重新加载环境变量
source ~/.bashrc

# 安装 Node.js 22
nvm install 22
nvm use 22

# 验证安装
node --version  # 应显示 v22.x.x
```

### 第三步：安装 OpenClaw

```bash
npm install -g openclaw
```

等待安装完成（约 2-5 分钟）。

### 第四步：初始化配置

```bash
openclaw onboard --install-daemon
```

这个命令会启动一个交互式向导，引导你完成：

1. **选择 AI 模型提供商**：推荐 Anthropic（Claude）或 OpenAI，也支持国内的阿里云百炼、腾讯混元等
2. **输入 API Key**：去对应平台申请，通常几分钟就能拿到
3. **配置通讯渠道**：支持企业微信、微信、钉钉、飞书、Telegram 等

> 💡 **国内用户推荐用企业微信**：注册免费，接收消息体验最好，而且 OpenClaw 对企微的支持非常完善。

### 第五步：验证运行状态

```bash
openclaw doctor
```

看到绿色提示，说明一切正常：

```
✓ Gateway running
✓ Model connected
✓ Channel connected
```

### 第六步：让服务开机自启

```bash
# 用 PM2 管理进程
npm install -g pm2
pm2 start $(which openclaw) --name "openclaw" -- gateway
pm2 save
pm2 startup
```

完成后，服务器重启也不会中断 OpenClaw。

---

## 部署完成后能做什么

这是我目前在用的自动化任务，全部通过 OpenClaw 的 Cron 功能实现，每天自动运行：

| 时间 | 任务 | 说明 |
|------|------|------|
| 09:00 | 早间时政简报 | 搜集过去12小时国际要闻 |
| 12:00 | 午间速递 | 今天上午重要事件 Top5 |
| 19:30 | 加密市场日报 | ETH/BTC等主流币动态 |
| 21:00 | 晚间视野 | 前沿科技/生命科学/哲学轮换推送 |

这些任务每天自动执行、自动推送到手机，我什么都不用做。

---

## 常见问题

**Q：没有技术基础能部署吗？**
A：可以。我本人不写代码，整个部署过程跟着步骤来，大约需要 30-60 分钟。遇到问题用 AI 问就行。

**Q：API Key 怎么申请？**
A：Claude 在 [console.anthropic.com](https://console.anthropic.com) 申请，OpenAI 在 [platform.openai.com](https://platform.openai.com) 申请。国内用户建议用阿里云百炼，注册即送免费额度。

**Q：每月费用大概多少？**
A：云服务器约 ¥10-20/月，API 调用费用视使用量而定，轻度使用每月约 ¥20-50。

**Q：数据安全吗？**
A：OpenClaw 是开源项目，代码可以审查。注意不要把 API Key 暴露在公开场合，定期检查 openclaw doctor 的安全提示。

---

## 小结

部署 OpenClaw 改变了我和 AI 的关系——它不再是一个工具，而是一个真正在帮我干活的助手。

如果你也想让 AI 替你做事，而不只是和你聊天，OpenClaw 是目前最值得尝试的方向。

**相关阅读：**
- [OpenClaw + 企业微信：让 AI 每天自动推送日报](/posts/)
- [我用 Perplexity 替代了 80% 的搜索引擎需求](/posts/)

---

*有问题？在下方留言，或发邮件至 hi@dayou.tech*
