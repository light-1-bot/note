# Pass Auto Renew(具体自动续费)



## 1. 需求背景

- *需求地址：

- C端：https://cooper.didichuxing.com/knowledge/2204291401492/2204196789269

- B端：https://cooper.didichuxing.com/knowledge/2204291401492/2204666983514

- 支付：https://cooper.didichuxing.com/knowledge/2200838214095/2204624205369

- *望岳地址：

- C端：https://ddp.intra.xiaojukeji.com/requirement/story/R-IBG-503009?backUrl=/workbench/requirement

- B端：https://ddp.intra.xiaojukeji.com/requirement/story/R-IBG-540006?backUrl=/requirement/list?filterDirId=600

- 相关依赖：

- 购买触发：@庹嘉明

- 支付：@何琳琛

- 下单接口：https://wiki.intra.xiaojukeji.com/pages/viewpage.action?pageId=888132480

- 回调接口：https://wiki.intra.xiaojukeji.com/pages/viewpage.action?pageId=989888954

- OS2对外开放的email&push的能力

- https://cooper.didichuxing.com/knowledge/2203105555297/2203105394449

- 文案检查: 

- 购买邮件相关文案

- 技术方案&测试case中需要补充历史活动兼容，尤其是B端



## 2. 技术方案设计



### **2.1 需求分析拆解**

> 权益管理逻辑

| 功能         | 图例                                                         | 功能详情                                                     | 方案                                                         | 备注 |
| :----------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :--- |
| 自动续费能力 |                                                              | 续费订单不扣库存和频次，忽略上下架、售卖时间订阅计划是否打开续费开关（需要区分是一直未打开还是，开启变为关闭）下一次扣款时间下一次扣款金额支付方式续费生命周期的记录自动续费的各个环节都有通知用户逾期重新激活时，立即扣款发券包且打开自动续费自动续费时间的计算If the user completes the purchase with the checkbox selected, **future charges** will happen on the **same day of the month** as the original purchase.             Example: If the user signs up with auto-renewal on **April 10**, the next charges should happen on **May 10**, **June 10**, and so on, always using the **same payment method** used in the first purchase.              If the user signs up on a **day that doesn’t exist in the next month** (for example, **January 31**), the charge will happen on the **previous valid day** (like **February 28 or 29**, depending on the year). | 购买接口增加参数，根据此参数不扣库存和频次，忽略上下架、售卖时间订阅计划表：续费订阅的唯一key，便于查询一个订阅计划表，一个每次的续购记录表。计划表和记录表的关系是1对多记录表需要继续扣款的请求&结果相应环节产生MQ进行异步处理扣款发券包由业务处理，成功后由业务通知续费管理模块日期计算逻辑根据首次购买时间，计算下一次续费日期。1.31 -> 2.28(29)->3.31 -> 4.30 |      |
| 续费数据管理 |                                                              | 续订开关状态更新用户主动关闭用户主动开启风控发起关闭支付方式变更用户变更支付方式商品变更续费日期 - 商品修改日期小于15天使用修改前的版本进行续购续费日期 - 商品修改日期大于15天使用修改后的版本进行续购 | 上游调用接口上游调用接口接收商品变更MQ消息，根据日期间隔，更新数据 |      |
| Email & push | 已购买邮件``pushPlainText`` 自动续费成功邮件PlainText``pushPlainText`` 支付提醒邮件PlainText``pushPlainText`` 取消邮件PlainText``pushPlainText`` 续费失败邮件PlainText``pushPlainText`` 变更邮件PlainText``pushPlainText`` price变更PlainText``pushPlainText`` | 已购买邮件（包括成功续费，有些文案不一致）内容用户名扣款金额扣款方式有效期deeplink（券包详情页）时机商品购买成功时取消确认邮件内容用户名deeplink（商城页）时机用户勾选取消时支付扣款提醒邮件内容用户名xxx时候会扣款deeplink（券包详情页）时机提前5天提醒一次提前3天提醒一次提前1天提醒一次续费失败邮件内容用户名deeplink（购买历史页）时机续费失败券包变更邮件内容用户名提醒发生变更 or price变更deeplink（券包详情页）时机商品变更时（价格，有效期），用户提前15天收到email | apollo配置维度：国家+业务+节点内容：imc模板，通用的占位符    |      |
|              |                                                              |                                                              |                                                              |      |



### **2.2 模块交互：流程图、时序图**



#### 2.2.1 初次购买支持自动续费套餐的流程

> 订单完成后，调用续费管理服务创建订阅记录
>
> 只有用户开启的时候，才调用honors-g创建数据。

![image-20260219002915090](/Users/didi/Library/Application Support/typora-user-images/image-20260219002915090.png)

#### 2.2.2 系统流程触发节点

> 1. 提前5d、3d、1d的续费提醒

> 2. 到达指定日期的续费动作

自动续费流程 & 续费提醒流程

![image-20260219002949110](/Users/didi/Library/Application Support/typora-user-images/image-20260219002949110.png)

套餐变更

![image-20260219003018618](/Users/didi/Library/Application Support/typora-user-images/image-20260219003018618.png)

其它节点

![image-20260219003033769](/Users/didi/Library/Application Support/typora-user-images/image-20260219003033769.png)

2.2.3 自动续费逻辑

![image-20260219003049927](/Users/didi/Library/Application Support/typora-user-images/image-20260219003049927.png)

#### 2.2.4 续费管理失败场景流程

> 失败了需要退款，只能是续费管理那里进行退款

> 发货失败的重试机制，不能自动取消

![image-20260219003152721](/Users/didi/Library/Application Support/typora-user-images/image-20260219003152721.png)

#### 2.2.5 MQ生产场景

![image-20260219003210229](/Users/didi/Library/Application Support/typora-user-images/image-20260219003210229.png)

#### 2.2.6 邮件、push

> 5个用户维度的邮件和push通知

> 1个套餐维度的邮件和push通知，运营可决定是否发放邮件 or push，以及模板内容

![image-20260219003333891](/Users/didi/Library/Application Support/typora-user-images/image-20260219003333891.png)

#### 2.2.7 商城权益购买

> 1. 判断是否需要扣库存，频次

> 2. sku修改日期&用户续费日期， 会影响扣减金额和发放的券。

![image-20260219003352310](/Users/didi/Library/Application Support/typora-user-images/image-20260219003352310.png)

### **2.3 是否引入新技术组件，包括公司内部公共组件及开源组件**

无



### **2.4 接口文档方案**



#### 2.4.1 续费相关事件MQ

> topic:  dive_honors_subscription_renew_notice

续费模块的MQ信息

PlainText

```
{
  "user_info":{
    "uid":6241294461928,
    "role":1,
    "country":"BR",
    "city":55000002
  },
  "scene":"buy",    // "renew_success","remind","cancel","update","pay_failed"
  "subscription_info":{   // 订阅的信息
    "platform":"passenger.pass",
    "product":362141264781624,
    "renew_manage":{
        "amount":300, // 下一次扣款滴分
        "payment_method":{
          "channel_id": 150, // 渠道id 150 信用卡
          "card_index": "9873",
          "card_org": "visa", // 卡组织
          "card_suffix": "0893" // 卡的后四位
        }
    },
    "platform_extra":{   // 订阅时的业务信息, 订阅管理模块不使用这个数据，只是暂存
        "campaign_id":23412
    }
  },
  "renew_info":{        // 续购成功时的续购信息
    "third_part_id":72136812548121412,
    "status":30,
    "pay_info":{
        "req":{}  // 请求支付扣款的参数
    }
  }
}
```



#### 2.4.2 H5购买支持不扣减库存和频次

> 新增sku_list.#.decrease_type字段, 识别是否需要扣减频次和库存

> 新增sku_list.#.version字段, 识别使用是什么版本的信息做处理

| path     | /gulfstream/shopping-api/item/order/purchase                 |
| :------- | ------------------------------------------------------------ |
| disf     | disf!ibt-marvel-shopping_api                                 |
| method   | post                                                         |
| request  | reqPlainText`{    "user_info":{        "uid":369436225749551,   // 用户ID        "role":1,		 // 用户角色        "country":"BR",	         // 国家码        "city":55000041          // 城市ID    },    "sku_list":[        {            "sku_id":360287970190967823,  // sku_id            "sale_amount":300,             // 有些价格不在sku存储，比如外卖券包，存在sale_amount，不读deduct_price_type            "version":"lastest" ,          // 使用的sku版本，未找到& 默认走最新的            "ignore_list": ["quota","frequency","sale_time","status"]  // 忽略的动作        }    ],    "platform":"passenger.pass",          // 平台名    "uuid":"zx_5_1",                      // 本次购买的UUID    "expire_ts":600,                      // 失效时间（需要单独调用pay接口时才有用），为0则不失效    "local_time":"2024-01-18 08:12:34",   // 当地当前时间    "pay_by_shopping":false,              // false（仅创建待支付订单，需要另外调支付接口完成订单），true（走完创建订单，支付，发货等流程)    "extra_info":{        "buy_from": "food_order", // 购买来源        "pass_type": 4,          // 套餐类型        "routine_id": 156141, // 只有外卖侧需要        "activity_id": 25396, // 主套餐的dive主活动id        "strategy_id": 23570, // 只有外卖侧需要        "free_routine_id": 156167, // 只有外卖侧需要        "free_pass_activity_id": 25414 // 赠套餐的dive主活动id    } }` |
| response | respPlainText`{    "errno": 0,    "errmsg": "success",    "data": "7153640000356089856"  // 订单ID }` |



#### 2.4.3 新增订阅

| path     | /gulfstream/dive-honors/v3/api/subscription/create           |
| :------- | ------------------------------------------------------------ |
| disf     | disf!ibt-marvel-dive_honors_g                                |
| method   | post                                                         |
| request  | reqPlainText`{  "user_info":{    "uid":369436225749551,    "role":1,    "country":"BR",    "city":55000002  },  "subscription_info":{    "signal":"213124124",    // 创建唯一标识 本次    "platform":"passenger.pass",    "product_id":3613691264912,    "switch_status":0,  // 0 未开启自动续费，1 开启自动续费    "renewal_manage":{      "amount":300,  // 续购价格，滴分      "open_source":1  // 续订开启来源  1 单独购买（或者随单购），2 管理页面单独开启      "payment_method":{          "channel_id": 150, // 渠道id 150 信用卡          "card_index": "9873",          "card_org": "visa", // 卡组织          "card_suffix": "0893" // 卡的后四位        }    },    "first_buy_info": {        "mall_order_id":7491826349126423,    // 商城订单ID        "pay_datetime": "2025-08-14 01:12:23",        "amount": 300, // 下一次扣款滴分        "buy_from":1,  // 1 h5购买，2随单购        "validity_days":12, // 套餐购买后有效期（天数）        "expire_local_datetime":"2025-09-15 01:23:12",  // 套餐购买后的失效时间        "payment_method": {          "channel_id": 150, // 渠道id 150 信用卡          "card_index": "9873",          "card_org": "visa", // 卡组织          "card_suffix": "0893" // 卡的后四位        } // 支付方式    },    "platform_extra":{        "shopping_extra":{          "buy_from": "ride_order", // 购买来源          "pass_type": 1,          // 套餐类型          "activity_id": 25396, // 主套餐的dive主活动id      }    }  } }` |
| response | respPlainText`{  "errno":0,  "errmsg":"success",  "data":{    "plan_id":1    // 自动续费的订阅ID  } }` |



#### 2.4.4 订阅数据更新

> 取消、打开、修改支付方式

| path     | /gulfstream/dive-honors/v3/api/subscription/update/manage    |
| :------- | ------------------------------------------------------------ |
| disf     | disf!ibt-marvel-dive_honors_g                                |
| method   | post                                                         |
| request  | reqPlainText`{  "user_info":{    "uid":369436225749551,    "role":1,    "country":"BR",    "city":55000002  },  "plan_id":1,  "update_list":[    {      "type":"payment_method",  // 支付方式修改      "value":{          "channel_id": 150, // 渠道id 150 信用卡          "card_index": "9873",          "card_org": "visa", // 卡组织          "card_suffix": "0893" // 卡的后四位        } // 支付方式    },    {      "type":"switch_status",   // 订阅开关修改       "value":1                // 1为开启，0 为默认状态，2表示用户取消，4表示风控取消    }  ] }` |
| response | respPlainText`{  "errno":0,  "errmsg":"success",  "data":null }` |



#### 2.4.5 订阅激活

| path     | /gulfstream/dive-honors/v3/api/subscription/reactivate       |
| :------- | ------------------------------------------------------------ |
| disf     | disf!ibt-marvel-dive_honors_g                                |
| method   | post                                                         |
| request  | reqPlainText`{  "user_info":{    "uid":369436225749551,    "role":1,    "country":"BR",    "city":55000002  },  "plan_id":1,  "record":{    "mall_order_id":7857865764572123,   // 商城订单ID    "out_trade_id":"21312312131",   // 支付的ID，非必传    "amount":220,                   // 支付金额    "payment_method":{      "channel_id": 150, // 渠道id 150 信用卡      "card_index": "9873",      "card_org": "visa", // 卡组织      "card_suffix": "0893" // 卡的后四位    }, // 支付方式,     "pay_local_datetime":"2025-08-15 03:12:23"  // 支付当地时间  } }` |
| response | respPlainText`{  "errno":0,  "errmsg":"success",  "data":null }` |



#### 2.4.6 订阅查询

> 查询订阅记录

| path     | /gulfstream/dive-honors/v3/api/subscription/query/bySignals  |
| :------- | ------------------------------------------------------------ |
| disf     | disf!ibt-marvel-dive_honors_g                                |
| method   | post                                                         |
| request  | reqPlainText`{  "user_info":{    "uid":369436225749551,    "role":1,    "country":"BR",    "city":55000002  },  "signals":["12341241231"] }` |
| response | respPlainText`{    "errno": 0,    "errmsg": "success",    "data": {        "12341241231": { // 订阅的信息            "plan_id": 1,            "uid": 369436225749551,            "role": 1,            "signal": "36956784574567346",            "country": "BR",            "city": 55000002,            "switch_status": 1,            "platform": "passenger.pass",            "product": "362141264781624",            "renewal_manage": {                "amount": 300, // 下一次扣款滴分                "payment_method": {                  "channel_id": 150, // 渠道id 150 信用卡                  "card_index": "9873",                  "card_org": "visa", // 卡组织                  "card_suffix": "0893" // 卡的后四位                } // 支付方式            },            "extra_info": {                "platform_extra": {}, // 业务的一些信息,自动续费后业务需要创建记录                "first_buy_info": {                    "mall_order_id":7491826349126423,    // 商城订单ID                    "pay_datetime": "2025-08-14 01:12:23",                    "amount": 300, // 下一次扣款滴分                    "payment_method": {                      "channel_id": 150, // 渠道id 150 信用卡                      "card_index": "9873",                      "card_org": "visa", // 卡组织                      "card_suffix": "0893" // 卡的后四位                    } // 支付方式                }            },            "renew_status": 1,            "next_renew_time": "2025-09-08 10:02:12"        }    } }` |



### **2.5 存储设计方案**

**2.5.1 续费计划表（renewal_plan_xxx）**

DDL

PlainText

```
CREATE TABLE `renewal_plan_0`
(
    `id`                bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键id',
    `uid`               bigint(20)          NOT NULL DEFAULT '0' COMMENT '用户id',
    `role`              tinyint(4)          NOT NULL DEFAULT '0' COMMENT '角色id 1乘客 2司机',
    `country`           varchar(10)         NOT NULL DEFAULT '' COMMENT '国家码',
    `city_id`           bigint(20)          NOT NULL DEFAULT '0' COMMENT '城市ID',
    `source_signal`     varchar(128)        NOT NULL DEFAULT '' COMMENT '订阅来源',
    `platform`          varchar(64)         NOT NULL DEFAULT '' COMMENT '业务来源',
    `product`           bigint(20)          NOT NULL DEFAULT '0' COMMENT '续订商品',
    `switch_status`     tinyint(4)          NOT NULL DEFAULT '0' COMMENT '订阅开关状态，0未开启,1开启',
    `renewal_manage`    json                NOT NULL COMMENT '续订信息',
    `ext`               json                NOT NULL COMMENT '额外信息',
    `renewal_status`    int(10)             NOT NULL DEFAULT '0' COMMENT '续订执行状态 0待开始，1执行中，2执行失败',
    `next_renewal_time` datetime            NOT NULL DEFAULT '1970-01-01 00:00:00' COMMENT '下一次续订时间',
    `created_time`      timestamp           NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `modify_time`       timestamp           NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    `is_del`            tinyint(1)          NOT NULL DEFAULT '0' COMMENT '是否软删除 0-正常未被软删除 1-已被软删除',
    PRIMARY KEY (`id`),
    KEY `idx_uid_signal` (`uid`, `signal`),
    KEY `idx_product_switch_status` (`product`, `switch_status`),
    KEY `idx_next_renew_time_switch_status` (`next_renewal_time`, `switch_status`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4
  COMMENT ='用户订阅计划表';
```

| 字段 | id           | uid             | role   | source_signal                                         | platform       | product           | country | city     | switch_status                               | renewal_manage                                               | ext                                                          | renew_status                         | next_renew_time     | create_time         | modify_time         |
| :--- | :----------- | :-------------- | :----- | :---------------------------------------------------- | :------------- | :---------------- | :------ | :------- | :------------------------------------------ | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------- | :------------------ | :------------------ | :------------------ |
| 值   | 1            | 369436225749551 | 1      | 36956784574567346                                     | passenger.pass | 36956784574567346 | BR      | 55000002 | 1                                           | 续费管理数据PlainText`{  "amount":300,  // 下一次扣款滴分  "payment_method":{    "channel_id": 150, // 渠道id 150 信用卡    "card_index": "9873",    "card_org": "visa", // 卡组织    "card_suffix": "0893" // 卡的后四位  } // 支付方式 }` | 一些额外信息PlainText`{  "platfrom_extra": {}, // 业务的一些信息,自动续费后业务需要创建记录  "first_buy_info": {    "pay_datetime": "2025-08-14 01:12:23",    "amount": 300, // 下一次扣款滴分    "payment_method": {      "channel_id": 150, // 渠道id 150 信用卡      "card_index": "9873",      "card_org": "visa", // 卡组织      "card_suffix": "0893" // 卡的后四位    } // 支付方式  } }` | 1                                    | 2025-09-08 10:02:12 | 2025-06-08 10:02:12 | 2025-08-08 10:02:12 |
| 注释 | 自增ID计划id | 用户ID          | 角色ID | 用于避免重复创建&查询sku维度的续订管理，这个就是skuID |                | 产品ID            |         |          | 自动续费打开状态0 关闭，1开启, 3 开启了关闭 | 下一次支付逻辑                                               |                                                              | 最近一次的续费状态1 待执行2 执行失败 |                     |                     |                     |

**2.5.1 续费记录表（renewal_record_xxx）**

DDL

PlainText

```

```



| 字段 | id           | plan_id        | uid             | role   | pay_info                                         | ext                                                      | out_trade_id | mall_id          | source                                       | status                                                       | create_time | modify_time |
| :--- | :----------- | :------------- | :-------------- | :----- | :----------------------------------------------- | :------------------------------------------------------- | :----------- | :--------------- | :------------------------------------------- | :----------------------------------------------------------- | :---------- | :---------- |
| 值   | 1            | 3              | 369436225749551 | 1      | 本次购买信息PlainText`{  "req":{},  "resp":{} }` | PlainText`{   "purchase_info":{}  // 续购时使用的参数 }` |              | 7491826349126423 | 1                                            | 30                                                           |             |             |
| 注释 | 自增ID计划id | 续费计划表的ID | 用户ID          | 角色ID | 支付相关的信息                                   | 额外信息                                                 | 支付的订单ID | 商城订单ID       | 记录来源自动续费生成业务生成（重新激活场景） | 续费状态0 初始化10 支付成功20 支付失败30 续订成功40 续订失败 |             |             |



### **2.6 配置、灰度设计方案**



#### 2.6.1 邮件配置apollo

| key            | val                            |
| :------------- | :----------------------------- |
| passenger.pass | 国家+节点的通知配置PlainText`` |



### **2.7 \*设计方案完成后check点（或方案设计前考虑）**

 a. 端来源及版本控制

-  端来源：明确产品需求涉及的端（99、global）

-  版本控制：明确产品需求涉及的版本

-  产品线车型控制：明确产品需求涉及的具体产品线或车型，订单维度功能使用产品线，司机维度功能使用车型，严禁司机维度配置使用car_level转换成product_id实现。

 b. 有新增rpc调用

- 必须接dirpc、disf、inrouter

- 必须明确SLA（新接口可先设置一个可接受的值，后续放量过程中持续关注优化）

- 耗时评估：核心链路下游耗时不超过50ms，非核心链路标准100ms，最多不超过200ms，异常向上反馈审核

- 超时重试机制配置

 c. 新生产、消费mq

- 开启检查是否有积压，是否需要重置消费

- 默认生产限流是否需要调整

 d. 评审机制

- 开发量小于一周，或已在小组周会上完成技术方案评审

 e. 相关依赖时间点确认

- 文案中心

- 素材、物料

- 实验策略

- 各方联调环境确认

 f. 是否存在主从延迟的场景可能导致业务异常



## 3. 稳定性方案



### **3.1 系统稳定性**

1. 各类功能是否需要支持动态开关、灰度

1. 降级方式（统一降级服务）

1. 限流（统一限流服务）

1. 邮件和push通过MQ处理，和OS2沟通发送的QPS

1. 异常处理流程

1. 单个用户的逻辑通过MQ处理，重试机制

1. 扫表逻辑：每小时执行一次，捞取符合条件的数据

1. 是否需要开发特定脚本支持异常处理方案

1. 监控的考察点（流量监控等）和监控工具



### **3.2 业务稳定性（业务观察指标监控&报警）**

1. 容错性，能否承受外部不合预期的逻辑异常

1. 资损风险

1. 扣款金额是否符合预期，（sku价格变更后的扣款金额是否符合预期）

1. 数据安全

1. 自动续费处理结果的通知机制



### **3.3 性能评估**

1. 集群大小(服务器、db)

1. 吞吐能力(qps/延迟)



### **3.4 新老逻辑兼容性评估**

1. 用户正常购买非续订套餐



### **3.5 外部、上下游依赖评估**

| **核心功能评估** | **是否涉及** | **是否兼容** | 用途                 |
| :--------------- | :----------- | :----------- | :------------------- |
| 支付             | 是           | 新下游       | 扣款                 |
| passport         | 是           |              | 查询用户信息，发邮件 |



### **3.6 系统对账【必须】**

1. 订阅记录和biz套餐记录的对账

1. 扣款对账



## 4. 开发排期



### **4.1 排期**

包括前端开发、后端开发、风控、外部合作方、联调、测试、上线时间，给的是端到端最终的交付时间。

| **模块**      | **业务方** | **模块owner** | **开发** | **联调** | **准入case** | **测试** | **上线** | **放量** | **全量** |
| :------------ | :--------- | :------------ | :------- | :------- | :----------- | :------- | :------- | :------- | :------- |
| dive-honors-g | dive       | 张雄          |          |          |              |          |          |          |          |
| shopping-api  | dive       | 张雄          |          |          |              |          |          |          |          |
|               |            |               |          |          |              |          |          |          |          |
|               |            |               |          |          |              |          |          |          |          |
|               |            |               |          |          |              |          |          |          |          |
|               |            |               |          |          |              |          |          |          |          |
|               |            |               |          |          |              |          |          |          |          |



### **4.2 数据埋点排期**







### **4.3 最终交付整体排期**

例：

| **事件** | **时间**    |
| :------- | :---------- |
|          | 1.01 - 1.20 |
|          |             |





## 5. 测试相关



### **5.1 是否压测**



## 6. 上线相关



### **6.1 准出后填写上线计划** 

上线计划-监控模板 https://cooper.didichuxing.com/knowledge/2200018798158/2203962082907







## 7. 实验相关

1. 新feature上线先小流量观察，再交由运营开启实验。

1. 各方向遵守对应实验sop，保证实验策略、实验过程、实验结果的有效性。



## 8. 注意事项



### **8.1 开发过程**



### **8.2 上线过程**