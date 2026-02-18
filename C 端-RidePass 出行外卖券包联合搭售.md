# C 端-RidePass 出行外卖券包联合搭售



## 1. 需求背景

- *需求wiki：

- 出行 C 端：https://cooper.didichuxing.com/knowledge/2204291401492/2204637749189

- 出行 B 端：https://cooper.didichuxing.com/knowledge/2204291401492/2204853332184

- 外卖 C 端：https://cooper.didichuxing.com/knowledge/2204533921780/2204803728447

- 外卖 B 端：https://cooper.didichuxing.com/knowledge/2204533921780/2204840397460

- 埋点：https://cooper.didichuxing.com/docs2/sheet/2201144721064?sheetId=UyAv9

- 出行/外卖反作弊：https://cooper.didichuxing.com/docs2/sheet/2204924629165?sheetId=MODOC

- *望岳地址：https://ddp.intra.xiaojukeji.com/requirement/story/R-IBG-537809?backUrl=/requirement/list?filterDirId=110339

- 相关依赖：

- 端：@邹优欢@林堃

- 接口文档：https://cooper.didichuxing.com/knowledge/share/page/llCf9IwB3YUb#Pages《[Android&iOS] SIMBA 套餐支持外卖+买赠》

- 在线 API：@师远

- 价格 API：@张兵

- 资源位：@徐梓桉

- 外卖增长：@刘允鹏

- 套餐商城：@杨淑璟

- OS2：@黄俞婷

- 反作弊：@王雅芳

- 支付风控：@潘必成

- 券系统：@任建清

- 物料中台：@张雄@郭恒

- 算奖服务：@杨磊

- B 端配置后台-后端：@涂显锋

- 出行侧方案

- 外卖侧方案

- 出行 B 端配置后台-前端：@刘昱君

- 外卖 B 端配置后台-前端：@周丽君

- 支付：@廖培科

- 接口文档：https://cooper.didichuxing.com/knowledge/2200124585383/2205113043602

- 翻译

- 套餐商城+弹窗+搭售条文案部分：https://i18n.intra.didiglobal.com/product/view/30500

- 侧边栏+搭售详情弹窗：https://i18n.intra.didiglobal.com/product/view/30522

- 文案检查: 在技术评审前需要补充文案

- 是否需要数据埋点需求以及排期确认

- 技术方案&测试case中需要补充历史活动兼容，尤其是B端



## 2. 技术方案设计



### **2.1 需求分析拆解**



#### 2.1.1 前置说明

1. 本次需求涉及 4 类套餐的售卖：出行独立套餐、出行套餐赠外卖套餐、外卖独立套餐、外卖套餐赠送出行套餐
2. 涉及国家：BR（部分内容涉及 global，会在下面标注），涉及语言 pt-BR
3. 推荐差异点说明

推荐差异点说明

|      |      |                                                              |                                                              |
| :--- | :--- | :----------------------------------------------------------- | :----------------------------------------------------------- |
|      |      | ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1752915231730e52f6a699417b3a26ece4869ceb106d0%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp) | ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1752831690054f32e0e66ab4b4a8c913ff89f6891b2d0%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp) |
|      |      |                                                              |                                                              |

商城套餐Hot Deal-买赠推荐逻辑举例

商城套餐Hot Deal-买赠推荐逻辑举例

#### 2.1.2 通用部分

| 功能说明     | 涉及范围 | 方案说明                                                     | 待确认点                                                     |
| ------------ | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 套餐推荐     | 通用域   | **涉及方：**atom-app -> atom-api -> os2**改动点：**/recomend 推荐逻辑（改动）对接os2更换入参（activity_types），支持活动类型多选返回值新增字段，区分出行套餐、外卖套餐，套餐是否可赠过滤赠送频次为0 or 购买频次=0的套餐风控桩点新增字段外卖获取出行推荐套餐（新增）单独起一个bff api，做入参隔离提供过滤【频次/不可售套餐】的能力 **新提供给外卖的推荐套餐接口 & 流程图：【pass】-pass-recommend-套餐推荐 **反作弊桩点新增字段-文档：**https://cooper.didichuxing.com/docs2/sheet/2204924629165?sheetId=MODOC 出行extra_infoJSON`{    "source_type":"ride", // 本次新增：区分出行/外卖    "campaign_id": 22901,    "pass_type": "standard",    "sku_info_list": [        {            "add_on_purchase_quota_id": "360287970197465499",            "add_on_purchase_wholetrip_quota_id": "360287970197728263",            "banner_quota_id": "360287970197480607",            "sku_id": "360287970197984306",            "touch_info": [],            "attribute_type": "normal / master / gift_able"  // 本次新增：区分普通/联合/可赠        }    ],    "sub_campaign_id": 21671,    "city_names":["sao paulo","andradia"] }`外卖extra_infoJSON`{    "source_type": "food",    "food_info": {        "campaign_info": {            "campaign_id": 10002, // 活动id            "sub_campaign_id": 2001, // 策略id            "score": 100, // 策略权重            "bubble_content": "", // 营销气泡            "attribute_type": "normal / master / gift_able",  // 本次新增：区分普通/联合/可赠            "city_names":["sao paulo","andradia"]        },        "sku_info_map":{            3602321315213: {                "sku_id":3602321315213,                "price_type": 1,                "online_payment_price": 1200 // 滴分            }, // key=sku_id, value=sku_info        },    } }` | ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1753855582088a1bd5f4f4de4f49852108186a1ef1bcd%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp)TODO：协议补充 |
| 随单套餐购买 | 出行域   | **涉及方：**atom-api -> shopping**改动点：**shopping一阶段购买接口：/shopping-api/item/order/purchase，单个sku_info接口切换为sku_list **shopping侧协议：**https://cooper.didichuxing.com/knowledge/2204275398448/2204900576543 | 在biz中区分：出行、外卖、出行买赠、外卖买赠@彭家鑫@赵述佳 进线风险点：![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1753794136548877ce55f2268746dceda377446caf5e2%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp) |



#### 2.1.3 出行弹窗&随单购买部分

| 页面说明                                                     | 涉及范围       | 方案说明                                                     | 待确认点                                                     |
| ------------------------------------------------------------ | -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=17537943168722fc03a7c90614774a95509363694f982%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp) | 出行域国家：BR | **涉及方：**端 -> bff -> 资源位 -> atom-app -> atom-api **资源位弹窗接口（改动）**在/resource/judge接口上改动，区分弹窗下发序列化的json数据**推荐逻辑（新增）**有买赠 & 有可赠：则展示买赠，赠送外卖权重最高的套餐无买增 / 无可增：出行(根据最高价值) > 外卖（根据权重）  **接口文档 & 流程图：**https://cooper.didichuxing.com/knowledge/2204915991613/2204862885664**端侧的协议：**https://cooper.didichuxing.com/knowledge/share/page/llCf9IwB3YUb#Pages *其他：富文本内容需要增长做处理 |                                                              |
| ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=175317547608689d889e01b3c3d75cb7e0bdebc72cbfb%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp) ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=17531754904014cfbdb36ab1f5e73a76380e3cb3c1617%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp) |                | **涉及方：**端、bff、在线 api、价格 api、OS2、biz、shopping **推荐逻辑（改动）**排序优先级一致时，优先推买赠套餐过滤当单不可用券的套餐**搭售绿条（改动）**副标题内容更新，区分纯出行和买赠套餐**半浮层弹窗（改动）**样式变更，新增tab选项（区分外卖、出行） **接口文档 & 流程图：**https://cooper.didichuxing.com/knowledge/2204915991613/2204279784265 | BR、global 一起修改@张舒宁@王喜 pm是否接受、qa回归面半浮层弹窗协议确认 |
| 发单                                                         |                | 支付风控新增字段                                             | 新增的字段是否需要增长额外提供@潘必成 走风控的服务确认@彭家鑫 |
| ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=17531755384444858677bb88fdacfae526bf57fd06248%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp) |                | **涉及方：**端、bff、在线 api、价格 api、OS2、biz、shopping **推荐逻辑（改动）**排序优先级一致时，优先推买赠套餐过滤当单不可用券的套餐**搭售绿条（改动）**副标题内容更新，区分纯出行和买赠套餐**半浮层弹窗（改动）**样式变更，新增tab选项（区分外卖、出行） **接口文档 & 流程图：**https://cooper.didichuxing.com/knowledge/2204275398448/2204426378434 | 半浮层弹窗的tab区分处理协议确认（BFF、端）                   |
| 一阶段/二阶段发套餐（数据需求）                              |                | 要说明，发券补充的字段 （待定）活动类型：出行独立套餐/出行买赠套餐/外卖独立套餐/外卖买赠套餐套餐类型：主套餐 or 附套餐套餐 ID套餐售价 | 跟shopping确认协议@张雄 具体需要的字段list待确认@张钟月      |
| ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=17531759954048c33db53a51adc93b722e38f94c134fd%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp) |                | **涉及方：**乘客完单：MQ、atom、biz、dos、shopping、engine-g弹窗：atom-api、OS2、dmc、端 **改动：**套餐的wanliu_order_paid事件中，主动调用os2的push能力，推送弹窗数据（新增） **具体流程**：https://cooper.didichuxing.com/knowledge/2204275398448/2204965314373**端侧的协议：**https://cooper.didichuxing.com/knowledge/share/page/llCf9IwB3YUb#Pages**触发dmc的接口协议：**https://cooper.didichuxing.com/knowledge/2204275398448/2204903187342 *其他：富文本内容需要增长做处理 |                                                              |



#### 2.1.4 侧边栏入口&商城部分

| 页面说明                                                     | 需求点细化                                                   | 方案说明                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 侧边栏![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1753772724379845fcf6af7f0ee7d026493a5f445778f%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp) | 套餐入口改为常驻入口，无论有没有推荐套餐都展示商城中有套餐售卖时，对应文案展示所有套餐中总价值最高的套餐价值，其中联合套餐价值为两个套餐之和商城中无套餐售卖时不展示副文案 | 通过api通用推荐接口获取套餐推荐结果（包含出行主套餐、出行附套餐、外卖主套餐、外卖附套餐、纯出行套餐、纯外卖套餐）根据prd的通用套餐排序逻辑，筛选出最优套餐（跟之前的筛选逻辑的区别时优先进行套餐之间的组合） **联合套餐组合逻辑：**case1：主套餐：出行A、出行B、外卖A附套餐：外卖1、出行1最终会组合出3个联合套餐（出行A+外卖1、出行B+外卖1、外卖A+出行1）case2：主套餐：出行A、外卖A附套餐：外卖1最终会组合出2个套餐，其中1个联合套餐（出行A+外卖1），1个单独的外卖A套餐 |
| 商城列表页![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1753760766486f839a2fc5e233faff13f197fd748efd3%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp) | 商城新增推荐区，优先推荐联合套餐，其次推荐出行套餐、外卖套餐各一常规套餐区按tab对套餐进行分类（所有套餐、出行套餐、外卖套餐） | 通过api通用推荐接口获取套餐推荐结果（包含出行主套餐、出行附套餐、外卖主套餐、外卖附套餐、纯出行套餐、纯外卖套餐）按套餐优先级构建推荐区数据如果有联合套餐则展示1个联合套餐如果既有纯出行也有纯外卖，则各选取一个进行展示，出行在上外卖在下如果只有一个业务线的套餐，则选取top2展示按套餐业务线构建常规套餐列表（排除推荐区的套餐） |
| 套餐详情页![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=17537610051029fd1cfdc7076abecde64a377b6edb396%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp)![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1753761036710d4b348480d6609172948e2f570976de1%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp) | 套餐详情页分为未购买、已购买未购买情况下，按联合套餐/出行/外卖套餐展示不同权益及数量，点击下方按钮拉起支付已购买情况下，展示套餐中权益的使用情况，点击下方按钮进行跳转 | 接口新增套餐状态字段，根据套餐状态走不同的查询逻辑（未购买、已购买）未购买状态下，调用os2接口查询当前指定活动的可见性，然后调用shopping接口查询sku详情已购买状态下，通过order_id调用shopping查询订单详情。联合套餐情况下，因为是1个单号对应2个套餐，所以在解析返回值时要根据sku_id识别出当前是哪个套餐。然后调用dive-honors查询当前套餐的权益使用情况 todo：未购买状态下调用os2接口查询指定活动时，如果是联合套餐，那2个活动id怎么传 |
| 我的套餐页![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=17537611137619b6e44b294150bc2c05912573acfdb80%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp)![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1753761166971fda48f328a393d3da3e14776413be532%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp) | 页面头部展示当前套餐已核销的金额按不同tab（生效中、已过期）对套餐进行分类展示生效中的套餐展示对应权益的使用情况，点击下方按钮跳转到不同业务线页面已过期的套餐展示从2024年起买过的所有套餐，支持滑动加载，一次加载10个 | 查询接口新增当前tab以及对应页码字段，接口每次返回一个tab的数据，每次切tab重新请求接口从dive-biz中查询用户的套餐购买记录，通过sku_order_id查询shopping的订单详情接口，然后解析接口返回（如果当前sku_order_id对应的是联合套餐，则接口会返回多个子单套餐信息）从dive-honors中获取每个出行/外卖套餐对应的核销信息![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1753792713965d7b06dfda452a194cc923e1c873a6ad9%2Fh5%E5%95%86%E5%9F%8E-%E6%88%91%E7%9A%84%E5%A5%97%E9%A4%90.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp) 联合套餐随单购情况下，shopping父单会有3个子单，展示的时候需要把随单购的两个子单聚合 |
| 购买套餐                                                     | 1、支持购买4种套餐，出行赠外卖/外卖赠出行/纯出行/纯外卖      | 购买套餐接口新增附活动id字段如果当前套餐为联合套餐，则api调用shopping生单接口时传2个skuId（主套餐skuId、附套餐skuId），并标识主/附关系如果当前套餐为联合套餐，调用dive-biz保存购买记录时会插入1条记录，其中target_id为shopping生单接口返回的sku_order_id，并在extra字段中标识主sku_id、附sku_id供后续查询使用![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1753688567164e0a1284c35c1e22ffde027ce175ebcbb%2F%E4%B9%B0%E8%B5%A0%E5%A5%97%E9%A4%90-h5%E8%B4%AD%E4%B9%B0.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp) todo：外卖的product_id取值逻辑 |
| 消费外卖套餐购买mq                                           | 为了确保我的套餐页面能展示用在所有渠道购买的套餐，所以在外卖随单购买套餐成功后，dive-biz需要存储对应套餐的购买信息shopping-api新增套餐购买消息，atom-api消费后把数据写入dive-biz消费成功后需要根据套餐有效期计算延迟时间，并发送延迟消息。atom-api消费延迟消息时判断当前套餐状态，同时更新套餐有效期 | ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1753865216682b51b9166650f0fb10a3a6a4baa4aff4c%2F%E8%81%94%E5%90%88%E5%A5%97%E9%A4%90-%E6%B6%88%E8%B4%B9%E5%A4%96%E5%8D%96%E5%A5%97%E9%A4%90%E8%B4%AD%E4%B9%B0mq.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjczMTg0MDUiLCJpYXQiOjE3NzE0MzQ3NzcsImV4cCI6MTc3MjAzOTU3N30.UhZH5k5BFfK__BN2EqGZBAElEoOt_vto0GpInWrL4U0&x-s3-process=image%2Fformat%2Cwebp) |



#### 2.1.5 风控反作弊部分

|                        | H5 独立购买 | 随单购（出行侧）                                             | 随单购（外卖侧）                                             | 首页弹窗                                                  |            |                               |                                            |                                            |              |                                          |
| ---------------------- | ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------------------------------------------- | ---------- | ----------------------------- | ------------------------------------------ | ------------------------------------------ | ------------ | ---------------------------------------- |
|                        | 纯出行套餐  | 纯外卖套餐                                                   | 出行->外卖                                                   | 外卖->出行                                                | 纯出行套餐 | 出行->外卖                    | 纯外卖套餐                                 | 外卖->出行                                 | 出行侧       | 外卖侧                                   |
| 出行反作弊（王雅芳）   | follow现状  | -不涉及                                                      | -不涉及                                                      | follow现状仅券核销时处理                                  | follow现状 | follow现状                    | -不涉及                                    | -不涉及                                    | 不需要过桩点 | -不涉及                                  |
| 出行支付风控（潘必成） | -不涉及     | -不涉及                                                      | 需要新增字段调起收银台 2.0 SDK 时传入right_add_on_type=food_voucherright_add_on_fee=0 | follow现状仅券核销时处理                                  | follow现状 | 需要新增字段foodvoucher_fee=0 | -不涉及                                    | -不涉及                                    | -不涉及      | -不涉及                                  |
| 外卖反作弊（竺芷羽）   | -不涉及     | 需要新增字段，h5 下单购买桩点传入rightsTyperightsEffectPeriodrightsStartTimerightsExpireTime | -不涉及                                                      | 推荐时，正常过展示桩点                                    | -不涉及    | 推荐时，正常过展示桩点        | 推荐时，正常过展示桩点外卖新增：展示的桩点 | 推荐时，正常过展示桩点外卖新增：展示的桩点 | -不涉及      | 推荐时，正常过展示桩点外卖新增：展示桩点 |
| 外卖支付风控（刘兴家） | -不涉及     | 需要新增字段调起收银台 2.0 SDK 时传入voucher_package_type    | -不涉及                                                      | 需要新增字段调起收银台 2.0 SDK 时传入voucher_package_type | -不涉及    | -不涉及                       | ？                                         | ？                                         | -不涉及      | -不涉及                                  |

当前反作弊接入的桩点

- ride pass 权益展示校验：https://newton.intra.didiglobal.com/v4/#/event/list/overview/basic?bizType=GLOBAL_GM&cmd=GLOBAL_RIDE_PASS_RIGHTS_DISPLAY&sync=true&menu=GRAVITATION 

- ride pass h5 页下单购买：https://newton.intra.didiglobal.com/v4/#/event/list/overview/basic?bizType=GLOBAL_GM&cmd=GLOBAL_RIDE_PASS_H5_CREATE_ORDER&sync=true&menu=GRAVITATION



出行/外卖反作弊：https://cooper.didichuxing.com/docs2/sheet/2204924629165?sheetId=MODOC

出行支付风控：https://cooper.didichuxing.com/docs2/sheet/2205097421228?sheetId=Hr1u4

外卖支付风控：https://cooper.didichuxing.com/docs2/sheet/2205054050301?sheetId=yZ7hF



### 2.2 模块交互：流程图、时序图



#### 行后外卖导流弹窗





### 2.3 是否引入新技术组件，包括公司内部公共组件及开源组件

| 名称        | 分类     | 说明       |
| :---------- | :------- | :--------- |
| fission-sdk | 公共组件 | 自动化开品 |
| elvish-sdk  | 公共组件 | 时间转化   |
| gift-s3     | 公共组件 | 图片存储   |



### 2.4 接口文档方案

1. 接口的命名规范、接口文档跟实现要保持一致

1. 接口是否支持幂等

1. 是否涉及到金额的字段？(请确认清楚金额的单位，是滴分还是当地货币？) 货币军规：http://sailing.intra.didiglobal.com/#/knowledgeDetail?72057594042023450

1）首页接口

- 接口名：

- 入参调整：

- 参数名

- 返回值调整：

- **返回值** 展开源码



2）H5商城-首页列表接口

![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAHAAAABwCAYAAADG4PRLAAAAAXNSR0IArs4c6QAAAARzQklUCAgICHwIZIgAAAdkSURBVHic7Z1NiBzHGYbfr6q6e6Z3dna1Wls/9loicWwjcA5CPggcQ4iJIQQfbCQI+CILkmBITj6IYLNjMLnEt5CQi84BCV98sB1IgiEHHyJ0DDgH4yAF/60jabXq3Z7p7i+H3bVX3p2pnpnq7iqmntvOVn/9wUP11NtdM0MYxRWWuLYWYzEK0RcBQGLkeI8huEBaDJCnfZxZTnCe8mEj6cBXz7HEaXSRJu3KevSUJ4o3cR3ruLpf5H6BPY4BdIHEzzariAsA6+hRsvfV+wX2uAMk3Trb8oxLvI4ebez+9c0s63Hs5blA0t25SgLYFXiOJQAvzx26O852BJ7273lukYhtZ4DAFZZ+tekgadLGFZYC19Zi/WiPlVxbiwVkFDbdh2dCFqNQIBJB0314JqQvAuFvj7kMCS/PcbxAx/ECHccLdBwv0HG8QMfxAh3HC3QcL9BxvEDH8QIdxwt0HFX5GdY/lZWfw2a6x4bu6TSBn4GO4wU6jhfoONW/B7pI+MBRpVrPMslTROIEUCgGPiHk/0ae/jXb/OI/Tbe4ixe4l3D5cRV2XgWJ54C929YFCDgLCEAFq6rT+ZAH997K08//2VivX3fmAQCouZVXVNR9b1feSAhnKZx7W3YeWUUoG11le4EAZOeRVYjgEsa8IhGpizJY+UOTEmdeoOqsXCRSFyc9noT4iQwfes1kT+Mw2wKjpZNAcGnaMkTqooyOPGWgo7GZaYEy6P4KhMhELQrmXjVRZ1xmV6BaiAnieWP1CGdVtHTSWL2SzKxAGcU/MDX7viaY+5HReiWYWYFE4WOmazKU8Zo6ZlYgQzxsuiYRr5iuqWNmBYK4isc8WQU1RzKzAgn5TdM1GTBeU4f7AuPD31Wdk2/L+e/8XcUrL5U9jLPiX6ZbIc6N19ThtsBQSiUW/gQSTxHwKGTw27KBOk/WPgQ40Y8sT7Z1928m65XBaYFKHf8ZCI/vfY3CeLXUwZSmzPxnY81w8Rdktz81Vq8k7goM5rqQwQF3P+j7Yu7hc2VK5OlXvwewPnUvzDmKjbemrjMBzgqU4eFfA1g66H+CgktQC/ovbxjc/V+R938zdTOU/S5L1j6aus4EuCkwWjpJQr089P9ED6j24itlShXJzXfAgzcmbYW5uJzdvfHHSY+fFicFqqC7Ct2zO5a/QHSoVFjPNm5cLrj/S4xzOSWk4MEb+cYnE8s3gXMCZfv40yChv+dIiFS4UPpRUbFx891s68tnmIvLI1enhJS5uJqlt36Ybdy4XLZ+VRB6945XegaTG3tDKVV44v1vrzxHwf17L469d4WjSMbLZ0mJUwx5FICiIv+MkX2Up8k/kN0pHz8q3tjr1Kamg2KDDgrjVaT46VgnojTNN//7AYAPxjquAdy5hA6NDTrKxwoXcUbgqNigo3SscBA3BOpig44xYoVrOCGwVGzQMUascAnrBZaODTrGjBWuYLfAUEqSrXI3p0shni/1tELMH1Kto08odfQxRItz5s5vHqsFThIbdGifVnAUSdl5iJkClhQpap1A0ez2+VHYK3Di2KBjdKxQrYVj259l2YaZJNrzR8z3YQZrBU4TG3QMjRVqscOQ8/t6KdpL4MjsFkRD2Clw2tigY0isUGgdfFtRgFRr4Vhl/UyBlQKNxAYd344VsnuYJQ2dZQw5D7XYqbSnCbBOoLHYoGNvrCikVEGsfZ8bOkMbxC6BxmODjp1Y0Z4/wkzalSZLiiC7h+vorCxWCawiNuigYK4ni3bpxZIK4iM2xQp7BFYWGzQQnqSw9WzZ4bbFCmsEVhkbtBBdAKhVdrhNscIOgVXHBh2EQ1Bh+WeGFsUKKwTWEhs0kBAvgIIHy463JVY0LrC22KAnJCUujHOADbGiWYG1xwYNJJ4hEZ4qO9yGWNGowCZigxYpfz7O8KZjRXMCm4oNOgjfI9UqfUlvOlY0JrDR2KDDoVjRjMCmY4MOh2JFIwJtiA06XIkVtQu0KDbocCJW1CvQttigw4FYUatAK2ODDstjRX0CbY0NOiyPFbUJtDo26CC6AKD0z7XLor0EGdby8+71zUChflzbuUxDOEQqfLT0eAGCiGpZkdYmUHDR+BeET8EWZ4OPxzqCs62KermP2rLYYOvW67K9cItYPFnXOQEABAKEAjPpBx8E3y6K7KpA+lXJ8UVG/VvINo1+idAw6gvT2Z0kv3vnzdrOZ5ii6QaG0PjzQM90eIGO4wU6TvXvgRV/zcas42eg43iBjuMFOo4X6DheoON4gY7jBTqOF+g4XqDjeIGO4wU6jhfoOAJgW59VerRwIRAWg6bb8ExIWgwEbqf9pvvwTEie9gXOLNey+cZTAWeWE4HzlCOKN5vuxTMmUbyJ85Rvr0KvYx2I/WLGGeJi29lujLhKOUz8DJunLtZ3nO3JgT1KgNhLtJ54fdvVNvt3K/c4BtAFEh/yrSIuANwnDzhIIACcY4nT6CJN2nW05tEQxZu4/s1lcy+jPy9whSWurcWQUYhIBAD5WVkLXCAsBrid9nFmOcH5/eJ2+T89crbxyJTmQAAAAABJRU5ErkJggg==)【H5】-pass-api-h5-mallHomepage  - 套餐商城列表

3）H5商城-我的套餐接口

![img](data:image/png;base64,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)【H5】-pass-api-h5-myPasses - 我的套餐

4）H5商城-套餐详情页接口

![img](data:image/png;base64,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)【H5】-pass-api-h5-passDetail - 套餐详情



### 2.5 存储设计方案

1. 给出整个系统数据库表的ER图或表之间的关系图

1. 库表设计：数据库表设计准则（包含数据字典、分库分表、字段设计要求）

1. 严格遵守sql规范：https://base.xiaojukeji.com/docs/rds/7505

- 是否涉及到主从延迟问题（端到端幂等设计）

  

- 是否涉及到金额、利率、时间字段，请按照货币/时间军规设计 货币军规：http://sailing.intra.didiglobal.com/#/knowledgeDetail?72057594042023450

  

- 是否涉及数仓看板相关改动

  



### 2.6 配置、灰度设计方案

1. 是否有新增apollo、代码配置

1. 新功能是否需要新增灰度，多模块如何共享同一个开关等



### 2.7 *设计方案完成后check点（或方案设计前考虑）

a. 端来源及版本控制

- 端来源：明确产品需求涉及的端（99、global）

  

- 版本控制：明确产品需求涉及的版本

  

- 产品线车型控制：明确产品需求涉及的具体产品线或车型，订单维度功能使用产品线，司机维度功能使用车型，严禁司机维度配置使用car_level转换成product_id实现。

  

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

g. 是否涉及时序问题



## 3. 稳定性方案



### 3.1 系统稳定性

1. 各类功能是否需要支持动态开关、灰度

1. 降级方式（统一降级服务）

1. 限流（统一限流服务）

1. 异常处理流程

1. 是否需要开发特定脚本支持异常处理方案

1. 监控的考察点（流量监控等）和监控工具



### 3.2 业务稳定性（业务观察指标监控&报警）

1. 容错性，能否承受外部不合预期的逻辑异常

1. 资损风险

1. 数据安全

1. x小时未发奖报警是否添加



### 3.3 性能评估

1. 集群大小(服务器、db)

1. 吞吐能力(qps/延迟)



### 3.4 新老逻辑兼容性评估

1. 老逻辑是否能兼容新流量
2. 新逻辑是否能兼容老流量



### 3.5 外部、上下游依赖评估

| 核心功能评估 | 是否涉及 | 是否兼容 |
| :----------- | :------- | :------- |
| 司机api      |          |          |
| 乘客api      |          |          |
| bff          |          |          |
| 收银         |          |          |
| 反作弊       |          |          |



### 3.6 系统对账【必须】

是否涉及各系统对账，营销主要依赖反作弊、收银、账单、订单等对账



## 4. 开发排期



### 4.1 排期

包括前端开发、后端开发、风控、外部合作方、联调、测试、上线时间，给的是端到端最终的交付时间。

| 模块           | 模块owner             | 开发                       | 联调                                                         | 准入case | QA负责人     | 单独技术评审时间 |
| :------------- | :-------------------- | :------------------------- | :----------------------------------------------------------- | :------- | :----------- | :--------------- |
| 端             | @邹优欢@林堃          | 5d                         |                                                              |          | @王喜        |                  |
| BFF            | @张雨轩               | 2d                         |                                                              |          |              |                  |
| 在线 API       | @师远                 | 无开发量（风控部分待评估） |                                                              |          | @王婷英      |                  |
| 价格 API       | @张兵                 | 不到 1 天工作量            |                                                              |          | @张世远      |                  |
| 资源位         | @熊强                 | 5d8.4-8.8                  |                                                              |          | @汪国强@冯昊 |                  |
| atom-app       | @陈晓霞@彭家鑫@赵述佳 | 家鑫 7.31-8.15             | 5d                                                           | 3d       | @王喜        |                  |
| atom-api       | 述佳 7.31-8.12        |                            |                                                              |          |              |                  |
| 物料中台       | @郭恒@王骞晧          | 骞晧：8.1 - 8.11郭恒：     |                                                              |          |              |                  |
| OS2            | @黄俞婷               | 2d                         |                                                              |          | @任倩        |                  |
| 算奖服务       | @杨磊@王味帅          | 3d8.13-8.15                |                                                              |          |              |                  |
| 券系统         | @任建清@徐志鸽        | 3d                         |                                                              |          | @汪国强      |                  |
| 商城 H5 前端   | @杨淑璟               | 8d8.4-8.13                 |                                                              |          |              |                  |
| 出行反作弊     | @王雅芳               | 无开发量                   |                                                              |          | @崔雅超      |                  |
| 外卖反作弊     | @竺芷羽               |                            |                                                              |          | @崔雅超      |                  |
| 出行风控       | @潘必成@鲁志鹏        |                            |                                                              |          | @杨文东      |                  |
| 外卖风控       | @刘兴家@鲁志鹏        |                            |                                                              |          | @杨文东      |                  |
| 外卖增长       | @王凯梅               |                            |                                                              |          | @孔凡玲      |                  |
|                |                       |                            |                                                              |          |              |                  |
| 出行B 端-前端  | @刘昱君               | 5d 8.4 - 8.8               |                                                              |          |              |                  |
| 出行 B 端-后端 | @刘润英               | 8d 8.1 - 8.12              | 8.13 - 8.15 创建、审核通过8.18 -                             |          |              |                  |
| 外卖 B 端-前端 | @周丽君@李泓男        | 12d 8.1 - 8.18             |                                                              |          |              |                  |
| 外卖 B 端-后端 | @涂显锋               | 14d 8.1 - 8.208.18         | 8.21 - 8.26 创建、审核通过8.27 - 8.29 其他部分 8.19-8.228.25-8.27 |          |              |                  |

https://cooper.didichuxing.com/knowledge/2204275398448/2204894785111





### 4.2 数据埋点排期







### 4.3 最终交付整体排期

| 事件       | 时间        |
| :--------- | :---------- |
| 开发       | 7.31 - 8.18 |
| 出行联调   | 8.18 - 8.27 |
| 端联调     | 8.19 - 8.27 |
| 外卖侧联调 | 8.25 - 9.1  |
| 准入       | 8.28 - 9.1  |
| 测试       |             |

sim测试：9.4-9.17 

上预发：9.18-9.19 

预发回归：9.22-9.23 

上线：9.24-9.25 

线上回归：9.26 

集成包回归：9.28-9.29





最终排期：

上预发：9.15-9.16 

预发回归：9.17-9.18 

上线：9.19-9.22 

线上回归：9.23 

出集成包：9.23

集成包回归：9.24-9.25

版本回归：9.26-9.28

灰度：10.9（灰度包）

提包：10.10（正式包）

开始放量：10.10-10.11（提包审核通过后）



风险：

1. 十一前提前封线

1. 提前几天做版本放量计划



预期：9.29  交付，运营可配置活动、运营可自测活动

10.9 端推送放量

一般经验：9.27-10.9 封线



## 5. 测试相关



### 5.1 是否压测



## 6. 上线相关



### 6.1 准出后填写上线计划

上线计划-https://cooper.didichuxing.com/knowledge/2200018798158/2205110631186







## 7. 实验相关

1. 新feature上线先小流量观察，再交由运营开启实验。

1. 各方向遵守对应实验sop，保证实验策略、实验过程、实验结果的有效性。



## 8. 注意事项



### 8.1 开发过程



### 8.2 上线过程