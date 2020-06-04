# 百融大额对接开发设计文档

## 需求背景

 市场增量，合作大额产品，为后期合规化道路做准备

## 需求范围

榕树 APP

## 概要设计

### 描述

此需求对接的产品是大额授信产品，涉及的流程依次为：准入、进件、授信、用款确认、绑卡（包括新网开户）、订单审核、放款、还款

### 业务流程设计

#### 一. 待开发功能列表

##### 1.用户过滤接口

 对方接口文档地址：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/product/1.1.html>
  注：使用 MD5 方式做校验

##### 2.借款试算接口

 对方接口文档地址：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/product/1.2.html>

##### 3.协议列表查询接口

 对方接口文档地址：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/product/1.14.html>

##### 4.订单信息推送接口

 对方接口文档地址：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/product/1.3.html>
  业务实现：
  1.初始化用户信息、三方表信息、三方转换表信息、三方数据表（数据上传 OSS ）
  2.初始化百融运营商数据拉取记录表（新建），记录榕树的 uid

##### 5.授信数据拉取定时任务

 对方接口文档地址：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/platform/1.11.html>
  业务实现：
  1.定时任务名称：RongshuOperatorPullJob
  2.定时任务查询运营商数据拉取记录表中待拉取的记录，分别拉取运营商数据、信用卡数据、设备信息
  3.更新三方表状态为待解析( 2 )

##### 6.榕树大额订单解析定时任务（新增）

 业务实现：
  1.定时任务名称：RongshuPushOrderJob
  2.具体实现参照 融360 订单解析定时任务

##### 7.发起授信申请定时任务修改

##### 8.榕树授信结果回调处理消费者（新增）

业务实现：
 1.消费者名称：RongshuCreditResultProcessor
 2.业务实现参照 Rong360CreditResultProcessor
 3.授信通过，三方表状态从授信中（ 11 ） 变为授信通过待确认（ 12 ）;  授信拒绝，三方表状态从授信中（ 11 ） 变为取消（ 4 ），并回调榕树审批拒绝
 涉及的对方接口文档地址：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/platform/1.1.html>

##### 9.审核结果查询接口

对方接口文档地址： <https://dymapi.xiaqiu.cn/groot/_book/chapter2/product/1.8.html> 业务实现：
 1.查询三方订单表，未授信状态（ 0，10，11 ），返回初审处理中（ 6003 ）；拒绝状态（ 4，5 ），返回初审拒绝（ 6004 ），失败原因：资质不符；授信通过的状态（ 12，1 ），返回初审通过 （ 6001 ）

##### 10.支持银行卡列表接口

对方接口文档地址：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/product/1.15.html>
 业务实现：
 1.查询 bank 表，已有实现的代码，参照即可

##### 11.绑卡接口

对方接口文档地址：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/product/1.4.html>
 业务实现：
 1.无验证码绑卡
 2.首次绑卡时，调用绑卡 API 绑卡，且绑定的卡既是默认放款卡也是默认还款卡
 3.重复绑卡，即当前卡已绑过，更新默认卡标识和绑卡时间
 4.绑卡成功后，更新三方订单表 authInfo 字段的为已绑卡
 5.具体实现参照 融360 绑卡接口

##### 12.  用户确认借款接口开发

对方接口文档地址：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/product/1.7.html>
 业务实现：
 1.如果用户已新网开户，接口逻辑如下：
 a.检查用户是否可借，如果可借，继续往下执行；不可借，接口返回拒贷
 b.根据用户输入的借款金额、期限等初始化借款订单表数据
 c.更新三方订单表状态，从 授信通过待确认（ 12 ）改为已处理（ 1 ）
 d.b 和 c 两个操作必须保证数据一致性，同时成功或同时失败
 2.如果用户未新网开户成功，接口逻辑如下：
 a.表 cl_borrow_confirm_log 中记录该接口的业务参数；用户重复点击，这张表有对 orderNo 的唯一约束，捕获异常，做幂等。
 b.接口中返回新网开户页地址

##### 13.新网开户状态检查定时任务（新增）

业务实现：
 1.定时任务名称：OpenapiXWAuthStateCheckJob
 2.查询三方表中的状态是授信通过待确认（ 12 ），且新网认证申请表（ cl_depository_register_apply ）中的状态是 1 小时前发起申请的、状态没变更过的记录
 3.查询表 cl_borrow_confirm_log 中的数据，执行功能列表 9 中的步骤 1。
 4.如果和 9 出现并发，即初始化的借款订单重复，需要捕获异常做幂等，打出日志信息即可

##### 14.订单风控审核...

##### 15.榕树订单审核结果处理消费者（新增）

业务实现：
 1.消费者名称：RongshuAuditResultProcessor
 2.如果订单审核通过，直接 return；审核拒绝，生成榕树放款失败-拒件处理（ 7003 ）的回调
 涉及的对方接口文档地址：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/platform/1.2.html>

##### 16.榕树放款结果处理消费者（新增）

业务实现：
 1.消费者名称：RongshuLoanResultProcessor
 2.该消费者的业务功能只有一个：处理放款失败后，运营在管理后台拒单的业务，即判断订单状态是否等于人工复审拒绝（ 27 ），如果是，生成榕树放款失败-拒件处理（ 7003 ）的回调
 涉及的对方接口文档地址：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/platform/1.2.html>

##### 17.放款结果查询接口

对方接口文档地址：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/product/1.9.html>
 业务实现：
 1.查询借款订单表（ cl_borrow ）,如果是审核中和放款失败的状态（ 0，10，13，20，31 ），返回放款中（ 7002  ）；如果是人工复审拒绝（ 27 ），返回放款失败-拒单处理（ 7003 ），如果是已放款状态（ 30，40，41 ），返回放款成功

##### 18.还款计划查询接口

对方接口文档地址：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/product/1.13.html> 业务实现：
 1.查询还款计划，按照接口要求返回数据

##### 19.生成还款计划消费者（ RepayPlanProcessor ）（修改）

业务实现：
 1.在还款计划生成后的回调处理业务中，增加生成榕树放款成功的回调
 2.代码位置：repayPlanOpenAPI.callBackWhenLoanSuccess
 涉及对方接口文档地址：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/platform/1.2.html>

##### 20.榕树订单逾期处理消费者（新增）

对方接口文档：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/platform/1.4.html>
 1.消费者名称：RongshuOverdueProcessor 2.生成还款计划的回调，具体方法在 API 中实现，其他地方还要用该方法

##### 21.主动还款试算接口

对方接口文档：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/product/1.10.html>
 业务实现：
 1.查询订单的还款计划再判断
 2.参照接口描述

##### 22.主动还款接口

对方接口文档：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/product/1.11.html>
 业务实现：
 1.参照 融360 还款接口

##### 23.还款结果处理消费者（新增）

对方接口文档https://dymapi.xiaqiu.cn/groot/_book/chapter2/platform/1.3.html
 业务实现：
 1.还款后，回调处理用
 2.还需要回调还款计划

##### 24.还款结果查询接口 对方接口文档：<https://dymapi.xiaqiu.cn/groot/_book/chapter2/product/1.12.html>

业务实现：
 1.参照接口描述

##### 25.其他

1.拦截器开发、生成回调的公共方法、回调方法
 2.产品基础数据配置，准入规则配置、新网开户页配置
 3.授信&审核被拒后的可借日期配置、短信签名配置

### 表结构设计

```
 新建表 ：rongshu_credit_data_pull_log
```

### 历史数据处理

无

### 流程图

> [点击跳转](/flowChart/百融渠道对接.md)