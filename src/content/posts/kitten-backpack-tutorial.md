---
AIGC:
    Label: "1"
    ContentProducer: 001191440300708461136T1XGW3
    ProduceID: 3c041fe1738ba2bde15a4dc9097ec384_46c86e9fa93011f1be88525400aeaaa3
    ReservedCode1: MMh1A2rBMsRTUHbMz9xg2Wwthy1tYYtNYDkQU7bu8clPznlnyY53T2XSZ1AodX9tiikxfP0PX9brBmJ6tv/C76pPQ/TSBjh85O9xHl3dKScTk1YINd1ANrezQ5mFJHsiBpx7UBD4XU7hCPy9fnk9MBRa895y1BSEjI00yV3pntYvVyaB7eSjuP5tDk0=
    ContentPropagator: 001191440300708461136T1XGW3
    PropagateID: 3c041fe1738ba2bde15a4dc9097ec384_46c86e9fa93011f1be88525400aeaaa3
    ReservedCode2: MMh1A2rBMsRTUHbMz9xg2Wwthy1tYYtNYDkQU7bu8clPznlnyY53T2XSZ1AodX9tiikxfP0PX9brBmJ6tv/C76pPQ/TSBjh85O9xHl3dKScTk1YINd1ANrezQ5mFJHsiBpx7UBD4XU7hCPy9fnk9MBRa895y1BSEjI00yV3pntYvVyaB7eSjuP5tDk0=
---





# Coding Cat Online Backpack Tutorial: Cross-Device Backpacks with Cloud Variables

## The Short Version

Want to build an **online backpack** in Coding Cat Kitten 4.0 — where players gather resources, exchange items and visit a shop, with data synced across devices? The core idea is one sentence: **store the string in a cloud variable, split it into a list to calculate, then join it back together.**

This idea comes from my friend Kevin, who worked out a very clean implementation. I have turned it into a full tutorial with all the block screenshots included — follow along and you will have it running.

## Core Idea: Cloud Variables Can't Store Lists, So Store a String

Coding Cat's cloud variables can only store numbers or strings — **you cannot store a list directly.** But a backpack is essentially "a bunch of item counts." So what do you do?

Kevin's solution is clever: **pack the backpack data into a single string, separated by `/`.**

For example, `云背包 = "10/5/20"` means:

- Item 1 = 10 (say, wood)
- Item 2 = 5 (say, stone)
- Item 3 = 20 (say, iron)

Every time you want to change the backpack, split the string into a list by `/`, edit the list, then join it back into a string and store it in the cloud variable. That way the cloud variable always holds a consistently formatted string, and online sync is worry-free.

> Note: this uses a **private cloud variable**, so each player keeps their own backpack without interference. If all players share one public cloud variable, everyone shares the same backpack — whichever gameplay you prefer.

## Step 1: Initialize the Cloud Variable

At the start of the game, check whether the cloud variable is in its initial state; if so, set it to `0/0/0`, then split it into a list for later use.

![初始化云变量](/C130AIR-Blog/kitten-backpack-init.png)

Logic breakdown:

1. **When flag clicked**, first check whether `云背包 == 0` (not initialized yet)
2. If so, set `云背包` to `0/0/0`, with all three item counts at 0
3. Split `云背包` into a list by `/`, storing it in `背包_`
4. Join items 1, 2 and 3 of the list back together with `/`, ensuring the format is always `count/count/count`

This step's purpose is **format standardisation** — no matter what the cloud variable was set to before, running this normalises it to an `x/y/z` shape. All subsequent logic builds on the convention of "item 1, item 2, item 3."

## Step 2: Getting Items (Gathering)

Click a gathering button and the corresponding item count increases by 1.

![获取物品](/C130AIR-Blog/kitten-backpack-get.png)

There is really just one block here:

- **Replace item 1 of 背包_ with item 1 of 背包_ + 1**

That is, "wood + 1." Want stone? Edit item 2. Iron? Item 3. Duplicate the block a few times and you have gathering for all three resources.

> Tip: after editing the list, remember to join `背包_` back into `云背包` (see the join method in step 1, sub-step 4) — otherwise the data only lives in the local list and never syncs to the cloud.

## Step 3: Exchange / Shop Logic

Once the backpack is full enough, you can exchange. For example, "10 wood for 1 stone":

![兑换逻辑](/C130AIR-Blog/kitten-backpack-exchange.png)

Logic breakdown:

1. **When self clicked**, check whether `背包_ 第 1 项 >= 10` (is there enough wood?)
2. If yes: item 1 -10, item 2 +1, then repeat "exchange successful" 5 times
3. If not: repeat "not enough iron" prompt 5 times

This is the complete version of what Kevin calls the shop idea — **a condition check plus add/subtract quantities.** To make it a shop purchase, just reverse the condition: when item 2 > 1, item 1 +1 and item 2 -1 — that is "spend 1 stone to buy 1 wood."

## Kevin's Original Words

This is where the idea comes from — Kevin's chat log, straight up:

![Kevin 的聊天记录](/C130AIR-Blog/kitten-backpack-chat.png)

The core of what he said, in two lines:

> Set the private cloud variable to the `0/0/0` format. To add one to the first, do it like this: `云以/分为列表的第一项 +1 / 云以/分为列表的第二项 / 云以/分为列表的第三项`.

> The shop, for example: when "item 2" > 1, you can buy 1 of item 1, which means: `云以/分为列表的第一项 +1 / 云以/分为列表的第二项 -1 / 云以/分为列表的第三项`.

In plain terms: **to change a position, only touch that one item in the list, and join the other two back unchanged.** This pattern scales indefinitely — add a fourth item and you add one more `/` and one more field; make a multi-page backpack and you store one more page-number field.

## The Full Flow

Chain the three blocks together and the whole online backpack loop looks like this:

1. **Start** → initialize `云背包 = 0/0/0`, split into list `背包_`
2. **Gather** → click button → `背包_ 第 N 项 + 1` → join back into `云背包`
3. **Exchange/Shop** → click button → check if enough → add/subtract if so → join back into `云背包`
4. **Cloud sync** → all players' `云背包` sync automatically, and backpack data stays consistent across devices

## Common Pitfalls

- **Always join the list back into the cloud variable after editing**: if you only change `背包_` without writing back to `云背包`, the data will not sync and will be lost on refresh.
- **Keep the format consistent**: do not mix the `/` separator — use the same one for splitting and joining, or the list items will misalign.
- **Private vs public cloud variables**: use private for "each player has their own backpack," public for "a server-wide shared warehouse" — do not mix them up.
- **Validate quantities**: always check whether there is enough before an exchange, or you will end up with a negative backpack. Kevin's `>= 10` check is exactly for this.

## Summary

The essence of Kevin's approach is using a **string to simulate an array**, bypassing the limitation that cloud variables cannot store lists. Then, with the fixed pattern of "split → change one item → join back," gathering, exchanging and shopping are all unified into the same operation. The code footprint is small, but the extensibility is huge — a genuinely practical trick for Coding Cat online projects.

Once you have learned it, go add a backpack to your online mini-game — and do not forget to give Kevin a thumbs up.
*（内容由AI生成，仅供参考）*
