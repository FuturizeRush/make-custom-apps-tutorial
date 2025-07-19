# REST API 整合範例

## 本章概要

本章提供一個完整的 REST API 整合範例，展示如何建立一個功能完整的 Make Custom App。我們將以一個任務管理 API 為例，實作包含 CRUD 操作、認證、分頁、錯誤處理等功能的應用程式。

## 學習目標

- 建立完整的 REST API 整合應用程式
- 實作所有 CRUD 操作（Create、Read、Update、Delete）
- 處理認證和授權
- 實作分頁和搜尋功能
- 設計良好的錯誤處理機制

## 前置知識

- 了解 REST API 基本概念
- 熟悉 HTTP 方法（GET、POST、PUT、DELETE）
- 掌握基本的模組開發技巧

## 內容主體

### 應用程式概覽

我們將建立一個名為「Task Manager Pro」的應用程式，整合虛擬的任務管理 API。

**API 基本資訊**：
- Base URL: `https://api.taskmanager.example.com/v1`
- 認證方式: API Key（Header）
- 支援功能: 任務的 CRUD 操作、專案管理、使用者管理

### 專案結構

```
task-manager-pro/
├── app.json                    # 應用程式配置
├── base.imljson               # Base 配置
├── connections/
│   └── apiKey.imljson         # API Key 連接
├── modules/
│   ├── createTask.imljson     # 建立任務
│   ├── getTask.imljson        # 取得任務
│   ├── updateTask.imljson     # 更新任務
│   ├── deleteTask.imljson     # 刪除任務
│   ├── listTasks.imljson      # 列出任務
│   └── watchNewTasks.imljson  # 監控新任務
├── rpc/
│   ├── listProjects.imljson   # 列出專案
│   └── listUsers.imljson      # 列出使用者
└── functions/
    └── taskHelpers.iml        # 輔助函數
```

### 步驟 1：應用程式配置

#### app.json

```json
{
    "name": "task-manager-pro",
    "label": "Task Manager Pro",
    "version": 1,
    "description": "完整的任務管理系統整合",
    "theme": "#2196F3",
    "language": "en",
    "private": false,
    "countries": ["US", "EU"]
}
```

### 步驟 2：Base 配置

#### base.imljson

```json
{
    "baseUrl": "https://api.taskmanager.example.com/v1",
    "headers": {
        "X-API-Key": "{{connection.apiKey}}",
        "Content-Type": "application/json",
        "User-Agent": "Make-TaskManager/1.0"
    },
    "response": {
        "error": {
            "type": "{{if(statusCode === 401, 'InvalidCredentials', if(statusCode === 403, 'InsufficientPermissions', if(statusCode === 404, 'DataError', if(statusCode === 429, 'RateLimitError', 'ConnectionError'))))}}",
            "message": "[{{statusCode}}] {{if(body.error, body.error.message, if(body.message, body.message, '未知錯誤'))}}",
            "code": "{{body.error.code}}"
        }
    },
    "log": {
        "sanitize": [
            "request.headers.X-API-Key",
            "response.body.users[].email",
            "response.body.users[].password"
        ]
    }
}
```

### 步驟 3：連接配置

#### connections/apiKey.imljson

```json
{
    "name": "apiKey",
    "label": "Task Manager API Connection",
    "type": "basic",
    "help": "請輸入您的 Task Manager API 金鑰。您可以在帳戶設定中找到 API 金鑰。",
    "parameters": [
        {
            "name": "apiKey",
            "type": "password",
            "label": "API Key",
            "required": true,
            "help": "您的 Task Manager API 金鑰"
        },
        {
            "name": "environment",
            "type": "select",
            "label": "Environment",
            "default": "production",
            "options": [
                {
                    "label": "Production",
                    "value": "production"
                },
                {
                    "label": "Sandbox",
                    "value": "sandbox"
                }
            ]
        }
    ],
    "communication": {
        "url": "/auth/verify",
        "method": "GET",
        "response": {
            "uid": "{{body.account.id}}",
            "metadata": {
                "type": "text",
                "value": "{{body.account.email}}"
            },
            "valid": "{{body.valid === true}}",
            "error": {
                "message": "無效的 API 金鑰或權限不足"
            }
        }
    }
}
```

### 步驟 4：CRUD 模組實作

#### modules/createTask.imljson

```json
{
    "name": "createTask",
    "label": "Create a Task",
    "description": "在指定專案中建立新任務",
    "type": "action",
    "typeOptions": {
        "actionType": "create"
    },
    "connection": "apiKey",
    "parameters": [
        {
            "name": "projectId",
            "type": "select",
            "label": "Project",
            "required": true,
            "help": "選擇要建立任務的專案",
            "options": {
                "rpc": {
                    "name": "listProjects",
                    "parameters": {}
                }
            }
        }
    ],
    "mappable": [
        {
            "name": "title",
            "type": "text",
            "label": "Task Title",
            "required": true,
            "validate": {
                "max": 200,
                "message": "標題不能超過 200 字元"
            }
        },
        {
            "name": "description",
            "type": "text",
            "label": "Description",
            "multiline": true,
            "help": "任務的詳細描述"
        },
        {
            "name": "priority",
            "type": "select",
            "label": "Priority",
            "default": "medium",
            "options": [
                {
                    "label": "High",
                    "value": "high"
                },
                {
                    "label": "Medium",
                    "value": "medium"
                },
                {
                    "label": "Low",
                    "value": "low"
                }
            ]
        },
        {
            "name": "assignee",
            "type": "select",
            "label": "Assignee",
            "options": {
                "rpc": {
                    "name": "listUsers",
                    "parameters": {
                        "projectId": "{{parameters.projectId}}"
                    }
                }
            }
        },
        {
            "name": "dueDate",
            "type": "date",
            "label": "Due Date",
            "time": true
        },
        {
            "name": "tags",
            "type": "array",
            "label": "Tags",
            "spec": {
                "type": "text",
                "label": "Tag"
            }
        },
        {
            "name": "customFields",
            "type": "collection",
            "label": "Custom Fields",
            "spec": [
                {
                    "name": "key",
                    "type": "text",
                    "label": "Field Name"
                },
                {
                    "name": "value",
                    "type": "text",
                    "label": "Field Value"
                }
            ]
        }
    ],
    "communication": {
        "url": "/projects/{{parameters.projectId}}/tasks",
        "method": "POST",
        "body": {
            "title": "{{parameters.title}}",
            "description": "{{parameters.description}}",
            "priority": "{{parameters.priority}}",
            "assignee_id": "{{parameters.assignee}}",
            "due_date": "{{formatDate(parameters.dueDate, 'YYYY-MM-DD HH:mm:ss')}}",
            "tags": "{{parameters.tags}}",
            "custom_fields": "{{toObject(parameters.customFields, 'key', 'value')}}"
        },
        "response": {
            "output": {
                "id": "{{body.task.id}}",
                "title": "{{body.task.title}}",
                "status": "{{body.task.status}}",
                "priority": "{{body.task.priority}}",
                "createdAt": "{{parseDate(body.task.created_at, 'YYYY-MM-DD HH:mm:ss')}}",
                "url": "{{body.task.url}}"
            }
        }
    },
    "interface": [
        {
            "name": "id",
            "type": "text",
            "label": "Task ID"
        },
        {
            "name": "title",
            "type": "text",
            "label": "Title"
        },
        {
            "name": "status",
            "type": "text",
            "label": "Status"
        },
        {
            "name": "priority",
            "type": "text",
            "label": "Priority"
        },
        {
            "name": "createdAt",
            "type": "date",
            "label": "Created Date"
        },
        {
            "name": "url",
            "type": "url",
            "label": "Task URL"
        }
    ]
}
```

#### modules/updateTask.imljson

```json
{
    "name": "updateTask",
    "label": "Update a Task",
    "description": "更新現有任務的資訊",
    "type": "action",
    "typeOptions": {
        "actionType": "update"
    },
    "connection": "apiKey",
    "mappable": [
        {
            "name": "taskId",
            "type": "text",
            "label": "Task ID",
            "required": true,
            "help": "要更新的任務 ID"
        },
        {
            "name": "title",
            "type": "text",
            "label": "Title"
        },
        {
            "name": "description",
            "type": "text",
            "label": "Description",
            "multiline": true
        },
        {
            "name": "status",
            "type": "select",
            "label": "Status",
            "options": [
                {
                    "label": "To Do",
                    "value": "todo"
                },
                {
                    "label": "In Progress",
                    "value": "in_progress"
                },
                {
                    "label": "Review",
                    "value": "review"
                },
                {
                    "label": "Done",
                    "value": "done"
                }
            ]
        },
        {
            "name": "priority",
            "type": "select",
            "label": "Priority",
            "options": [
                {
                    "label": "High",
                    "value": "high"
                },
                {
                    "label": "Medium",
                    "value": "medium"
                },
                {
                    "label": "Low",
                    "value": "low"
                }
            ]
        }
    ],
    "communication": {
        "url": "/tasks/{{parameters.taskId}}",
        "method": "PUT",
        "body": {
            "title": "{{if(parameters.title, parameters.title, undefined)}}",
            "description": "{{if(parameters.description, parameters.description, undefined)}}",
            "status": "{{if(parameters.status, parameters.status, undefined)}}",
            "priority": "{{if(parameters.priority, parameters.priority, undefined)}}"
        },
        "response": {
            "output": "{{body.task}}"
        }
    },
    "interface": "{{getTask.interface}}"
}
```

#### modules/listTasks.imljson

```json
{
    "name": "listTasks",
    "label": "List Tasks",
    "description": "搜尋和列出任務",
    "type": "search",
    "connection": "apiKey",
    "parameters": [
        {
            "name": "projectId",
            "type": "select",
            "label": "Project",
            "options": {
                "store": true,
                "rpc": {
                    "name": "listProjects",
                    "parameters": {}
                }
            }
        },
        {
            "name": "limit",
            "type": "number",
            "label": "Limit",
            "default": 10,
            "validate": {
                "min": 1,
                "max": 100
            }
        }
    ],
    "mappable": [
        {
            "name": "query",
            "type": "text",
            "label": "Search Query",
            "help": "搜尋任務標題或描述"
        },
        {
            "name": "filters",
            "type": "filter",
            "label": "Filters",
            "options": {
                "fields": [
                    {
                        "name": "status",
                        "type": "select",
                        "label": "Status",
                        "options": ["todo", "in_progress", "review", "done"]
                    },
                    {
                        "name": "priority",
                        "type": "select",
                        "label": "Priority",
                        "options": ["high", "medium", "low"]
                    },
                    {
                        "name": "assignee",
                        "type": "text",
                        "label": "Assignee ID"
                    },
                    {
                        "name": "created_after",
                        "type": "date",
                        "label": "Created After"
                    }
                ],
                "operators": [
                    {
                        "label": "equals",
                        "value": "="
                    },
                    {
                        "label": "not equals",
                        "value": "!="
                    },
                    {
                        "label": "greater than",
                        "value": ">"
                    },
                    {
                        "label": "less than",
                        "value": "<"
                    }
                ]
            }
        }
    ],
    "communication": {
        "url": "/tasks",
        "method": "GET",
        "qs": {
            "project_id": "{{parameters.projectId}}",
            "q": "{{parameters.query}}",
            "limit": "{{parameters.limit}}",
            "page": "{{ifempty(pagination.page, 1)}}",
            "filters": "{{buildFilterQuery(parameters.filters)}}"
        },
        "response": {
            "iterate": "{{body.tasks}}",
            "output": {
                "id": "{{item.id}}",
                "title": "{{item.title}}",
                "status": "{{item.status}}",
                "priority": "{{item.priority}}",
                "assignee": {
                    "id": "{{item.assignee.id}}",
                    "name": "{{item.assignee.name}}",
                    "email": "{{item.assignee.email}}"
                },
                "dueDate": "{{parseDate(item.due_date, 'YYYY-MM-DD HH:mm:ss')}}",
                "tags": "{{item.tags}}",
                "createdAt": "{{parseDate(item.created_at, 'YYYY-MM-DD HH:mm:ss')}}"
            },
            "pagination": {
                "qs": {
                    "page": "{{body.pagination.current_page + 1}}"
                },
                "condition": "{{body.pagination.has_more === true}}"
            }
        }
    },
    "interface": [
        {
            "name": "id",
            "type": "text",
            "label": "Task ID"
        },
        {
            "name": "title",
            "type": "text",
            "label": "Title"
        },
        {
            "name": "status",
            "type": "text",
            "label": "Status"
        },
        {
            "name": "priority",
            "type": "text",
            "label": "Priority"
        },
        {
            "name": "assignee",
            "type": "collection",
            "label": "Assignee",
            "spec": [
                {
                    "name": "id",
                    "type": "text",
                    "label": "ID"
                },
                {
                    "name": "name",
                    "type": "text",
                    "label": "Name"
                },
                {
                    "name": "email",
                    "type": "email",
                    "label": "Email"
                }
            ]
        },
        {
            "name": "dueDate",
            "type": "date",
            "label": "Due Date"
        },
        {
            "name": "tags",
            "type": "array",
            "label": "Tags",
            "spec": {
                "type": "text"
            }
        },
        {
            "name": "createdAt",
            "type": "date",
            "label": "Created Date"
        }
    ]
}
```

#### modules/watchNewTasks.imljson

```json
{
    "name": "watchNewTasks",
    "label": "Watch New Tasks",
    "description": "監控新建立的任務",
    "type": "trigger",
    "connection": "apiKey",
    "parameters": [
        {
            "name": "projectId",
            "type": "select",
            "label": "Project",
            "options": {
                "rpc": {
                    "name": "listProjects",
                    "parameters": {}
                }
            }
        },
        {
            "name": "includeStatuses",
            "type": "select",
            "label": "Include Statuses",
            "multiple": true,
            "default": ["todo", "in_progress"],
            "options": [
                {
                    "label": "To Do",
                    "value": "todo"
                },
                {
                    "label": "In Progress",
                    "value": "in_progress"
                },
                {
                    "label": "Review",
                    "value": "review"
                },
                {
                    "label": "Done",
                    "value": "done"
                }
            ]
        }
    ],
    "epoch": {
        "communication": {
            "url": "/tasks",
            "method": "GET",
            "qs": {
                "project_id": "{{parameters.projectId}}",
                "limit": 1,
                "sort": "created_at",
                "order": "desc"
            }
        },
        "response": {
            "iterate": "{{body.tasks}}",
            "output": {
                "id": "{{item.id}}",
                "date": "{{item.created_at}}"
            }
        }
    },
    "communication": {
        "url": "/tasks",
        "method": "GET",
        "qs": {
            "project_id": "{{parameters.projectId}}",
            "created_after": "{{epoch}}",
            "status": "{{join(parameters.includeStatuses, ',')}}",
            "sort": "created_at",
            "order": "asc"
        },
        "response": {
            "iterate": "{{body.tasks}}",
            "output": {
                "id": "{{item.id}}",
                "title": "{{item.title}}",
                "status": "{{item.status}}",
                "priority": "{{item.priority}}",
                "createdAt": "{{parseDate(item.created_at, 'YYYY-MM-DD HH:mm:ss')}}"
            },
            "trigger": {
                "id": "{{item.id}}",
                "date": "{{item.created_at}}"
            }
        }
    },
    "interface": "{{listTasks.interface}}"
}
```

### 步驟 5：RPC 實作

#### rpc/listProjects.imljson

```json
{
    "name": "listProjects",
    "label": "List Projects",
    "connection": "apiKey",
    "communication": {
        "url": "/projects",
        "method": "GET",
        "qs": {
            "active": true,
            "limit": 100
        },
        "response": {
            "iterate": "{{body.projects}}",
            "output": {
                "label": "{{item.name}}",
                "value": "{{item.id}}"
            }
        }
    }
}
```

#### rpc/listUsers.imljson

```json
{
    "name": "listUsers",
    "label": "List Users",
    "connection": "apiKey",
    "parameters": [
        {
            "name": "projectId",
            "type": "text",
            "label": "Project ID"
        }
    ],
    "communication": {
        "url": "/projects/{{parameters.projectId}}/members",
        "method": "GET",
        "response": {
            "iterate": "{{body.members}}",
            "output": {
                "label": "{{item.name}} ({{item.email}})",
                "value": "{{item.id}}"
            }
        }
    }
}
```

### 步驟 6：輔助函數

#### functions/taskHelpers.iml

```javascript
/**
 * 建構篩選查詢字串
 */
function buildFilterQuery(filters) {
    if (!filters || filters.length === 0) {
        return '';
    }
    
    return filters[0].map(filter => {
        const field = filter.field;
        const operator = filter.operator;
        const value = filter.value;
        
        // 處理日期格式
        let processedValue = value;
        if (field.includes('date') || field.includes('_at')) {
            processedValue = formatDate(value, 'YYYY-MM-DD');
        }
        
        return `${field}${operator}${processedValue}`;
    }).join('&');
}

/**
 * 將陣列轉換為物件
 */
function toObject(array, keyField, valueField) {
    if (!array || array.length === 0) {
        return {};
    }
    
    const result = {};
    array.forEach(item => {
        if (item[keyField]) {
            result[item[keyField]] = item[valueField];
        }
    });
    
    return result;
}

/**
 * 驗證並格式化日期
 */
function validateAndFormatDate(date, format) {
    if (!date) {
        return null;
    }
    
    try {
        const parsed = parseDate(date);
        return formatDate(parsed, format || 'YYYY-MM-DD');
    } catch (error) {
        debug('Invalid date:', date);
        return null;
    }
}

/**
 * 處理批次操作結果
 */
function processBatchResults(results) {
    const successful = [];
    const failed = [];
    
    results.forEach(result => {
        if (result.success) {
            successful.push(result.data);
        } else {
            failed.push({
                id: result.id,
                error: result.error
            });
        }
    });
    
    return {
        successful: successful,
        failed: failed,
        totalSuccess: successful.length,
        totalFailed: failed.length
    };
}
```

### 使用範例

#### 場景 1：自動化任務建立

```
觸發器: 收到電子郵件
↓
解析郵件內容
↓
Task Manager Pro - Create Task
  - Title: 從郵件主題擷取
  - Description: 從郵件內容擷取
  - Priority: 根據關鍵字判斷
↓
發送通知
```

#### 場景 2：任務狀態同步

```
觸發器: Task Manager Pro - Watch New Tasks
↓
檢查任務詳情
↓
更新其他系統
↓
Task Manager Pro - Update Task
  - Status: 更新為 "in_progress"
```

#### 場景 3：批次任務處理

```
排程觸發器: 每天早上 9:00
↓
Task Manager Pro - List Tasks
  - Filter: status = "todo"
  - Filter: created_after = 昨天
↓
Iterator
↓
處理每個任務
```

### 最佳實踐

1. **模組化設計**
   - 每個 API 操作對應一個模組
   - 共用邏輯放在 functions 中
   - RPC 處理動態選項

2. **錯誤處理**
   - 在 Base 層級統一處理
   - 提供有意義的錯誤訊息
   - 記錄足夠的除錯資訊

3. **效能優化**
   - 實作分頁避免大量資料
   - 使用適當的快取策略
   - 避免不必要的 API 呼叫

4. **安全性**
   - 敏感資訊使用 sanitize
   - 驗證所有輸入參數
   - 使用 HTTPS 連接

5. **使用者體驗**
   - 提供清晰的參數說明
   - 設定合理的預設值
   - 使用 RPC 簡化選擇

## 實作練習

### 練習 1：加入批次操作
實作批次建立、更新和刪除任務的模組。

### 練習 2：進階搜尋
加入全文搜尋和複雜篩選條件支援。

### 練習 3：檔案附件
實作任務附件上傳和管理功能。

## 重點整理

- REST API 整合需要完整的 CRUD 操作
- 良好的錯誤處理提升穩定性
- 分頁和篩選是必要功能
- 模組化設計便於維護
- 安全性考量不可忽視

## 延伸閱讀

- [OAuth 認證範例](OAuth_認證範例.md) - 更安全的認證方式
- [複雜資料處理範例](複雜資料處理範例.md) - 處理巢狀資料
- [API 參考手冊](../06_參考資料/API_參考手冊.md) - 完整 API 文檔

---

[← 上一章：錯誤處理與除錯](../04_進階功能/03_錯誤處理與除錯.md) | [下一章：OAuth 認證範例 →](OAuth_認證範例.md)