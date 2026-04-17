

## 2. 执行抽奖接口

### 接口信息
- **请求方式**: POST
- **接口路径**: `draw.php`
https://h5.cyol.com/certificate/tcj/draw.php
- **功能说明**: 执行抽奖，返回抽奖结果，使用Redis做每日抽奖次数限制

### 请求参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| user_id | string | 是 | 用户ID（企业微信用户ID） |
| name | string | 是 | 用户姓名 |

**请求示例**:
```json
{
  "user_id": "zhangsan",
  "name": "张三"
}
```

### 返回示例

**中奖**:
```json
{
  "code": 200,
  "message": "ok",
  "data": {
    "result": "win",
    "id": 123,
    "prize_name": "小米空气净化器"
  }
}
```

**未中奖**:
```json
{
  "code": 200,
  "message": "ok",
  "data": {
    "result": "lose"
  }
}
```

**今日抽奖次数已达上限**:
```json
{
  "code": 4004,
  "message": "今日抽奖次数已达上限",
  "data": {
    "result": "limit"
  }
}
```

**参数错误**:
```json
{
  "code": 4001,
  "message": "参数错误",
  "data": null
}
```

---

## 3. 检查用户状态接口

### 接口信息
- **请求方式**: POST
- **接口路径**: `check-status.php`
- **功能说明**: 检查用户是否有已中奖但未提交收货地址的记录
- **缓存策略**: 优先查询Redis缓存，无缓存再查询MySQL

### 请求参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| user_id | string | 是 | 用户ID（企业微信用户ID） |
| name | string | 是 | 用户姓名 |

**请求示例**:
```json
{
  "user_id": "zhangsan",
  "name": "张三"
}
```

### 返回示例

**有未提交地址的中奖记录**:
```json
{
  "code": 200,
  "message": "ok",
  "data": {
    "has_pending_prize": true,
    "id": 123,
    "prize_name": "小米空气净化器"
  }
}
```

**没有未提交地址的中奖记录**:
```json
{
  "code": 200,
  "message": "ok",
  "data": {
    "has_pending_prize": false
  }
}
```

**参数错误**:
```json
{
  "code": 4001,
  "message": "参数错误",
  "data": null
}
```

---

## 4. 提交收货地址接口

### 接口信息
- **请求方式**: POST
- **接口路径**: `submit-address.php`
https://h5.cyol.com/certificate/tcj/submit-address.php
- **功能说明**: 提交中奖用户的收货地址

### 请求参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| user_id | string | 是 | 用户ID（企业微信用户ID） |
| name | string | 是 | 用户姓名 |
| recipient | string | 是 | 收件人姓名 |
| mobile | string | 是 | 手机号码 |
| address | string | 是 | 详细收货地址 |



**请求示例（不指定ID，自动查找）**:
```json
{
  "user_id": "zhangsan",
  "name": "张三",
  "recipient": "张三",
  "mobile": "13800138000",
  "address": "北京市朝阳区XX街道XX小区XX号楼XX单元XX室"
}
```

### 返回示例

**提交成功**:
```json
{
  "code": 200,
  "message": "ok",
  "data": {
    "submitted": true
  }
}
```

**未找到可提交地址的中奖记录**:
```json
{
  "code": 4002,
  "message": "未找到可提交地址的中奖记录",
  "data": []
}
```

**未中奖，不能提交地址**:
```json
{
  "code": 4005,
  "message": "未中奖，不能提交地址",
  "data": []
}
```

**地址已提交，不能重复提交**:
```json
{
  "code": 4006,
  "message": "地址已提交，不能重复提交",
  "data": []
}
```

**参数错误**:
```json
{
  "code": 4001,
  "message": "参数错误",
  "data": null
}
```

---

## 业务逻辑说明

### 过期处理
每个接口在执行业务逻辑前，都会自动清理超过1天未提交地址的中奖记录：
1. 恢复奖品库存
2. 将中奖记录标记为已取消
3. 删除Redis中的待领奖缓存
这样可以释放库存，恢复用户中奖资格

### Redis缓存说明
- **抽奖次数限制**: 按用户+日期缓存，每日零点自动过期
- **待领奖缓存**: 中奖后缓存7天，提交地址后自动删除

### 中奖规则
- 每个用户最多只能中一次奖
- 整体中奖概率为50%
- 每日抽奖次数限制由 `DAILY_DRAW_LIMIT` 配置项控制

---

## 前后端交互流程

```
1. 用户进入页面
   ↓
2. 前端调用 check-status 接口
   ↓
3. 如果有未提交地址 → 跳转到地址提交页面
   如果没有未提交地址 → 进入答题页面
   ↓
4. 用户提交答案 → 调用 submit-answers 接口
   ↓
5. 如果答案不全对 → 提示错误，允许重新作答
   如果答案全对 → 调用 draw 接口抽奖
   ↓
6. 抽奖完成
   ↓
7. 如果中奖 → 跳转到地址提交页面
   如果未中奖 → 显示未中奖结果
   ↓
8. 用户提交地址 → 调用 submit-address 接口
   ↓
9. 完成流程
```

---

## 注意事项

1. 所有接口都需要传递 `user_id` 和 `name` 参数
2. 请求体必须是 JSON 格式，`Content-Type` 需要设置为 `application/json`
3. 地址提交后，Redis缓存会自动删除，不需要前端处理
4. 如果用户中奖后超过1天未提交地址，系统会自动取消中奖资格并恢复库存
5. 每个用户只能中一次奖，中奖后即使地址过期被取消，也不能再次中奖