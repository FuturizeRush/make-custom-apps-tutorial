# API 參考手冊

## 本章概要

本章提供 Make Custom Apps API 的完整參考文檔，包含所有可用的配置選項、參數類型、指令和屬性的詳細說明。這是開發過程中的重要查閱資料。

## 基礎配置

### app.json

應用程式的主要配置檔案。

```json
{
    "name": "string",              // 必填，應用程式識別名稱
    "label": "string",             // 必填，顯示名稱
    "version": "number",           // 必填，版本號
    "description": "string",       // 選填，應用程式描述
    "theme": "string",            // 選填，主題顏色 (HEX)
    "language": "string",         // 選填，預設語言
    "private": "boolean",         // 選填，是否為私有應用
    "countries": ["string"],      // 選填，可用地區
    "apiVersion": "number"        // 選填，API 版本
}
```

### Base 配置

所有模組共用的基礎設定。

```json
{
    "baseUrl": "string",          // API 基礎 URL
    "headers": {                  // 共用請求標頭
        "key": "value"
    },
    "qs": {                      // 共用查詢參數
        "key": "value"
    },
    "body": {},                  // 共用請求主體
    "response": {                // 回應處理
        "error": {},             // 錯誤處理
        "valid": "boolean"       // 驗證邏輯
    },
    "log": {                     // 日誌設定
        "sanitize": ["path"]     // 敏感資訊清理
    }
}
```

## 模組類型

### Action

執行單一操作的模組。

```json
{
    "name": "string",
    "label": "string",
    "description": "string",
    "type": "action",
    "typeOptions": {
        "actionType": "create|read|update|delete|other"
    },
    "connection": "string|null",
    "parameters": [],
    "mappable": [],
    "communication": {},
    "interface": []
}
```

### Search

搜尋並返回多筆資料的模組。

```json
{
    "type": "search",
    "communication": {
        "response": {
            "iterate": "string",      // 陣列路徑
            "output": {},            // 輸出映射
            "limit": "number",       // 結果限制
            "pagination": {}         // 分頁設定
        }
    }
}
```

### Trigger

定期檢查新資料的模組。

```json
{
    "type": "trigger",
    "epoch": {},                     // 初始時間點設定
    "communication": {
        "response": {
            "trigger": {
                "id": "string",      // 唯一識別
                "date": "date"       // 時間戳記
            }
        }
    }
}
```

### Instant Trigger

即時接收事件的模組。

```json
{
    "type": "trigger",
    "typeOptions": {
        "instant": true
    },
    "webhook": "string",             // 關聯的 Webhook
    "parameters": []                 // 只能使用靜態參數
}
```

### Responder

回應 Webhook 請求的模組。

```json
{
    "type": "responder",
    "communication": {
        "response": {
            "status": "number",      // HTTP 狀態碼
            "headers": {},          // 回應標頭
            "body": {}              // 回應內容
        }
    }
}
```

### Universal

自訂 API 呼叫的模組。

```json
{
    "type": "universal",
    "expects": [],                   // 預期的輸入參數
    "interface": []                  // 動態介面定義
}
```

## 參數類型

### 基本類型

| 類型 | 說明 | 屬性 |
|------|------|------|
| `text` | 文字輸入 | `multiline`, `maxLength`, `minLength` |
| `number` | 數字輸入 | `min`, `max`, `step` |
| `boolean` | 布林值 | - |
| `date` | 日期時間 | `time`, `timezone` |
| `email` | 電子郵件 | - |
| `url` | URL 網址 | - |
| `password` | 密碼 | - |
| `color` | 顏色選擇器 | - |

### 選擇類型

#### Select

```json
{
    "type": "select",
    "multiple": false,               // 是否多選
    "options": [                     // 靜態選項
        {
            "label": "string",
            "value": "any"
        }
    ],
    "options": {                     // 動態選項 (RPC)
        "rpc": {
            "name": "string",
            "parameters": {}
        }
    }
}
```

### 複合類型

#### Array

```json
{
    "type": "array",
    "spec": {                        // 陣列項目規格
        "type": "string",
        "label": "string"
    },
    "min": "number",                 // 最少項目數
    "max": "number"                  // 最多項目數
}
```

#### Collection

```json
{
    "type": "collection",
    "spec": [                        // 子欄位定義
        {
            "name": "string",
            "type": "string",
            "label": "string"
        }
    ]
}
```

#### Buffer

```json
{
    "type": "buffer",                // 二進位資料
    "semantic": "file"               // 語意類型
}
```

#### Filter

```json
{
    "type": "filter",
    "options": {
        "fields": [],                // 可篩選欄位
        "operators": []              // 可用運算子
    }
}
```

## 通訊配置

### 請求設定

```json
{
    "url": "string",                 // 端點 URL
    "method": "GET|POST|PUT|DELETE|PATCH",
    "headers": {},                   // 請求標頭
    "qs": {},                       // 查詢參數
    "body": {},                     // 請求主體
    "type": "json|urlencoded|multipart|raw",
    "aws": {},                      // AWS 簽名
    "oauth": {},                    // OAuth 簽名
    "cert": {},                     // 憑證設定
    "ca": "string",                 // CA 憑證
    "condition": "boolean"          // 執行條件
}
```

### 回應處理

```json
{
    "response": {
        "output": {},                // 輸出映射
        "iterate": "string",         // 迭代路徑
        "error": {                   // 錯誤處理
            "type": "string",
            "message": "string"
        },
        "valid": "boolean",          // 驗證邏輯
        "limit": "number"            // 結果限制
    }
}
```

### 臨時變數

```json
{
    "temp": {
        "variableName": {            // 臨時請求
            "url": "string",
            "method": "string",
            "response": {}
        }
    }
}
```

## 連接類型

### Basic

```json
{
    "type": "basic",
    "parameters": [],                // 連接參數
    "communication": {}              // 驗證邏輯
}
```

### OAuth 2.0

```json
{
    "type": "oauth2",
    "scope": [],                     // 權限範圍
    "communication": {
        "authorize": {},             // 授權設定
        "token": {},                 // Token 交換
        "refresh": {},               // Token 更新
        "info": {},                  // 使用者資訊
        "invalidate": {}             // 撤銷連接
    }
}
```

### OAuth 1.0

```json
{
    "type": "oauth1",
    "communication": {
        "requestToken": {},          // 請求 Token
        "authorize": {},             // 授權
        "accessToken": {},           // 存取 Token
        "info": {}                   // 使用者資訊
    }
}
```

## IML 變數

### 基本變數

| 變數 | 說明 | 範例 |
|------|------|------|
| `{{parameters}}` | 模組參數 | `{{parameters.name}}` |
| `{{connection}}` | 連接資料 | `{{connection.apiKey}}` |
| `{{body}}` | 回應主體 | `{{body.data}}` |
| `{{headers}}` | 回應標頭 | `{{headers['content-type']}}` |
| `{{statusCode}}` | HTTP 狀態碼 | `{{statusCode}}` |
| `{{now}}` | 當前時間 | `{{now}}` |
| `{{timestamp}}` | Unix 時間戳 | `{{timestamp}}` |

### 特殊變數

| 變數 | 說明 | 使用位置 |
|------|------|----------|
| `{{item}}` | 迭代項目 | iterate 內部 |
| `{{webhook}}` | Webhook 資訊 | Webhook 模組 |
| `{{epoch}}` | 觸發器時間點 | Trigger 模組 |
| `{{pagination}}` | 分頁資訊 | Search 模組 |
| `{{temp}}` | 臨時變數 | 多步驟請求 |

### OAuth 變數

| 變數 | 說明 |
|------|------|
| `{{oauth.scope}}` | 請求的權限範圍 |
| `{{oauth.redirectUri}}` | 回調 URL |
| `{{oauth.state}}` | 狀態參數 |
| `{{query}}` | 查詢參數 |
| `{{data}}` | 連接儲存的資料 |

## 驗證規則

### 文字驗證

```json
{
    "validate": {
        "pattern": "string",         // 正規表達式
        "min": "number",            // 最小長度
        "max": "number",            // 最大長度
        "enum": ["values"],         // 允許值
        "message": "string"         // 錯誤訊息
    }
}
```

### 數字驗證

```json
{
    "validate": {
        "min": "number",            // 最小值
        "max": "number",            // 最大值
        "integer": "boolean",       // 是否整數
        "message": "string"
    }
}
```

## 錯誤類型

| 類型 | 說明 | 使用場景 |
|------|------|----------|
| `DataError` | 資料錯誤 | 一般資料問題 |
| `ConnectionError` | 連接錯誤 | 網路或服務問題 |
| `RateLimitError` | 速率限制 | 請求過於頻繁 |
| `IncompleteDataError` | 資料不完整 | 缺少必要資料欄位 |
| `DuplicateDataError` | 重複資料 | 資料已存在 |

## 日誌清理

```json
{
    "log": {
        "sanitize": [
            "request.headers.authorization",
            "request.headers.x-api-key",
            "response.body.password",
            "response.body.users[].email",
            "response.body.data.*.token"
        ]
    }
}
```

## 進階功能

### 條件執行

```json
{
    "condition": "{{parameters.enabled === true}}",
    "communication": {
        // 只在條件成立時執行
    }
}
```

### 動態 URL

```json
{
    "url": "{{if(parameters.sandbox, 'https://sandbox.api.com', 'https://api.com')}}/endpoint"
}
```

### 批次處理

```json
{
    "response": {
        "iterate": "{{body.items}}",
        "output": {
            "success": "{{item.status === 'success'}}",
            "data": "{{if(item.status === 'success', item.result, null)}}",
            "error": "{{if(item.status === 'failed', item.error, null)}}"
        }
    }
}
```

## 最佳實踐建議

1. **命名規範**
   - 使用 camelCase 命名
   - 名稱要有描述性
   - 避免使用保留字

2. **錯誤處理**
   - 總是提供錯誤處理
   - 使用適當的錯誤類型
   - 提供有意義的錯誤訊息

3. **效能優化**
   - 使用分頁處理大量資料
   - 實作適當的快取
   - 避免不必要的請求

4. **安全性**
   - 清理敏感資訊
   - 驗證所有輸入
   - 使用 HTTPS

## 版本相容性

| Make 版本 | API 版本 | 支援功能 |
|-----------|----------|----------|
| 2.0+ | 2 | 所有功能 |
| 1.0-1.9 | 1 | 基本功能 |

---

[← 上一章：複雜資料處理範例](../05_範例程式/複雜資料處理範例.md) | [下一章：IML 函數速查表 →](IML_函數速查表.md)