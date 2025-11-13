# 5. 个人中心 API 文档
“个人信息”页面所需的 API，它们都必须登录后才能访问。

### 5.1 获取用户信息 (get_userinfo)
Action: get_userinfo

功能描述: (对应 get_userinfo 流程) 获取当前登录用户的详细资料，用于“个人信息”页面展示。必须登录。

请求 (Request) data 内容： 
| 参数名 | 类型    |   必填  | 说明 | 
| token  | string | ✅   | 用户登录凭证。 (不使用 账号) |

{
  "action": "get_userinfo",
  "data": {
    "token": "a-valid-user-session-token-obtained-at-login"
  }
}
success:
{
  "status": "success",
  "code": 200,
  "message": "获取用户信息成功",
  "data": {
    "user_id": 1001,
    "username": "zhangsan123",
    "phone_number": "138...000",
    "email": "124@qq.com",
    "security_question": "我的小学名称是？"
  }
}
fail:
{
  "status": "error",
  "code": 401,
  "message": "用户未登录或会话已过期",
  "data": null
}

### 5.2 获取我的订单列表 (get_my_orders)
(流程图称之为 get_order，但它指向 "我的订单" 页面，这是一个列表。 )

Action: get_my_orders

功能描述: (对应流程图 "我的订单" -> get_order) 获取当前登录用户的所有订单列表。必须登录。

请求 (Request) data 内容： 
| 参数名 | 类型      | 必填 | 说明 | 
| token  | string    | ✅ | 用户登录凭证。 (不使用账号) |
| page   | integer    | ✅ |请求的页码，默认为 1 |
| page_size | integer | ✅ |每页的数据数，默认为 10 |

{
  "action": "get_my_orders",
  "data": {
    "token": "a-valid-user-session-token-obtained-at-login",
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

### 5.3 修改用户名 (change_username)
Action: change_username

功能描述: (对应 change_username 流程) 修改当前登录用户的用户名。必须登录。

请求 (Request) data 内容： 
| 参数名         | 类型   | 必填  | 说明 |
| token         | string | ✅   | 用户登录凭证。 (已修正) | 
| new_username  | string | ✅   | 想要设置的新用户名。 |

{
  "action": "change_username",
  "data": {
    "token": "a-valid-user-session-token-obtained-at-login",
    "new_username": "zhangsan_new"
  }
}

success:
{
  "status": "success",
  "code": 200,
  "message": "用户名修改成功",
  "data": {
    "new_username": "zhangsan_new"
  }
}
fail:
{
  "status": "error",
  "code": 409,
  "message": "用户名 'zhangsan_new' 已被占用",
  "data": null
}

### 5.4 退出登录 (logout)
Action: logout

功能描述: (对应 退出登录 流程) 使当前用户的 token 立即失效。必须登录。

请求 (Request) data 内容： 
| 参数名 | 类型 | 必填 | 说明 | 
| token | string | ✅ | 需要被注销的用户登录凭证。 |

{
  "action": "logout",
  "data": {
    "token": "a-valid-user-session-token-obtained-at-login"
  }
}
success:
{
  "status": "success",
  "code": 200,
  "message": "退出登录成功",
  "data": null
}
fail:
{
  "status": "error",
  "code": 401,
  "message": "Token 无效或已过期",
  "data": null
}