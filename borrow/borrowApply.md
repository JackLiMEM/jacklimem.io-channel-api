# 借款申请接口文档
###	**1.接口说明**
用于短信验证通过后，确认借款时使用。

### **2. 接口 URL**
~~~java
/api/beidou/borrow/confirm
~~~

### **3.请求参数**{#请求参数}

| 参数    |  类型   | 是否必须 | 说明    |
| ------  | ------ | ------ | ------  |
|   orderNo  		|  String |  Y   		|  订单号  	|
|   loanAmount  |  String |  Y   		|  借款金额  |
|   loanTerm  	|  String |  Y   		|  借款期限  |
|   loanUnit  	|  int |  Y  		|  期限单位 <br/>1-日 2-月 3-年 |
|   repayReturnUrl  |  String |  N   |  确认借款回跳地址|


>示例：
~~~
略
~~~

### **4. 响应参数**{#响应参数}

| 参数    |  类型   | 是否必须 | 说明    |
| ------  | ------ | ------ | ------  |
| code | integer | 是 | 响应码，如200|
| msg | integer | 是 | 响应信息，如成功 |
| data | Object| 是 | json数据|

>示例：
~~~java
{
	"code": 200,
	"msg": "成功",
	"data": {"result":"200","remark":"success"}
}
~~~
