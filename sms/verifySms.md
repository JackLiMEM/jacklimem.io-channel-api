# 短信确认接口文档

# 1. 接口说明 {#接口说明}

**本接口主要针对借款前验证短信验证码。**

借款确认前调用此接口处理保险签约和连资贷签约相关短信发送业务

# 2. 接口标识 {#接口标识}
> 请求方式： post 

> 请求地址：${domain}/api/h5/openapi/borrow/verifyBorrowConfirmSms

# 3. 请求参数 {#请求参数}

| 参数 | 类型 | 是否必选 | 描述 |
| :--- | :--- | :--- | :--- |
| orderNo | string | 是 | 我方订单号 |
| optType | int | 是 | 操作类型1保险签约2支付认证 |
| captcha |string|是|验证码


# 4. 响应参数 {#响应参数}

| 参数 | 类型 | 是否必选 | 描述 |
| :--- | :--- | :--- | :--- |
| code | integer | 是 | 响应码，如200|
| msg | integer | 是 | 响应信息，如成功 |
| data | Object| 是 | json数据|

# 5. 请求示例 {#请求示例}

```
略
```

# 6. 响应示例 {#响应示例}

```
{
  "code": 200,
  "data": {
    "jumpUrl": "https://gateway-openapi-test01.XXX.XXX",
    "optType": 2,
    "orderNo": "HK76190517450442",
    "phone": "15313288116",
    "state": false
  },
  "msg": "成功"
}
```






