---
layout: post
title:  "如何讓Google Gemini API 相容OpenAI格式"
date:   2025-02-13 23:42:00 +0900
categories: google
---

網址：[Google AI Studio](https://aistudio.google.com/)

網路上有很多服務都是希望你填寫OpenAI相容的網址，但Gemini API原本的格式太過獨特，不是每個服務都有相容。
<img src="/assets/img/2025-02-13-google-ai-api-openai-compatibility-01.png" width="60%">

在Google AI Studio裡並不會直接提供OpenAI相容的網址，需要到文件裡去找
<img src="/assets/img/2025-02-13-google-ai-api-openai-compatibility-02.png" width="60%">

### Case 1
如果你看到的服務，是寫 `https://api.openai.com/v1/chat/completions`
那你需要改成 `https://generativelanguage.googleapis.com/v1beta/openai/chat/completions`

### Case 2
如果你看到的是寫 `https://api.openai.com/v1`
那就改成 `https://generativelanguage.googleapis.com/v1beta/openai/`

### Case 3
金鑰 = Api Key，如果你看到的服務要填寫Apikey的話，OpenAI的格式是 `sk-1122334455667788`
你直接貼上從Google AI Studio取得的Key即可

如果你看到的是 `Bearer sk-1122334455667788`
你就改成 `Bearer Google的Key`

### Case 4
模型名稱的部分，Google的是長這樣
```
gemini-2.0-pro-exp
gemini-2.0-flash
gemini-2.0-flash-exp
gemini-2.0-flash-thinking-exp
gemini-2.0-flash-lite-preview
gemini-2.0-flash-lite-preview-02-05
```
有非常多種，而且3個月左右會更新一批，如果你使用的是 preview 或是 有帶日期的模型名稱，有時會被淘汰
請隨時注意 [模型列表](https://ai.google.dev/gemini-api/docs/models/gemini?hl=zh-tw)

目前推薦使用：`gemini-2.0-flash` `gemini-2.0-flash-exp` `gemini-2.0-pro-exp` 

參考來源：

[OpenAI 相容性](https://ai.google.dev/gemini-api/docs/openai?hl=zh-tw#rest)
