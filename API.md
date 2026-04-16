# 阅读挑战活动接口说明

## 说明

- 基础路径：`/api/readingweek`
- 请求方式：除特别说明外，统一使用 `POST`
- 返回格式统一为：

```json
{
  "code": 200,
  "message": "ok",
  "data": {}
}
```

- `code = 200` 表示成功，其它表示失败
- 所有接口请求统一携带 `user_id` 和 `name`

## 1. 进入活动时检查用户状态

用途：页面初始化时调用。判断用户是否已有中奖但未填写地址的记录，如果有，前端直接跳到中奖页。

### 接口

`POST /api/readingweek/check-status`

### 请求参数

```json
{
  "user_id": "5cc7a149-2eb8-11ea-a5e3-dcf401e72500",
  "name": "iOS开发"
}
```

### 返回示例

```json
{
  "code": 200,
  "message": "ok",
  "data": {
    "has_pending_prize": true,
    "draw_id": "draw_20260416_0001",
    "prize_name": "《郑和下西洋与海上丝路》"
  }
}
```

### 字段说明

- `has_pending_prize`：是否存在已中奖但未填写地址的记录
- `draw_id`：中奖记录 ID；提交地址时要带上
- `prize_name`：中奖书名；没有中奖记录时可返回空字符串

### 前端处理

- `has_pending_prize = true`：直接跳到中奖页，并显示 `prize_name`
- `has_pending_prize = false`：正常开始活动

## 2. 提交三个选择题答案

用途：用户点击“开始挑战”后，完成三道选择题时提交。

### 接口

`POST /api/readingweek/submit-answers`

### 请求参数

```json
{
  "user_id": "5cc7a149-2eb8-11ea-a5e3-dcf401e72500",
  "name": "iOS开发",
  "answers": [0, 1, 2]
}
```

### 参数说明

- `answers`：长度固定为 `3` 的整数数组
- `0` 表示选 `A`
- `1` 表示选 `B`
- `2` 表示选 `C`

例如：

- `[0, 1, 2]` 表示第 1 题选 A，第 2 题选 B，第 3 题选 C

### 返回示例

```json
{
  "code": 200,
  "message": "ok",
  "data": {
    "saved": true
  }
}
```

## 3. 执行抽奖

用途：用户点击“立即抽奖”时调用。后台返回三种结果：

- 今日抽奖次数已达上限
- 未中奖
- 中奖，并返回书名

### 接口

`POST /api/readingweek/draw`

### 请求参数

```json
{
  "user_id": "5cc7a149-2eb8-11ea-a5e3-dcf401e72500",
  "name": "iOS开发"
}
```

### 返回示例 1：今日已抽满

```json
{
  "code": 200,
  "message": "ok",
  "data": {
    "result": "limit"
  }
}
```

### 返回示例 2：未中奖

```json
{
  "code": 200,
  "message": "ok",
  "data": {
    "result": "lose"
  }
}
```

### 返回示例 3：中奖

```json
{
  "code": 200,
  "message": "ok",
  "data": {
    "result": "win",
    "draw_id": "draw_20260416_0002",
    "prize_name": "《郑和下西洋与海上丝路》"
  }
}
```

### 字段说明

- `result`：抽奖结果
- `limit`：今日抽奖次数已达上限
- `lose`：未中奖
- `win`：中奖
- `draw_id`：中奖记录 ID，仅 `win` 时返回
- `prize_name`：中奖书名，仅 `win` 时返回

## 4. 提交中奖收货地址

用途：中奖用户填写收件人、手机号、地址后提交。

### 接口

`POST /api/readingweek/submit-address`

### 请求参数

```json
{
  "user_id": "5cc7a149-2eb8-11ea-a5e3-dcf401e72500",
  "name": "iOS开发",
  "draw_id": "draw_20260416_0002",
  "recipient": "张三",
  "mobile": "13800138000",
  "address": "北京市朝阳区XX路XX号"
}
```

### 参数说明

- `draw_id`：中奖记录 ID，来自抽奖接口或状态检查接口
- `recipient`：收件人姓名
- `mobile`：收件手机号
- `address`：详细地址

### 返回示例

```json
{
  "code": 200,
  "message": "ok",
  "data": {
    "saved": true
  }
}
```

## 建议错误码

可选，后端如果需要统一错误处理，可以约定以下错误码：

- `200`：成功
- `4001`：参数错误
- `4002`：用户不存在或未登录
- `4003`：答案格式错误
- `4004`：今日抽奖次数已达上限
- `4005`：未中奖，不能提交地址
- `4006`：地址已提交，不能重复提交
- `5000`：服务器内部错误
