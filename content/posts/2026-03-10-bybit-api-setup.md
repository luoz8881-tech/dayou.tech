---
title: "How to Create Your Bybit API Key and Connect It to Your Server"
date: 2026-03-09T23:30:00+08:00
description: "The bridge between your exchange account and your trading system — set up in minutes."
tags: [Bybit, API, server, tutorial, algo trading]
categories: [Quant Trading]
draft: false
---

*The bridge between your exchange account and your trading system — set up in minutes.*

---

You have a Bybit account. You have a server.

Now you need them to talk to each other.

An API Key is how that happens. Your program uses it to place orders, check positions, and read market data — without ever touching the withdrawal function.

---

## Step 1: Open API Management

Log in to Bybit → Profile icon (top right) → Account & Security → API Management → Create New Key → **System-generated API Key**

---

## Step 2: Set Permissions

Only check these two:

- ✅ Read
- ✅ Contract Trading

**Never enable withdrawals.** There is no reason a trading system needs withdrawal access. Keeping it off protects your funds even if the key is compromised.

---

## Step 3: Bind Your Server IP

Enter your Google Cloud server's external IP address.

Once bound, this API Key only works from your server. Even if someone gets the key, they can't use it from anywhere else.

To find your server IP: Google Cloud Console → VM Instances → External IP column.

---

## Step 4: Save Your Keys

After confirming, you'll see your API Key and API Secret.

**The Secret is only shown once. Copy it immediately.**

SSH into your server and run this command — paste your actual Key and Secret into the two lines:

```bash
mkdir -p ~/trading && cat > ~/trading/.env << 'EOF'
BYBIT_API_KEY=paste_your_api_key_here
BYBIT_API_SECRET=paste_your_api_secret_here
EOF
```

Replace `paste_your_api_key_here` and `paste_your_api_secret_here` with your actual values. Everything else stays the same.

---

## Step 5: Test the Connection

Copy and run this entire block — no modifications needed:

```bash
pip3 install pybit python-dotenv && python3 - << 'EOF'
from pybit.unified_trading import HTTP
from dotenv import load_dotenv
import os

load_dotenv(os.path.expanduser('~/trading/.env'))
session = HTTP(
    testnet=False,
    api_key=os.environ["BYBIT_API_KEY"],
    api_secret=os.environ["BYBIT_API_SECRET"]
)
result = session.get_wallet_balance(accountType="CONTRACT")
print(result)
EOF
```

If you see your account balance returned, the connection works.

---

## Common Errors

**`10003: Invalid api_key`**
Key was copied incorrectly or has extra spaces. Check the `.env` file carefully.

**`33004: apikey is not allow access`**
IP whitelist mismatch. Verify your server's current external IP matches what you entered in Bybit.

**`KeyError: BYBIT_API_KEY`**
The `.env` file didn't load. Check the file path is correct.

---

## Connection Confirmed.

Your server and Bybit can now communicate.

Next: pulling historical market data — the first step in building your trading system.

---

Don't have a Bybit account yet? [Sign up here](https://partner.bybit.com/b/156103) — new users get up to **$30,000 USDT** in rewards.

---

*All content is for informational purposes only and does not constitute financial or investment advice. Cryptocurrency trading involves significant risk of loss. Past performance is not indicative of future results. Trade responsibly and never risk more than you can afford to lose.*

*Questions: hi@dayou.tech*
