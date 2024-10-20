---
layout: post
title:  "一起來用 Structured Outputs"
date:   2024-10-20 23:42:00 +0900
categories: chatgpt
---

[一起來用 LLM Function Calling](./function-calling)

[OpenAI Structured Outputs](https://platform.openai.com/docs/guides/structured-outputs)

這是OpenAI在2024，是Function Calling推出一年後再一次對輸出格式的重大更新。

我主要做翻譯工具為主，所以介紹的案例會偏向翻譯應用這塊。

### Structured Outputs

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

#### 情境：使用Structured Outputs，將翻譯結果指定為多個語系。

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
            "content": "句子\"請問最近的車站怎麼走?\"，完整翻譯，不要發音、注解。"
        }
    ],
    "response_format": {
        "type": "json_schema",
        "json_schema": {
            "name": "split_text",
            "schema": {
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
                        "description": "Englisg reslut"
                    }
                },
                "required": [
                    "zh-CN"
                ]
            }
        }
    }
}
```

就規格上可以看到`response_format`是這次輸出的重點參數。

基本上就都是固定參數，會變動的就 `json_schema.name` 跟 `json_schema.schema`

`json_schema.name` 顧名思義就是這個 json_schema 的名稱。

而 `json_schema.schema` 就很重要了，你要決定json輸出是 object 或 array！

`json_schema.schema.properties` 開始，就是描述複雜的json回應格式了，就看每個情境來決定。

以我的情況來說，我需要每個語種的翻譯結果.

來看輸出，耗時1339ms
```
{
    "message": {
        "role": "assistant",
        "content": "{\"zh-CN\":\"请问最近的车站怎么走？\",\"jp-JP\":\"最近の駅に行くにはどうすればいいですか？\",\"en-US\":\"How do I get to the nearest station?\"}",
        "refusal": null
    },
    "usage": {
        "prompt_tokens": 120,
        "completion_tokens": 45,
        "total_tokens": 165,
    }
}

//特別抽出結果
{
  'zh-CN': '请问最近的车站怎么走？',
  'jp-JP': '最近の駅に行くにはどうすればいいですか？',
  'en-US': 'How do I get to the nearest station?'
}

```

可以看到，他會按照我指定的樣子產出給我，方便我們做後處理。

有趣的事，我的prompt裡並沒有指定要輸出的語言，但他也能夠判斷要輸出的語言給我

#### 情境：使用Structured Outputs，將翻譯結果指定沒有在輸出格式裡的語系。

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
            "content": "句子\"請問最近的車站怎麼走?\"，請用'th-TH泰語'完整翻譯，不要發音、注解。"
        }
    ],
    "response_format": {
        "type": "json_schema",
        "json_schema": {
            "name": "split_text",
            "schema": {
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
                        "description": "Englisg reslut"
                    }
                },
                "required": [
                    "zh-CN"
                ]
            }
        }
    }
}
```
我刻意要求輸出泰語的翻譯，並強制要求`zh-CN`是JSON輸出的必要項目時。

輸出，耗時873ms
```
{
    "message": {
        "role": "assistant",
        "content": "{\"en-US\":\"How do I get to the nearest station?\",\"zh-CN\":\"请问最近的车站怎么走?\",\"jp-JP\":\"最近の駅にはどうやって行けばいいですか?\",\"th-TH\":\"บอกทางไปสถานีที่ใกล้ที่สุดหน่อยได้ไหมครับ/ค่ะ?\"}",
        "refusal": null
    },
    "usage": {
        "prompt_tokens": 129,
        "completion_tokens": 70,
        "total_tokens": 199
    }
}

//特別抽出結果
{
  'en-US': 'How do I get to the nearest station?',
  'zh-CN': '请问最近的车站怎么走?',
  'jp-JP': '最近の駅にはどうやって行けばいいですか?',
  'th-TH': 'บอกทางไปสถานีที่ใกล้ที่สุดหน่อยได้ไหมครับ/ค่ะ?'
}
```

依照我要的格式輸出外，還把指定格式內的參數也產出了。

為什麼跟上一次實驗結果一樣，沒有指定的參數也出來了？

原因在這：[Tips and best practices](https://platform.openai.com/docs/guides/structured-outputs/best-practices)

OpenAI有規定，要強制把所有參數都放到`required`裡，看來他們後端有自己也做了這個動作。

#### 情境：使用Structured Outputs，沒有指定的語言就不要輸出。

根據OpenAI的說明，可以把 `properties.paramter.type`指定成 `["string", "null"]`

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
            "content": "句子\"請問最近的車站怎麼走?\"，請用'th-TH泰語'完整翻譯，不要發音、注解。"
        }
    ],
    "response_format": {
        "type": "json_schema",
        "json_schema": {
            "name": "split_text",
            "schema": {
                "type": "object",
                "properties": {
                    "zh-CN": {
                        "type": "string",
                        "description": "Chinese reslut"
                    },
                    "jp-JP": {
                        "type": ["string", "null"],
                        "description": "Japanese reslut"
                    },
                    "en-US": {
                        "type": ["string", "null"],
                        "description": "Englisg reslut"
                    }
                },
                "required": [
                    "zh-CN"
                ]
            }
        }
    }
}

```

輸出：耗時1603ms
```
{
    "message": {
        "role": "assistant",
        "content": "{\"zh-CN\":\"请问最近的车站怎么走?\",\"jp-JP\":null,\"en-US\":null,\"th-TH\":\"กรุณาถามว่าไปสถานีที่ใกล้ที่สุดได้อย่างไร?\"}",
        "refusal": null
    },
    "usage": {
        "prompt_tokens": 135,
        "completion_tokens": 48,
        "total_tokens": 183
    }
}
//特別抽出結果
{
    "zh-CN":"请问最近的车站怎么走?",
    "jp-JP":null,
    "en-US":null,
    "th-TH":"กรุณาถามว่าไปสถานีที่ใกล้ที่สุดได้อย่างไร?"
}
```

在不屬於我指定範圍內的語言，他就會轉成null了

#### 情境：使用Structured Outputs，但不請他翻譯，我問他別的事情

```
{
    "model": "gpt-4o-mini",
    "messages": [
        {
            "role": "user",
            "content": "今天的台北好冷，你有沒有推薦的保暖方法？"
        }
    ],
    "response_format": {
        "type": "json_schema",
        "json_schema": {
            "name": "split_text",
            "schema": {
                "type": "object",
                "properties": {
                    "zh-CN": {
                        "type": "string",
                        "description": "Chinese reslut"
                    },
                    "jp-JP": {
                        "type": ["string", "null"],
                        "description": "Japanese reslut"
                    },
                    "en-US": {
                        "type": ["string", "null"],
                        "description": "Englisg reslut"
                    }
                },
                "required": [
                    "zh-CN","jp-JP","en-US"
                ]
            }
        }
    }
}
```

輸出：2170ms
```
{
    "message": {
        "role": "assistant",
        "content": "{\"en-US\":[\"Wear layers of clothing to trap heat, such as thermal shirts, sweaters, and insulating jackets.\",\"Use a warm scarf, hat, and gloves to keep extremities warm.\",\"Drink hot beverages like tea or coffee to stay warm from the inside.\",\"Consider using a space heater or electric blanket at home.\",\"Snuggle up with a warm blanket while watching movies or reading a book.\"],\"jp-JP\":[null],\"zh-CN\":\"可以尝试以下几种保暖方法：\\n\\n1. 多穿几层衣服，选择保暖的内衣、毛衣和外套。\\n2. 戴上围巾、帽子和手套，以保暖手脚部位。\\n3. 喝热饮如茶或咖啡，让身体从内部保持温暖。\\n4. 在家使用取暖器或电热毯。\\n5. 看电影或阅读时，裹上毛毯。\"}",
        "refusal": null
    },
    "usage": {
        "prompt_tokens": 97,
        "completion_tokens": 200,
        "total_tokens": 297
    }
}

```

他還是回傳了我要的json格式！因為我的任務其實很單純，所以LLM會理解成要一個方案後，給我一些語系上的回傳。

不過 OpenAI文件有提到例外處理還是要做！ [拒絕結構化輸出](https://platform.openai.com/docs/guides/structured-outputs/refusals)

#### Structured Outputs + Function Calling = 要你命3000

每個都能獨當一面的功能摻在一起後，會變成`要你命3000`還是`撒尿牛丸`？

```
{
    "model": "gpt-4o-mini",
    "messages": [
        {
            "role": "user",
            "content": "今天的台北好冷，我想買最大的外套。你那邊有沒有推薦的保暖方法？"
        }
    ],
    "tools": [
        {
            "type": "function",
            "function": {
                "name": "pick_size",
                "description": "Call this if the user specifies which size they want",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "size": {
                            "type": "string",
                            "enum": [
                                "S",
                                "M",
                                "L",
                                "XL"
                            ],
                            "description": "The size of the t-shirt that the user would like to order"
                        }
                    },
                    "required": [
                        "size"
                    ],
                    "additionalProperties": false
                }
            }
        }
    ],
    "tool_choice": "auto",
    "response_format": {
        "type": "json_schema",
        "json_schema": {
            "name": "split_text",
            "schema": {
                "type": "object",
                "properties": {
                    "zh-CN": {
                        "type": "string",
                        "description": "Chinese reslut"
                    },
                    "jp-JP": {
                        "type": [
                            "string",
                            "null"
                        ],
                        "description": "Japanese reslut"
                    },
                    "en-US": {
                        "type": [
                            "string",
                            "null"
                        ],
                        "description": "Englisg reslut"
                    }
                },
                "required": [
                    "zh-CN",
                    "jp-JP",
                    "en-US"
                ]
            }
        }
    }
}

```

輸出：耗時 3370ms
```
{
    "choices": [
        {
            "index": 0,
            "message": {
                "role": "assistant",
                "content": "{\"en-US\":[null],\"jp-JP\":[null],\"zh-CN\":\"在台北寒冷的天气中，除了购买最大的外套，你可以尝试以下几种保暖的方法：\\n\\n1. **分层穿衣**：穿多层衣物，如长袖衬衫、毛衣和外套，可以更好地保持体温。\\n2. **使用围巾和手套**：围巾可以保护脖子，手套则保暖双手。\\n3. **选用保暖材料**：如羊毛、羽绒或厚棉材料的衣物，能有效隔绝寒冷。\\n4. **保暖鞋袜**：穿保暖鞋和厚袜子，确保脚部的温暖。\\n5. **喝热饮**：喝热茶、热巧克力等可以让身体内部升温。\\n6. **活动身体**：适当运动可以帮助提升体温，避免长时间静坐。 \\n\\n希望这些方法能帮助你在寒冷的台北保持温暖！\"}",
                "refusal": null
            },
            "logprobs": null,
            "finish_reason": "stop"
        }
    ],
    "usage": {
        "prompt_tokens": 168,
        "completion_tokens": 242,
        "total_tokens": 410
    }
}
```

換個問法

```
{
    "choices": [
        {
            "index": 0,
            "message": {
                "role": "assistant",
                "content": null,
                "tool_calls": [
                    {
                        "id": "call_WshjoFjxhiN84EFE3S8lFGHt",
                        "type": "function",
                        "function": {
                            "name": "pick_size",
                            "arguments": "{\"size\": \"XL\"}"
                        }
                    }
                ],
                "refusal": null
            },
            "logprobs": null,
            "finish_reason": "tool_calls"
        }
    ],
    "usage": {
        "prompt_tokens": 170,
        "completion_tokens": 30,
        "total_tokens": 200
    }
}

```

Seeeeee！ 看到沒，`要你命3000`啊！

特別把 `choices[0].finish_reason` 拿出來講

如果這一步採`Structured Outputs`的話，`finish_reason`是 stop

如果是採用`Function Calling`的話，`finish_reason`是tool_calls

#### 結論

Structured Outputs是號稱100%輸出成你想要的樣子，我的案例實測下來確實是這個樣子，還能給出一些些驚喜。

這樣能有效的讓AI介入開發的環節之內，有效的利用回傳內容做後處理。

Structured Outputs與Function Calling比較

共同點
1. 能夠有效產出json格式，並指定自己的參數上去
2. 都會照成token數上昇，但比Structured Outputs節省一些
3. 都會造成回應時間拉長，原本880ms，變成1300~1600ms左右

不同點
1. Function Calling會因為LLM選擇tool的不同，產出不同的結果
2. Function Calling產出結果的處理步驟會複雜一些
3. Function Calling彈性較大，tools的工具庫可以自由開發

Structured Outputs與Function Calling同時使用的話，不能保證會是哪一個結果被輸出哦！