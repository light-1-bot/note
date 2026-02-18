# 【C端技术方案】套餐自动续费Rider Pass Auto renew

## 1. 需求背景

- *需求wiki：

- C端PRD：https://cooper.didichuxing.com/knowledge/2204291401492/2204196789269

- B端PRD：https://cooper.didichuxing.com/knowledge/2204291401492/2204666983514

- 支付PRD：https://cooper.didichuxing.com/knowledge/2200838214095/2204624205369

- 支付风控PRD：https://cooper.didichuxing.com/knowledge/2202214386146/2204776268053

- *望岳地址：

- C端望岳：https://ddp.intra.xiaojukeji.com/requirement/story/R-IBG-503009?backUrl=/workbench/requirement

- B端望岳：https://ddp.intra.xiaojukeji.com/requirement/story/R-IBG-540006?backUrl=/requirement/list?filterDirId=600

- 支付望岳：https://ddp.intra.xiaojukeji.com/requirement/story/R-IBGFIN-504682?backUrl=/requirement/list?filterDirId=110630



- 相关依赖：包括前置依赖，后置数据分析、数据埋点等，如有均需写明如何支持

- 端：

- iOS：@陈星辰

- 安卓：@阮严冬

- BFF：@张雨轩

- 在线API：@师远

- 价格API：@李浩

- 资源位：不涉及

- 治理：@林科颢

- 支付：@何琳琛 @吴圣中@代东锋

- 收银台 2.0 前端接口：https://cooper.didichuxing.com/knowledge/2199431322830/2199707829158

- 收银接口：http://wiki.intra.xiaojukeji.com/pages/viewpage.action?pageId=916213853

- 支付回调接口：https://wiki.intra.xiaojukeji.com/pages/viewpage.action?pageId=989888954

- 查询用户所有的支付方式接口报文：https://cooper.didichuxing.com/docs2/document/2205083748700

- 支付风控：@鲁志鹏

- 套餐商城H5：@许钰晗@杨淑璟

- 麦哲伦：@刘昱君

- DIVE B端：@卢秋呈

- 物料中台：@张雄

- 技术方案：https://cooper.didichuxing.com/knowledge/2204915991613/2205028565457

- 反作弊：不涉及



- 18n：文案翻译 https://i18n.intra.didiglobal.com/product/view/31012

- UI：https://mastergo.com/goto/Ly8bIDPz?page_id=3:5816&file=157881039380210

- 埋点文档：https://cooper.didichuxing.com/docs2/sheet/2205022254454



**范围： 巴西，99乘客端**



## 2. 技术方案设计



### **2.1 需求分析拆解**



### 2.1.1 随单购

12345

| **Page description**                                         | 需要的修改                                                   | 接口变更                                                     | 开发进度 |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :------- |
| ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=17542902882092d2750abd10cff5c1e8ae132b2bb66eb%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20&x-s3-process=image%2Fformat%2Cwebp)![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=175429043675210c7c24599ee83383ba2acf136f738df%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20&x-s3-process=image%2Fformat%2Cwebp) | 预估套餐中增加套餐是否支持自动续费；套餐支持自动续费，则展示开关套餐不支持自动续费，则不展示基于当前用户选中的支付方式判断是否能让用户开启或者关闭自动续费开关用户选中的支付方式非借记卡或者信用卡，则关闭开关且点击后出现弹窗报错用户选中的支付方式为借记卡或者信用卡，用户可自由选择是否开启自动续费开关 支付方式不正确报错：**Auto Renew only works with credit/debit payment method** 自动续费按钮旁边的文本未选中**To enable auto-renew (R$0.99/month), a credit/debit card is required.**选中**Auto-bill monthly with R$0.99/m. Cancel any time in your Account. T&Cs** T&C应该有特殊标识，端应该知道能点击 todo： 确认前置弹出的逻辑已确认弹窗逻辑：点击小感叹号弹窗选中套餐单选框的时候弹窗 @李浩修改预估自动续费开关状态的逻辑：如果套餐本身不支持自动续费，即is_auto_renew为false，则不展示这部分属性，即无需额外处理；套餐本身支持自动续费，即is_auto_renew为true，则自动续费开关会展示，此时需要处理如果最终支付方式是support_pay_type_list中的，则无需修改自动续费信息；如果最终支付方式不在支持列表中，则需要关闭开关，即修改auto_renew_switch_default_status为false，且同时修改上报数据log_data中的is_auto_renew字段为2 | **[atom-api]** **recommend**返回值sku_info增加字段：is_auto_renew： bool 套餐是否支持自动续费 | done     |
| **[atom-app] getEstimateInfo->**返回值增加JSON`// 增加每个预估套餐 自动续费信息和支持的支付方式， // 端增加用户选中的支付方方式和支持的支付方式进行匹配后判断是否能够让用户点击自动续费开关 {  "data": {    "new_pass_info": {      "{estimate_id}": {        "pass_info": {          "recommend_pass": {              // 预计价格侧进行处理支付方式匹配              "support_pay_type_list": [256],                "is_support_auto_renew": true, // 套餐是否支持自动续费              "is_open_auto_renew": true, // 自动续费默认状态 是否开启              "is_operate_auto_renew": true, // 是否可以操作自动续费开关              "detail": {                "auto_renew_module": {                  "auto_renew_title": "Auto-renew",                  // 开关能点击时左侧的文案展示                  "auto_renew_desc": "Auto-bill monthly with R$0.99/m. Cancel any time in your Account.", // 如果支付方式不满足则展示另一个                  "auto_renew_link_desc": "check T&c",                  "tc_link": "https://h5.didiglobal.com/silver-bullet-online/igOyir1XA5FxVj09qSkH7",                  // 开关的默认状态                  "auto_renew_switch_default_status": true,                   // 不能点击时自动续费开关左侧的文案展示                  "disable_auto_renew_desc": "To enable auto-renew ({{total}}/month), a credit/debit card is required. ",                  // 选中支付方式不是卡支付的时候报错                                    "disable_auto_renew_pop_up_desc": "Auto Renew only works with credit/debit payment method"               },                        }          }        }      }    }  } }` | done                                                         |                                                              |          |
| 发单 MQ                                                      | 新增订单信息套餐用户是否选中自动续费                         | 发单MQ    cg_pass_new_order:wanliu_order_new 新增字段cg_pass_new_order:wanliu_order_newJSON`{  "data": {    "tying_sale_choice_info": {      "pass_choice_info": { // 来自端上        "select_auto_renew": 1 // 是否开启自动续费：1标识开启；2标识关闭；      }    }  } }` | done     |
| 行程中搭售套餐                                               | atom-app行中推荐新增筛掉支持自动续费的套餐即可               | // 参数不变                                                  | done     |
| 完单MQ                                                       | 支付完成时确认自动续费的支付方式 目前只支持巴西的套餐自动续费行中切换支付方式只能线下转线上，所以并不会导致前面选择的支付方式有效，切换后的支付方式无效的情况 完单mq里面有相关参数，需要新增接受解析参数类型即可 | 完单MQ cg_pass_trip_order_pay:wanliu_order_paidJSON`{    "data": {        "request_data": {            "allChannels": {                "150": {                    "channel": 173,                    "cost": 450,                    "wxtype": "2",                    "cardIndex": "772106",                    "cardSuffix": "6822",                    "cardPrefix": "546997",                    "cardPrefix8": "54699700",                    "formatCardGroup": "mc",                    "usedPayChannel": "167",                    "usedMerchantId": "99InAppPaymentCapture"                }            }        }    } }` | done     |





### 2.1.2 套餐商城

123456789101112

| **Page description**                                         | **说明**                                                     | **接口变更**                                                 | 开发进度 |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :------- |
| **侧边栏**![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=175199403551631382762a765990fe3cb89aabaa958cb%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20&x-s3-process=image%2Fformat%2Cwebp) | 头像下面是否展示reactiavte按钮判断条件检查用户所有购买的套餐中开启自动续费的套餐订单是否逾期：存在逾期套餐，按钮变成reactivate没有就正常展示 view | **【atom-app】****getsideinfo**增加逻辑检查最新的开启自动续费的套餐订单们是否逾期：如果有套餐逾期，新增跳转到my clube99页面链接，跳过去了要展示冒泡正常view跳转clube99商城页面；出参变更PlainText`{    "data": {        "passProfile": {            "jumpLink": "https://page.didiglobal.com/global/silver-bullet-online/ridepass-details?entry=sidebar", // 重新激活进入我的套餐页面            "cardText": "reactivate", // 重新激活按钮字样            "logData": {              "button_type": "reactivate"// 枚举值：check、reactivate            }        }    } }` | done     |
| **商城中心页**![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=175523998962124c8d8569041429fd901a92d2905787f%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20&x-s3-process=image%2Fformat%2Cwebp)![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=17550539812835b3624a088cdc2b39bd506809645293e%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20&x-s3-process=image%2Fformat%2Cwebp) | 商城中心页 clube99page 新增套餐是否能够自动续费；可续订的套餐，点击 buy 按钮，进入套餐详情页不可续订的套餐点击buy则调用h5/buy接口直接购买 clube99 page 套餐介绍页文案统一使用 已经和舒宁拉起**Pack with auto renew option for R$ 10,00 every month, cancel any time** | **[atom-app]** **H5/mallHomepage****返回值新增**PlainText`{    "data": {        "ridepass_hot_deal_module": {            "type": "ridepass_hot_deal_module",            "data": {                "hot_deal_passes": [                    {                        "is_auto_renew": 1, // 套餐是否支持自动续费，0 为不支持 1支持默认打开 2支持默认关闭                        "desc": "Pack with {auto renew} option for R$ 10,00 every month, cancel any time",                    }                ]            }        },        "ridepass_content_module": {            "data": {                "tab_list": [                    {                        "passes": [                            {                                "campaign_id": 1,                                // 套餐是否支持自动续费，0 为不支持 1支持默认打开 2支持默认关闭                                "is_auto_renew": 1,                                 "desc": "Pack with auto renew option for R$ 10,00 every month, cancel any time"                            }                        ]                    },                    {                        "tab_code": "Ride",                        "passes": [                            {                               "campaign_id": 1,                                // 套餐是否支持自动续费，0 为不支持 1支持默认打开 2支持默认关闭                                "is_auto_renew": 1,                                 "desc": "Pack with auto renew option for R$ 10,00ry month, cancel any time"                            }                        ]                    }                ]            }        }    } }` |          |
|                                                              |                                                              |                                                              |          |
| **购买前-套餐详情页**![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1756715940247b1a6e12e8d3294d2bbe9ef897c3ec451%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20&x-s3-process=image%2Fformat%2Cwebp) | 自动续费按钮旁边的文本自动续费按钮禁用置为灰色**To enable auto-renew (R$0.99/month), a credit/debit card is required.**自动续费按钮可点击**Auto-bill monthly with R$0.99/m. Cancel any time in your Account. T&Cs** 因支付方式未通过校验而导致的自动续费按钮展示但禁用点击开关跳出提示文案：**Auto Renew only works with credit/debit payment method** | 【atom-app】h5/passDetail PlainText`{    "data": {        "detail_page_content_module": {            "type": "detail_page_content_module",            "data": {                "pass_detail": {                 }            }        "auto_renew_module": {            "type": "auto_renew_module",            "data": {                // 套餐是否支持自动续费，0 为不支持 1支持默认打开 2支持默认关闭                "is_auto_renew": 1,                 "auto_renew_title": "Auto-renew",                // 开关的默认状态                "auto_renew_switch_default_status": true,                 // true标识用户能点击开关，false标识不能                "enable_auto_renew_switch": true,                 // 自动续费开关左侧展示文案                "auto_renew_desc": "Auto-bill monthly with R$0.99/m. Cancel any time in your Account.",                          // 不能点击时点击的弹窗内容 不返回则不弹窗                "disable_auto_renew_pop_up_desc": "Auto Renew only works with credit/debit payment method" // 不能点击的时候点击报错弹窗信息                // 拼接在开关左侧描述最后的check T&C                "auto_renew_link_desc": "View Terms and Conditions"                "log_data": {                    "is_auto_renew": 1,                    "is_operate_auto_renew": true                }            }        }    } }` | done     |
| **点击购买**                                                 |                                                              | **【atom-app】****h5/buy:**请求中带上是否选中自动续费PlainText`{    "select_auto_renew": true // 选中 }` 购买的返回参数没有修改  **【atom-api】****payCallBack**请求新增字段：支付方式详情返回PlainText`{    "pay_detail": {      "channel_id": 150, // 渠道id 150 信用卡      "card_index": "9873",      "card_org": "visa", // 卡组织      "card_suffix": "0893" // 卡的后四位    } }` |          |
| **我的套餐页-onging**![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=175373535510488d91f8988ef36dc2619f6dc5201fb6b%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20&x-s3-process=image%2Fformat%2Cwebp) | 标签箭头 **auto renewal/Renovação Automática**点击进入管理自动续费页面如果套餐被renew的情况下，该套餐会被放在ongoing如果能够renew的套餐已经过期了 自动续费管理入口逻辑：ongoing中的套餐支持自动续费则展示入口expired的自动续费套餐仅最新的该种套餐订单展示入口用户在续费开启生效后手动关闭自动续费则不再展示入口套餐若自动续费失败，入口置为红色，且更换文案 | 【atom-app】mypasses接口新增入参PlainText`{    "entry": "sidebar" // 或者是email，    "need_pop": 1 // 1需要弹窗 0不需要弹窗 }`新增出参PlainText`{    "data": {        "my_passes_page_content_module": {            "data": {                "tab_detail": {                    "passes": [                        {                               // 套餐是否支持自动续费，0 为不支持 1支持默认打开 2支持默认关闭                            "is_auto_renew": 1,                             // 开启自动续费重新激活 为true时 按钮文案应该为Reactivate                            "is_open_auto_renew_reactivate": true,                             "auto_renew_info": {                                "is_show_entry": true, //是否展示入口,                                "is_auto_renew_valid": true, // 自动续费生效中，false标识自动续费失败，需要展示红色文案                            }                        }                    ]                }            }        },         "my_passes_page_pop_up_module": {            "type": "my_passes_page_pop_up_module",            "data": {                "status": 1, //1标识弹出 0标识不弹出                "title": "Clube99 Renewal Expired",                "desc": "We didn't received your renewal payment. Reactivate now to continue keep your benefits.",                "button_words": "Reactivate",                "action_after_click": "reactivate", // 点击按钮后的动作定义枚举 reacitvate则调用重新激活接口 close则关闭弹窗                "pass": {                    "price": "{R$}2.99",                    "order_id": "1234567890",                    "pass_sku_id": "360287970191749711",                    "pass_os2_activity_id": 55530,                    "sku_detail": [{                        "pass_sku_id": "360287970191749711",                        "pass_os2_activity_id": 55530,                        "source": "ride",                        "attribute": "normal"                    }]                },                "log_data": {                    "is_auto_renew": 1,                    "activity_id": 12345                }            }        }    } }` | done     |
| **我的套餐页-expired**![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=17549945496891121a9aee747fe7e7e7d0fe3895a037e%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20&x-s3-process=image%2Fformat%2Cwebp) | 过期套餐中只能激活每种套餐第一个（过期时要选中自动续费） 自动续费管理入口逻辑：ongoing中的套餐支持自动续费则展示入口expired的自动续费套餐仅最新的该种套餐订单展示入口用户在续费开启生效后手动关闭自动续费则不再展示入口套餐若自动续费失败，入口置为红色，且更换文案 |                                                              |          |
| ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1755000668464d5142fe0b84093cd966e97baaafdbf15%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20&x-s3-process=image%2Fformat%2Cwebp)![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=175500077999011c6c25c71b219e69a6468aadd137994%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20&x-s3-process=image%2Fformat%2Cwebp) | 从邮件或者侧边栏reactivate进入时：如果只有一个套餐逾期：展示该套餐的冒泡如果多个套餐逾期：冒泡改为提示多个套餐逾冒泡页点击reactivate 调用重新激活接口checkdetail弹窗关闭后，用户能看到所有的过期自动续费的套餐（不折叠起来） 进来跳转是ongoing页面对自动续费的套餐进行激活七天没有重新激活的话则视为过期 |                                                              |          |
| ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1751486694507ca70c31afd30271675da6ab23c75356e%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20&x-s3-process=image%2Fformat%2Cwebp)![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1756462558152a77f02668c1658a623e3e5e8bc946e8a%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20)![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=175627772911317519506841fe6a1818ca483c8c8c9a2%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20&x-s3-process=image%2Fformat%2Cwebp) | 能够点击T&C点击取消时弹出取消弹窗点击更换支付方式弹出收银台 待确认新接口 文案：**Stay in Clube99 Without Interruptions** **With automatic renewal, you will keep enjoying discounts and perks every month—without needing to sign up again every month. Check T&Cs** | **新增接口****【atom-app】**![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAHAAAABwCAYAAADG4PRLAAAAAXNSR0IArs4c6QAAAARzQklUCAgICHwIZIgAAAdkSURBVHic7Z1NiBzHGYbfr6q6e6Z3dna1Wls/9loicWwjcA5CPggcQ4iJIQQfbCQI+CILkmBITj6IYLNjMLnEt5CQi84BCV98sB1IgiEHHyJ0DDgH4yAF/60jabXq3Z7p7i+H3bVX3p2pnpnq7iqmntvOVn/9wUP11NtdM0MYxRWWuLYWYzEK0RcBQGLkeI8huEBaDJCnfZxZTnCe8mEj6cBXz7HEaXSRJu3KevSUJ4o3cR3ruLpf5H6BPY4BdIHEzzariAsA6+hRsvfV+wX2uAMk3Trb8oxLvI4ebez+9c0s63Hs5blA0t25SgLYFXiOJQAvzx26O852BJ7273lukYhtZ4DAFZZ+tekgadLGFZYC19Zi/WiPlVxbiwVkFDbdh2dCFqNQIBJB0314JqQvAuFvj7kMCS/PcbxAx/ECHccLdBwv0HG8QMfxAh3HC3QcL9BxvEDH8QIdxwt0HFX5GdY/lZWfw2a6x4bu6TSBn4GO4wU6jhfoONW/B7pI+MBRpVrPMslTROIEUCgGPiHk/0ae/jXb/OI/Tbe4ixe4l3D5cRV2XgWJ54C929YFCDgLCEAFq6rT+ZAH997K08//2VivX3fmAQCouZVXVNR9b1feSAhnKZx7W3YeWUUoG11le4EAZOeRVYjgEsa8IhGpizJY+UOTEmdeoOqsXCRSFyc9noT4iQwfes1kT+Mw2wKjpZNAcGnaMkTqooyOPGWgo7GZaYEy6P4KhMhELQrmXjVRZ1xmV6BaiAnieWP1CGdVtHTSWL2SzKxAGcU/MDX7viaY+5HReiWYWYFE4WOmazKU8Zo6ZlYgQzxsuiYRr5iuqWNmBYK4isc8WQU1RzKzAgn5TdM1GTBeU4f7AuPD31Wdk2/L+e/8XcUrL5U9jLPiX6ZbIc6N19ThtsBQSiUW/gQSTxHwKGTw27KBOk/WPgQ40Y8sT7Z1928m65XBaYFKHf8ZCI/vfY3CeLXUwZSmzPxnY81w8Rdktz81Vq8k7goM5rqQwQF3P+j7Yu7hc2VK5OlXvwewPnUvzDmKjbemrjMBzgqU4eFfA1g66H+CgktQC/ovbxjc/V+R938zdTOU/S5L1j6aus4EuCkwWjpJQr089P9ED6j24itlShXJzXfAgzcmbYW5uJzdvfHHSY+fFicFqqC7Ct2zO5a/QHSoVFjPNm5cLrj/S4xzOSWk4MEb+cYnE8s3gXMCZfv40yChv+dIiFS4UPpRUbFx891s68tnmIvLI1enhJS5uJqlt36Ybdy4XLZ+VRB6945XegaTG3tDKVV44v1vrzxHwf17L469d4WjSMbLZ0mJUwx5FICiIv+MkX2Up8k/kN0pHz8q3tjr1Kamg2KDDgrjVaT46VgnojTNN//7AYAPxjquAdy5hA6NDTrKxwoXcUbgqNigo3SscBA3BOpig44xYoVrOCGwVGzQMUascAnrBZaODTrGjBWuYLfAUEqSrXI3p0shni/1tELMH1Kto08odfQxRItz5s5vHqsFThIbdGifVnAUSdl5iJkClhQpap1A0ez2+VHYK3Di2KBjdKxQrYVj259l2YaZJNrzR8z3YQZrBU4TG3QMjRVqscOQ8/t6KdpL4MjsFkRD2Clw2tigY0isUGgdfFtRgFRr4Vhl/UyBlQKNxAYd344VsnuYJQ2dZQw5D7XYqbSnCbBOoLHYoGNvrCikVEGsfZ8bOkMbxC6BxmODjp1Y0Z4/wkzalSZLiiC7h+vorCxWCawiNuigYK4ni3bpxZIK4iM2xQp7BFYWGzQQnqSw9WzZ4bbFCmsEVhkbtBBdAKhVdrhNscIOgVXHBh2EQ1Bh+WeGFsUKKwTWEhs0kBAvgIIHy463JVY0LrC22KAnJCUujHOADbGiWYG1xwYNJJ4hEZ4qO9yGWNGowCZigxYpfz7O8KZjRXMCm4oNOgjfI9UqfUlvOlY0JrDR2KDDoVjRjMCmY4MOh2JFIwJtiA06XIkVtQu0KDbocCJW1CvQttigw4FYUatAK2ODDstjRX0CbY0NOiyPFbUJtDo26CC6AKD0z7XLor0EGdby8+71zUChflzbuUxDOEQqfLT0eAGCiGpZkdYmUHDR+BeET8EWZ4OPxzqCs62KermP2rLYYOvW67K9cItYPFnXOQEABAKEAjPpBx8E3y6K7KpA+lXJ8UVG/VvINo1+idAw6gvT2Z0kv3vnzdrOZ5ii6QaG0PjzQM90eIGO4wU6TvXvgRV/zcas42eg43iBjuMFOo4X6DheoON4gY7jBTqOF+g4XqDjeIGO4wU6jhfoOAJgW59VerRwIRAWg6bb8ExIWgwEbqf9pvvwTEie9gXOLNey+cZTAWeWE4HzlCOKN5vuxTMmUbyJ85Rvr0KvYx2I/WLGGeJi29lujLhKOUz8DJunLtZ3nO3JgT1KgNhLtJ54fdvVNvt3K/c4BtAFEh/yrSIuANwnDzhIIACcY4nT6CJN2nW05tEQxZu4/s1lcy+jPy9whSWurcWQUYhIBAD5WVkLXCAsBrid9nFmOcH5/eJ2+T89crbxyJTmQAAAAABJRU5ErkJggg==)【H5】-pass-api-h5-renewalDetail - 套餐自动续费详情页 返回值PlainText`{    "errno": 0,    "errmsg": "success",    "data": {        "renewal_page_header_module": {            "type": "renewal_page_header_module",            "data": {                "title": "Stay in {{Clube99}} without interruptions. ", //or "Your auto renew for {{Clube99}} has expired "                "desc": "With automatic renewal, you will keep enjoying discounts and perks every month-without needing to sing up again every month."                // or "Your auto renewal stopped because you payment method is unavailable. To continue enjoying discounts  rates, please update your payment informations and reativate your auto renew. ",                "tc_link": "https://xiaojukeji.com"            }        },        "renewal_page_content_module": {            "type": "renewal_page_content_module",            "data": {                // 开关标题                "auto_renew_title": "Auto-renew",                // 开关默认状态 关只展示三个内容，开则要展示四个                "auto_renew_switch_default_status": true,                // true标识用户能操作开关，false标识不能                "enable_auto_renew_switch": true,                // 自动续费开关左侧展示文案                "auto_renew_desc": "ablabal",                "auto_renew_payment_detail":{                    "title": "auto renewl details",                    "next_bill_date": "May 21, 2025",                    "paid_date": "May 2, 2025",                    "schedule": "Every Month",                    "bill_amount": "R$99",                    "payment_method": "visa 0834 is invalid",                    // 卡组织icon url,如有则展示在payment_method中                    "payment_method_icon_url": "http://xxx",                    // 标识支付方式是否可用，可用为灰色，不可用为红色                    "is_payment_method_valid": true,                     // true能够更换支付方式，false标识不能，前端警用                    "is_payment_method_support_change": false,                     "schedule": "Every Month",                    "order_id": "xxxxxxxxx",                },                "pass_sku_id": "360287970191749711",                "pass_os2_activity_id": 55530,                "sku_detail": [{                    "pass_sku_id": "360287970191749711",                    "pass_os2_activity_id": 55530,                    "source": "ride",                    "attribute": "normal"                }]                // 控制底部reactivate开关的展示                "is_show_reactivate": false,                "log_data": {                  "is_auto_renew": 0, // 1 或者 2                  "pass_renew_type": "active" // 或者 delinquent                }            }        },        "cancel_renewal_pop_up_module": {            "type": "cancel_renewal_pop_up_module",            "data": {                "title": "Are you sure to give 8 coupons up to xxx?",                 "desc": "If you cancel Clube99s auto renew, you won't receive any more coupons, and we won't to change your saved payment method again. ",                "confirm_button_content": "Confirm cancellation",                "no_confirm_button_content: "Maybe later":, // 点击后关闭冒泡            }        },    } }` | done     |
| ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1754980256650272753ff7e884726e20e73c801785a87%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20&x-s3-process=image%2Fformat%2Cwebp) | 收银台只展示借记卡和信用卡切换支付卡必须选中一张卡只能选中一张卡能够新增卡 | // 待确认收银确认参数绑卡来源 pm申请页面来源 支付侧给 todo：待瑜洁老板确定 // todo： 切换支付方式的接口 atom-app更换支付方式![img](data:image/png;base64,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)【H5】-pass-api-h5-updateAutoRenewal - 更改套餐自动续费状态接口  更换支付方式回调![img](data:image/png;base64,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)【H5】-pass-api-h5-updateAutoRenewPaymentCallBack |          |
| ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1747770571943149e7811b77ee2ef81991eb18d04f152%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20&x-s3-process=image%2Fformat%2Cwebp) | 点击确认取消时请求新接口确认文案 **Are you sure to give up 8 coupons up to R$xxx?** **If you cancel Clube99’s auto-renewal, you won’t receive any more coupons, and we won’t charge your saved payment methods again.** **Confirm Cancellation** **Maybe later** | **新增接口****【atom-app】****pass/api/h5/updateAutoRenewal**![img](data:image/png;base64,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)【H5】-pass-api-h5-updateAutoRenewal - 更改套餐自动续费状态接口 |          |
| ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=17537362761678fb8417476fe38c597fd2b68accaaf64%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20&x-s3-process=image%2Fformat%2Cwebp) | reactivate 按钮导向收银台auto renew expired 标签箭头导向auto renew 管理页面去更换支付方式 | **新增接口** **【atom-app】**![img](data:image/png;base64,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)【H5】-pass-api-h5-reacitvatePassRenewal - 重新激活自动续费 |          |





### 2.1.3 通知 deeplink

|       | **Email proposal**                                           | **Push notification proposal**                               | 要返回的跳转地址           |
| :---- | :----------------------------------------------------------- | :----------------------------------------------------------- | :------------------------- |
| **1** | 1Renewal reminder Hello, [user_name]!We hope you've been enjoying your savings this month and we'd like to remind you that on ww/ww we'll be renewing your Club99! Bottom: Find out more about its benefits: (direct to “package details” in the app) | 1It is a pleasure to have you with us and we would like to remind you that on xx/xx we will be renewing your clube99. | myclube99                  |
| **2** | 1-Welcome to Clube99! Hello, [user_name]! Welcome, your Clube99 payment has been successfully completed.We're excited for you to be able to save even more on your trips. Total paid: R$[total]Payment method: [payment_method] Valid until: [coupous_validity] The package can be used immediately after purchase and can be used within [coupons_validity] days.Bottom: Find out more about its benefits: (direct to “package details” in the app) | **1**Welcome to Clube99! Your Club99 payment has been successfully completed. | myclube99                  |
| **3** | **1**Hello, [user_name]! Your Club99 payment has been successfully completed.We're excited for you to be able to save even more on your trips. Total paid: R$[total]Payment method: [payment_method] Valid until: [coupous_validity] Bottom: Find out more about its benefits: (direct to “package details” in the app) | **1**Welcome to Clube99! Your Club99 payment has been successfully completed. | myclube99                  |
| **4** | **1**See you soon Cancellation confirmationHello, [user_name]! We have successfully canceled your Clube99.It's a pleasure to have you as a customer and we hope you'll enjoy more benefits soon.reactivate club99 - Direct the user to the clube99 page in the main menu | **1**Cancellation confirmation.We have successfully canceled your Clube99. | clube 99 中心页            |
| **5** | **1**We're waiting for you Hello, [user_name]We've noticed that you haven't paid for your Clube99 renewal. Change your payment method and enjoy exclusive Clube99 discounts. Reactivate club99 - Direct the user to the myclube99 pag3 | **1**We're waiting for you Try to reactivate your Clube99    | my clube 99 重新激活冒泡页 |
| **6** | **1****Plan updates**Hello, [user_name] We would like to inform you that your plan has undergone some adjustments. Currently, the number of coupons available has changed from {{amount}} to {{amount}} per month. Bottom: Find out more about your benefits → (link to "package details")  2**Plan updates**Hello, [user_name]We would like to inform you that your plan has undergone some adjustments. Currently, the **discount per coupon** has changed from **{{discount}} to {{discount}}**.**Bottom:** Find out more about your benefits → *(link to "package details")*3**Plan updates**Hello, [user_name]We would like to inform you that your plan has undergone some adjustments. Currently, the **coupon validity period** has changed from {{period}} **to {{period}}**.**Bottom:** Find out more about your benefits → *(link to "package details")*4**Plan updates**Hello, [user_name]We would like to inform you that your plan has undergone some adjustments:Coupon quantity: **from {{amount}} to {{amount}}**Validity: **from {{period}} to {{period}} days**Discount: **from {{discount}} to {{discount}} per coupon** | 1Plan update!Your plan was updated: **coupon quantity** changed from {{amount}} **to {{amount}}**.2Plan update! Your **coupon validity** changed from **{{period}} to {{period}} days**.3Plan update! The **discount per coupon** changed from **{{discount}} to {{discount}}**.4Plan update!Your plan was updated: **new quantity, validity, and coupon value**. | myclube99                  |





### 2.2 模块交互：流程图、时序图

- 涉及多个模块、多个团队的项目，都需要有流程图

- 交互逻辑调整较大、逻辑较复杂的，推荐有时序图

- 新旧模块流程差异及兼容（技术迁移、改造适用）



### 2.2.1 随单购流程

### ![image-20260219002336033](/Users/didi/Library/Application Support/typora-user-images/image-20260219002336033.png)

![image-20260219002409797](/Users/didi/Library/Application Support/typora-user-images/image-20260219002409797.png)



### 2.2.2 侧边栏信息展示![image-20260219002425719](/Users/didi/Library/Application Support/typora-user-images/image-20260219002425719.png)

### 2.2.3 H5购买套餐流程 todo：下单购买不用检查支付方式

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1755234817768bef3625b98aa5b343db2d5da3547682d%2Fh5%3Abuy%E5%BC%80%E5%90%AF%E8%87%AA%E5%8A%A8%E7%BB%AD%E8%B4%B9%E6%B5%81%E7%A8%8B.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0MTU4NDgiLCJpYXQiOjE3NzE0MzA5MDEsImV4cCI6MTc3MjAzNTcwMX0._4k_PgngUyO8VSCjq3Pr84LNNBX1hi60-kdhhBG5N20&x-s3-process=image%2Fformat%2Cwebp)



### 2.2.4 我的套餐页面流程 （ongoing tag展示逻辑待确认）

### ![image-20260219002449579](/Users/didi/Library/Application Support/typora-user-images/image-20260219002449579.png)

### 2.2.5 自动续费管理详情页 todo：// 直接用honors的价格

![image-20260219002636299](/Users/didi/Library/Application Support/typora-user-images/image-20260219002636299.png)

### 2.2.6 取消或开启自动续费

![image-20260219002658992](/Users/didi/Library/Application Support/typora-user-images/image-20260219002658992.png)

### 2.2.7 更换支付方式

![image-20260219002714685](/Users/didi/Library/Application Support/typora-user-images/image-20260219002714685.png)

### 2.2.8 重新激活

![image-20260219002733770](/Users/didi/Library/Application Support/typora-user-images/image-20260219002733770.png)

### 2.3 是否引入新技术组件，包括公司内部公共组件及开源组件

| 名称 | 分类 | 说明 |
| :--- | :--- | :--- |
|      |      |      |
|      |      |      |
|      |      |      |



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

- 



### 2.5 存储设计方案

1. 给出整个系统数据库表的ER图或表之间的关系图

1. 库表设计：数据库表设计准则（包含数据字典、分库分表、字段设计要求）

1. 严格遵守sql规范：https://base.xiaojukeji.com/docs/rds/7505

- 是否涉及到主从延迟问题（端到端幂等设计）

  

- 是否涉及到金额、利率、时间字段，请按照货币/时间军规设计 货币军规：http://sailing.intra.didiglobal.com/#/knowledgeDetail?72057594042023450

  

- 是否涉及数仓看板相关改动

  



### biz数据结构改动

biz数据结构

PlainText

```
{
    "extra_info": {
        "buy_method": "h5_buy",
        "amount": 300,  // 本次新增
        "payment_method": {
            "channelID": 150, // 渠道id 150 信用卡
            "cardIndex": "9873",
            "format_card_group": "visa", // 卡组织
            "card_suffix": "0893" // 卡的后四位
        },  // 本次新增
        "pay_datetime": "2025-08-13 00:01:23" // 本次新增
    },
    "campaign_id": 6789,
    "sku_info_list": [
        {
            "sku_id": 360287970193372770,
            "banner_quota_id": "360287970193433265",
            "add_on_purchase_quota_id": "360287970193376925",
        }
    ],
    "os2_activity_id": 181554,
    "sub_campaign_id": 4689,
}
```





### 2.6 配置、灰度设计方案

1. 是否有新增apollo、代码配置

1. 新功能是否需要新增灰度，多模块如何共享同一个开关等



新增短链配置

【prod】

我的套餐 重新激活冒泡  https://page.didiglobal.com/global/silver-bullet-online/mypass-v2?entry=push&need_pop=1 https://oia.99app.com/99/6yoyPoU 

我的套餐 无重新激活 https://page.didiglobal.com/global/silver-bullet-online/mypass-v2?entry=push https://oia.99app.com/99/vbyN6d5 

套餐中心页 [https:///page.didiglobal.com/global/silver-bullet-online/ridepass-v2?entry=push](https://page.didiglobal.com/global/silver-bullet-online/ridepass-v2?entry=push) https://oia.99app.com/99/5cvCxjV



【pre】

我的套餐 重新激活冒泡  [https:///prepage.didiglobal.com/global/silver-bullet-online/mypass-v2?entry=push&need_pop=1](https://prepage.didiglobal.com/global/silver-bullet-online/mypass-v2?entry=push&need_pop=1) https://oia.99app.com/99/P49dxVV 

我的套餐 无重新激活 [https:///prepage.didiglobal.com/global/silver-bullet-online/mypass-v2?entry=push](https://prepage.didiglobal.com/global/silver-bullet-online/mypass-v2?entry=push) https://oia.99app.com/99/LbKBNNo 

套餐中心页 [https:///prepage.didiglobal.com/global/silver-bullet-online/ridepass-v2?entry=push](https://prepage.didiglobal.com/global/silver-bullet-online/ridepass-v2?entry=push) https://oia.99app.com/99/i853Qhi



【sim】

我的套餐（待重新续订冒泡） url: https://dsim244-page.intra.xiaojukeji.com/global/silver-bullet-online/mypass-v2?entry=push&need_pop=1 deeplink: https://oia.99app.com/99/8XsVEbE 

我的套餐（正常无冒泡） url: https://dsim244-page.intra.xiaojukeji.com/global/silver-bullet-online/mypass-v2?entry=push deeplink: https://oia.99app.com/99/v4QJgnc 

套餐中心页面 url: https://dsim244-page.intra.xiaojukeji.com/global/silver-bullet-online/ridepass-v2?entry=push deeplink: https://oia.99app.com/99/XtVupaq





新增ab test apollo配置实验名称 RIdes_Pass_Auto_Renew

atom-app

新增配置

auto_renew_expired_days 标识过期多少天后不再展示重新激活样式

jumplink中新增跳转到我的套餐页面 key： h5_my_passes





### 2.7 *设计方案完成后check点（或方案设计前考虑）

a. 端来源及版本控制

- 端来源：明确产品需求涉及的端（99、global）

  

- 版本控制：明确产品需求涉及的版本

  

- 产品线车型控制：明确产品需求涉及的具体产品线或车型，订单维度功能使用产品线，司机维度功能使用车型，严禁司机维度配置使用car_level转换成product_id实现。

  

b. 有新增rpc调用 没有新增rpc

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

| 模块 | 业务方      | 模块owner      | 开发          | 联调        | 准入case      | 测试 | 上线 | QA负责人 | 放量 | 全量 |
| :--- | :---------- | :------------- | :------------ | :---------- | :------------ | :--- | :--- | :------- | :--- | :--- |
|      | 增长C端     | 庹嘉明         | 08.18 ~ 09.05 | 09.08~09.16 | 09.17 ~ 09.19 |      |      |          |      |      |
|      | IOS端       | 安然           | follow        |             |               |      |      |          |      |      |
|      | 安卓端      | 杨成栋         | follow        |             |               |      |      |          |      |      |
|      | BFF         | 张雨轩         | follow        |             |               |      |      |          |      |      |
|      | 在线API     | 师远           | follow        |             | @王婷英       |      |      |          |      |      |
|      | 价格API     | 朱鹏           | follow        |             | @杨玉赐       |      |      |          |      |      |
|      | 治理        | 林科灏         | 没有开发量    |             |               |      |      |          |      |      |
|      | 支付后端    | 何林琛         | follow        |             | @张改         |      |      |          |      |      |
|      | 支付端      | 代东锋、吴圣中 |               |             |               |      |      |          |      |      |
|      | 支付风控    | 鲁志鹏         | follow        |             | 杨文东        |      |      |          |      |      |
|      | H5          | 许钰晗、杨淑璟 | follow        |             |               |      |      |          |      |      |
|      | 算奖        | 杨磊           | follow        |             |               |      |      |          |      |      |
|      | 增长B端     | 卢秋呈         | 08.20 ~ 08.27 | 9.1-9.5     |               |      |      |          |      |      |
|      | Magellan FE | 刘昱君、杨晨辉 | 08.20 ~ 08.27 | 9.1-9.5     |               |      |      |          |      |      |
|      | 权益        | 张雄           | 08.18 ~ 09.05 | 9.1-9.5     |               |      |      |          |      |      |



### 4.2 据埋点排期







### 4.3 最终交付整体排期

跟版：1030

| 事件                                                         | 时间                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 开发时间                                                     | 8.18 - 9.5                                                   |
| 联调                                                         | 9.8 - 9.16                                                   |
| 准入                                                         | 9.17 - 9.19                                                  |
| sim 测试                                                     | 9.22-10.14（13d，跨十一假期，有休假风险，3天buffer）         |
| 走查                                                         | 第一波：9.24给走查 语言9.28返回、UI9.29返回 问题修复+验证+二次验证：9.29-10.10 |
| 第二波：9.28给走查，语言10.9返回、UI10.10返回 问题修复+验证+二次验证：10.10-10.13 |                                                              |
| 第三波：10.11给走查，语言10.14返回 UI10.15返回 问题修复+验证+二次验证：10.15-10.17 |                                                              |
| 上预发                                                       | 10.15                                                        |
| 预发回归                                                     | 10.16                                                        |
| 上线                                                         | 10.20-10.21                                                  |
| 线上回归                                                     | 10.22-10.23                                                  |
| 出集成包                                                     | 10.24                                                        |
| 集成包回归                                                   | 10.27-10.28                                                  |
| 交付                                                         | 10.29                                                        |











## 5. 测试相关



### 5.1 是否压测



## 6. 上线相关



### 6.1 准出后填写上线计划

上线计划-监控模板 https://cooper.didichuxing.com/knowledge/2200018798158/2203952515917







## 7. 实验相关

1. 新feature上线先小流量观察，再交由运营开启实验。

1. 各方向遵守对应实验sop，保证实验策略、实验过程、实验结果的有效性。



### AB Test

- **测试分组设置**：

- **控制组（Control Group）**：平台上配置了有效借记卡或信用卡的用户，其默认会看到自动续费开关处于开启状态。若这些用户不想进行自动续费，可随时将其关闭。

- **处理组（Treatment Group）**：同样是拥有有效借记卡或信用卡的用户，但默认会看到自动续费开关处于关闭状态，若他们选择开启自动续费功能，则可以将其启用。

- **测试目的**：通过该A/B测试，能够更好地了解用户行为，尤其是在出现非预期购买情况或用户对所提供的自动续费功能不够清晰明确时。借此方式，可以缓解潜在问题，避免联系率、取消率的增加以及对转化率可能产生的负面影响。



## 8. 注意事项



### 8.1 开发过程



### 8.2 上线过程





**待确认信息：**

1. 自动续费的方案是按月自动购买新同种套餐还是套餐能够长期存在但重新发券 - 建议是按月自动购买，套餐就不能设置超长购买后有效期，B端prd、工具等需要重新考虑

1. 按照月自动续费

1. my clube 99 ongoing tag 中 自动续费管理入口的出现与续订套餐多次购买、续订的逻辑

1. 只在ongoing tag中出现自动续费管理页面

1. 增加预算只剩10%的警报 —— 算奖侧支持