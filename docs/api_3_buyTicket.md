# 4. 航班订票 API 文档

## 通用失败响应 (401 - 未认证): 所有需要 token 的接口，在 token 缺失、无效或过期时，都会返回此错误。
{
  "status": "error",
  "code": 401,
  "message": "用户未登录或会话已过期",
  "data": null
}

## 订单状态说明
- pending_payment: 待支付（订单已创建，等待支付）
- confirmed: 已确认（已支付）
- canceled: 已取消
- expired: 已过期（超时未支付）

### 4.1获取航班信息 (get_airlines)
Action: get_airlines

功能描述: (对应 get_airlines 流程) 用户根据筛选条件搜索航班。这是一个公共接口，无需登录。

请求 (Request) data 内容： 
| 参数名                | 类型  | 必填 | 说明 |  
| departure_city      | string | ✅ | 出发城市 (例如 "北京") | 
| arrival_city        | string | ✅ | 到达城市 (例如 "上海") | 
| departure_date      | string | ✅ | 出发日期 (例如 "2025-11-13") | 
| airline             | string | ❌ | (可选) 航空公司 (例如 "中国国际航空") | 
| time_range_start    | string | ❌ | (可选) 起飞时间范围-开始 (例如 "08:00") | 
| time_range_end      | string | ❌ | (可选) 起飞时间范围-结束 (例如 "12:00") | 
| price_max           | float  | ❌ | (可选) 最高票价 (例如 1000.0) |
| page                | integer | ✅ | 请求的页码，默认为 1 |
| page_size           | integer | ✅ | 每页的数据数，默认为 10 |

{
  "action": "get_airlines",
  "data": {
    "departure_city": "北京",
    "arrival_city": "上海",
    "departure_date": "2025-11-13",
    "airline": "中国国际航空",
    "time_range_start": "08:00",
    "time_range_end": "12:00",
    "price_max": 1500.0,
    "page": 1,
    "page_size": 10
  }
}

success:
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
      },
      {
        "flight_id": "MU5102",
        "airline": "中国东方航空",
        "departure_city": "北京",
        "arrival_city": "上海",
        "departure_airport": "北京首都国际机场 (PEK)",
        "arrival_airport": "上海浦东国际机场 (PVG)",
        "departure_time": "2025-11-13T10:30:00",
        "arrival_time": "2025-11-13T12:50:00",
        "duration_minutes": 140,
        "seat_prices": {
          "economy": 880.00,
          "first_class": 4200.00
        },
        "available_seats": {
          "economy": 32,
          "first_class": 1
        }
      }
    ]
  }
}

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

fail:
{
  "status": "error",
  "code": 400,
  "message": "参数错误：出发日期格式不正确，应为 YYYY-MM-DD 格式",
  "data": null
}

fail:
{
  "status": "error",
  "code": 400,
  "message": "参数错误：出发城市和到达城市不能相同",
  "data": null
}

fail:
{
  "status": "success",
  "code": 200,
  "message": "查询成功，未找到符合条件的航班",
  "data": []
}


### 4.2 创建订单 (create_order)
Action: create_order

功能描述: (对应create_order 流程) 用户选择航班、座位并填写乘客信息后，创建订单（预扣库存）。必须登录。

请求 (Request) data 内容： 
| 参数名 |          类型 | 必填 | 说明 | 
| token |           string | ✅ | 用户登录凭证。 | 
| flight_id |       string | ✅ | 用户选择的航班ID (来自 get_airlines )。| 
| seat_type |       string | ✅ | 座位等级，可选值：economy（经济舱）、first_class（头等舱） |
| passenger |       object | ✅ | 乘客信息（JSON 对象）。 |
    passenger 对象内部结构:
    | 键名 |            类型    |必填 | 说明 |
    passenger.name      (string, ✅): 乘客姓名 (例如 "李四")。
    passenger.id_card   (string, ✅): 乘客身份证号。
    passenger.phone     (string, ✅): 乘客电话。

{
  "action": "create_order",
  "data": {
    "token": "a-valid-user-session-token-obtained-at-login",
    "flight_id": "CA1831",
    "seat_type": "economy",
    "passenger": {
      "name": "李四",
      "id_card": "110101199003071234",
      "phone": "13912345678"
    }
  }
}
success:
{
  "status": "success",
  "code": 200,
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
fail:
{
  "status": "error",
  "code": 400,
  "message": "创建订单失败，该航班经济舱已售罄",
  "data": null
}


### 4.3 支付订单 (pay_order)
Action: pay_order

功能描述: (对应pay_order 流程) 模拟支付。将"待支付"订单更新为"已支付"。必须登录。用户只能支付自己的订单。

请求 (Request) data 内容： 
| 参数名 |      类型 |      必填 | 说明 | 
| token |       string |    ✅ | 用户登录凭证。 |
| order_id |    integer |   ✅ | 要支付的订单ID (来自 create_order)。 |

{
  "action": "pay_order",
  "data": {
    "token": "a-valid-user-session-token-obtained-at-login",
    "order_id": 10001
  }
}
success:
{
  "status": "success",
  "code": 200,
  "message": "支付成功",
  "data": {
    "order_id": 10001,
    "status": "confirmed"
  }
}
fail:
{
  "status": "error",
  "code": 400,
  "message": "支付失败，订单已超时并被取消",
  "data": null
}

fail:
{
  "status": "error",
  "code": 400,
  "message": "支付失败，订单状态不是待支付状态",
  "data": null
}

fail:
{
  "status": "error",
  "code": 403,
  "message": "无权操作此订单",
  "data": null
}

fail:
{
  "status": "error",
  "code": 404,
  "message": "订单不存在",
  "data": null
}

### 4.4 获取我的订单列表 (get_my_orders)
Action: get_my_orders

功能描述: (对应“我的订单”页面) 获取当前登录用户的所有订单列表。必须登录。

请求 (Request) data 内容： 
| 参数名 | 类型 |   必填 | 说明 |         
| token | string | ✅ | 用户登录凭证。 |
| page | integer | ✅ | 请求的页码，默认为 1 |
| page_size | integer | ✅ | 每页的数据数，默认为 10 |

{
  "action": "get_my_orders",
  "data": {
    "token": "a-valid-user-session-token-obtained-at-login",
    "page": 1,
    "page_size": 10
  }
}

success：
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
      },
      {
        "order_id": 9987,
        "flight_id": "ZH1234",
        "departure_city": "北京",
        "arrival_city": "深圳",
        "departure_time": "2025-11-10T14:00:00",
        "status": "canceled",
        "total_price": 1200.00
      }
    ]
  }
}

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

fail:
{
  "status": "error",
  "code": 401,
  "message": "用户未登录或会话已过期",
  "data": null
}

fail:
{
  "status": "error",
  "code": 400,
  "message": "参数错误：页码必须大于0",
  "data": null
}

fail:
{
  "status": "error",
  "code": 400,
  "message": "参数错误：每页数据数必须在1-100之间",
  "data": null
}

### 4.5 获取订单详情 (get_order)
Action: get_order

功能描述: (对应get_order 流程) 获取单个订单的详细信息。必须登录。用户只能查看自己的订单。

请求 (Request) data 内容： 
| 参数名 |      类型 |      必填 | 说明 | 
| token |       string |    ✅ | 用户登录凭证。 | 
| order_id |    integer |   ✅ | 要查询的订单ID。 |

{
  "action": "get_order",
  "data": {
    "token": "a-valid-user-session-token-obtained-at-login",
    "order_id": 10001
  }
}

success:
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

fail:
{
  "status": "error",
  "code": 403,
  "message": "无权查看此订单",
  "data": null
}

fail:
{
  "status": "error",
  "code": 404,
  "message": "订单不存在",
  "data": null
}

### 4.6 取消订单 (cancel_order)
Action: cancel_order

功能描述: (对应你的 cancel_order 流程) 取消一个订单（释放库存）。必须登录。用户只能取消自己的订单。

请求 (Request) data 内容： 
| 参数名 |      类型 |      必填 | 说明 | 
| token |       string |    ✅ | 用户登录凭证。 | 
| order_id |    integer |   ✅ | 要取消的订单ID。 |

{
  "action": "cancel_order",
  "data": {
    "token": "a-valid-user-session-token-obtained-at-login",
    "order_id": 10001
  }
}

success:
{
  "status": "success",
  "code": 200,
  "message": "订单已取消",
  "data": {
    "order_id": 10001,
    "status": "canceled"
  }
}
fail:
{
  "status": "error",
  "code": 403,
  "message": "无权操作此订单",
  "data": null
}

fail:
{
  "status": "error",
  "code": 400,
  "message": "订单已起飞，无法取消",
  "data": null
}

fail:
{
  "status": "error",
  "code": 400,
  "message": "订单已处于取消状态，请勿重复操作",
  "data": null
}

fail:
{
  "status": "error",
  "code": 404,
  "message": "订单不存在",
  "data": null
}
