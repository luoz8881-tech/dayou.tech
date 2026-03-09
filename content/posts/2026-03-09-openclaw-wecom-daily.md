---
title: "OpenClaw + 企业微信：让 AI 每天自动推送日报"
date: 2026-03-09
description: "从部署到每天自动收到日报，我是怎么把 OpenClaw 和企业微信打通的——完整配置流程记录。"
tags: [OpenClaw, 企业微信, 自动化, 教程, 日报]
categories: [技术实战]
draft: false
---

我每天早上 9 点，手机会收到一条来自企业微信的消息。

不是人发的，是我的 AI 管家「豆豆」——一个运行在云服务器上的 OpenClaw 实例——自动搜集了过去 12 小时的重要新闻，整理成简报，推送给我。

这篇文章记录我是怎么做到的。

---

## 整体架构

```
OpenClaw（云服务器）
    ↓
每天定时触发 Cron 任务
    ↓
调用 tavily_search 搜索新闻
    ↓
AI 整理成简报
    ↓
调用企业微信 Webhook API
    ↓
推送到我的企业微信群
```

整个流程不需要你手动触发，设置一次，每天自动运行。

---

## 前置条件

- 已部署好 OpenClaw（参考：[OpenClaw 完全上手指南](/posts/2026-03-09-openclaw-deploy-guide/)）
- 一个企业微信账号（个人可以免费注册）

---

## 第一步：创建企业微信群机器人

企业微信支持"群机器人"功能，通过 Webhook 地址可以往群里发消息。

1. 打开企业微信，创建一个群（或用已有群）
2. 点击右上角 **"..."** → **"群机器人"** → **"添加机器人"**
3. 给机器人起个名字，点击确认
4. 复制生成的 **Webhook 地址**，格式如下：

```
https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

这个地址就是你的推送入口，保存好。

---

## 第二步：创建推送脚本

在服务器上创建推送脚本 `~/scripts/send_wecom.py`：

```python
#!/usr/bin/env python3
import sys
import json
import urllib.request

WEBHOOK_URL = "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=你的key"
MAX_BYTES = 4096

def send(content):
    content = content.strip()
    # 超长自动分段
    encoded = content.encode('utf-8')
    if len(encoded) <= MAX_BYTES:
        chunks = [content]
    else:
        chunks = []
        current = ""
        for line in content.split('\n'):
            test = (current + '\n' + line).encode('utf-8')
            if len(test) > MAX_BYTES and current:
                chunks.append(current.strip())
                current = line
            else:
                current = current + '\n' + line if current else line
        if current:
            chunks.append(current.strip())

    total = len(chunks)
    for i, chunk in enumerate(chunks):
        text = chunk if total == 1 else f"({i+1}/{total})\n{chunk}"
        payload = {"msgtype": "text", "text": {"content": text}}
        data = json.dumps(payload).encode('utf-8')
        req = urllib.request.Request(
            WEBHOOK_URL,
            data=data,
            headers={'Content-Type': 'application/json'}
        )
        urllib.request.urlopen(req)

if __name__ == "__main__":
    content = sys.stdin.read()
    if content.strip():
        send(content)
```

给脚本添加执行权限：
```bash
chmod +x ~/scripts/send_wecom.py
```

测试推送：
```bash
echo "测试消息，豆豆上线了" | python3 ~/scripts/send_wecom.py
```

如果企业微信群里收到消息，说明配置成功。

---

## 第三步：配置定时日报任务

打开 OpenClaw 界面，或者通过命令行添加 Cron 任务：

```bash
openclaw cron add \
  --name "早间时政简报" \
  --cron "0 9 * * *" \
  --tz "Asia/Shanghai" \
  --session isolated \
  --no-deliver \
  --message "执行早间时政简报任务。用 tavily_search 搜索过去 12 小时的国际国内重大事件，信源优先路透社、FT、BBC、新华社、财新。覆盖：国际政治、中美关系、全球经济、重要人事变动。写成 800 字以上的深度简报，每条新闻有背景和影响分析，严格控制在 4000 字节内。写入 daily-logs/news-morning-$(date +%Y%m%d).md，然后用 python3 ~/scripts/send_wecom.py 推送。"
```

这条命令设置了每天北京时间 9:00 自动运行，搜集新闻、生成简报、推送到企业微信。

---

## 我目前的推送计划表

| 时间（北京） | 任务 | 内容 |
|------------|------|------|
| 09:00 | 早间时政 | 国际国内重大事件，约 800 字 |
| 12:00 | 午间速递 | 今天上午 Top5 新闻，单条发完 |
| 13:30 | 午间精神食粮 | 一条有启发性的故事或思考 |
| 19:30 | 加密市场日报 | BTC/ETH 等主流币动态 |
| 21:00 | 晚间视野 | 前沿科技/生命科学/文化思想轮换 |

全部自动运行，我什么都不用做。

---

## 推送效果示例

```
📰 午间时政速递（2026年03月09日）

1. 伊朗最高领袖继任确定
哈梅内伊在美以联合打击中身亡后，伊朗宣布由其子莫吉塔巴接任最高领袖。
→ 影响：政权合法性存疑，中东局势将进一步不稳定。（路透社）

2. 商务部明确支持横琴深合区
商务部长助理袁晓明在两会表态，十五五期间将深化CEPA开放，支持横琴面向澳门扩大改革。
→ 影响：横琴营商环境有望进一步改善，对港澳资本利好。（新华社）

3. GPT-5.4 发布：可直接操作电脑
OpenAI 发布 GPT-5.4，原生支持 OpenClaw，被评为"AI 第一次比人类更会操作电脑"。
→ 影响：Agent 能力门槛大幅降低，将加速普通用户进入 AI 实战阶段。（TechCrunch）

4. 深圳龙岗出台"龙虾十条"
龙岗区发布支持 OpenClaw 发展政策，最高补贴 400 万元，首次将 AI Agent 纳入区级政策扶持。
→ 影响：地方政府开始实质性布局 Agent 生态，产业化提速。（证券时报）

5. 比特币跌破 65000 美元
受中东局势影响，加密市场整体回落，BTC 日内下跌 4.2%。
→ 影响：短期恐慌情绪明显，但历史上地缘冲突引发的跌幅往往在 1-2 周内修复。（CoinDesk）

详情问豆豆
```

---

## 进阶：让推送更智能

基础配置做好之后，可以继续优化：

**1. 按重要性过滤**
在 Cron 任务的 message 里加上指令：只保留 Top5 最重要的，不重要的舍弃，保持每条简报单条发完。

**2. 周轮换话题**
比如我的"晚间视野"，周一四讲前沿科技，周二五讲生命科学，周三六讲文化思想，周日推荐一篇长文——这样每天内容不重复，又有系统性。

**3. 异常告警**
添加一个"看门狗"任务，每天晚上检查当天所有日报文件是否正常生成，如果有任务失败就推送告警。

---

## 常见问题

**Q：企业微信个人可以用吗？**
A：可以。个人注册企业微信免费，群机器人功能不需要任何资质。

**Q：消息太长怎么办？**
A：推送脚本已经处理了自动分段，超过 4096 字节会拆成多条发送，并标注 (1/N)(2/N)。

**Q：推送失败怎么排查？**
A：在服务器上执行 `openclaw cron list` 查看任务状态，`lastRunStatus` 显示 ok 表示正常，error 则查看错误日志。

---

## 小结

OpenClaw + 企业微信的组合，本质上是给自己配了一个不睡觉的信息助理。

它每天花在信息收集上的时间是零（对我来说），但我每天收到的信息质量，比自己刷手机强得多——因为它是按照我的需求定向搜集的，不是算法推荐的。

这是 AI Agent 最实用的形态之一：**不是替你思考，而是替你做那些重复但必要的事。**

**相关阅读：**
- [OpenClaw 完全上手指南：普通人的 AI 员工部署实录](/posts/2026-03-09-openclaw-deploy-guide/)
- [n8n 入门：用自动化工作流消灭重复劳动](/posts/2026-03-09-n8n-automation-guide/)

---
*有问题？在下方留言，或发邮件至 hi@dayou.tech*
