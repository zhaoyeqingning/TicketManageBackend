# 1. 核心约定
### 1.1 通信方式
协议： TCP Socket

数据格式： JSON

### 1.2 客户端请求格式
所有从客户端发往服务器的请求都使用此 JSON 结构：
{
  "action": "String (e.g., sign_up)",
  "data": {
    "param1": "value1",
    "param2": "value2"
  }
}

### 1.3 服务器响应格式
所有从服务器返回给客户端的响应都使用此 JSON 结构：
{
  "status": "String ('success' or 'error')",
  "code": "Integer (e.g., 200, 401, 500)",
  "message": "String (描述信息)",
  "data": "Object or Null (返回的具体数据)"
}


# 2. 通用状态码 (Status Code)
code 字段是前端判断如何响应的关键。以下是通用代码表：

错误码,   错误类型,        典型场景,                   客户端处理建议
200,      SUCCESS,       操作成功,                    显示成功提示
400,      BAD_REQUEST,   参数格式错误,                提示用户检查输入（例如手机号非11位）
401,      UNAUTHORIZED,  认证失败,                    提示“用户名或密码错误”
403,      FORBIDDEN,     权限不足 / 账户被禁用,        提示“账户已被禁用，请联系管理员”
404,      NOT_FOUND,     资源不存在,                  提示“用户不存在”
409,      CONFLICT,      资源冲突,                    提示“用户名已存在”或“手机号已注册”
500,      SERVER_ERROR,  服务器内部错误,               提示“服务器错误，请稍后重试”

# 3. API 接口详情
### 3.1 用户注册 (sign_up)
Action: sign_up

功能描述: 新用户注册为乘客。
参数名,         类型,       必填,       说明
username      string,       ✅,     用户名 (3-50字符)
password,     string,       ✅,     密码 (6-20字符)
phone_number, string,       ✅,     手机号 (11位数字)
email         string        yes      用户邮箱(3-50字符)
security_question	string	  ✅	用户设置的安全问题 (例如 "你就读的小学叫什么？")
security_answer	string	    ✅	对应的答案 (例如 "XX小学")

json:
{
  "action": "sign_up",
  "data": {
    "username": "zhangsan123",
    "password": "password123",
    "phone_number": "13800138000",
    "email": "124@qq.com",
    "security_question": "我的小学名称是？",
    "security_answer": "阳光小学"
  }
}
success:
{
  "status": "success",
  "code": 200,
  "message": "注册成功",
  "data": {
    "user_id": 1001,
    "username": "zhangsan123"
  }
}
fail:
{
  "status": "error",
  "code": 409,
  "message": "用户名 'zhangsan123' 已被占用",
  "data": null
}

### 3.2 用户登录 (sign_in)
Action: sign_in

功能描述: 乘客使用用户名或手机号登录。

请求 data 内容：
参数名,      类型,      必填,       说明
identifier, string,     ✅,     用户名 或 手机号
password,   string,     ✅,     密码

{
  "action": "sign_in",
  "data": {
    "identifier": "zhangsan123",
    "password": "password123"
  }
}

success:
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

fail:
{
  "status": "error",
  "code": 401,
  "message": "用户名或密码错误",
  "data": null
}


### 3.3 已经登陆的用户修改密码 (change_password)
Action: change_password

功能描述: 已登录用户通过提供旧密码来设置新密码。

请求 data 内容：
参数名,               类型,       必填,         说明
token,              string,       ✅,         用户登录后获取的会话凭证。
old_password,       string,       ✅,         用户的当前（旧）密码。
new_password,       string,       ✅,         新密码。
confirm_password,   string,       ✅,         确认新密码（服务器必须再次校验）。

{
  "action": "change_password",
  "data": {
    "token": "a-valid-user-session-token-obtained-at-login",
    "old_password": "MyOldPassword123",
    "new_password": "MyNewSecurePassword456",
    "confirm_password": "MyNewSecurePassword456"
  }
}

success:
{
  "status": "success",
  "code": 200,
  "message": "密码修改成功",
  "data": null
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
  "message": "旧密码不正确",
  "data": null
}

fail:
{
  "status": "error",
  "code": 400,
  "message": "新密码和确认密码不一致",
  "data": null
}

### 3.4 忘记密码第一步：获取安全问题 (get_security_question)

Action: get_security_question
功能描述: 用户输入账号，服务器返回该账号注册时设置的安全问题。

请求 (Request) data 内容：
参数名,         类型,     必填,     说明
username,       string,   ✅,     尝试找回密码的账号。

{
  "action": "get_security_question",
  "data": {
    "username": "zhangsan123"
  }
}
success:
{
  "status": "success",
  "code": 200,
  "message": "获取成功",
  "data": {
    "question": "我的小学名称是？"
  }
}
fail:
{
  "status": "error",
  "code": 404,
  "message": "用户不存在",
  "data": null
}

### 3.5 验证答案并重置密码 (reset_password_with_answer)

Action: reset_password_with_answer
功能描述: 用户提交安全问题的答案和新密码。如果答案正确，则更新密码。  

参数名,             类型,       必填,       说明
username,           string,     ✅,       目标账号。
security_answer,    string,     ✅,       用户对安全问题的回答。
new_password,       string,     ✅,       新密码。
confirm_password,   string,     ✅,       确认新密码 (服务器校验)。

{
  "action": "reset_password_with_answer",
  "data": {
    "username": "zhangsan123",
    "security_answer": "阳光小学",
    "new_password": "MyNewSecurePassword789",
    "confirm_password": "MyNewSecurePassword789"
  }
}
success:
{
  "status": "success",
  "code": 200,
  "message": "密码重置成功，请使用新密码登录",
  "data": null
}
fail:
{
  "status": "error",
  "code": 400,
  "message": "安全问题答案不正确",
  "data": null
}
fail:
{
  "status": "error",
  "code": 404,
  "message": "用户不存在",
  "data": null
}