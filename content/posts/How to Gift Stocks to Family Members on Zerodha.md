---
author: ["Nishant Nikhil"]
title: "How to Gift Stocks to Family Members on Zerodha (and Avoid Losing Your Original Buy Price)"
date: "2025-10-25"
description: "Gifting stocks on Zerodha (Kite) such that accounting is easy"
summary: ""
tags: ["kite", "zerodha", "indian equity"]
categories: ["indian equity"]
series: ["Guide"]
ShowToc: true
TocOpen: true
---

# How to Gift Stocks to Family Members on Zerodha (and Avoid Losing Your Original Buy Price)

I recently decided to move to the U.S. and wanted to simplify my finances — that meant parting ways with my Indian equity holdings. Instead of selling them off, I decided to **gift my stocks to my mom** using Zerodha’s gifting feature.

If you’re in a similar situation, this guide walks you through **how to gift stocks via Zerodha**, and more importantly, **how to preserve your original buy prices** (so you don’t lose track of your actual capital gains in the process).

---

## Step 1: Open a Demat Account for the Recipient

First things first — your family member (the recipient) will need a **Zerodha account**.

You can start the process here:

👉 [https://zerodha.com/open-account/](https://zerodha.com/open-account/)

Once the account is active, you can initiate the gifting process.

---

## Step 2: Gift Stocks via Zerodha Console

Go to Zerodha’s gifting page:

👉 [https://console.zerodha.com/gift](https://console.zerodha.com/gift)

The process is quite simple — select the stocks you wish to gift, verify via **a couple of CDSL OTPs**, and that’s it!

Your stocks are now transferred.

---

## Step 3: The Hidden Catch — Your Buy Price Gets Reset

Here’s where things get tricky.

Once the gift is processed, **Zerodha resets the buy price** for the recipient to the *closing price on the day of transfer* — not your original acquisition price.

This means the **P&L (Profit & Loss)** and **LTCG/STCG (Long/Short Term Capital Gains)** shown in the Console will be *incorrect*.

I reached out to Zerodha Support about this, and here’s what they said:

> “When stocks are gifted, the closing price of the stock on the date of transfer is considered the exit price for the sender and the entry price for the recipient.
> 
> 
> For income tax purposes, however, the acquisition price may be interpreted differently when filing returns. Please consult your CA.”
> 
> You can also read their official article [here](https://support.zerodha.com/category/your-zerodha-account/transfer-of-shares-and-conversion-of-shares/gifting-securities/articles/gift-shares).
> 

---

## Step 4: Get the Original Buy Breakdown

To ensure you don’t lose your **true acquisition data**, it’s a good idea to **save your current holdings breakdown** before initiating the gift.

Zerodha’s **Console** displays this breakdown under:

👉 [https://console.zerodha.com/portfolio/holdings](https://console.zerodha.com/portfolio/holdings)

You’ll see something like this:

![image.png](/posts/gift_stocks/image.png)

Click **“View breakdown”** to get details of all your buy transactions for a stock:

![image.png](/posts/gift_stocks/image%201.png)

---

## Step 5: Export Your Holding Breakdowns

Unfortunately, Zerodha’s public API doesn’t expose this breakdown.

But, you can access it from the **network inspector** in your browser.

1. Open the holdings page on Console.
2. Open **Developer Tools → Network tab**.
3. Filter for `/portfolio`.
4. Copy the JSON response (it contains your equity and mutual fund holdings).

![image.png](/posts/gift_stocks/image%202.png)

You can then parse it with the following Python snippet:

```python
import json
import pandas as pd

data_json = json.loads(data_json)
data_equity = pd.DataFrame(data_json['data']['result']['eq'])
data_equity["type"] = "EQ"
data_mf = pd.DataFrame(data_json['data']['result']['mf'])
data_mf["type"] = "MF"
data_all = pd.concat([data_equity, data_mf])
```

---

## Step 6: Fetch Each Stock’s Purchase Breakdown

Now, when you click “View breakdown,” a request is sent to Zerodha’s backend API.

You can replicate that API call with Python to download all breakdowns programmatically.

Here’s the function you can use:

```python
import requests

def get_holding_breakdown(instrument_id: str, segment: str):
    """
    Fetch holdings breakdown for a given instrument_id from Zerodha Console.
    """
    url = "https://console.zerodha.com/api/reports/holdings/breakdown"
    params = {
        "instrument_id": instrument_id,
        "segment": segment
    }

    headers = {
        "accept": "application/json, text/plain, */*",
        "referer": "https://console.zerodha.com/",
        "user-agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)",
        "x-csrftoken": "<ADD THIS HERE from your Network call>"
    }

    cookies = {
        "session": "<ADD THIS HERE from your Network call>",
        "public_token": "<ADD THIS HERE from your Network call>"
    }

    response = requests.get(url, headers=headers, cookies=cookies, params=params)

    if response.status_code == 200:
        return response.json()['data']['result']
    else:
        print(f"Error {response.status_code}: {response.text}")
        return None

```

Then apply this function to all your holdings:

```python
breakdowns = data_all.apply(lambda x: get_holding_breakdown(x["instrument_id"], x["type"]), axis=1)
data_all["breakdowns"] = breakdowns
data_all.to_csv("2025_10_07_all_equity_breakdown.csv")
```

This will give you a **comprehensive CSV file** containing every stock’s purchase breakdown — a lifesaver for accurate tax reporting later.

---

## Step 7: Restore Buy Prices in the Gifted Account

Once the transfer is complete, you can **ask Zerodha Support to reset the buy prices** in the recipient’s account.

You can then use your exported CSV as the source of truth for updating their trade history manually (if needed).

(This is a cumbersome 

---

## TL;DR

| Step | Action | Link |
| --- | --- | --- |
| 1 | Open family member’s Zerodha account | [zerodha.com/open-account](https://zerodha.com/open-account/) |
| 2 | Gift your stocks | [console.zerodha.com/gift](https://console.zerodha.com/gift) |
| 3 | Save your holdings breakdown | [console.zerodha.com/portfolio/holdings](https://console.zerodha.com/portfolio/holdings) |
| 4 | Export buy breakdowns | Use Python script above |
| 5 | Ask Zerodha to reset buy prices post-transfer | — |

---

## Final Thoughts

Zerodha’s gifting feature makes transferring stocks to loved ones straightforward — but the **buy price reset issue** can cause trouble when filing taxes or tracking returns.

By exporting your breakdowns beforehand, you can ensure your family members’ holdings reflect the true cost basis. It’s a bit of extra effort — but it keeps your financial record-keeping clean and audit-proof.
