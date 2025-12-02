# 个人中心 API 文档
用户信息管理相关的 RESTful API，所有接口都需要用户登录后才能访问。

**认证方式**: 在请求头中携带 `Authorization: Bearer {token}`

### 5.1 获取用户信息
**接口**: `GET /api/users/profile`

**功能描述**: 获取当前登录用户的详细资料，用于"个人信息"页面展示。

**请求头**:
```
Authorization: Bearer {token}
```

**成功响应** (200 OK):
```json
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
```

**失败响应示例** (401 Unauthorized):
```json
{
  "status": "error",
  "code": 401,
  "message": "用户未登录或会话已过期",
  "data": null
}
```

### 5.2 获取我的订单列表
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

**失败响应示例** (401 Unauthorized):
```json
{
  "status": "error",
  "code": 401,
  "message": "用户未登录或会话已过期",
  "data": null
}
```

### 5.3 修改用户名
**接口**: `PUT /api/users/username`

**功能描述**: 修改当前登录用户的用户名。

**请求头**:
```
Authorization: Bearer {token}
```

**请求参数** (JSON Body):
```json
{
  "new_username": "zhangsan_new"
}
```

**参数说明**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| new_username | string | ✅ | 想要设置的新用户名 |

**成功响应** (200 OK):
```json
{
  "status": "success",
  "code": 200,
  "message": "用户名修改成功",
  "data": {
    "new_username": "zhangsan_new"
  }
}
```

**失败响应示例** (409 Conflict):
```json
{
  "status": "error",
  "code": 409,
  "message": "用户名 'zhangsan_new' 已被占用",
  "data": null
}
```

### 5.4 退出登录
**接口**: `POST /api/auth/logout`

**功能描述**: 使当前用户的 token 立即失效。

**请求头**:
```
Authorization: Bearer {token}
```

**成功响应** (200 OK):
```json
{
  "status": "success",
  "code": 200,
  "message": "退出登录成功",
  "data": null
}
```

**失败响应示例** (401 Unauthorized):
```json
{
  "status": "error",
  "code": 401,
  "message": "Token 无效或已过期",
  "data": null
}
```