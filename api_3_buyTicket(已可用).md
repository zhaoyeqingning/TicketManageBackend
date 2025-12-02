# 航班订票 API 文档

**认证方式**: 需要认证的接口在请求头中携带 `Authorization: Bearer {token}`

## 通用认证失败响应
所有需要认证的接口在 token 缺失、无效或过期时返回：
```json
{
  "status": "error",
  "code": 401,
  "message": "用户未登录或会话已过期",
  "data": null
}
```

## 订单状态说明
- `pending_payment`: 待支付（订单已创建，等待支付）
- `confirmed`: 已确认（已支付）
- `canceled`: 已取消
- `expired`: 已过期（超时未支付）

### 4.1 搜索航班
**接口**: `GET /api/flights`

**功能描述**: 用户根据筛选条件搜索航班。这是一个公共接口，无需登录。

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| departure_city | string | ✅ | 出发城市 (例如 "北京") |
| arrival_city | string | ✅ | 到达城市 (例如 "上海") |
| departure_date | string | ✅ | 出发日期 (例如 "2025-11-13") |
| airline | string | ❌ | (可选) 航空公司 (例如 "中国国际航空") |
| time_range_start | string | ❌ | (可选) 起飞时间范围-开始 (例如 "08:00") |
| time_range_end | string | ❌ | (可选) 起飞时间范围-结束 (例如 "12:00") |
| price_max | float | ❌ | (可选) 最高票价 (例如 1000.0) |
| page | integer | ❌ | 请求的页码，默认为 1 |
| page_size | integer | ❌ | 每页的数据数，默认为 10 |

**成功响应** (200 OK):
```json
{
  "status": "success",
  "code": 200,
  "message": "查询成功",
  "data": {
    "total_items": 125,
    "total_pages": 13,
    "current_page": 1,
    "page_size": 10,
    "list": [
      {
        "flight_id": "CA1831",
        "airline": "中国国际航空",
        "departure_city": "北京",
        "arrival_city": "上海",
        "departure_airport": "北京首都国际机场 (PEK)",
        "arrival_airport": "上海虹桥国际机场 (SHA)",
        "departure_time": "2025-11-13T08:00:00",
        "arrival_time": "2025-11-13T10:25:00",
        "duration_minutes": 145,
        "seat_prices": {
          "economy": 950.00,
          "first_class": 4500.00
        },
        "available_seats": {
          "economy": 45,
          "first_class": 2
        }
      }
    ]
  }
}
```

**空结果响应** (200 OK):
```json
{
  "status": "success",
  "code": 200,
  "message": "查询成功，未找到符合条件的航班",
  "data": {
    "total_items": 0,
    "total_pages": 0,
    "current_page": 1,
    "page_size": 10,
    "list": []
  }
}
```

**失败响应示例**:
- 400 Bad Request: 参数错误（出发日期格式不正确、出发城市和到达城市不能相同等）


### 4.2 创建订单
**接口**: `POST /api/orders`

**功能描述**: 用户选择航班、座位并填写乘客信息后，创建订单（预扣库存）。

**请求头**:
```
Authorization: Bearer {token}
```

**请求参数** (JSON Body):
```json
{
  "flight_id": "CA1831",
  "seat_type": "economy",
  "passenger": {
    "name": "李四",
    "id_card": "110101199003071234",
    "phone": "13912345678"
  }
}
```

**参数说明**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| flight_id | string | ✅ | 用户选择的航班ID |
| seat_type | string | ✅ | 座位等级，可选值：economy、first_class |
| passenger | object | ✅ | 乘客信息 |
| passenger.name | string | ✅ | 乘客姓名 |
| passenger.id_card | string | ✅ | 乘客身份证号 |
| passenger.phone | string | ✅ | 乘客电话 |

**成功响应** (201 Created):
```json
{
  "status": "success",
  "code": 201,
  "message": "订单创建成功，请在15分钟内支付",
  "data": {
    "order_id": 10001,
    "status": "pending_payment",
    "flight_id": "CA1831",
    "passenger_name": "李四",
    "total_price": 950.00,
    "expires_at": "2025-11-12T17:35:00"
  }
}
```

**失败响应示例** (400 Bad Request):
```json
{
  "status": "error",
  "code": 400,
  "message": "创建订单失败，该航班经济舱已售罄",
  "data": null
}
```


### 4.3 支付订单
**接口**: `POST /api/orders/{order_id}/payment`

**功能描述**: 模拟支付，将"待支付"订单更新为"已支付"。用户只能支付自己的订单。

**请求头**:
```
Authorization: Bearer {token}
```

**路径参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| order_id | integer | ✅ | 要支付的订单ID |

**成功响应** (200 OK):
```json
{
  "status": "success",
  "code": 200,
  "message": "支付成功",
  "data": {
    "order_id": 10001,
    "status": "confirmed"
  }
}
```

**失败响应示例**:
- 400 Bad Request: 订单已超时并被取消 / 订单状态不是待支付状态
- 403 Forbidden: 无权操作此订单
- 404 Not Found: 订单不存在

### 4.4 获取我的订单列表
**接口**: `GET /api/users/orders`

**功能描述**: 获取当前登录用户的所有订单列表。

**请求头**:
```
Authorization: Bearer {token}
```

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| page | integer | ❌ | 请求的页码，默认为 1 |
| page_size | integer | ❌ | 每页的数据数，默认为 10 |

**成功响应** (200 OK):
```json
{
  "status": "success",
  "code": 200,
  "message": "查询成功",
  "data": {
    "total_items": 105,
    "total_pages": 11,
    "current_page": 1,
    "page_size": 10,
    "list": [
      {
        "order_id": 10001,
        "flight_id": "CA1831",
        "departure_city": "北京",
        "arrival_city": "上海",
        "departure_time": "2025-11-13T08:00:00",
        "status": "confirmed",
        "total_price": 950.00
      }
    ]
  }
}
```

**空结果响应** (200 OK):
```json
{
  "status": "success",
  "code": 200,
  "message": "查询成功，未找到订单",
  "data": {
    "total_items": 0,
    "total_pages": 0,
    "current_page": 1,
    "page_size": 10,
    "list": []
  }
}
```

**失败响应示例**:
- 400 Bad Request: 参数错误（页码必须大于0、每页数据数必须在1-100之间）
- 401 Unauthorized: 用户未登录或会话已过期

### 4.5 获取订单详情
**接口**: `GET /api/orders/{order_id}`

**功能描述**: 获取单个订单的详细信息。用户只能查看自己的订单。

**请求头**:
```
Authorization: Bearer {token}
```

**路径参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| order_id | integer | ✅ | 要查询的订单ID |

**成功响应** (200 OK):
```json
{
  "status": "success",
  "code": 200,
  "message": "查询成功",
  "data": {
    "order_id": 10001,
    "status": "confirmed",
    "total_price": 950.00,
    "created_at": "2025-11-12T17:20:00",
    "paid_at": "2025-11-12T17:25:00",
    "seat_type": "economy",
    "flight_info": {
      "flight_id": "CA1831",
      "airline": "中国国际航空",
      "departure_city": "北京",
      "arrival_city": "上海",
      "departure_time": "2025-11-13T08:00:00",
      "arrival_time": "2025-11-13T10:25:00"
    },
    "passenger_info": {
      "name": "李四",
      "id_card": "110101...1234",
      "phone": "139...678"
    }
  }
}
```

**失败响应示例**:
- 403 Forbidden: 无权查看此订单
- 404 Not Found: 订单不存在

### 4.6 取消订单
**接口**: `DELETE /api/orders/{order_id}`

**功能描述**: 取消一个订单（释放库存）。用户只能取消自己的订单。

**请求头**:
```
Authorization: Bearer {token}
```

**路径参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| order_id | integer | ✅ | 要取消的订单ID |

**成功响应** (200 OK):
```json
{
  "status": "success",
  "code": 200,
  "message": "订单已取消",
  "data": {
    "order_id": 10001,
    "status": "canceled"
  }
}
```

**失败响应示例**:
- 400 Bad Request: 订单已起飞，无法取消 / 订单已处于取消状态，请勿重复操作
- 403 Forbidden: 无权操作此订单
- 404 Not Found: 订单不存在
