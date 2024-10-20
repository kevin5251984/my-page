---
layout: post
title:  "一起來用 Function Calling"
date:   2024-10-20 11:42:00 +0900
categories: chatgpt
---

[一起來用 LLM Function Calling](./2024-10-21-structured-outputs)

[OpenAI Function calling](https://platform.openai.com/docs/guides/function-calling)

這是OpenAI在2023/06推出的功能，基於文字生成後，能給相對穩定的輸出格式。

我主要做翻譯工具為主，所以介紹的案例會偏向翻譯應用這塊。

### Function Calling

#### 情境：需要將中文，翻譯成指定的多個語系。

先不使用工具提問
```
{
    "model": "gpt-4o-mini",
    "messages": [
        {
            "role": "system",
            "content": "You are a translation expert. I will ask you to translate a language into setting languages"
        },
        {
            "role": "user",
            "content": "句子\"請問最近的車站怎麼走?\"，請翻譯成'zh-CN簡體中文' 'jp-JP日本語' 'en-US美式英文' 完整翻譯，不要發音、注解。"
        }
    ]
}
```

回應，耗時 886ms
```
{
    "message": {
        "role": "assistant",
        "content": "
            zh-CN简体中文：请问最近的车站怎么走？
        
            jp-JP日本語：最近の駅へはどうやって行きますか？
        
            en-US美式英文：Excuse me, how do I get to the nearest train station?",
        "refusal": null
    },
    "usage": {
        "prompt_tokens": 82,
        "completion_tokens": 55,
        "total_tokens": 137
    }
}
```

他確實回應我要的答案，但這個答案你很難回到你的程式碼裡去做二次應用。比方說只顯示中文，或是要只顯示中文與日文。

接下來看 function calling

#### 情境：需要將中文，翻譯成指定的多個語系，並使用function calling。

```
{
    "model": "gpt-4o-mini",
    "messages": [
        {
            "role": "system",
            "content": "You are a translation expert. I will ask you to translate a language into setting languages"
        },
        {
            "role": "user",
            "content": "句子\"請問最近的車站怎麼走?\"，請翻譯成'zh-CN簡體中文' 'jp-JP日本語' 'en-US美式英文' 完整翻譯，不要發音、注解。"
        }
    ],
    "tools": [
        {
            "type": "function",
            "function": {
                "name": "split_text",
                "description": "Identify Chinese and other languages in text.",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "zh-CN": {
                            "type": "string",
                            "description": "Chinese reslut"
                        },
                        "jp-JP": {
                            "type": "string",
                            "description": "Japanese reslut"
                        },
                        "en-US": {
                            "type": "string",
                            "description": "English reslut"
                        },
                        "howManyLangs":{
                            "type": "number",
                            "description": "How many languages have been produced in total?"
                        }
                    },
                    "required": [
                        "zh-CN"
                    ]
                }
            }
        }
    ],
    "tool_choice": "auto"
}

```

就規格上可以看到，跟過往單純使用 `messages` 做為input來說，變的複雜不少，所以一般情況下不會使用到 function calling 的功能。

`tools` 工具箱，在這邊你可以定義各式個樣的工具，讓AI的回應時使用你的工具來回答。

這邊我做的是，將翻譯好的句字分類。

回應的結果，耗時1139ms

```
{
    "message": {
        "role": "assistant",
        "content": null,
        "tool_calls": [
            {
                "id": "call_IcRHiDV4SkZPx1Jyswmopw6u",
                "type": "function",
                "function": {
                    "name": "split_text",
                    "arguments": "{\"zh-CN\": \"请问最近的车站怎么走?\", \"jp-JP\": \"最近の駅へはどう行けばいいですか?\", \"en-US\": \"How do I get to the nearest station?\"}"
                }
            }
        ],
        "refusal": null
    },
    "usage": {
        "prompt_tokens": 161,
        "completion_tokens": 70,
        "total_tokens": 231
    },
}

//回應重點
{
  'zh-CN': '请问最近的车站怎么走?',
  'jp-JP': '最近の駅へはどう行けばいいですか?',
  'en-US': 'How do I get to the nearest station?'
}
```

他確實把翻譯結果用我指定的JSON格式回傳。但少了一個 `howManyLangs`，因為這不是我必定要輸出的格式。

但，function calling一定是萬能的嗎？ 讓我們看下去。

#### 情境：使用function calling計算有多少個輸出語種。

當我把 `howManyLangs` 加到 `required`後，就變這樣

輸入，省略部份
```
"required": [
    "zh-CN",
    "howManyLangs"
]
```

回應，耗時 1386ms
```
{
    "message": {
        "role": "assistant",
        "content": null,
        "tool_calls": [
            {
                "id": "call_MEGQYUKN3eSIoXpS0h0KCXkK",
                "type": "function",
                "function": {
                    "name": "split_text",
                    "arguments": "{\"zh-CN\": \"请问最近的车站怎么走?\"}"
                }
            },
            {
                "id": "call_Or3CdU0zhRFsAAAEQK3gdBtR",
                "type": "function",
                "function": {
                    "name": "split_text",
                    "arguments": "{\"jp-JP\": \"最近の駅への行き方を教えてください。\"}"
                }
            },
            {
                "id": "call_kHgrVkfrIF7NbcOWDoIAPijx",
                "type": "function",
                "function": {
                    "name": "split_text",
                    "arguments": "{\"en-US\": \"How do I get to the nearest station?\"}"
                }
            }
        ],
        "refusal": null
    }
}
```

看可看見，他一樣沒有把 `howManyLangs` 回傳給我，而且還把function拆成了三個回應給我...

所以程式的後處理很變得也很重要！

#### 情境：使用function calling，tools中有多種工具

再來，我們將 function 拆成三個好了，翻簡中、翻日文、翻英文，三個工具

```
{
    "model": "gpt-4o-mini",
    "messages": [
        {
            "role": "system",
            "content": "You are a translation expert. I will ask you to translate a language into setting languages"
        },
        {
            "role": "user",
            "content": "句子\"請問最近的車站怎麼走?\"，請翻譯成'zh-CN簡體中文' 'jp-JP日本語' 'en-US美式英文' 完整翻譯，不要發音、注解。"
        }
    ],
    "tools": [
        {
            "type": "function",
            "function": {
                "name": "split_chinese_text",
                "description": "Identify Chinese.",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "zh-CN": {
                            "type": "string",
                            "description": "Chinese reslut"
                        }
                    },
                    "required": [
                        "zh-CN"
                    ]
                }
            }
        },
        {
            "type": "function",
            "function": {
                "name": "split_japanese_text",
                "description": "Identify Japanese in text.",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "jp-JP": {
                            "type": "string",
                            "description": "Japanese reslut"
                        }
                    },
                    "required": [
                        "jp-JP"
                    ]
                }
            }
        },
        {
            "type": "function",
            "function": {
                "name": "split_englist_text",
                "description": "Identify English in text.",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "en-US": {
                            "type": "string",
                            "description": "English reslut"
                        }
                    },
                    "required": [
                        "en-US"
                    ]
                }
            }
        }
    ],
    "tool_choice": "auto"
}
```

回應，耗時 1666ms
```
{
    "message": {
        "role": "assistant",
        "content": null,
        "tool_calls": [
            {
                "id": "call_NJrlogGVEXKnNNBjXnBXZZK3",
                "type": "function",
                "function": {
                    "name": "split_chinese_text",
                    "arguments": "{\"zh-CN\": \"请问最近的车站怎么走?\"}"
                }
            },
            {
                "id": "call_bzatO4BZ4dhLwD4PVjIYDcCz",
                "type": "function",
                "function": {
                    "name": "split_japanese_text",
                    "arguments": "{\"jp-JP\": \"最近の駅にはどうやって行きますか？\"}"
                }
            },
            {
                "id": "call_QVAjPv33bpXizBwxxm0v9wbU",
                "type": "function",
                "function": {
                    "name": "split_englist_text",
                    "arguments": "{\"en-US\": \"How do I get to the nearest station?\"}"
                }
            }
        ],
        "refusal": null
    },
    "usage": {
        "prompt_tokens": 176,
        "completion_tokens": 96,
        "total_tokens": 272
    }
}
```

確確實實的用了三個function來回應我的問題，在做後處理上會方便不少。

#### 情境：使用function calling，只執行單一個工具

再來，我們如果有很多工具，但只要指定其中一個工作時，可以對`tool_choice`做改變

```
"tool_choice": {
    "type": "function", 
    "function": {
        "name": "split_chinese_text"
     }
}
```

回應，耗時770ms
```
{
    "message": {
        "role": "assistant",
        "content": null,
        "tool_calls": [
            {
                "id": "call_1loM3SzD5aGvILsHYcPCT2bf",
                "type": "function",
                "function": {
                    "name": "split_chinese_text",
                    "arguments": "{\"zh-CN\":\"请问最近的车站怎么走?\"}"
                }
            }
        ],
        "refusal": null
    },
    "usage": {
        "prompt_tokens": 187,
        "completion_tokens": 15,
        "total_tokens": 202
    }
}
```

#### 情境：任務與function calling無關時。

最後，我們再實驗，如果你的題目壓根與翻譯無關時，會發生什麼事

```
{
    "model": "gpt-4o-mini",
    "messages": [
        {
            "role": "user",
            "content": "今天的台北好冷，你有沒有推薦的保暖方法？"
        }
    ],
    "tools": [
        {
            "type": "function",
            "function": {
                "name": "split_text",
                "description": "Identify Chinese and other languages in text.",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "zh-CN": {
                            "type": "string",
                            "description": "Chinese reslut"
                        },
                        "jp-JP": {
                            "type": "string",
                            "description": "Japanese reslut"
                        },
                        "en-US": {
                            "type": "string",
                            "description": "English reslut"
                        }
                    },
                    "required": [
                    ]
                }
            }
        }
    ],
    "tool_choice": "auto"
}

```

回應，耗時 3920ms

```
{
    "message": {
        "role": "assistant",
        "content": "在寒冷的台北，這裡有一些保暖的方法可以幫助你保持溫暖：\n\n1. **穿著層疊衣物**：多層次的穿著可以幫助保持體熱，內層可以選擇保暖內衣，中層穿著毛衣或厚襯衫，外層則選擇防風或防水的外套。\n\n2. **使用熱水袋**：在睡覺或坐在沙發上時，可以使用熱水袋來保暖，放在身邊或腳下，能感覺到很舒適。\n\n3. **喝熱飲**：熱茶、熱巧克力或湯品都能讓身體感覺更暖和，特別是生薑茶，還有助於驅寒。\n\n4. **保持活動**：適當的運動可以促進血液循環，讓身體更溫暖，即使是在室內做一些簡單的伸展運動或瑜伽也是不錯的選擇。\n\n5. **使用厚毛毯**：在家中有一條厚毛毯可以隨時裹住自己，特別是在看電視或閱讀時，能有效保暖。\n\n6. **窗戶密封**：檢查窗戶是否有漏風的地方，可以使用窩心的窗簾或窗戶密封條來減少冷空氣的進入。\n\n希望這些方法能讓你在寒冷的天氣中感覺更舒適！",
        "refusal": null
    },
    "usage": {
        "prompt_tokens": 86,
        "completion_tokens": 353,
        "total_tokens": 439
    }
}
```

他就完全不會照你的設定來回答，因為題目與文字識別沒有關係了。

#### 情境：指定輸出的文字

OpenAI裡給了一個有趣的案例是，設定衣服的大小 function，如果沒有指定的話，function可能就是回應最大、很大、特大這些文字

```
{
    "model": "gpt-4o-mini",
    "messages": [
        {
            "role": "user",
            "content": "我剛剛去店裡買的衣服太大了，請問能換貨嗎？"
        },
        {
            "role": "assistant",
            "content": "請問你買的是什麼尺吋?"
        },
        {
            "role": "user",
            "content": "吊牌不見了，但我記得是最大的"
        }
    ],
    "tools": [
        {
            "type": "function",
            "function": {
                "name": "pick_tshirt_size",
                "description": "Call this if the user specifies which size t-shirt they want",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "size": {
                            "type": "string",
                            "enum": ["S", "M", "L", "XL"],
                            "description": "The size of the t-shirt that the user would like to order"
                        }
                    },
                    "required": ["size"],
                    "additionalProperties": false
                }
            }
        }
    ],
    "tool_choice": "auto"
}
```

有加上 enum的回應
```
{
    "message": {
        "role": "assistant",
        "content": null,
        "tool_calls": [
            {
                "id": "call_v0NLwZPkfhCoZWDUIW80HlY6",
                "type": "function",
                "function": {
                    "name": "pick_tshirt_size",
                    "arguments": "{\"size\":\"XL\"}"
                }
            }
        ],
        "refusal": null
    }
}
```

沒有加上 enum的回應

`笑死`

```
{
    "message": {
        "role": "assistant",
        "content": null,
        "tool_calls": [
            {
                "id": "call_BiojkLcAsOUQ6ImBHOYex4JF",
                "type": "function",
                "function": {
                    "name": "pick_tshirt_size",
                    "arguments": "{\"size\":\"最大\"}"
                }
            }
        ],
        "refusal": null
    }
}
```

#### 總結

Function calling確實能夠大大提升AI回答的可處理性，但必要的例外處理也一定要加上，避免非遇期的錯誤發生。

但是實驗中有發現兩件要注意的事

1. function calling也確實會造成 token的數量上升，一開始可能不會有影響，但多輪談話後，成本上的增長也是要注意的。

2. function calling會造成回應時間拉長，從原本的800ms，增長到1200~1700ms不等，如果是做即時應用的話，要把這個因素考量進去。