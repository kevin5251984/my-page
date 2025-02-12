---
layout: post
title:  "如何申請Groq的API key"
date:   2025-02-12 19:42:00 +0900
categories: ai
---

網址：[Groq console](https://console.groq.com/playground)

Groq aka AI大善人，提供不少速度極快，品質極高的AI模型

<img src="/assets/img/2025-02-12-how-to-get-groq-api-key-01.png" width="60%">

還有最新的千問2.5 跟 Deepseek R1蒸餾版

<img src="/assets/img/2025-02-12-how-to-get-groq-api-key-05.png" width="60%">

註冊與登入，[Login](https://console.groq.com/login)

<img src="/assets/img/2025-02-12-how-to-get-groq-api-key-02.png" width="60%">

到Email去收信

<img src="/assets/img/2025-02-12-how-to-get-groq-api-key-06.png" width="60%">

進入後台，選`API keys`，然後`Create API Key`

<img src="/assets/img/2025-02-12-how-to-get-groq-api-key-03.png" width="60%">

注意！注意！注意！ 一定要Copy Key，不然後面會找不到

<img src="/assets/img/2025-02-12-how-to-get-groq-api-key-04.png" width="60%">

以上，就完成Groq的申請了。 如何使用？ 請看下面

### Case 1

如果你看到的服務，是寫 `https://api.openai.com/v1/chat/completions`
那你需要改成 `https://api.groq.com/openai/v1/chat/completions`

### Case 2

如果你看到的是寫 `https://api.openai.com/v1`
那就改成 `https://api.groq.com/openai/v1`

### Case 3

金鑰 = Api Key，如果你看到的服務要填寫Apikey的話，OpenAI的格式是 `sk-1122334455667788`
你直接貼上從Groq取得的Key即可，格式像這樣`gsk_1122334455667788`

如果你看到的是 `Bearer sk-1122334455667788`
你就改成 `Bearer Groq的Key`

### Case 4

模型名稱的部分，Groq的是長這樣
```
gemma2-9b-it
llama-3.3-70b-versatile
qwen-2.5-32b
deepseek-r1-distill-qwen-32b
deepseek-r1-distill-llama-70b-specdec
```
有非常多種，而且3個月左右會更新一批，如果你使用的是 preview 有時會被淘汰
請隨時注意 [模型列表](https://console.groq.com/docs/models)

目前推薦使用：`gemma2-9b-it` `qwen-2.5-32b` `deepseek-r1-distill-qwen-32b` 