# SA 首页 where to 升级



## 1. 需求背景

- *需求wiki：https://cooper.didichuxing.com/knowledge/2204291401492/2204610186148

- *望岳地址：https://ddp.intra.xiaojukeji.com/requirement/story/R-IBG-535522?backUrl=/requirement/list?filterDirId=600

- 文案地址：

- 埋点地址：https://cooper.didichuxing.com/docs2/apply/2205131339014

- 相关依赖：

- 在线 API：@冉成

- 价格 API：@刘明炜



## 2. 技术方案设计



### **2.1 需求分析拆解**

涉及国家：SSL 国家

增长涉及点：where to 上方感知栏

涉及奖励类型：新流营（不含促活营）、第二单立减、套餐

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=175619075822139b82416f83b6a98de1c27e8e1c5b051%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0NTc0MTIiLCJpYXQiOjE3NzE0MzQ3MTgsImV4cCI6MTc3MjAzOTUxOH0.wVkokEkSNcjL2Pio5LEkaXjd_pzJ9nnHYalflUW_y0E&x-s3-process=image%2Fformat%2Cwebp)

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1756190797836a6c5e7ca84f7460a29e728dd607a28e0%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc0NTc0MTIiLCJpYXQiOjE3NzE0MzQ3MTgsImV4cCI6MTc3MjAzOTUxOH0.wVkokEkSNcjL2Pio5LEkaXjd_pzJ9nnHYalflUW_y0E&x-s3-process=image%2Fformat%2Cwebp)



| 需求点                             | 修改点                                                       | 接口、链路、确认项                                           |
| ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 不出增长营销感知栏场景             | 有特殊安全感知情况下，不出营销感知，有特殊安全感知条件当地时间晚上21点-次日早上7点用户当前定位在危险区有存量券情况下，不出营销感知，有存量券条件：当前账户内有>=5比索（立减）or  5% off（折扣）的优惠用户进入sa首页的时间符合券的星期、日期、时段条件存量券不是 surge 券 | 待确认项是否可以，有特殊安全时，不请求增长获取营销数据？可以是否可以，乘客身上有优惠券展示时，不请求增长获取营销数据？可以需要有版本控制吗？SA 首页流量会是多少？709qps出行首页流量会是多少？1710qps |
| 新流营（不含促活营）               | 当前需求没有促活营功能，后期做促活营时，需要加上此过滤只针对进行中的 dxgy 节点判断（未开始的、已完成的、反作弊的不关注）用户是否有下一节点券 | **待确认项**【新流营】首页有绑定逻辑，所以新活动上线时，会存在绑定&查询同时进行的情况，会有锁冲突，当前首页营销位一样的问题，所以会有一定概率where to 营销位不出新流营活动 @俞钦文 产品接受当前情况**接口定义**https://cooper.didichuxing.com/knowledge/2204915991613/2205213849608 |
| 第二单立减                         | 时机当用户参与该活动后（不区分主动&被动参与）第一单未完成时（即：Joined     TaskStatus = 1 // 已参与-未得券 状态） |                                                              |
| rider pass 套餐（不含 surge 套餐） | 时机过滤出最优套餐（和首页营销位逻辑一致）特别注意：最优套餐，不含 surge 套餐 |                                                              |
| 其他                               | 三类奖励类型，需要并发获取活动数据优先级：新流营 > 第二单立减 > rider pass 套餐埋点 |                                                              |



### **2.2 模块交互：流程图、时序图**



<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="741px" viewBox="-0.5 -0.5 741 483" style="height: auto;"><defs></defs><g><rect x="0" y="2" width="100" height="40" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"></rect><path d="M 50 42 L 50 482" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="all"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 98px; height: 1px; padding-top: 22px; margin-left: 1px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">端</div></div></div></foreignObject></switch></g><rect x="45" y="62" width="10" height="410" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"></rect><rect x="160" y="2" width="100" height="40" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"></rect><path d="M 210 42 L 210 482" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="all"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 98px; height: 1px; padding-top: 22px; margin-left: 161px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">BFF</div></div></div></foreignObject></switch></g><rect x="205" y="72" width="10" height="400" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"></rect><path d="M 205 471 L 152.5 471 L 51.74 471" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="stroke"></path><path d="M 59.62 466.5 L 50.62 471 L 59.62 475.5" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-end; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 468px; margin-left: 127px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">return</div></div></div></foreignObject></switch></g><rect x="320" y="2" width="100" height="40" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"></rect><path d="M 370 42 L 370 482" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="all"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 98px; height: 1px; padding-top: 22px; margin-left: 321px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 10px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;"><p style="font-size: 10px;">(SA 首页)在线 API/<br style="font-size: 10px;">(出行首页)在线 API</p></div></div></div></foreignObject></switch></g><rect x="365" y="82" width="10" height="380" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"></rect><path d="M 365 461 L 312.5 461 L 211.74 461" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="stroke"></path><path d="M 219.62 456.5 L 210.62 461 L 219.62 465.5" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-end; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 458px; margin-left: 287px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">return</div></div></div></foreignObject></switch></g><rect x="480" y="2" width="100" height="40" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"></rect><path d="M 530 42 L 530 482" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="all"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 98px; height: 1px; padding-top: 22px; margin-left: 481px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">atom-app</div></div></div></foreignObject></switch></g><rect x="525" y="92" width="10" height="360" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"></rect><path d="M 535 352 L 565 352 L 565 382 L 548.12 382" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"></path><path d="M 541.12 382 L 548.12 378.5 L 548.12 385.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe flex-start; width: 1px; height: 1px; padding-top: 365px; margin-left: 569px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: left;"><div style="display: inline-block; font-size: 11px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">优先级排序<br>新流营 &gt; 第二单立减 &gt; rider pass&nbsp;</div></div></div></foreignObject></switch></g><rect x="640" y="2" width="100" height="40" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"></rect><path d="M 690 42 L 690 322" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="all"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 98px; height: 1px; padding-top: 22px; margin-left: 641px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">atom-api</div></div></div></foreignObject></switch></g><rect x="685" y="167" width="10" height="25" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"></rect><rect x="685" y="222" width="10" height="25" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"></rect><path d="M 534 225 L 609 225 L 675.88 225" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"></path><path d="M 682.88 225 L 675.88 228.5 L 675.88 221.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-end; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 222px; margin-left: 609px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">获取第二单立减活动数据</div></div></div></foreignObject></switch></g><path d="M 685 247 L 670 247 L 532.24 247" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="stroke"></path><path d="M 540.12 242.5 L 531.12 247 L 540.12 251.5" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-end; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 244px; margin-left: 607px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">return</div></div></div></foreignObject></switch></g><rect x="685" y="272" width="10" height="25" fill="rgb(255, 255, 255)" stroke="rgb(0, 0, 0)" pointer-events="all"></rect><path d="M 534 275 L 609 275 L 675.88 275" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"></path><path d="M 682.88 275 L 675.88 278.5 L 675.88 271.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-end; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 272px; margin-left: 609px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">获取 rider pass 活动数据</div></div></div></foreignObject></switch></g><path d="M 685 297 L 670 297 L 532.24 297" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="stroke"></path><path d="M 540.12 292.5 L 531.12 297 L 540.12 301.5" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-end; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 294px; margin-left: 607px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">return</div></div></div></foreignObject></switch></g><path d="M 49.5 72 L 127.25 72 L 196.88 72" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"></path><path d="M 203.88 72 L 196.88 75.5 L 196.88 68.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-end; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 69px; margin-left: 128px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">用户开端</div></div></div></foreignObject></switch></g><path d="M 209.5 82 L 287.25 82 L 356.88 82" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"></path><path d="M 363.88 82 L 356.88 85.5 L 356.88 78.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-end; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 79px; margin-left: 288px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">获取安全/营销数据</div></div></div></foreignObject></switch></g><path d="M 369.5 92 L 447.25 92 L 516.88 92" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="stroke"></path><path d="M 523.88 92 L 516.88 95.5 L 516.88 88.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-end; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 89px; margin-left: 448px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">获取营销数据</div></div></div></foreignObject></switch></g><path d="M 530 452 L 475 452 L 371.74 452" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="stroke"></path><path d="M 379.62 447.5 L 370.62 452 L 379.62 456.5" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="all"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-end; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 449px; margin-left: 450px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; background-color: rgb(255, 255, 255); white-space: nowrap;">return</div></div></div></foreignObject></switch></g><path d="M 490 112 L 550 112 L 550 127 L 540 142 L 490 142 Z" fill="#e1d5e7" stroke="#9673a6" stroke-miterlimit="10" pointer-events="all"></path><path d="M 550 112 L 740 112 L 740 332 L 490 332 L 490 142" fill="none" stroke="#9673a6" stroke-miterlimit="10" pointer-events="none"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 58px; height: 1px; padding-top: 127px; margin-left: 491px;"><div data-drawio-colors="color: rgb(0, 0, 0); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: none; white-space: normal; overflow-wrap: normal;">par</div></div></div></foreignObject></switch></g><path d="M 534 170 L 609 170 L 675.88 170" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="none"></path><path d="M 682.88 170 L 675.88 173.5 L 675.88 166.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="none"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-end; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 167px; margin-left: 609px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: none; background-color: rgb(255, 255, 255); white-space: nowrap;">获取新流营活动数据</div></div></div></foreignObject></switch></g><path d="M 685 192 L 670 192 L 531.74 192" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="none"></path><path d="M 539.62 187.5 L 530.62 192 L 539.62 196.5" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="none"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-end; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 189px; margin-left: 607px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 11px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: none; background-color: rgb(255, 255, 255); white-space: nowrap;">return</div></div></div></foreignObject></switch></g><path d="M 530 402 L 560 402 L 560 432 L 543.12 432" fill="none" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="none"></path><path d="M 536.12 432 L 543.12 428.5 L 543.12 435.5 Z" fill="rgb(0, 0, 0)" stroke="rgb(0, 0, 0)" stroke-miterlimit="10" pointer-events="none"></path><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe flex-start; width: 1px; height: 1px; padding-top: 415px; margin-left: 564px;"><div data-drawio-colors="color: rgb(0, 0, 0); background-color: rgb(255, 255, 255); " style="box-sizing: border-box; font-size: 0px; text-align: left;"><div style="display: inline-block; font-size: 11px; font-family: &quot;Comic Sans MS&quot;; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: none; background-color: rgb(255, 255, 255); white-space: nowrap;">埋点信息处理</div></div></div></foreignObject></switch></g></g><switch><g requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"></g></switch></svg>

![icon](https://cooper.didichuxing.com/cooper_gateway/flow_chart/plugin/api/doc/MjIwNTI2NDA5OTc3OQ/thumbnail?accessToken=undefined&hash=id_2381219790&version=4)



### **2.3 是否引入新技术组件，包括公司内部公共组件及开源组件**

| 名称        | 分类     | 说明       |
| ----------- | -------- | ---------- |
| fission-sdk | 公共组件 | 自动化开品 |
| elvish-sdk  | 公共组件 | 时间转化   |
| gift-s3     | 公共组件 | 图片存储   |



### **2.4 接口文档方案**

1. 接口的命名规范、接口文档跟实现要保持一致

1. 接口是否支持幂等

1. 是否涉及到金额的字段？(请确认清楚金额的单位，是滴分还是当地货币？)  货币军规：http://sailing.intra.didiglobal.com/#/knowledgeDetail?72057594042023450

 1）首页接口

- 接口名：

- 入参调整：

- 返回值调整：



### **2.5 存储设计方案**

1. 无



### **2.6 配置、灰度设计方案**

1. 无，遵守线上已有的开城规则



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

 g. 是否涉及时序问题



## 3. 稳定性方案



### **3.1 系统稳定性**

1. 各类功能是否需要支持动态开关、灰度

1. 降级方式（统一降级服务）

1. 限流（统一限流服务）

1. 异常处理流程

1. 是否需要开发特定脚本支持异常处理方案

1. 监控的考察点（流量监控等）和监控工具



### **3.2 业务稳定性（业务观察指标监控&报警）**

1. 容错性，能否承受外部不合预期的逻辑异常

1. 资损风险

1. 数据安全

1. x小时未发奖报警是否添加



### **3.3 性能评估**

1. 集群大小(服务器、db)

1. 吞吐能力(qps/延迟)

1. 



### 3.4 新老逻辑兼容性评估

1. 老逻辑是否能兼容新流量

1. 新逻辑是否能兼容老流量



### **3.5 外部、上下游依赖评估**

|         |      |      |
| ------- | ---- | ---- |
| 司机api |      |      |
| 乘客api |      |      |
| bff     |      |      |
| 收银    |      |      |
| 反作弊  |      |      |



### **3.6 系统对账【必须】**

是否涉及各系统对账，营销主要依赖反作弊、收银、账单、订单等对账



## 4. 开发排期



### **4.1 排期**

包括前端开发、后端开发、风控、外部合作方、联调、测试、上线时间，给的是端到端最终的交付时间。

| 模块 | 业务方 | 开发 | 联调 | 准入 | QA 负责人 | 放量 | 全量 |
| ---- | ------ | ---- | ---- | ---- | --------- | ---- | ---- |
|      |        |      |      |      |           |      |      |



### **4.2 数据埋点排期**







### **4.3 最终交付整体排期**



| 开发时间 |      |
| -------- | ---- |
| 联调时间 |      |
| 准入时间 |      |
| 测试时间 |      |











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