# IML 函數速查表

## 本章概要

本章提供所有內建 IML（Integromat Markup Language）函數的快速參考，包含函數語法、參數說明和使用範例。這是開發過程中經常查閱的重要資源。

## IML 基礎語法

### 基本語法規則

```
{{expression}}                    // 基本表達式
{{variable.property}}            // 存取屬性
{{function(parameter)}}          // 函數呼叫
{{condition ? true : false}}     // 三元運算子
```

## 條件函數

### if()
條件判斷函數。

**語法**: `if(condition, trueValue, falseValue)`

```json
{{if(parameters.status === 'active', 'Yes', 'No')}}
{{if(body.count > 0, body.data, [])}}
```

### ifempty()
檢查空值並提供預設值。

**語法**: `ifempty(value, defaultValue)`

```json
{{ifempty(parameters.name, 'Anonymous')}}
{{ifempty(body.description, 'No description')}}
```

### switch()
多條件判斷。

**語法**: `switch(value, case1, result1, case2, result2, ..., default)`

```json
{{switch(parameters.priority, 'high', 1, 'medium', 2, 'low', 3, 0)}}
```

## 日期時間函數

### now
取得當前時間。

```json
{{now}}                          // 當前時間
{{formatDate(now, 'YYYY-MM-DD')}} // 格式化當前日期
```

### formatDate()
格式化日期。

**語法**: `formatDate(date, format, [timezone])`

**常用格式**:
- `YYYY` - 4位數年份
- `MM` - 月份（01-12）
- `DD` - 日期（01-31）
- `HH` - 24小時制小時
- `mm` - 分鐘
- `ss` - 秒
- `X` - Unix時間戳（秒）
- `x` - Unix時間戳（毫秒）

```json
{{formatDate(parameters.date, 'YYYY-MM-DD')}}
{{formatDate(now, 'YYYY-MM-DD HH:mm:ss', 'Asia/Taipei')}}
{{formatDate(body.timestamp, 'X')}}
```

### parseDate()
解析日期字串。

**語法**: `parseDate(dateString, format, [timezone])`

```json
{{parseDate('2024-01-15', 'YYYY-MM-DD')}}
{{parseDate(body.created_at, 'YYYY-MM-DD HH:mm:ss')}}
{{parseDate('1705305600', 'X')}}
```

### addDays()
增加天數。

**語法**: `addDays(date, days)`

```json
{{addDays(now, 7)}}              // 7天後
{{addDays(parameters.date, -30)}} // 30天前
```

### addMonths()
增加月份。

**語法**: `addMonths(date, months)`

```json
{{addMonths(now, 1)}}            // 下個月
{{addMonths(parameters.date, -3)}} // 3個月前
```

### addYears()
增加年份。

**語法**: `addYears(date, years)`

```json
{{addYears(now, 1)}}             // 明年
```

### dateDifference()
計算日期差異。

**語法**: `dateDifference(date1, date2, unit)`

**單位**: `years`, `months`, `days`, `hours`, `minutes`, `seconds`

```json
{{dateDifference(parameters.endDate, parameters.startDate, 'days')}}
```

## 字串函數

### toString()
轉換為字串。

**語法**: `toString(value)`

```json
{{toString(parameters.number)}}
{{toString(body.data)}}
```

### length()
取得字串長度。

**語法**: `length(string)`

```json
{{length(parameters.description)}}
{{if(length(parameters.title) > 100, 'Too long', 'OK')}}
```

### toUpper()
轉換為大寫。

**語法**: `toUpper(string)`

```json
{{toUpper(parameters.code)}}
{{toUpper(body.status)}}
```

### toLower()
轉換為小寫。

**語法**: `toLower(string)`

```json
{{toLower(parameters.email)}}
```

### capitalize()
首字母大寫。

**語法**: `capitalize(string)`

```json
{{capitalize(parameters.name)}}
```

### trim()
移除首尾空白。

**語法**: `trim(string)`

```json
{{trim(parameters.input)}}
```

### substring()
截取子字串。

**語法**: `substring(string, start, [length])`

```json
{{substring(parameters.text, 0, 100)}}
{{substring(body.id, 5)}}
```

### replace()
替換字串。

**語法**: `replace(string, searchValue, replaceValue)`

```json
{{replace(parameters.text, ' ', '_')}}
{{replace(body.phone, '-', '')}}
```

### split()
分割字串。

**語法**: `split(string, separator, [limit])`

```json
{{split(parameters.tags, ',')}}
{{split(body.path, '/', 3)}}
```

### join()
連接陣列為字串。

**語法**: `join(array, separator)`

```json
{{join(parameters.items, ', ')}}
{{join(body.tags, '|')}}
```

### contains()
檢查是否包含子字串。

**語法**: `contains(string, searchValue)`

```json
{{contains(parameters.email, '@')}}
{{if(contains(body.status, 'error'), 'Failed', 'Success')}}
```

### startsWith()
檢查是否以特定字串開頭。

**語法**: `startsWith(string, searchValue)`

```json
{{startsWith(parameters.url, 'https')}}
```

### endsWith()
檢查是否以特定字串結尾。

**語法**: `endsWith(string, searchValue)`

```json
{{endsWith(parameters.filename, '.pdf')}}
```

## 數字函數

### parseNumber()
解析數字。

**語法**: `parseNumber(value)`

```json
{{parseNumber(parameters.price)}}
{{parseNumber(body.quantity)}}
```

### round()
四捨五入。

**語法**: `round(number, [decimals])`

```json
{{round(parameters.price, 2)}}
{{round(body.average)}}
```

### ceil()
無條件進位。

**語法**: `ceil(number)`

```json
{{ceil(parameters.value)}}
```

### floor()
無條件捨去。

**語法**: `floor(number)`

```json
{{floor(parameters.value)}}
```

### abs()
絕對值。

**語法**: `abs(number)`

```json
{{abs(parameters.difference)}}
```

### min()
最小值。

**語法**: `min(number1, number2, ...)`

```json
{{min(parameters.price, body.maxPrice)}}
{{min(1, 2, 3, 4, 5)}}
```

### max()
最大值。

**語法**: `max(number1, number2, ...)`

```json
{{max(parameters.quantity, 1)}}
{{max(body.scores)}}
```

### sum()
陣列總和。

**語法**: `sum(array)`

```json
{{sum(parameters.amounts)}}
{{sum(body.items.map(item => item.price))}}
```

### avg()
陣列平均值。

**語法**: `avg(array)`

```json
{{avg(parameters.scores)}}
{{avg(body.ratings)}}
```

## 陣列函數

### length()
陣列長度。

**語法**: `length(array)`

```json
{{length(parameters.items)}}
{{if(length(body.results) > 0, 'Has results', 'No results')}}
```

### first()
取得第一個元素。

**語法**: `first(array)`

```json
{{first(parameters.items)}}
{{first(body.data).id}}
```

### last()
取得最後一個元素。

**語法**: `last(array)`

```json
{{last(parameters.items)}}
{{last(body.history).timestamp}}
```

### get()
取得指定索引的元素。

**語法**: `get(array, index)`

```json
{{get(parameters.items, 0)}}
{{get(body.results, 2)}}
```

### map()
映射陣列元素。

**語法**: `map(array, key)`

```json
{{map(body.users, 'email')}}
{{map(parameters.items, 'id')}}
```

### filter()
篩選陣列元素。

**語法**: `filter(array, key, operator, value)`

```json
{{filter(body.items, 'status', '=', 'active')}}
{{filter(parameters.products, 'price', '>', 100)}}
```

### sort()
排序陣列。

**語法**: `sort(array, key, order)`

**排序**: `asc` (升序), `desc` (降序)

```json
{{sort(body.items, 'name', 'asc')}}
{{sort(parameters.products, 'price', 'desc')}}
```

### distinct()
去除重複值。

**語法**: `distinct(array)`

```json
{{distinct(parameters.tags)}}
{{distinct(map(body.items, 'category'))}}
```

### merge()
合併陣列。

**語法**: `merge(array1, array2, ...)`

```json
{{merge(parameters.items1, parameters.items2)}}
{{merge(body.results, body.additionalResults)}}
```

### slice()
截取陣列片段。

**語法**: `slice(array, start, [end])`

```json
{{slice(body.items, 0, 10)}}
{{slice(parameters.data, 5)}}
```

### contains()
檢查陣列是否包含元素。

**語法**: `contains(array, value)`

```json
{{contains(parameters.roles, 'admin')}}
{{if(contains(body.tags, 'urgent'), 'High', 'Normal')}}
```

### flatten()
展平多維陣列。

**語法**: `flatten(array, [depth])`

```json
{{flatten(body.nestedArray)}}
{{flatten(parameters.categories, 1)}}
```

## 物件函數

### keys()
取得物件的所有鍵。

**語法**: `keys(object)`

```json
{{keys(parameters.data)}}
{{length(keys(body.attributes))}}
```

### values()
取得物件的所有值。

**語法**: `values(object)`

```json
{{values(parameters.settings)}}
{{join(values(body.labels), ', ')}}
```

### omit()
排除指定屬性。

**語法**: `omit(object, keys)`

```json
{{omit(body.user, ['password', 'token'])}}
{{omit(parameters.data, 'internal')}}
```

### pick()
選取指定屬性。

**語法**: `pick(object, keys)`

```json
{{pick(body.user, ['id', 'name', 'email'])}}
{{pick(parameters.data, 'public')}}
```

## 編碼函數

### base64()
Base64 編碼。

**語法**: `base64(string)`

```json
{{base64(parameters.data)}}
{{base64(connection.username + ':' + connection.password)}}
```

### sha256()
SHA256 雜湊。

**語法**: `sha256(string, [encoding])`

```json
{{sha256(parameters.secret)}}
{{sha256(body.data, 'hex')}}
```

### md5()
MD5 雜湊。

**語法**: `md5(string)`

```json
{{md5(parameters.input)}}
```

### encodeURL()
URL 編碼。

**語法**: `encodeURL(string)`

```json
{{encodeURL(parameters.query)}}
{{encodeURL(body.redirect_url)}}
```

### decodeURL()
URL 解碼。

**語法**: `decodeURL(string)`

```json
{{decodeURL(query.callback)}}
```

### parseJSON()
解析 JSON 字串。

**語法**: `parseJSON(jsonString)`

```json
{{parseJSON(body.metadata)}}
{{parseJSON(parameters.config)}}
```

## 特殊函數

### uuid()
生成 UUID。

**語法**: `uuid()`

```json
{{uuid()}}
```

### timestamp
Unix 時間戳（秒）。

```json
{{timestamp}}
```

### random()
生成隨機數。

**語法**: `random()`

```json
{{random()}}                     // 0-1 之間
{{floor(random() * 100)}}        // 0-99 之間整數
```

### jwt()
生成 JWT Token。

**語法**: `jwt(payload, secret, algorithm, [options])`

```json
{{jwt(temp.payload, connection.secret, 'HS256')}}
{{jwt(body.claims, parameters.key, 'RS256', temp.options)}}
```

### debug()
除錯輸出到開發者控制台。

**語法**: `debug(value)`

**注意事項**: 
- 只在開發環境中有效
- 輸出會顯示在瀏覽器的開發者控制台
- 在 VS Code 本地測試時需要改為 `console.log()`

```javascript
function processData(input) {
    debug(input);  // 在 Make 中使用
    // 在 VS Code 中測試時改為：console.log(input);
    
    // 處理邏輯
    return result;
}
```

## 自定義函數

### 定義函數

```javascript
// functions/myHelpers.iml
function formatCurrency(amount, currency) {
    const formatted = parseNumber(amount).toFixed(2);
    return `${currency} ${formatted}`;
}

function validateEmail(email) {
    const pattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return pattern.test(email);
}
```

### 使用自定義函數

```json
{
    "output": {
        "price": "{{formatCurrency(body.amount, 'USD')}}",
        "validEmail": "{{validateEmail(parameters.email)}}"
    }
}
```

## 實用技巧

### 鏈式呼叫

```json
{{toUpper(trim(parameters.input))}}
{{round(sum(map(body.items, 'price')), 2)}}
```

### 條件運算

```json
{{parameters.discount ? body.price * 0.9 : body.price}}
{{body.user.role === 'admin' && body.user.active}}
```

### 空值處理

```json
{{body.data?.user?.name || 'Unknown'}}
{{ifempty(body.results, []).length}}
```

### 陣列操作

```json
{{body.items.filter(item => item.active).map(item => item.id)}}
{{parameters.values.reduce((sum, val) => sum + val, 0)}}
```

## 常見錯誤

1. **型別錯誤**
   ```json
   // 錯誤
   {{length(123)}}
   
   // 正確
   {{length(toString(123))}}
   ```

2. **空值處理**
   ```json
   // 錯誤
   {{body.user.name}}
   
   // 正確
   {{body.user?.name || ''}}
   ```

3. **日期格式**
   ```json
   // 錯誤
   {{formatDate('2024-01-15', 'YYYY-MM-DD')}}
   
   // 正確
   {{formatDate(parseDate('2024-01-15', 'YYYY-MM-DD'), 'YYYY-MM-DD')}}
   ```

---

[← 上一章：API 參考手冊](API_參考手冊.md) | [下一章：常見問題集 →](常見問題集.md)