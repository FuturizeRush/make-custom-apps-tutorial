# Webhook 實作

## 本章概要

本章深入探討 Webhook 和 Instant Trigger 的實作方式。您將學會如何建立即時觸發器，處理 Webhook 驗證，以及實作專用（Dedicated）和共享（Shared）Webhook。

## 學習目標

- 理解 Webhook 在 Make 中的運作機制
- 實作 Instant Trigger 模組
- 處理 Webhook 驗證流程
- 區分並實作專用和共享 Webhook
- 使用 Responder 模組回應 Webhook

## 前置知識

- 了解 HTTP Webhook 的基本概念
- 熟悉 Instant Trigger 模組類型
- 具備 API 整合經驗

## 內容主體

### Webhook 概述

Webhook 讓 Make 能夠即時接收外部服務的事件通知，是實現即時自動化的關鍵技術。

**Webhook 的優勢**：
- 即時回應，無需輪詢
- 減少 API 呼叫次數
- 提升效能和使用者體驗
- 支援雙向通訊

**使用原則**：
> Webhook 必須搭配 Instant Trigger 使用，不能單獨存在。

### Webhook 類型

#### 1. 專用 Webhook（Dedicated）

每個使用者有獨立的 Webhook URL，只接收該使用者的通知。

**特點**：
- 90% 以上的服務使用此類型
- 每個連接有獨立的 URL
- 支援自動註冊（Attached）或手動註冊

#### 2. 共享 Webhook（Shared）

所有使用者共用一個 Webhook URL，需要識別通知屬於哪個使用者。

**特點**：
- 需要實作 uid 識別機制
- 必須在應用程式發布後才能使用
- 適用於服務只支援單一 Webhook 的情況

### 實作 Instant Trigger

#### 基本結構

```json
{
    "name": "watchNewOrder",
    "label": "Watch New Orders",
    "description": "Triggers when a new order is created",
    "type": "trigger",
    "typeOptions": {
        "instant": true
    },
    "webhook": "orderWebhook"
}
```

#### 靜態參數

Instant Trigger 只能使用靜態參數：

```json
{
    "parameters": [
        {
            "name": "eventTypes",
            "type": "select",
            "label": "Event Types",
            "multiple": true,
            "options": [
                {"label": "Order Created", "value": "order.created"},
                {"label": "Order Updated", "value": "order.updated"},
                {"label": "Order Cancelled", "value": "order.cancelled"}
            ]
        }
    ]
}
```

#### 定義介面

```json
{
    "interface": [
        {
            "name": "id",
            "type": "text",
            "label": "Order ID"
        },
        {
            "name": "status",
            "type": "text",
            "label": "Status"
        },
        {
            "name": "createdAt",
            "type": "date",
            "label": "Created Date"
        }
    ]
}
```

### 建立 Webhook

#### 1. 基本 Webhook 定義

```json
{
    "name": "orderWebhook",
    "label": "Order Events Webhook",
    "type": "webhook"
}
```

#### 2. 處理 Webhook 資料

```json
{
    "communication": {
        "output": {
            "id": "{{body.order_id}}",
            "status": "{{body.status}}",
            "amount": "{{body.total_amount}}",
            "items": "{{body.line_items}}"
        }
    }
}
```

#### 3. 使用 iterate 處理批次資料

當 Webhook 一次傳送多筆資料時：

```json
{
    "communication": {
        "iterate": "{{body.events}}",
        "output": {
            "id": "{{item.id}}",
            "type": "{{item.event_type}}",
            "data": "{{item.payload}}"
        }
    }
}
```

### Webhook 驗證

許多服務在註冊 Webhook 前需要驗證端點。

#### 驗證流程實作

```json
{
    "verification": {
        "condition": "{{if(body.challenge, true, false)}}",
        "respond": {
            "status": 200,
            "type": "json",
            "body": {
                "challenge": "{{body.challenge}}"
            }
        }
    }
}
```

#### 複雜驗證範例

```json
{
    "verification": {
        "condition": "{{body.type === 'url_verification'}}",
        "respond": {
            "status": 200,
            "headers": {
                "Content-Type": "application/json"
            },
            "body": {
                "challenge": "{{body.challenge}}",
                "token": "{{body.token}}",
                "type": "url_verification"
            }
        }
    }
}
```

### 專用 Webhook - 自動註冊（Attached）

#### 1. 定義 Attach 程序

```json
{
    "attach": {
        "url": "https://api.example.com/webhooks",
        "method": "POST",
        "body": {
            "url": "{{webhook.url}}",
            "events": "{{parameters.eventTypes}}",
            "secret": "{{common.webhookSecret}}"
        },
        "response": {
            "data": {
                "webhookId": "{{body.id}}",
                "secret": "{{body.secret}}"
            }
        }
    }
}
```

**重要**：必須在 `response.data` 中保存 webhook ID，以供 detach 使用。

#### 2. 定義 Detach 程序

```json
{
    "detach": {
        "url": "https://api.example.com/webhooks/{{webhook.webhookId}}",
        "method": "DELETE"
    }
}
```

#### 3. 完整範例

```json
{
    "name": "eventWebhook",
    "label": "Event Webhook",
    "type": "webhook",
    "connection": "oauth2",
    "attach": {
        "url": "/webhooks",
        "method": "POST",
        "headers": {
            "Authorization": "Bearer {{connection.accessToken}}"
        },
        "body": {
            "target_url": "{{webhook.url}}",
            "event_types": "{{parameters.eventTypes}}",
            "description": "Make automation webhook"
        },
        "response": {
            "data": {
                "id": "{{body.webhook_id}}",
                "createdAt": "{{body.created_at}}"
            },
            "error": {
                "message": "[{{statusCode}}] {{body.error.message}}"
            }
        }
    },
    "detach": {
        "url": "/webhooks/{{webhook.id}}",
        "method": "DELETE",
        "headers": {
            "Authorization": "Bearer {{connection.accessToken}}"
        }
    },
    "communication": {
        "iterate": "{{if(isArray(body), body, [body])}}",
        "output": {
            "eventId": "{{item.id}}",
            "eventType": "{{item.type}}",
            "timestamp": "{{item.created_at}}",
            "data": "{{item.data}}"
        }
    }
}
```

### 專用 Webhook - 手動註冊

當 API 不支援自動註冊時：

```json
{
    "name": "manualWebhook",
    "label": "Manual Webhook",
    "type": "webhook",
    "connection": null,
    "communication": {
        "output": {
            "id": "{{body.id}}",
            "event": "{{body.event}}",
            "data": "{{body.payload}}"
        }
    }
}
```

使用者需要手動複製 Webhook URL 到外部服務。

### 共享 Webhook 實作

#### 1. Connection 中定義 uid

```json
{
    "response": {
        "uid": "{{body.user_id}}",
        "data": {
            "accessToken": "{{body.access_token}}"
        }
    }
}
```

#### 2. Webhook 中使用 uid

```json
{
    "name": "sharedWebhook",
    "label": "Shared Events",
    "type": "webhook",
    "communication": {
        "uid": "{{body.user_id}}",
        "iterate": "{{body.events}}",
        "output": {
            "id": "{{item.id}}",
            "userId": "{{body.user_id}}",
            "data": "{{item}}"
        }
    }
}
```

### 在 Instant Trigger 中擷取額外資料

有時需要根據 Webhook 資料擷取更多資訊：

```json
{
    "name": "watchDetailedEvents",
    "type": "trigger",
    "typeOptions": {
        "instant": true
    },
    "webhook": "eventWebhook",
    "communication": {
        "url": "/events/{{payload.eventId}}/details",
        "method": "GET",
        "headers": {
            "Authorization": "Bearer {{connection.accessToken}}"
        },
        "response": {
            "output": {
                "id": "{{payload.eventId}}",
                "summary": "{{payload.data}}",
                "details": "{{body}}"
            }
        }
    }
}
```

### Responder 模組

Responder 用於回應 Webhook 發送者，實現雙向通訊。

#### 基本實作

```json
{
    "name": "respondToWebhook",
    "label": "Respond to Webhook",
    "type": "responder",
    "communication": {
        "response": {
            "status": 200,
            "headers": {
                "Content-Type": "application/json"
            },
            "body": {
                "received": true,
                "processedAt": "{{now}}",
                "result": "{{parameters.result}}"
            }
        }
    }
}
```

#### 使用場景範例

1. **Instant Trigger** 接收訂單
2. **Action 模組** 處理訂單
3. **Responder** 回傳處理結果

### 安全性考量

#### 1. 驗證 Webhook 簽名

```javascript
function verifyWebhookSignature(payload, signature, secret) {
    const expectedSignature = crypto
        .createHmac('sha256', secret)
        .update(JSON.stringify(payload))
        .digest('hex');
    
    return signature === expectedSignature;
}
```

#### 2. 在 Webhook 中實作

```json
{
    "verification": {
        "condition": "{{verifySignature(body, headers['x-signature'], webhook.secret)}}",
        "respond": {
            "status": 401,
            "body": {
                "error": "Invalid signature"
            }
        }
    }
}
```

### 除錯技巧

1. **檢查 Webhook URL**
   - 確認 URL 正確且可存取
   - 使用工具測試 Webhook 端點

2. **記錄原始資料**
   ```json
   {
       "output": {
           "raw": "{{body}}",
           "headers": "{{headers}}",
           "method": "{{method}}"
       }
   }
   ```

3. **處理錯誤**
   ```json
   {
       "respond": {
           "status": "{{if(error, 500, 200)}}",
           "body": {
               "success": "{{if(error, false, true)}}",
               "error": "{{error.message}}"
           }
       }
   }
   ```

## 實作練習

### 練習 1：建立完整的 Webhook 整合
實作一個包含驗證、自動註冊和取消註冊的 Webhook。

### 練習 2：處理批次 Webhook
建立能處理批次事件的 Webhook，每個事件產生一個 bundle。

### 練習 3：雙向通訊
使用 Instant Trigger 和 Responder 實作雙向通訊流程。

## 重點整理

- Webhook 必須搭配 Instant Trigger 使用
- 專用 Webhook 是最常見的類型
- 自動註冊需要實作 attach 和 detach
- 共享 Webhook 需要 uid 識別機制
- 驗證流程確保 Webhook 的可靠性
- Responder 實現雙向通訊

## 延伸閱讀

- [模組類型詳解](../02_核心概念/02_模組類型詳解.md) - 深入了解 Instant Trigger
- [錯誤處理與除錯](03_錯誤處理與除錯.md) - Webhook 除錯技巧
- [REST API 整合範例](../05_範例程式/REST_API_整合範例.md) - 完整的 Webhook 實作

---

[← 上一章：IML 函數開發](01_IML函數開發.md) | [下一章：錯誤處理與除錯 →](03_錯誤處理與除錯.md)