---
title: "Bybit API 实盘踩坑全记录"
date: 2026-03-09T23:00:00+08:00
description: "实盘第一周踩遍了 Bybit API 的细节坑，整理成清单，帮你省掉这些弯路。"
tags: [量化交易, Bybit, API, 实盘, 踩坑]
categories: [量化实战]
draft: false
---

回测通过，系统部署完毕，以为万事俱备。

实盘第一天，问题一个接一个出现。

---

## 坑一：分批止盈 TP2 失效

**现象：** TP1 触发正常，TP2 从不触发。

**原因：** Bybit 默认 `tpslMode="Full"`，设第二个 TP 会直接覆盖第一个，不是叠加。

**解决：** 改用 `tpslMode="Partial"`，每个止盈指定对应数量。

```python
session.set_trading_stop(
    symbol="SOLUSDT",
    tpslMode="Partial",
    takeProfit="160.0",
    tpSize="0.5",    # 平一半
)
```

---

## 坑二：追踪止损只在 K 线收盘时检查

**现象：** 单根 K 线内大幅震荡，止损没有触发。

**原因：** 程序每根 15m K 线收盘才检查一次，中间波动完全不管。

**解决：** 双重保障——程序端逻辑 + Bybit 服务端止损兜底。

```python
# 开仓时同时设置服务端止损
session.place_order(
    symbol="SOLUSDT",
    side="Buy",
    qty=str(qty),
    stopLoss=str(sl_price),
    takeProfit=str(tp_price),
    ...
)
```

服务端止损不依赖你的程序是否在线，是最后一道防线。

---

## 坑三：多仓失控，杠杆拉满

**现象：** 信号持续触发，仓位不断叠加，总杠杆远超预期。

**原因：** 没有总仓位上限检查，每次信号都无条件开仓。

**解决：** 开仓前强制检查：

```python
def can_open(symbol, side, max_pos=3):
    positions = get_positions(symbol)
    same_side = sum(1 for p in positions if p['side'] == side)
    return same_side < max_pos
```

最多 3 个同向仓位，硬限制。

---

## 坑四：加载旧模型，400 根 K 线零信号

**现象：** 系统运行整整一天，无任何交易。日志正常，没有报错。

**原因：** 模型更新后特征从 100+ 扩展到 148，但 live_signal.py 还在加载旧 checkpoint。旧模型静默失败，输出默认概率，永远达不到阈值。

**解决：** 启动时做特征维度一致性检查：

```python
assert model.input_dim == len(feature_columns), \
    f"特征维度不匹配！模型 {model.input_dim}，实际 {len(feature_columns)}"
```

每次更新模型后先在测试环境跑一遍信号生成，确认有输出。

---

## 坑五：手续费侵蚀利润

Bybit Taker 往返手续费 0.11%。高频策略每天 10 次开平仓：

```
0.11% × 10次 × 30天 = 33%/月
```

月收益必须超过 33% 才是在赚钱。很多回测漂亮的策略，加上真实手续费就变成亏损。

**解决：** 回测必须含手续费。不含手续费的数字没有参考价值。

---

## 实盘前检查清单

- [ ] 模型特征维度一致性验证
- [ ] 服务端止损/止盈已设置
- [ ] 最大持仓数已配置
- [ ] 回测已含手续费
- [ ] 止损更新有重试机制
- [ ] 企微异常告警已接通

---

**系列文章：**
- 上一篇 → [回测 600 倍收益背后的真相](/posts/2026-03-10-backtest-600x-trap/)
- 从头读起 → [我为什么放弃手工指标，转向 AI 量化](/posts/2026-03-10-why-i-switched-to-ai-quant/)
- [回归预测为什么做不了交易信号](/posts/2026-03-10-regression-vs-classification/)

---

**关于服务器：**

量化系统需要 24 小时不间断运行的服务器。我用[阿里云 ECS](https://www.aliyun.com/daily-act/ecs/activity_selection?userCode=132pbq5s)，稳定性好，国内延迟低，对 Bybit API 的响应速度也没有影响。新用户首购有优惠。

---
*有问题？发邮件至 hi@dayou.tech 或在下方留言*
