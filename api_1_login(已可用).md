# 1. 核心约定
### 1.1 通信方式
协议： HTTP/HTTPS

数据格式： JSON

### 1.2 请求格式
RESTful API 使用标准的 HTTP 方法和 URL 路径。

**认证方式：**
- 需要认证的接口在请求头中携带 `Authorization: Bearer {token}`
- token 通过登录接口获取

### 1.3 响应格式
所有 API 响应都使用统一的 JSON 结构：
```json
{
  "status": "String ('success' or 'error')",
  "code": "Integer (HTTP status code)",
  "message": "String (描述信息)",
  "data": "Object or Null (返回的具体数据)"
}
```


# 2. HTTP 状态码
API 使用标准的 HTTP 状态码，code 字段与 HTTP 状态码保持一致：

状态码,   含义,           典型场景,                   客户端处理建议
200,      成功,          操作成功,                    显示成功提示
201,      已创建,        资源创建成功,                显示成功提示并跳转
400,      请求错误,      参数格式错误,                提示用户检查输入
401,      未授权,        认证失败或token无效,         提示"请重新登录"
403,      禁止访问,      权限不足或账户被禁用,        提示"账户已被禁用，请联系管理员"
404,      未找到,        资源不存在,                  提示"用户不存在"
409,      冲突,          资源冲突,                    提示"用户名已存在"或"手机号已注册"
422,      验证错误,      请求数据不符合业务规则,      提示具体的验证错误信息
500,      服务器错误,    服务器内部错误,               提示"服务器错误，请稍后重试"

# 3. API 接口详情
### 3.1 用户注册
**接口**: `POST /api/users`

**功能描述**: 新用户注册为乘客。

**请求参数** (JSON Body):
```json
{
  "username": "zhangsan123",
  "password": "password123",
  "phone_number": "13800138000",
  "email": "124@qq.com",
  "security_question": "我的小学名称是？",
  "security_answer": "阳光小学"
}
```

**参数说明**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| username | string | ✅ | 用户名 (3-50字符) |
| password | string | ✅ | 密码 (6-20字符) |
| phone_number | string | ✅ | 手机号 (11位数字) |
| email | string | ✅ | 用户邮箱 (3-50字符) |
| security_question | string | ✅ | 用户设置的安全问题 |
| security_answer | string | ✅ | 对应的答案 |

**成功响应** (201 Created):
```json
{
  "status": "success",
  "code": 201,
  "message": "注册成功",
  "data": {
    "user_id": 1001,
    "username": "zhangsan123"
  }
}
```

**失败响应示例** (409 Conflict):
```json
{
  "status": "error",
  "code": 409,
  "message": "用户名 'zhangsan123' 已被占用",
  "data": null
}
```

### 3.2 用户登录
**接口**: `POST /api/auth/login`

**功能描述**: 乘客使用用户名或手机号登录。

**请求参数** (JSON Body):
```json
{
  "identifier": "zhangsan123",
  "password": "password123"
}
```

**参数说明**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| identifier | string | ✅ | 用户名 或 手机号 |
| password | string | ✅ | 密码 |

**成功响应** (200 OK):
```json
{
  "status": "success",
  "code": 200,
  "message": "登录成功",
  "data": {
    "token": "a-long-and-unique-session-token-generated-by-server",
    "user_id": 1001,
    "username": "zhangsan123"
  }
}
```

**失败响应示例** (401 Unauthorized):
```json
{
  "status": "error",
  "code": 401,
  "message": "用户名或密码错误",
  "data": null
}
```


### 3.3 修改密码
**接口**: `PUT /api/users/password`

**功能描述**: 已登录用户通过提供旧密码来设置新密码。

**请求头**:
```
Authorization: Bearer {token}
```

**请求参数** (JSON Body):
```json
{
  "old_password": "MyOldPassword123",
  "new_password": "MyNewSecurePassword456",
  "confirm_password": "MyNewSecurePassword456"
}
```

**参数说明**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| old_password | string | ✅ | 用户的当前（旧）密码 |
| new_password | string | ✅ | 新密码 |
| confirm_password | string | ✅ | 确认新密码（服务器必须再次校验） |

**成功响应** (200 OK):
```json
{
  "status": "success",
  "code": 200,
  "message": "密码修改成功",
  "data": null
}
```

**失败响应示例**:
- 401 Unauthorized: 用户未登录或会话已过期
- 400 Bad Request: 旧密码不正确 / 新密码和确认密码不一致

### 3.4 获取安全问题
**接口**: `GET /api/users/{username}/security-question`

**功能描述**: 用户输入账号，服务器返回该账号注册时设置的安全问题。

**路径参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| username | string | ✅ | 尝试找回密码的账号 |

**成功响应** (200 OK):
```json
{
  "status": "success",
  "code": 200,
  "message": "获取成功",
  "data": {
    "question": "我的小学名称是？"
  }
}
```

**失败响应示例** (404 Not Found):
```json
{
  "status": "error",
  "code": 404,
  "message": "用户不存在",
  "data": null
}
```

### 3.5 重置密码
**接口**: `POST /api/auth/reset-password`

**功能描述**: 用户提交安全问题的答案和新密码。如果答案正确，则更新密码。

**请求参数** (JSON Body):
```json
{
  "username": "zhangsan123",
  "security_answer": "阳光小学",
  "new_password": "MyNewSecurePassword789",
  "confirm_password": "MyNewSecurePassword789"
}
```

**参数说明**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| username | string | ✅ | 目标账号 |
| security_answer | string | ✅ | 用户对安全问题的回答 |
| new_password | string | ✅ | 新密码 |
| confirm_password | string | ✅ | 确认新密码 (服务器校验) |

**成功响应** (200 OK):
```json
{
  "status": "success",
  "code": 200,
  "message": "密码重置成功，请使用新密码登录",
  "data": null
}
```

**失败响应示例**:
- 400 Bad Request: 安全问题答案不正确 / 新密码和确认密码不一致
- 404 Not Found: 用户不存在