# IML 函數開發

## 本章概要

本章深入探討 IML（Integromat Markup Language）函數的使用和開發。您將學會使用內建函數處理資料，以及如何使用 JavaScript 撰寫自定義 IML 函數來解決複雜的資料轉換需求。

## 學習目標

- 理解 IML 的作用和語法
- 掌握常用的內建 IML 函數
- 學會撰寫自定義 IML 函數
- 使用除錯工具優化函數
- 處理複雜的資料轉換邏輯

## 前置知識

- JavaScript 基礎知識
- 了解 JSON 資料結構
- 熟悉基本的資料處理概念

## 內容主體

### IML 簡介

IML 是 Make 的內部標記語言，用於在應用程式配置中進行動態資料處理。它使用雙大括號 `{{}}` 語法來標記表達式。

**IML 的主要用途**：
- 資料映射和轉換
- 條件邏輯處理
- 日期時間格式化
- 字串處理
- 數學運算

### IML 語法基礎

#### 1. 基本語法

```json
{
    "simple": "{{parameters.name}}",
    "nested": "{{parameters.user.email}}",
    "with_default": "{{ifempty(parameters.status, 'pending')}}",
    "expression": "{{parameters.price * 1.1}}"
}
```

#### 2. 存取資料

| 語法 | 說明 | 範例 |
|------|------|------|
| `{{variable}}` | 存取變數 | `{{parameters.id}}` |
| `{{object.property}}` | 存取物件屬性 | `{{body.data.name}}` |
| `{{array[0]}}` | 存取陣列元素 | `{{items[0].title}}` |
| `{{array.0}}` | 陣列元素（替代語法） | `{{items.0.title}}` |

### 常用內建函數

#### 1. 條件函數

**if() - 條件判斷**
```json
{
    "status": "{{if(parameters.isActive, 'active', 'inactive')}}",
    "priority": "{{if(parameters.urgent === true, 'high', 'normal')}}"
}
```

**ifempty() - 空值處理**
```json
{
    "name": "{{ifempty(parameters.name, 'Anonymous')}}",
    "tags": "{{ifempty(parameters.tags, [])}}"
}
```

#### 2. 日期時間函數

**formatDate() - 格式化日期**
```json
{
    "date": "{{formatDate(parameters.date, 'YYYY-MM-DD')}}",
    "time": "{{formatDate(parameters.date, 'HH:mm:ss')}}",
    "timestamp": "{{formatDate(parameters.date, 'X')}}"
}
```

常用格式：
- `YYYY-MM-DD` - 2024-01-15
- `DD/MM/YYYY` - 15/01/2024
- `X` - Unix 時間戳（秒）
- `x` - Unix 時間戳（毫秒）

**parseDate() - 解析日期**
```json
{
    "parsed": "{{parseDate(body.created_at, 'YYYY-MM-DD HH:mm:ss')}}",
    "fromTimestamp": "{{parseDate(body.timestamp, 'X')}}"
}
```

**now - 當前時間**
```json
{
    "current": "{{now}}",
    "tomorrow": "{{addDays(now, 1)}}",
    "formatted": "{{formatDate(now, 'YYYY-MM-DD')}}"
}
```

#### 3. 字串函數

**基本字串操作**
```json
{
    "uppercase": "{{toUpper(parameters.text)}}",
    "lowercase": "{{toLower(parameters.email)}}",
    "trimmed": "{{trim(parameters.input)}}",
    "length": "{{length(parameters.description)}}"
}
```

**字串組合**
```json
{
    "fullName": "{{parameters.firstName + ' ' + parameters.lastName}}",
    "joined": "{{join(parameters.tags, ', ')}}",
    "split": "{{split(parameters.csv, ',')}}"
}
```

#### 4. 陣列函數

**陣列操作**
```json
{
    "count": "{{length(parameters.items)}}",
    "first": "{{first(parameters.items)}}",
    "last": "{{last(parameters.items)}}",
    "unique": "{{distinct(parameters.tags)}}",
    "merged": "{{merge(array1, array2)}}"
}
```

**陣列篩選和轉換**
```json
{
    "filtered": "{{filter(parameters.items, 'status', 'active')}}",
    "mapped": "{{map(parameters.users, 'email')}}",
    "sorted": "{{sort(parameters.items, 'name', 'asc')}}"
}
```

#### 5. 數學函數

```json
{
    "rounded": "{{round(parameters.price, 2)}}",
    "ceiling": "{{ceil(parameters.value)}}",
    "floor": "{{floor(parameters.value)}}",
    "sum": "{{sum(parameters.numbers)}}",
    "average": "{{avg(parameters.scores)}}"
}
```

#### 6. 編碼函數

```json
{
    "base64": "{{base64(parameters.data)}}",
    "urlEncoded": "{{encodeURL(parameters.query)}}",
    "jsonString": "{{toString(parameters.object)}}",
    "parsed": "{{parseJSON(parameters.jsonString)}}"
}
```

### 自定義 IML 函數

當內建函數無法滿足需求時，可以使用 JavaScript 撰寫自定義函數。

#### 1. 基本結構

```javascript
function myFunction(input, param1, param2) {
    // 函數邏輯
    return result;
}
```

#### 2. 實作範例

**範例 1：複雜的篩選邏輯**
```javascript
function filterTasks(tasks, filters) {
    if (!tasks || !filters) return [];
    
    return tasks.filter(task => {
        // 檢查狀態
        if (filters.status && task.status !== filters.status) {
            return false;
        }
        
        // 檢查優先級
        if (filters.priority && task.priority !== filters.priority) {
            return false;
        }
        
        // 檢查日期範圍
        if (filters.startDate && new Date(task.dueDate) < new Date(filters.startDate)) {
            return false;
        }
        
        return true;
    });
}
```

**範例 2：資料轉換**
```javascript
function transformUserData(user) {
    return {
        id: user.user_id,
        name: `${user.first_name} ${user.last_name}`.trim(),
        email: user.email_address.toLowerCase(),
        isActive: user.status === 'ACTIVE',
        createdAt: new Date(user.created_timestamp * 1000).toISOString()
    };
}
```

**範例 3：動態查詢建構**
```javascript
function buildQuery(filters) {
    if (!filters || filters.length === 0) return '';
    
    return filters[0].map(condition => {
        if (!condition.field || !condition.operator || !condition.value) {
            return null;
        }
        return `${condition.field}~${condition.operator}~${condition.value}`;
    })
    .filter(Boolean)
    .join('|AND|');
}
```

#### 3. 在模組中使用

```json
{
    "url": "/api/tasks",
    "qs": {
        "filter": "{{buildQuery(parameters.filters)}}"
    },
    "response": {
        "iterate": "{{filterTasks(body.data, parameters.advancedFilters)}}",
        "output": "{{transformUserData(item)}}"
    }
}
```

### 除錯技巧

#### 1. 使用 debug() 函數

```javascript
function calculatePrice(items, discount) {
    debug(items);  // 在開發者控制台顯示
    
    let total = items.reduce((sum, item) => {
        debug(`Processing item: ${item.name}`);
        return sum + (item.price * item.quantity);
    }, 0);
    
    debug(`Total before discount: ${total}`);
    
    if (discount) {
        total = total * (1 - discount / 100);
    }
    
    return total;
}
```

#### 2. VS Code 除錯

1. 建立測試檔案
2. 準備測試資料
3. 設定中斷點
4. 執行除錯

**測試範例**：
```javascript
// 測試資料
const testData = {
    items: [
        { name: "Product A", price: 100, quantity: 2 },
        { name: "Product B", price: 50, quantity: 3 }
    ],
    discount: 10
};

// 呼叫函數
const result = calculatePrice(testData.items, testData.discount);
console.log('Result:', result);
```

### 最佳實踐

#### 1. 錯誤處理

```javascript
function safeParseJSON(jsonString) {
    try {
        return JSON.parse(jsonString);
    } catch (error) {
        debug(`JSON parse error: ${error.message}`);
        return null;
    }
}
```

#### 2. 型別檢查

```javascript
function processArray(input) {
    // 確保輸入是陣列
    if (!Array.isArray(input)) {
        return [];
    }
    
    return input.map(item => {
        // 處理每個項目
        return processItem(item);
    });
}
```

#### 3. 效能考量

```javascript
function optimizedSearch(items, searchTerm) {
    if (!searchTerm || searchTerm.length < 2) {
        return items;
    }
    
    const lowerSearchTerm = searchTerm.toLowerCase();
    
    return items.filter(item => {
        // 只轉換一次，避免重複運算
        const lowerName = item.name.toLowerCase();
        return lowerName.includes(lowerSearchTerm);
    });
}
```

### 複雜應用範例

#### 範例：動態欄位映射

```javascript
function mapFields(source, mapping) {
    const result = {};
    
    for (const [targetField, sourceField] of Object.entries(mapping)) {
        // 支援巢狀路徑
        const value = getNestedValue(source, sourceField);
        setNestedValue(result, targetField, value);
    }
    
    return result;
}

function getNestedValue(obj, path) {
    return path.split('.').reduce((current, key) => {
        return current?.[key];
    }, obj);
}

function setNestedValue(obj, path, value) {
    const keys = path.split('.');
    const lastKey = keys.pop();
    
    const target = keys.reduce((current, key) => {
        if (!current[key]) {
            current[key] = {};
        }
        return current[key];
    }, obj);
    
    target[lastKey] = value;
}
```

使用方式：
```json
{
    "response": {
        "output": "{{mapFields(body, parameters.fieldMapping)}}"
    }
}
```

## 實作練習

### 練習 1：日期處理函數
撰寫一個函數，將各種日期格式統一轉換為 ISO 8601 格式。

### 練習 2：資料驗證函數
建立一個驗證函數，檢查必填欄位並回傳錯誤訊息。

### 練習 3：批次處理函數
實作一個函數，能夠批次處理陣列資料並加入錯誤處理。

## 重點整理

- IML 使用 `{{}}` 語法進行資料處理
- 內建函數涵蓋大部分常用功能
- 自定義函數使用 JavaScript 撰寫
- debug() 函數有助於除錯
- 良好的錯誤處理能提升穩定性
- 效能優化對大量資料處理很重要

## 延伸閱讀

- [資料流程與處理](../02_核心概念/03_資料流程與處理.md) - IML 在資料處理中的應用
- [錯誤處理與除錯](03_錯誤處理與除錯.md) - 進階除錯技巧
- [IML 函數速查表](../06_參考資料/IML_函數速查表.md) - 完整函數參考

---

[← 上一章：模組開發步驟](../03_開發實戰/03_模組開發步驟.md) | [下一章：Webhook 實作 →](02_Webhook實作.md)