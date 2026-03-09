---
title: "Bybit API 实盘踩坑全记录"
date: 2026-03-10
description: "实盘第一周，踩遍了 Bybit API 的所有细节坑。这篇文章帮你省掉这些弯路。"
tags: [量化交易, Bybit, API, 实盘, 踩坑]
categories: [量化实战]
draft: false
---

回测好看，模型跑通，系统部署完毕——我以为万事俱备了。

实盘第一天，问题一个接一个。

这篇文章把我踩过的 Bybit API 实盘坑全部记录下来，按重要程度排序。

---

## 坑一：分批止盈 TP2 失效

**现象：** 设置了两个止盈位，TP1 触发没问题，TP2 从来不触发。

**原因：** Bybit 的止盈止损默认是 `tpslMode="Full"`，意思是止盈/止损会平掉全部仓位。设置第二个 TP 时，它会直接覆盖第一个，不是叠加，是替换。

**解决方案：** 改用 `tpslMode="Partial"`，同时指定每个止盈对应的数量：

```python
# 错误方式：第二个 TP 会覆盖第一个
session.set_trading_stop(
    symbol="SOLUSDT",
    takeProfit="160.0",   # TP1
    ...
)
session.set_trading_stop(
    symbol="SOLUSDT", 
    takeProfit="165.0",   # TP2：覆盖了 TP1，TP1 消失
    ...
)

# 正确方式：Partial 模式，分别指定数量
session.set_trading_stop(
    symbol="SOLUSDT",
    tpslMode="Partial",
    takeProfit="160.0",
    tpSize="0.5",    # TP1 平一半
    ...
)
```

---

## 坑二：追踪止损只在 K 线收盘时检查

**现象：** 价格在一根 K 线内剧烈波动，触及了止损位，但程序没有反应。等到下一根 K 线确认时已经来不及了。

**原因：** 我的实盘逻辑是每根 15m K 线收盘时检查一次，中间的价格波动完全不管。遇到单根 K 线内大幅震荡，止损逻辑失效。

**解决方案：** 双重保障——程序端的止损 + Bybit 服务端的止损。

服务端止损直接在交易所执行，不依赖你的程序是否在线、是否在检查：

```python
# 开仓时同时设置服务端止损
session.place_order(
    symbol="SOLUSDT",
    side="Buy",
    orderType="Market",
    qty=str(qty),
    stopLoss=str(sl_price),      # 服务端止损
    takeProfit=str(tp_price),    # 服务端止盈
    tpslMode="Full",
    positionIdx=0
)
```

程序端的止损逻辑作为补充，服务端止损作为兜底。两层保障，不依赖单点。

---

## 坑三：多仓失控，杠杆拉满

**现象：** 策略开了第一仓，信号继续触发，又开了第二仓、第三仓……总杠杆不断叠加，超出了预期风险。

**原因：** 早期没有做总仓位检查，每次信号触发就无条件开仓。在 BNB 时期曾经把总杠杆拉到接近 50 倍。

**解决方案：** 开仓前强制检查总敞口：

```python
def can_open_position(symbol, side, max_positions=3):
    """检查是否可以开新仓"""
    positions = get_current_positions(symbol)
    same_side_count = sum(1 for p in positions if p['side'] == side)
    
    if same_side_count >= max_positions:
        print(f"已达最大持仓数 {max_positions}，跳过开仓")
        return False
    
    # 检查总名义价值
    total_notional = sum(abs(p['notional']) for p in positions)
    if total_notional > MAX_TOTAL_NOTIONAL:
        print(f"总敞口超限，跳过开仓")
        return False
    
    return True
```

最多 3 个同向仓位，总名义价值硬上限，两个条件都要满足才开仓。

---

## 坑四：加载了旧模型，400 根 K 线零信号

**现象：** 实盘运行了整整一天，没有产生任何交易信号，完全静默。

**原因：** 模型更新了，特征数量从 100+ 增加到 148，但 live_signal.py 还在加载旧的 checkpoint。旧模型期望 100 个特征，收到 148 个特征，在内部静默失败，不报错，直接输出默认概率，永远达不到阈值。

这个坑最气——系统看起来在正常运行，日志也没报错，实际上什么都没做。

**解决方案：**

1. 模型文件加版本号，`sol_transformer_v2.ckpt`，不用 `latest.ckpt` 这种名字
2. 启动时打印模型期望的特征数量和实际输入的特征数量，做一致性检查：

```python
# 启动时验证
expected_features = model.input_dim
actual_features = len(feature_columns)
assert expected_features == actual_features, \
    f"特征维度不匹配！模型期望 {expected_features}，实际输入 {actual_features}"
```

3. 每次更新模型后，先在测试环境跑一遍完整的信号生成流程，确认有输出

---

## 坑五：手续费累积侵蚀利润

**数字：** Bybit 永续合约 Taker 手续费 0.055%/边，开平仓往返 0.11%。

看起来很小。但一个高频策略如果每天开平仓 10 次，每次用满仓位：

```
0.11% × 10次/天 × 30天 = 33%/月
```

一个月手续费 33%——这意味着你的策略月收益必须超过 33%，才是真正在赚钱。

很多看起来收益不错的策略，加上真实手续费之后变成亏损。

**解决方案：**

- 回测必须包含手续费，不含手续费的回测数字没有参考价值
- 控制交易频率，低信噪比的信号宁可不交易
- 用 Maker 单替代 Taker 单（Maker 手续费通常更低，甚至负费率）

---

## 坑六：动态止损更新的时机

**现象：** 动态止损（每根 K 线更新一次 SL/TP 位置）有时候更新失败，接口返回成功但实际没生效。

**原因：** Bybit 的 `set_trading_stop` 接口有时有延迟，或者在特定情况下（仓位状态刚发生变化）会静默失败。

**解决方案：**

```python
def update_stop_loss(symbol, new_sl, new_tp, retries=3):
    for attempt in range(retries):
        try:
            result = session.set_trading_stop(
                symbol=symbol,
                stopLoss=str(new_sl),
                takeProfit=str(new_tp),
                ...
            )
            if result['retCode'] == 0:
                return True
        except Exception as e:
            print(f"第 {attempt+1} 次更新失败: {e}")
            time.sleep(1)
    
    print(f"止损更新失败，已重试 {retries} 次")
    return False
```

加重试机制，更新失败时不静默，记录日志并报警。

---

## 实盘前的检查清单

总结成一份清单，每次上实盘前过一遍：

- [ ] 模型版本号和特征数量一致性验证
- [ ] 服务端止损/止盈已设置
- [ ] 最大持仓数和总敞口上限已配置
- [ ] 手续费已纳入回测验证
- [ ] 止损更新有重试机制
- [ ] 关键操作有日志记录
- [ ] 企微告警已接通（异常时能收到通知）

---

## 小结

实盘和回测最大的区别，不是市场条件，而是**你的代码要面对真实世界的各种意外**：接口延迟、静默失败、版本不一致、边界条件……

这些问题在回测里完全看不到，但在实盘里会一一浮现。

提前把这些坑走一遍，比在真金白银上学费要划算得多。

**相关阅读：**
- [回测 600 倍收益背后的真相](/posts/2026-03-10-backtest-600x-trap/)
- [我的量化系统架构](/posts/)

---
*有问题？在下方留言，或发邮件至 hi@dayou.tech*
