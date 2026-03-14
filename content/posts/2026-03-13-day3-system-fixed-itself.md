---
title: "Day 3: The System Fixed Itself (After We Fixed the System)"
date: 2026-03-13T20:00:00+08:00
description: "Equity went from $701 to $928 in a single day. Not because the market was kind. Because the data was finally clean."
tags: [quant, algotrading, crypto, buildinpublic, debugging]
---

Day 3. Equity went from $701 to $928.

Not because the market was kind. Because the data was finally clean.

---

## One line of code. Running for two days.

Something felt off on day one. I was holding long positions, the market was moving up, but my equity was bleeding faster than it should. I couldn't find the cause.

On day three, I pulled the logs. Found a keyword repeating every 15 minutes:

```
NaN features (7): ['cvd_net', 'cvd_ratio', 'cvd_zscore', 'global_ls_ratio', 'global_ls_zscore']
```

I had assumed it was occasional API noise. It wasn't. This line had been printing for nearly two days straight.

The root cause was in `data_fetcher.py`. CoinGlass returns UTC timestamps. But somewhere in the code, there was this:

```python
df['timestamp'] = pd.to_datetime(..., utc=True).dt.tz_localize(None) - timedelta(hours=8)
```

Subtracting 8 hours. From a UTC timestamp.

This meant cvd, global_ls, and top_position — three core features — were timestamped 8 hours in the past. The `merge_asof` tolerance is 2 hours. Every merge failed silently. Features became NaN, then got filled with zeros.

The model didn't know. It kept reasoning on garbage data. It kept opening positions.

---

## Silent failure is worse than a crash

The system didn't crash. Logs showed INFO throughout. Cron fired every 15 minutes like clockwork. From the outside, everything looked fine.

The real problem was deeper: data quality. Were timestamps aligned? Were features producing NaN? What does the model do when you fill NaN with zero? None of this was monitored.

After removing that one line, the afternoon sessions ran on real data. BTC, XRP, DOGE found genuine momentum signals. Equity climbed from $701 to $928.

What changed wasn't the market. It was the data.

After this, I added critical NaN detection to all five live scripts: if ≥3 core features are NaN, an immediate alert fires. This should have existed on day one.

---

## The fix introduced the next bug

Same day. Fixed three vulnerabilities. Then created a fourth.

While patching the close-position quantity precision in `common.py`, I wrote an f-string with nested same-type quotes:

```python
# This fails on Python 3.10
f"{qty:.{len(str(step).rstrip('0').split('.')[-1])}f}"
```

Python 3.12+ supports nested quotes inside f-strings. The server runs 3.10. It doesn't.

Deployed. 10:45 cron fired. Syntax error. All five symbols went silent for 15 minutes.

*"Just fixed it"* is exactly when you stop being careful.

New rule: every file gets `python3 -m py_compile` before upload. No exceptions. No matter how trivial the change looks.

---

## What day 3 actually looked like

- Start of day: $831
- Intraday low: $701 (NaN bug period, false signals)
- End of day: $928
- Swing: -16% → +11%, same day

This is not a smooth system. This is a system being debugged in production, with real money, in real time.

Three bugs found and fixed. One introduced and fixed. Net: cleaner data, better signal quality, one rule added.

Three months from now I'll know if any of this actually works. Right now, the only job is to fix what I find, log what breaks, and not pretend I know more than I do.

---

*This content is for informational purposes only and does not constitute financial or investment advice. Cryptocurrency trading involves significant risk of loss. Trade responsibly.*
