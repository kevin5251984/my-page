
使用思維鏈，提高問題回答的正確率

[參考來源](https://github.com/bklieger-groq/g1)

這是個python的範例，我改用Javascript寫，並抽出重點

```
const messages = [
    {
        role: "system",
        content: `你是一個專業的 AI 助手，會一步一步解釋你的推理過程。對每一步，請提供一個描述你在該步驟中所做工作的標題，並附上內容。決定你是否需要再進行一步，或是已經準備好給出最終答案。請在句尾回應'next_action'（'continue' 或 'final_answer'）。盡可能使用多的推理步驟，至少 3 步。意識到你作為大型語言模型（LLM）的限制，以及你能做和不能做的事情。在你的推理過程中，包含對替代答案的探索。考慮到你可能是錯的，如果你的推理有錯，錯在哪裡。徹底測試所有其他可能性。你可能是錯的。當你說你正在重新檢視時，實際上要重新檢視，並使用另一種方法來進行。不要只是說你正在重新檢視。使用至少 3 種方法來推導答案。使用最佳實踐。`,
    },
    { 
        role: "user", 
        content: ${prompt} 
    },
    {
        role: "assistant",
        content: "謝謝！我現在將按照我的指示一步一步思考，從分解問題後的開始部分開始。",
    },
]

```

上面的system prompt，讓AI使用3種方式驗證自己寫出來的東西對不對。

在System prompt後，就是發問並請AI開始整理回答

```
// 思維鏈程式碼

let stepCount = 1;
let totalThinkingTime = 0;

while (true) {
    const startTime = Date.now();
    const stepData = await makeApiCall(messages, 300);
    
    const endTime = Date.now();
    const thinkingTime = (endTime - startTime) / 1000;
    totalThinkingTime += thinkingTime;

    messages.push({ role: "assistant", content: stepData });

    if (stepData.indexOf("final_answer") >= 0 || stepCount > 3) {
        break;
    }
    
    stepCount += 1;
}

```

整段邏實其實很簡單，就是把AI回答內容再丟回給他反思，直到AI覺得答案是正確的為止。

下面兩個近期用來評論AI的問題，在使用思維鏈的方式後， `llama-3.1-8b` `gemma2-9b-it` 甚至 `gemma-7b-it` 都有不錯的正確率

```
`strawberry`單字中裡有幾個`r`
```

與

```
3.11 與 3.8 這兩個數字誰比較大？
```

已下是回傳的對話記錄：

**步驟 1：問題理解**

我需要計算 `strawberry` 這個單字中有多少個 `r`。這個問題很簡單，但是我會一步一步地思考。

**步驟 2：字串分解**

我會把 `strawberry` 這個單字分解成每個字母：s-t-r-a-w-b-e-r-r-y。現在，我可以開始計算 `r` 的數量了。

**步驟 3：計算 `r` 的數量**

從分解的字串中，我可以看到有 3 個 `r`。因此，我可以得出結論：`strawberry` 這個單字中有 3 個 `r`。

**步驟 4：檢查替代答案**

我會檢查一下是否有其他可能的答案。例如，如果我錯過了某個 `r`，或者如果我計算錯誤了。然而，根據我的計算，答案是 3。

**步驟 5：確認答案**\n我會確認一下我的答案是否正確。根據我的計算，答案是 3。因此，我可以得出結論：`strawberry` 這個單字中有 3 個 `r`。

**結論**

根據我的計算，我可以得出結論：`strawberry` 這個單字中有 3 個 `r`。

**最終答案是**

`strawberry` 這個單字中有 3 個 `r`。