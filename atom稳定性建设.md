# atom稳定性建设

## 1. TODO

1. atom-api 项目中的mq比较重要，需要单拆一期需求进行稳定性建设

1. 物料中台的稳定性建设



## 2. 会议目的

1. 区分接口的p0、p1、p2级别， 确认哪些接口需要做稳定性建设

1. 目前确认的指标：错误率 错误数量 p99 平均耗时 QPS，是否还有其他补充（订单的同比、环比、分布等业务性指标需要各负责人去做）

1. 聊一聊接口降级熔断的事情



## 3. 整体建设思路



### 3.1 报警级别

| **报警级别**    | **实例**                                  |
| :-------------- | :---------------------------------------- |
| **info（p2）**  | 暂时不做报警处理                          |
| **warn（p1）**  | 支付接口 P99 > 1s 触发 P1 严重告警        |
| **error（p0）** | 2s 触发 P0 紧急告警（若同时伴随高 QPS）。 |



### 3.2 服务监控指标

| **指标**                                                  | **作用**                                                     | **说明**                                                     |
| :-------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **错误率（失败个数、失败率计算，按照errorNo为500 配置）** | 错误请求数占总请求数的比例，反映服务 “异常的相对比例”，避免被绝对数量误导。 | 错误率、错误数量对于核心接口如下单、支付等，容忍率为0，所以SLA建设时，报警的全部为p0 |
| **错误数量**                                              | 单位时间内服务返回错误的请求总数（如 HTTP 4xx/5xx、GRPC 非 OK 状态），反映服务 “异常请求的绝对数量”。 |                                                              |
| **p99**                                                   | 将所有请求的响应时间按从小到大排序后，第 99% 位置的耗时（即 99% 的请求耗时 ≤ 该值，仅 1% 的请求耗时超过）。 |                                                              |
| **平均耗时**                                              | 所有请求的平均响应时间（从请求到达至响应返回的耗时），反映服务的 “整体性能水平”。 |                                                              |
| **QPS**                                                   | 判断服务是否过载（如 QPS 超过设计阈值可能导致响应变慢）      |                                                              |
|                                                           |                                                              |                                                              |



### 3.3 服务等级分类

1. 做报警时，以高峰期流量为标准，建立相关指标报警，未区分日常流量和高峰流量主要是考虑到国际化相关接口是与时间相关的、规律性的波峰、波谷变化；

1. 报警上线后，如果呈现出：报警频繁且无需特殊关注的场景，可以适当调整报警策略，降低相关指标

1. 持续时间：p99、平均耗时、qps等指标都需要包含【持续时间】指标

1. SLA建设只针对 p0、p1级别服务

| **服务等级**     | **适用场景**                                       | **核心保障要求**             |                                                              | **错误率 / 错误数量**                                       |                                        | **p99**                                  |                                          | **平均耗时（**以接口历史平均耗时的 95% 分位值 为基准**）**   |                                                              | **QPS****低流量阈值尝试一下使用环比报警**「历史日均 QPS 的 0.3 倍」为 “低流量阈值” （Q_min），低流量告警收益不太明显，只针对流量较大的接口可能有一些作用，低流量接口无意义 以「历史 3 个月（or 1个月）峰值 QPS 的 1.2 倍」为 “过载阈值” （Q_max） |
| :--------------- | :------------------------------------------------- | :--------------------------- | :----------------------------------------------------------- | :---------------------------------------------------------- | :------------------------------------- | :--------------------------------------- | :--------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **基础说明**     | **warn（p1）**                                     | **error（p0）**              | **warn（p1）**                                               | **error（p0）**                                             | **warn（p1）**                         | **error（p0）**                          | **warn（p1）**                           | **error（p0）**                                              |                                                              |                                                              |
| **核心级（P0）** | 支付、下单、登录等影响业务连续性的服务             | 最高可用性、最短故障恢复时间 | 错误率 > 0或 错误数量 > 0                                    | p99长尾 * 1.3倍持续时间：3分钟                              | p99长尾 * 1.8倍 // TODO持续时间：3分钟 | 超基准值 30% 预警持续时间：3分钟（待定） | 超 60% 严重告警持续时间：3分钟（待定）   | 过载告警：>1.2 * Q_max持续时间：3分钟 低流量告警：<  Q_min   //是否需要低流量报警，低流量报警如何设置时间段持续时间：5分钟 | 过载告警：> 1.5 * Q_max持续时间：1分钟低流量告警：< Q_min    |                                                              |
| **重要级（P1）** | 商品查询、订单查询等核心功能服务查询次数较高的服务 | 高可用性、较快故障恢复时间   | 错误率 > 0.1%，持续时间：3分钟或错误个数 > 50持续时间：5分钟 | 错误率 > 0.5%持续时间：3分钟或错误个数 > 100持续时间：5分钟 | p99长尾 * 1.3倍持续时间：5分钟         | p99长尾 * 2.0倍持续时间：5分钟           | 超基准值 50% 预警持续时间：5分钟（待定） | 超 100% 严重告警持续时间：5分钟（待定）                      | 过载告警：> 1.2* Q_max持续时间：5分钟 低流量告警：<  Q_min 持续时间：10分钟 | 过载告警：> 1.8 * Q_max持续时间：3分钟                       |
| **普通级（P2）** | 数据统计、基础数据查询、历史记录查询等非核心服务   | 基础可用性，常规故障恢复     |                                                              |                                                             | p99长尾 * 1.7倍持续时间：10分钟        | p99长尾 * 2.7倍持续时间：10分钟          | 超基准值 80% 预警持续时间：10分钟        | 超 150% 严重告警持续时间：10分钟                             | 过载告警：> 1.3* Q_max持续时间：15分钟 低流量告警：不监控    | 过载告警：> 2.0 * Q_max持续时间：5分钟                       |



### 3.4 指标获取



#### 3.4.1 聚合含义解析

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1762938967436c91b3486-a064-4c07-9dc3-87a858fd5aa2%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc2ODYwMzgiLCJpYXQiOjE3NzE0MDUyMjIsImV4cCI6MTc3MjAxMDAyMn0.1PN8A8B7_D05JRLoF_999yEADOmPGWFUXwmpZY48qxw)



图里面的聚合【最大值】，聚合维度【caller_func】获取指标为【p99】，一共5台机器，每台机器里面的某个api都有10次请求，先获取每台机器上的 p99的时间，于是获得5个时间数据，再从这5个时间数据中找到最大的



#### **3.4.2  错误率 / 错误数量**

**//errorNo 使用500**

通过LOG._com_request_out 建设新指标，计算错误率，错误数量，详情见【指标建设（新增指标）】

- AGGR.request.error.counter

- AGGR.request.error.rate



#### **3.4.3**  p99指标

统计数据时，当前系统接口、下游rpc接口在配置上唯一的不同就是聚合维度，统计当前系统时，聚合维度为【caller_func】统计下游rpc接口时，聚合维度为【callee_func】

- 指标：by_tags.rpc_dirpc_call.latency 

- 获取p99长尾数据，以7天为一个周期对数据进行统计，获取p99数据最大值作为接口 p99长尾

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1762938087875053d32a0-4584-40d0-8cf1-1a350391f80b%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc2ODYwMzgiLCJpYXQiOjE3NzE0MDUyMjIsImV4cCI6MTc3MjAxMDAyMn0.1PN8A8B7_D05JRLoF_999yEADOmPGWFUXwmpZY48qxw)





#### **3.4.4 平均耗时**

统计数据时，当前系统接口、下游rpc接口在配置上唯一的不同就是聚合维度，统计当前系统时，聚合维度为【caller_func】统计下游rpc接口时，聚合维度为【callee_func】

- 指标：by_tags.rpc_dirpc_call.latency

- 获取【p95】的平均耗时：聚合选择【均值】，聚合维度【caller-func】，*percentile* 选择【95】

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=176293964189240f30794-21ba-4c31-ac4b-3fbb483f74f3%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc2ODYwMzgiLCJpYXQiOjE3NzE0MDUyMjIsImV4cCI6MTc3MjAxMDAyMn0.1PN8A8B7_D05JRLoF_999yEADOmPGWFUXwmpZY48qxw)





#### **3.4.5** QPS

在做QPS指标时，有两类日志可选，下面做一些对比：

- LOG._com_request_in：从请求入口处进行记录，最准确

- by_tags.rpc_dirpc_call.counter：在请求完成后记录，会有请求被过滤掉，比如不符合业务规范、error等场景

- 注意：**callee-func、caller-func选项都需要加上 all**

- ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=176294241880480c9ea73-0560-4709-8f2c-373864d36bb0%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc2ODYwMzgiLCJpYXQiOjE3NzE0MDUyMjIsImV4cCI6MTc3MjAxMDAyMn0.1PN8A8B7_D05JRLoF_999yEADOmPGWFUXwmpZY48qxw)

  

综上，在记录当前项目的当前接口的QPS时，使用【LOG._com_request_in】，在统计下游rpc的QPS时，使用【by_tags.rpc_dirpc_call.counter】，下面的两张图是两类指标的请求QPS数据

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=17629423110630f2aa378-e8f4-425d-849d-e428174e5ec0%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc2ODYwMzgiLCJpYXQiOjE3NzE0MDUyMjIsImV4cCI6MTc3MjAxMDAyMn0.1PN8A8B7_D05JRLoF_999yEADOmPGWFUXwmpZY48qxw)

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=176294233300245e362c8-89d0-49b7-9042-8f8274c461be%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc2ODYwMzgiLCJpYXQiOjE3NzE0MDUyMjIsImV4cCI6MTc3MjAxMDAyMn0.1PN8A8B7_D05JRLoF_999yEADOmPGWFUXwmpZY48qxw)









### 3.5 新增指标

| 目录1      | 目录2                   | 新增指标                   | 原始指标             | 说明1                                                        |
| :--------- | :---------------------- | :------------------------- | :------------------- | :----------------------------------------------------------- |
| 指标计算   | 单指标聚合              | AGGR.request.error.counter | LOG._com_request_out | atom-api项目，errorNo不等于0，但是属于正常业务，errorNo如下：01050：命中反作弊1051 // 时间周期外50002 // 活动未命中50401 // 会员未开城 atom-app项目：1005 1050 // 命中反作弊 50002 50100 50101 50102 50103 50104 // 命中反作弊 50105 // 无需展示 50401 // 会员未开城80018002 // 无参与资格8003 // 手机号不存在8010 // 兑换码无效8011 // 兑换码已兑完100300 // 会员不能取消，100500 // 会员pdp不能取消， |
| 多指标运算 | AGGR.request.error.rate | LOG._com_request_out       |                      |                                                              |
|            |                         |                            |                      |                                                              |
|            |                         |                            |                      |                                                              |





## 4. 接口及指标统计（过时）

- 梳理出监控项，分为两类，自身和下游

- 监控大盘：做全局监控，不区分具体的接口

> 针对个别的日志中不存在的数据，可以考虑通过【指标计算】下的三种模式进行新指标的构建；将已有的一个或多个指标通过聚合、数学运算等方式进行

| **一级分类**           | **二级分类**                   | **名称**                                                     | **指标**                                                     | **作用**                                                     | **SLA**                                                      |
| :--------------------- | :----------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **基础指标**           | **cpu**                        | cpu空闲率                                                    | cpu.idle                                                     |                                                              |                                                              |
| cpu在一段时间内的负载  | loadavg1、loadavg5 、loadavg15 | 数值越低：系统越空闲，CPU 有充足能力处理任务                 |                                                              |                                                              |                                                              |
|                        |                                |                                                              |                                                              |                                                              |                                                              |
| **内存mem**            | 内存使用率（mem）              | mem.used                                                     |                                                              |                                                              |                                                              |
| mem使用率              | mem.used.percent               |                                                              |                                                              |                                                              |                                                              |
|                        |                                |                                                              |                                                              |                                                              |                                                              |
| **磁盘**               | 所有分区已使用字节数           | disk.total.used                                              |                                                              |                                                              |                                                              |
| 所有分区容量使用率     | disk.total.used.percent        |                                                              |                                                              |                                                              |                                                              |
| 磁盘io使用率           | io.util                        | 磁盘读写繁忙程度，过高会拖慢系统                             |                                                              |                                                              |                                                              |
|                        |                                |                                                              |                                                              |                                                              |                                                              |
| **网络**               | 某块网卡的接收流量             | net.in                                                       |                                                              |                                                              |                                                              |
| 某块网卡的接收丢包量   | net.in.dropped                 |                                                              |                                                              |                                                              |                                                              |
|                        |                                |                                                              |                                                              |                                                              |                                                              |
| **服务指标**           | **可用性指标（自身）**         | 运行时长                                                     | PROM.mesh_proxy_prometheus.envoy_server_uptime               |                                                              | 当单位时间内（如 1 小时）重启次数超过阈值（如 3 次），触发告警（可能存在稳定性风险） |
| 服务可用率             | agent.alive                    |                                                              | 服务正常运行时间占总时间的比例，计算公式为 `(总时间 - 故障时间) / 总时间 × 100%`，通常要求核心服务达到 99.9% 及以上。 |                                                              |                                                              |
|                        |                                |                                                              |                                                              |                                                              |                                                              |
| **可用性指标（下游）** | 运行时长                       | PROM.mesh_proxy_prometheus.envoy_server_uptime               |                                                              | 当单位时间内（如 1 小时）重启次数超过阈值（如 3 次），触发告警（可能存在稳定性风险） |                                                              |
| 服务可用率             | agent.alive                    |                                                              | 服务正常运行时间占总时间的比例，计算公式为 `(总时间 - 故障时间) / 总时间 × 100%`，通常要求核心服务达到 99.9% 及以上。 |                                                              |                                                              |
|                        |                                |                                                              |                                                              |                                                              |                                                              |
| **性能指标（自身）**   | 平均响应时间（ART）            |                                                              | 所有请求的响应时间平均值，反映整体处理效率                   |                                                              |                                                              |
| P95/P99 响应时间       |                                | 按响应时间从小到大排序后，第 95%/99% 位置的数值，更能体现 “慢请求” 的真实情况（避免被平均值掩盖极端值） |                                                              |                                                              |                                                              |
| qps                    |                                |                                                              |                                                              |                                                              |                                                              |
| 错误率（Error Rate）   |                                |                                                              |                                                              |                                                              |                                                              |
|                        |                                |                                                              |                                                              |                                                              |                                                              |
| **性能指标（下游）**   | 平均响应时间（ART）            |                                                              | 所有请求的响应时间平均值，反映整体处理效率                   |                                                              |                                                              |
| P95/P99 响应时间       |                                | 按响应时间从小到大排序后，第 95%/99% 位置的数值，更能体现 “慢请求” 的真实情况（避免被平均值掩盖极端值） |                                                              |                                                              |                                                              |
| qps                    |                                |                                                              |                                                              |                                                              |                                                              |
| 错误率（Error Rate）   |                                |                                                              |                                                              |                                                              |                                                              |





## 5. 接口指标阈值 



### 5.1 atom-app

‌

 atom-app接口数据统计文件_v1.xlsx (14.6KB)

‌



‌

 atom-app下游rpc接口数据统计文件_v1.xlsx (14.7KB)

‌





#### 5.1.1 两日内访问量曲线

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1762418014374ced0a216-f331-48b8-b120-451dbb4b7de2%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc2ODYwMzgiLCJpYXQiOjE3NzE0MDUyMjIsImV4cCI6MTc3MjAxMDAyMn0.1PN8A8B7_D05JRLoF_999yEADOmPGWFUXwmpZY48qxw)





#### 5.1.2 自身接口访问量数据，按照不同模块进行分类

| **业务**             | **服务等级**                                 | **接口**                           | **功能**                         | **Max（10s）** | **Min**   | **Avg**          | **说明**         | **p99** | **平均耗时** | **QPS**    |
| -------------------- | -------------------------------------------- | ---------------------------------- | -------------------------------- | -------------- | --------- | ---------------- | ---------------- | ------- | ------------ | ---------- |
| **等必赔**           | p2                                           | /slowpay/h5/info/v2                | 慢必赔H5页面                     | --             | --        | --               | 两日内无访问数据 |         |              |            |
| p1                   | /slowpay/resource/judge                      | 判定是否展示资源位                 | 351.652                          | 127.093        | 240.218   |                  | 270.08           | 26.526  | 426.364      |            |
|                      |                                              |                                    |                                  |                |           |                  |                  |         |              |            |
| **抽奖转盘**         | p3（忽略此接口）                             | /lucky_draw/launch                 | 发起抽奖                         | --             | --        | --               | 两日内无访问数据 |         |              |            |
| p3（忽略此接口）     | /lucky_draw/h5/info                          | 获取H5感知页数据                   | --                               | --             | --        | 两日内无访问数据 |                  |         |              |            |
| p3（忽略此接口）     | /lucky_draw/h5/records                       | 获取抽奖记录                       | --                               | --             | --        | 两日内无访问数据 |                  |         |              |            |
| p3（忽略此接口）     | /lucky_draw/resource/judge                   | 判定是否展示资源位                 | --                               | --             | --        | 两日内无访问数据 |                  |         |              |            |
|                      |                                              |                                    |                                  |                |           |                  |                  |         |              |            |
| **新乘客套餐**       | p1                                           | /pass/api/getSideInfo              | 套餐侧边栏                       | 59077.725      | 4504.594  | 29588.712        |                  | 473.15  | 34.986       | 73640.27   |
| p1                   | /pass/api/homepageRecommend                  | 首页营销位                         | 22870.997                        | 2120.829       | 11257.479 |                  | 574.75           | 65.837  | 29335.258    |            |
| p3（忽略此接口）     | /pass/mock/resourceLog                       | 资源位埋点日志consumer             | --                               | --             | --        | 两日内无访问数据 |                  |         |              |            |
| p1                   | /pass/api/h5/totalRevenueaz                  | 获取套餐首页信息                   | 81.75                            | 6              | 69.412    |                  | 497              | 13.462  |              |            |
| p1                   | /pass/api/h5/detail                          | 获取套餐详情                       | 27                               | 8              | 14.895    |                  | 191.25           | 16.672  |              |            |
| p1                   | /pass/api/h5/orderDetail                     | 获取兑换详情                       | 2                                | 1              | 1.333     |                  | 241              | 43.5    |              |            |
| p0                   | /pass/api/h5/buy                             | 套餐H5购买                         | 69                               | 3              | 42.9      |                  | 490              | 50.084  |              |            |
| p1                   | /pass/api/getPassDetailRule                  | 获取套餐详情页使用须知文案         | 90.333                           | 1              | 12.202    |                  |                  |         | 118.5        |            |
| p3（忽略此接口）     | /pass/api/saRecommend                        | SA首页营销位                       | --                               | --             | --        | 两日内无访问数据 |                  |         |              |            |
| p1                   | /pass/api/h5/userPassList                    | 获取兑换列表                       | 35                               | 1              | 22.5      |                  | 416              | 13.687  |              |            |
| p0                   | /pass/api/onTripBuy                          | 行程中按钮购买                     | 89.75                            | 1              | 43.653    |                  | 2539             | 170.733 | 160.781      |            |
| p1                   | /pass/api/resource/judge                     | 判定是否展示资源位                 | 593.061                          | 132.604        | 363.983   |                  | 243.61           | 38.343  | 8.018        |            |
| p1                   | /pass/api/h5/mallHomepage                    | 获取套餐商城首页数据               |                                  |                |           |                  | 204              | 19.799  |              |            |
| p1                   | pass/api/h5/myPasses                         | 获取我的套餐页面数据               |                                  |                |           |                  | 502              | 44.991  |              |            |
| p1                   | /pass/api/h5/passDetail                      | 获取套餐详情（包含未购买、已购买） |                                  |                |           |                  | 237              | 19.715  |              |            |
| p1                   | /pass/api/h5/autoRenew/detail                | 获取自动续费信息                   |                                  |                |           |                  | 40               | 24.333  |              |            |
| p1                   | /pass/api/h5/autoRenew/updateAutoRenewal     | 更新自动续费                       |                                  |                |           |                  | 34               | 18.5    |              |            |
| p1                   | /pass/api/h5/autoRenew/updatePaymentCallBack | 更新支付回调                       |                                  |                |           |                  | 64               | 36      |              |            |
| p1                   | /pass/api/h5/autoRenew/reactivate            | 重新激活自动续费                   |                                  |                |           |                  | 64               | 33.5    |              |            |
|                      |                                              |                                    |                                  |                |           |                  |                  |         |              |            |
| **didi club**        | p1                                           | /club/api/getJoinMemberInfo        | 获取可加入的会员信息（冒泡拦截） | 11450.458      | 940.569   | 5581.82          |                  | 282.78  | 20.95        | 14720.658  |
| p1                   | /club/api/cancelOrder                        | 乘客取消会员                       | 3                                | 1              | 1.211     |                  | 841              | 841     | 4            |            |
| p3（忽略此接口）     | /club/api/popup/set                          | 弹窗免打扰设置                     | 38                               | 1              | 17.262    |                  |                  |         |              |            |
|                      |                                              |                                    |                                  |                |           |                  |                  |         |              |            |
| **兑换码**           | p0                                           | /exchange_code/v1/redeem           | 使用兑换码                       | 48             | 2         | 31.142           |                  | 2010    | 131.776      |            |
|                      | p1                                           | /exchange_code/entrance            | 兑换码入口                       |                |           |                  |                  |         |              |            |
|                      | p1                                           | /exchange_code/ob/redeem           | 兑换码兑换                       |                |           |                  |                  | 1014    | 883          | 10         |
|                      |                                              |                                    |                                  |                |           |                  |                  |         |              |            |
| **virgin权益**       | p3（忽略此接口）                             | /virgin/h5/getBaseInfo             |                                  | 14             | 1         | 5.895            |                  |         |              |            |
| p3（忽略此接口）     | /virgin/h5/tripsDetail                       |                                    | 6                                | 1              | 3.071     |                  |                  |         |              |            |
| p1                   | /virgin/bind                                 |                                    | 16                               | 2              | 4.6       |                  | 7000             | 4591    |              |            |
| p3（忽略此接口）     | /virgin/changeType                           |                                    | --                               | --             | --        | 两日内无访问数据 |                  |         |              |            |
|                      |                                              |                                    |                                  |                |           |                  |                  |         |              |            |
| **dxgy**             |                                              | /dxgy/resource/judge               | 判定是否展示资源位               | --             | --        | --               | 老dxgy不需要处理 |         |              |            |
|                      | /dxgy/h5/getTaskDetails                      | 获取H5活动详情页信息               | --                               | --             | --        |                  |                  |         |              |            |
|                      | /dxgy/h5/getOrderInfo                        | 获取活动订单信息                   | --                               | --             | --        |                  |                  |         |              |            |
|                      | /dxgy/h5/getDescription                      | 获取H5活动文案                     | --                               | --             | --        |                  |                  |         |              |            |
|                      |                                              |                                    |                                  |                |           |                  |                  |         |              |            |
| **乘客营**           | p1                                           | /camp/api/homePopup                | 首页弹窗                         | 4182.055       | 344.787   | 2106.689         |                  | 310.78  | 41.953       | 4295.582   |
| p1                   | /camp/api/homePageTask                       | 首页营销位                         | 22069.875                        | 1116.947       | 10402.171 |                  | 405.7            | 32.78   | 28104.628    |            |
| p3（忽略此接口）     | /camp/h5/newUser                             | H5主页信息                         | 19                               | 2              | 8.526     |                  |                  |         |              |            |
| p2                   | /camp/api/touchHolder                        | 触达占位符                         | 213.5                            | 89             | 158.422   |                  |                  |         |              |            |
|                      |                                              |                                    |                                  |                |           |                  |                  |         |              |            |
| **通勤玩法**         | p1                                           | /commuting/api/resource/judge      | 判定是否展示资源位               | 35315.333      | 1901.208  | 17510.966        |                  | 354.93  | 85.714       | 43262.033  |
| p2                   | /commuting/api/resource/wholeTrip            | 行程中资源位                       | 1750.667                         | 198.297        | 1062.228  |                  |                  |         |              |            |
| p2                   | /commuting/api/touchHolder                   | 触达占位符                         | 5049.132                         | 3              | 3489.247  |                  |                  |         |              |            |
| p1                   | /commuting/h5/bind                           | 活动绑定                           | 149.171                          | 26.2           | 135.668   |                  | 276              | 47.85   |              |            |
| p2                   | /commuting/h5/main                           | 活动主页                           | 168.313                          | 39.65          | 150.534   |                  |                  |         |              |            |
|                      |                                              |                                    |                                  |                |           |                  |                  |         |              |            |
| **第二单立减**       | p3（忽略此接口）                             | /second_trip/h5/main               | 落地页详情&同时绑定活动          | --             | --        | --               | 两日内无访问数据 |         |              |            |
| p1                   | /second_trip/api/resource/judge              | 判定是否展示资源位                 | 2467.542                         | 230.471        | 1381.66   |                  | 327.56           | 46.058  | 3163.506     |            |
| p3（忽略此接口）     | /second_trip/api/touchHolder                 | 触达占位符                         | --                               | --             | --        | 两日内无访问数据 |                  |         |              |            |
|                      |                                              |                                    |                                  |                |           |                  |                  |         |              |            |
| **开端页**           | p3（忽略此接口）                             | /registerPage/greenBar             | 注册页绿条                       | 234.975        | 135.025   | 190.012          |                  |         |              |            |
|                      |                                              |                                    |                                  |                |           |                  |                  |         |              |            |
| **预估冒泡推荐活动** | p0                                           | /getEstimateInfo                   | 随单购-冒泡&预估推荐套餐         | 11464.458      | 955.911   | 5597.062         |                  | 450.2   | 46.751       | 15431.445  |
|                      |                                              |                                    |                                  |                |           |                  |                  |         |              |            |
| **行程中**           | p1                                           | /onTripCardBanner                  | 行程中banner卡片                 | 234410.583     | 15492.417 | 128427.985       |                  | 405.01  | 22.117       | 277607.328 |
|                      |                                              |                                    |                                  |                |           |                  |                  |         |              |            |
| **乘客组件**         | p3（忽略此接口）                             | /h5/component/data                 | 组件数据                         | --             | --        | --               | 两日内无访问数据 |         |              |            |
| p3（忽略此接口）     | /h5/component/lucky_draw/launch              | 组件抽奖                           | --                               | --             | --        | 两日内无访问数据 |                  |         |              |            |
|                      |                                              |                                    |                                  |                |           |                  |                  |         |              |            |
| **完单页**           | p1                                           | /orderPayPage                      |                                  | 6220.792       | 406.971   | 3198.052         |                  | 400.31  | 7.982        | 6781.141   |
|                      |                                              |                                    |                                  |                |           |                  |                  |         |              |            |
| **券详情列表**       | p2                                           | /coupon/passCard                   | 券列表数据                       | 140.788        | 20.5      | 127.603          |                  |         |              |            |
|                      |                                              |                                    |                                  |                |           |                  |                  |         |              |            |
| **通用**             |                                              | /common/mock/localTime             |                                  | 0              |           |                  |                  |         |              |            |
|                      | /common/mock/delTime                         |                                    | 0                                |                |           |                  |                  |         |              |            |
|                      | /common/mock/getTime                         |                                    | 0                                |                |           |                  |                  |         |              |            |
|                      | /common/mock/bizTime                         |                                    | 0                                |                |           |                  |                  |         |              |            |
|                      | /common/mock/testApi                         |                                    | 0                                |                |           |                  |                  |         |              |            |
|                      | /common/mock/sendIMC                         |                                    | 0                                |                |           |                  |                  |         |              |            |



#### 5.1.3 下游访问量数据

| **disf服务**                                       | **调用接口**                                                 | **服务等级**                                                 | **失败影响**                                | **强弱依赖**   | **Max**                                                      | **Min**    | **Avg**       | **Sum**      | p99    | 平均耗时 | QPS       |
| -------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------- | -------------- | ------------------------------------------------------------ | ---------- | ------------- | ------------ | ------ | -------- | --------- |
| disf!codis-fuproxy-normandy-br-dxgytasksim_fuproxy | redis                                                        | p0                                                           | 所有缓存依赖                                | 强（sim环境）  |                                                              |            |               |              |        |          |           |
| disf!common-plat-public-dos-dos_order              | /dos/getOrderInfo                                            | p3（忽略此接口）                                             | 无调用                                      | 弱             |                                                              |            |               |              |        |          |           |
| disf!common-plat-public-passport                   | ValidateTicket                                               | p3（忽略此接口）                                             | 所有入口校验不通过直接返回                  | 强             |                                                              |            |               |              |        |          |           |
| disf!daijia-nodejs-shortserver_i18n                | /admin/add                                                   | p2                                                           | 无法生成短链接                              | 强             | 641.259                                                      | 33         | 354.354       | 255488.954   |        |          |           |
| disf!ds-feature-mining-ditag-i18n-ditag_judge      | /phoenix_point_judge/judge                                   | p2                                                           | 无法判定是否符合tag                         | 强             | 519.874                                                      | 180        | 424.562       | 306109.345   |        |          |           |
| disf!ibt-falcon-ceres                              | /gulfstream/api/ceres/v1/notifyNewCoupon                     | p2                                                           | 无法通知ceres新券                           | 弱             | 419.357                                                      | 72         | 291.341       | 210056.781   |        |          |           |
| /gulfstream/api/ceres/v1/getPriceInfo              | p3（忽略此接口）                                             | 无法获取行程中的价格信息，目前无调用                         | 弱                                          | --             | --                                                           | --         | --            |              |        |          |           |
| disf!ibt-guarana-task_taiphon                      | /api/bubble_card/detail                                      | p1                                                           | 获取slowPay 预估卡片，冒泡页无法推荐slowpay | 强             | ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1762430723713c2389f9f-6980-4139-bd1d-b2ebd4670cb8%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc2ODYwMzgiLCJpYXQiOjE3NzE0MDUyMjIsImV4cCI6MTc3MjAxMDAyMn0.1PN8A8B7_D05JRLoF_999yEADOmPGWFUXwmpZY48qxw) | --         | --            | --           | 14     | 2.438    | 19        |
| disf!ibt-marvel-atom_api                           | /gulfstream/atom-api/pass/recommend                          | p3（忽略此接口）                                             | 无法获取推荐套餐                            | 强             | --                                                           | --         | --            | --           |        |          |           |
| /gulfstream/atom-api/slowpay/redirect              | p2                                                           | 调用os2获取可参与的SlowPay活动信息，页面查看异常-调用下游查询活动异常 | 强                                          | 1054.904       | 386                                                          | 721.476    | 520184.392    |              |        |          |           |
| /gulfstream/atom-api/pass/buy                      | p0                                                           | 无法购买套餐                                                 | 强                                          | 183.75         | 6                                                            | 79.799     | 57535.25      |              |        |          |           |
| /gulfstream/atom-api/pass/detail                   | p2                                                           | 无发展示套餐详情（未购买）                                   | 强                                          | 102            | 3                                                            | 40.39      | 29121         |              |        |          |           |
| /gulfstream/atom-api/pass/orderDetail              | p2                                                           | 无法展示套餐兑换详情（已购买）                               | 强                                          | 15             | 3                                                            | 4.614      | 1218          |              |        |          |           |
| /gulfstream/atom-api/club/getMemberInfo            | p1                                                           | 无法获取外卖获取会员信息                                     | 强                                          | 42325.38       | 2499.009                                                     | 24274.71   | 17502066.229  |              |        |          |           |
| /gulfstream/atom-api/club/cancelMemberOrder        | p0                                                           | 完单页无法取消会员购买                                       | 强                                          | 9              | 3                                                            | 3.6        | 72            |              |        |          |           |
| /gulfstream/atom-api/pass/updateUserPassStatusMq   | p2                                                           | 无法更新套餐过期状态                                         | 弱（err无ret）                              | 186            | 3                                                            | 36.945     | 25861.333     |              |        |          |           |
| /gulfstream/atom-api/pass/onTripBuy                | p0                                                           | 无法套餐行程中购买                                           | 强                                          | 266.25         | 3                                                            | 131.39     | 94600.75      |              |        |          |           |
| /gulfstream/atom-api/pass/getCoupons               | p1                                                           | 获取乘客套餐可用券，获取用户券，无券则不发push               | 弱                                          | 15939          | 3                                                            | 272.96     | 64964.55      |              |        |          |           |
| /gulfstream/atom-api/pass/getCampaignByID          | p2                                                           | 根据id获取活动信息，失败无法组装行程中banner卡片             | 强                                          | 443.452        | 21                                                           | 304.26     | 219371.369    |              |        |          |           |
| /gulfstream/atom-api/club/newOrder                 | p3（忽略此接口）                                             | 无法会员发单通知接口                                         | 强                                          | --             | --                                                           | --         | --            |              |        |          |           |
| /gulfstream/atom-api/camp/homeBind                 | p1                                                           | 乘客营首页绑定，无法绑定活动                                 | 强                                          | 11684.5        | 991.371                                                      | 6151.568   | 4435280.381   |              |        |          |           |
| /gulfstream/atom-api/camp/getCoupons               | p2                                                           | 获取乘客营券，无法渲染感知&最优券无法pk                      | 强                                          | 358            | 57                                                           | 232.75     | 167812.439    |              |        |          |           |
| /gulfstream/atom-api/commuting/bind                | p1                                                           | 通勤活动绑定，无法绑定                                       | 强                                          | 466.915        | 9                                                            | 225.534    | 162609.911    |              |        |          |           |
| /gulfstream/atom-api/commuting/info                | p1                                                           | 获取通勤活动数据，无法渲染感知信息&触达信息                  | 强                                          | 124812.879     | 7290.896                                                     | 63166.623  | 45543135.167  |              |        |          |           |
| /gulfstream/atom-api/second_trip/bind              | p3（忽略此接口）                                             | 第二单立减活动主页无法活动绑定                               | 强                                          | --             | --                                                           | --         | --            |              |        |          |           |
| /gulfstream/atom-api/second_trip/info              | p1                                                           | 获取第二单立减活动信息，无法渲染感知位                       | 强                                          | 202750.736     | 12335.397                                                    | 112287.739 | 80959459.643  |              |        |          |           |
| /gulfstream/atom-api/common/recommend              | p1                                                           | 套餐、慢必赔活动通用无法查询活动推荐                         | 强                                          | 274683.275     | 17816.497                                                    | 144635.228 | 104281999.201 |              |        |          |           |
| /gulfstream/atom-api/coupon/couponList             | p2                                                           | 无法获取券列表页面展示数据                                   | 强                                          | 499.291        | 303                                                          | 434.91     | 313570.129    |              |        |          |           |
| disf!ibt-os2-os2_action                            | /os2_action/api/send_imc                                     | p1                                                           | 触发imc，该接口os2是异步处理的              | 强             | 14875.5                                                      | 3          | 249.923       | 59481.65     | 150    | 112      | 8094.917  |
| disf!ibt-os2-os2_api                               | /os2_api/activity/batch/get_by_routine_key                   | p1                                                           | 仅推荐可用, 无法感知被推荐人进度查询        | 强             | 66653.999                                                    | 3460.725   | 31636.989     | 22810268.933 |        |          |           |
| disf!ibt-pay-ms-coupon                             | /foundation/coupon/v1/valuationinterface/valuateBatchWithAmount | p1                                                           | 获取券抵扣金额，无法进行最优券pk            | 强             | 118739.75                                                    | 5174.95    | 53746.09      | 38750931.003 | 407.85 | 14.413   | 55091.061 |
| disf!ibt-ring-seg_data_center                      | /psi/seg-data-center/api/v1/feature/query                    | p3（忽略此接口）                                             | 无调用                                      | 弱             | --                                                           | --         | --            | --           |        |          |           |
| disf!kproxy_vipcenter-kproxy                       | redis                                                        | p0                                                           | 所有缓存依赖                                | 强（线上环境） |                                                              |            |               |              |        |          |           |
| disf!os-volcano-task-api                           | /task/openApi/xpanel/taskInfoV2                              | p3（忽略此接口）                                             | 乘客DXGY活动详情页数据                      | 强             | --                                                           | --         | --            | --           |        |          |           |
| /task/openApi/xpanel/orderListV2                   | p3（忽略此接口）                                             | 乘客DXGY订单列表页查                                         | 强                                          | --             | --                                                           | --         | --            |              |        |          |           |
| /task/openApi/xpanel/detailDescription             | p3（忽略此接口）                                             | 乘客DXGY活动法律文案                                         | 强（弱？）                                  | --             | --                                                           | --         | --            |              |        |          |           |
| disf!sailing-growth-services-benefit_growth        | /benefitGrowth/getValidMemberBenefit                         | p2                                                           | 查询已购买的会员权益                        | 强             | 9                                                            | 3          | 3.369         | 438          |        |          |           |





### 5.2 atom-api

‌

 atom-api接口访问量统计文件_v1.xlsx (12.9KB)

‌



‌

 atom-api下游rpc接口统计文件_v1.xlsx (15.2KB)

‌



//TODO ：atom-app中调用了atom-api的接口，但是atom-app中rpc调用量与atom-api中统计的数据不一致，需要排查 （可能是重试导致的）

//TODO： 对redis的访问时长、error进行统计

//TODO ：数据库的访问异常直接在请求的error里面看？或者，error里面肯定有体现，然后对mysql的异常也独立报警，能立刻识别到mysql的问题，



#### 5.2.1 两日内访问曲线

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=1762248907295e4ce3f7c-f6e8-42c6-9ac1-2a37ba465534%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc2ODYwMzgiLCJpYXQiOjE3NzE0MDUyMjIsImV4cCI6MTc3MjAxMDAyMn0.1PN8A8B7_D05JRLoF_999yEADOmPGWFUXwmpZY48qxw)





#### 5.2.2 自身访问量统计

| **业务**                     | **接口**               | **服务等级**                             | **功能**                               | **Max**   | **Min**   | **Avg**                                                      |        | **p99**                                                      | **平均耗时** | **QPS**   |
| ---------------------------- | ---------------------- | ---------------------------------------- | -------------------------------------- | --------- | --------- | ------------------------------------------------------------ | ------ | ------------------------------------------------------------ | ------------ | --------- |
| **pass接口**                 | /pass/buy              | p0                                       | 套餐购买                               | 50        | 2         | 25.296                                                       |        | 509  1061(第二次记录的结果，但是这个结果高于平均水平，不作为最后阈值) | 13.359       | 68.842    |
| /pass/cashier/payCallBack    | p0                     | 支付回调                                 | 29                                     | 1         | 11.007    |                                                              | 1048   | 33.122                                                       | 58.617       |           |
| /pass/cashier/refund         | p3（忽略此接口）       |                                          | --                                     | --        | --        | 两日内无访问数据                                             |        |                                                              |              |           |
| /pass/isPassOrder            | p3（忽略此接口）       |                                          | --                                     | --        | --        | 两日内无访问数据                                             |        |                                                              |              |           |
| /pass/cancelOrder            | p3（忽略此接口）       |                                          | --                                     | --        | --        | 两日内无访问数据                                             |        |                                                              |              |           |
| /pass/detail                 | p1                     | 套餐详情                                 | 59.25                                  | 1         | 18.641    |                                                              | 77     | 5.661                                                        | 5.661        |           |
| /pass/orderList              | p1                     | 订单列表                                 | 48.833                                 | 6         | 27.695    |                                                              | 100    | 8.706                                                        | 72.514       |           |
| /pass/orderDetail            | p1                     | 订单详情                                 | 5                                      | 1         | 1.554     |                                                              | 94     | 12                                                           | 16           |           |
| /pass/updateUserPassStatusMq | p1                     | 更新用户套餐装填                         | 49                                     | 1         | 11.578    |                                                              | 33     | 3.057                                                        | 59.5         |           |
| /pass/onTripBuy              | p0                     | 行中购买                                 | 67.117                                 | 1         | 36.218    |                                                              | 1213.6 | 34.518                                                       | 81.578       |           |
| /pass/getCoupons             | p2                     | 获取套餐下的券                           | 5011.5                                 | 1         | 79.446    |                                                              |        |                                                              |              |           |
| /pass/getCampaignByID        | p0                     | 根据routine_id获取活动数据               | 92.323                                 | 7         | 61.544    |                                                              | 107.93 | 3.157                                                        | 99.548       |           |
| **slowPay接口**              | /slowpay/bind          | p1                                       | 在用户打车时，给用户绑定slowpay活动    | 11091.833 | 535.625   | 5317.768                                                     |        | 217                                                          | 17.017       | 11818.857 |
| /slowpay/redirect            | p1                     | 获取活动相关数据（不绑定）               | 336.55                                 | 68.251    | 192.386   |                                                              | 158.08 | 20.741                                                       | 405.902      |           |
| **club接口**                 | /club/getMemberInfo    | p1                                       | 获取当前用户会员信息（已购买or可购买） | 14687.875 | 802       | 8060.819                                                     |        | 252.38                                                       | 38.109       | 18214.639 |
| /club/newOrder               | p3（忽略此接口）       | 会员发单                                 | --                                     | --        | --        | 两日内无访问数据                                             |        |                                                              |              |           |
| /club/getMemberCancelInfo    | p2                     | 获取取消会员信息                         | 1175.792                               | 209.65    | 783.037   |                                                              |        |                                                              |              |           |
| /club/cancelMemberOrder      | p0                     | 取消会员订单                             | 4                                      | 1         | 1.227     |                                                              | 566    | 84.4                                                         | 7            |           |
| **乘客营相关接口**           | /camp/homeBind         | p1                                       | 首页弹窗绑定活动                       | 3917.792  | 303.123   | 2031.543                                                     |        | 250.24                                                       | 18.092       | 4171.965  |
| /camp/getCoupons             | p2                     | 查询乘客营发的券                         | 70.878                                 | 14.5      | 53.862    |                                                              |        |                                                              |              |           |
| /camp/script/bestAct         | p3（忽略此接口）       | 最优活动脚本                             |                                        |           |           | ![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc-upload-image-prod&k=17624849891803f8e0c87-155b-49e7-827d-dcbf97d2b4c1%2Fimage.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6Ijc2ODYwMzgiLCJpYXQiOjE3NzE0MDUyMjIsImV4cCI6MTc3MjAxMDAyMn0.1PN8A8B7_D05JRLoF_999yEADOmPGWFUXwmpZY48qxw)访问量极少 |        |                                                              |              |           |
| **通勤相关接口**             | /commuting/info        | p1                                       | 活动数据获取                           | 41563.958 | 2390.875  | 21356.568                                                    |        | 475.52 ｜ 318.45                                             | 22.982       | 45417.237 |
| /commuting/bind              | p1                     | 活动绑定                                 | 83.471                                 | 3         | 47.958    |                                                              | 221.29 | 41.827                                                       | 111.803      |           |
| **乘客通用接口**             | /common/trip/endCharge | p1                                       | 结束计费-返回费项                      | 2748.333  | 202.566   | 1565.146                                                     |        |                                                              |              | 11927.829 |
| /common/recommend            | p1                     | 套餐、slowpay 活动推荐                   | 95772.708                              | 5865.708  | 49768.426 |                                                              | 219.85 | 9.984                                                        | 112602.249   |           |
| /common/tradeCallBack        | p2                     | 收银代收单支付成功回调，目前不做任何处理 | 5                                      | 1         | 1.464     |                                                              |        |                                                              |              |           |
| **券列表页接口**             | /coupon/couponList     | p2                                       | 券列表页接口                           | 96.441    | 55.002    | 80.62                                                        |        | 198                                                          | 2.82         | 106.3     |
| **mock接口**                 | /mock/tripMQ           |                                          |                                        | 0         |           |                                                              |        |                                                              |              |           |
| /mock/delCache               |                        |                                          | 0                                      |           |           |                                                              |        |                                                              |              |           |
| /pass/mockPush               |                        |                                          | 0                                      |           |           |                                                              |        |                                                              |              |           |
| /pass/mockGetCache           |                        |                                          | 0                                      |           |           |                                                              |        |                                                              |              |           |



#### 5.2.3 下游访问量统计

disf服务

| **rpc服务**                                                  | **调用接口**                                                 | **强弱依赖**                               | **接口等级**                                   | **失败影响**                               | **Max**    | **Min**   | **Avg**          |                                                              | **p99**          | **平均耗时**   | **QPS**   |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------ | ---------------------------------------------- | ------------------------------------------ | ---------- | --------- | ---------------- | ------------------------------------------------------------ | ---------------- | -------------- | --------- |
| disf!codis-fuproxy-ibt-growth-eng-tech-ibtshopping_fuproxy   | fusion                                                       | 强                                         | p0                                             | 支付加锁防重入下单购买缓存                 |            |           |                  |                                                              |                  |                |           |
| disf!codis-fuproxy-ibt-growth-eng-tech-ibtshoppingsim_fuproxy | fusion                                                       | 强（sim）                                  | p3（忽略此接口）                               | 支付加锁防重入下单购买缓存                 |            |           |                  |                                                              |                  |                |           |
| disf!codis-fuproxy-normandy-br-dxgytask_fuproxy              | redis                                                        | 强                                         | p0                                             | 所有缓存依赖                               |            |           |                  |                                                              |                  |                |           |
| disf!common-plat-public-dos-dos_order                        | /dos/getOrderInfo                                            | 强                                         | p1                                             | 无法获取DOS订单系统订单信息                | 4453.926   | 369.503   | 2345.553         |                                                              | 294.3            | 7.645 ｜ 9.895 | 4805.767  |
| disf!engine-gs-locserver                                     | /engine/locsvr/get_city                                      | 强                                         | p2                                             | 无法获取城市信息                           | --         | --        | --               | 两日内无访问数据                                             |                  |                |           |
| disf!ibt-falcon-ceres                                        | /gulfstream/api/ceres/v1/updateFeeInfo                       | 强                                         | p2                                             | 代收成功后无法通知价格更新行程中感知       | 78.217     | 1         | 39.828           |                                                              |                  |                |           |
| /gulfstream/api/ceres/v1/getPriceInfo                        | 强                                                           | p1                                         | 无法获取费项信息                               | 2855.917                                   | 202.344    | 1570.795  |                  | 186.72                                                       | 23.453 ｜ 37.459 | 2990.663       |           |
| disf!ibt-falcon-dmc                                          | /gulfstream/dmc/push/send                                    | 强（协程异步通知，弱？）                   | p2                                             | 不能发送push信息                           | 1          | 1         | 1                |                                                              |                  |                |           |
| disf!ibt-falcon-price-cashier                                | /gulfstream/pay/v1/cashier/getBatchFeeDetail                 | 强                                         | p2                                             | 会员完单页无法获取券优惠折扣               | 10         | 1         | 2.774            |                                                              |                  |                |           |
| disf!ibt-guarana-task_taiphon                                | /api/revoke_coupon                                           | 强                                         | p1                                             | 无法撤回会员slp券                          | 1          | 1         | 1                |                                                              | 77 \| 27         | 27             | 3         |
| disf!ibt-marvel-dive_biz_g                                   | /gulfstream/dive-biz/v2/api/purchase/pre                     | 强                                         | p1                                             | 无法创建订单事件同步biz                    | 173.231    | 20        | 100.234          |                                                              | 200              | 12.268         | 73.783    |
| /gulfstream/dive-biz/v2/api/purchase/commit                  | 强                                                           | p1                                         | 支付订单事件无法同步biz                        | 191.998                                    | 15         | 81.839    |                  | 203                                                          | 12.787           | 203.043        |           |
| /gulfstream/dive-biz/v2/api/event/update                     | 强                                                           | p1                                         | 无法更新biz扩展信息                            | 94.898                                     | 12         | 60.797    |                  | 202                                                          | 13.605           | 73.774         |           |
| /gulfstream/dive-biz/v2/api/purchase/list                    | 强                                                           | p1                                         | 无法获取购买记录                               | 72161.145                                  | 5274.788   | 35646.146 |                  | 198                                                          | 3.475            | 82214.735      |           |
| /gulfstream/dive-biz/v2/api/event/list                       | 强                                                           | p1                                         | 无法根据ActivityIDs获取购买记录                | 50689.07                                   | 2469.294   | 22953.399 |                  | 144.56                                                       | 3.104            | 59076.181      |           |
| disf!ibt-marvel-dive_honors_g                                | /gulfstream/dive-honors/v3/api/query/bySignal                | 强                                         | p1                                             | 无法查询套餐订单的权益核销情况             | 5011.5     | 1         | 80.993           |                                                              | 33.26            | 13             | 8696.917  |
| disf!ibt-marvel-shopping_api                                 | /gulfstream/shopping-api/item/query/skuInfo/batch            | 强                                         | p1                                             | 无法批量查询物料信息                       | 50785.528  | 2482.294  | 22990.529        |                                                              | 135.48 \| 159    | 7.426          | 59165.241 |
| /gulfstream/shopping-api/item/query/orderInfo/batch          | 强                                                           | p2                                         | 无法批量查询订单                               | 176.862                                    | 20         | 98.574    |                  |                                                              |                  |                |           |
| /gulfstream/shopping-api/item/query/orderInfo/byOutTradeID   | 强                                                           | p3（忽略此接口）                           | 无法通过OutTradeID查询订单                     | --                                         | --         | --        | 两日内无访问数据 |                                                              |                  |                |           |
| /gulfstream/shopping-api/item/order/purchase                 | 强                                                           | p0                                         | 无法创建普通订单v2                             | 47.167                                     | 2          | 23.595    |                  | 326 1061（根据数据曲线图，这个值为异常数据，不采纳，需要报警感知） | 27.968           | 67.531         |           |
| /gulfstream/shopping-api/item/order/cancel                   | 强                                                           | p0                                         | 无法取消订单                                   | 56.2                                       | 1          | 16.649    |                  | 1032.49                                                      | 86.943           | 67.274         |           |
| /gulfstream/shopping-api/item/order/pay                      | 强                                                           | p0                                         | 无法支付普通订单                               | 95.53                                      | 8          | 60.551    |                  | 2000                                                         | 104.439          | 112.75         |           |
| /gulfstream/shopping-api/item/order/purchase/phase           | 强                                                           | p0                                         | 套餐无法随单购买                               | 130.297                                    | 12         | 76.363    |                  | 1213.6                                                       | 98.749           | 72.026         |           |
| disf!ibt-marvel-shopping_quota                               | /gulfstream/shopping-quota/query/batch                       | 强                                         | p1                                             | 无法批量查询购买频次信息                   | 39347.727  | 1499.083  | 17217.657        |                                                              | 122.15           | 2.698          | 46395.668 |
| disf!ibt-os2-os2_activity_control                            | /os2/activity_control/canvas/set_expire_day                  | 强                                         | p2                                             | 无法设置用户活动过期时间                   | 70.143     | 2         | 49.12            |                                                              |                  |                |           |
| /os2/activity_control/canvas/query_fusion                    | 弱（无调用）                                                 | p3（忽略此接口）                           |                                                | --                                         | --         | --        | 两日内无访问数据 |                                                              |                  |                |           |
| disf!ibt-os2-os2_api                                         | /os2_api/activity/redirect                                   | 强                                         | p1                                             | 无法活动推荐                               | 161084.315 | 10689.532 | 83174.184        |                                                              | 254.68           | 21.904         | 190708.12 |
| /os2_api/activity/preview                                    | 强                                                           | p2                                         | 无法根据活动ID获取活动信息                     | 19                                         | 1          | 2.766     |                  |                                                              |                  |                |           |
| /os2_api/routine/get_by_routine_ids                          | 强                                                           | p1                                         | 无法根据routineID获取活动信息                  | 408.092                                    | 119.583    | 294.749   |                  | 227                                                          | 6.109            | 144.651        |           |
| /os2_api/activity/get_by_routine_key                         | 强                                                           | p1                                         | 无法根据routineID获取活动信息                  | 83426.25                                   | 5466.875   | 46458.743 |                  | 475.52                                                       | 9.561            | 97866.843      |           |
| disf!ibt-os2-os2_event                                       | /os2-event/api/outer/standardEvent                           | 强                                         | p1                                             | 活动无法绑定                               | 4064.193   | 357.089   | 2155.794         |                                                              | 250.24           | 53.765         | 4222.518  |
| disf!ibt-pay-cashier                                         | /gulfstream/pay/v1/openapi/queryTrade                        | 强                                         | p3（忽略此接口）                               | 无法查询套餐订单的权益核销情况，影响H5购买 | --         | --        | --               | 两日内无访问数据                                             |                  |                |           |
| /gulfstream/pay/v1/openapi/createOrder                       | 强                                                           | p1                                         | 无法创建代收单                                 | 5                                          | 1          | 1.471     |                  | 502                                                          | 446              | 10             |           |
| /gulfstream/pay/v1/openapi/closeTrade                        | 强                                                           | p1                                         | 无法关闭代收单代收单                           | 1                                          | 1          | 1         |                  | 566                                                          | 566              | 3              |           |
| /gulfstream/pay/v1/cashier/stopCashPay                       | 强                                                           | p1                                         | 无法停止支付 【优惠解冻】                      | 1                                          | 1          | 1         |                  | 211                                                          | 211              | 3              |           |
| disf!ibt-pay-jcashier-wakanda                                | /gulfstream/cashier/wakanda/v1/payments/refund/refund        | 强                                         | p3（忽略此接口）                               | 套餐无法发起退款                           | --         | --        | --               | 两日内无访问数据                                             |                  |                |           |
| disf!ibt-pay-ms-coupon                                       | /foundation/coupon/v1/couponinterface/getAllCoupons          | 强                                         | p1                                             | 无法获取券信息                             | 39352.552  | 2092.5    | 19147.677        |                                                              | 266              | 31.984         | 45393.006 |
| disf!ibt-ring-seg_data_center                                | /psi/seg-data-center/api/v1/feature/query                    | 弱                                         | p2                                             | PDP特征无法查询                            | 7317.375   | 414.234   | 3034.681         |                                                              |                  |                |           |
| disf!ibt-ring-servicecontrol-sc_after_deal                   | /gp/sc-after-deal/api/v1/dFinishTrip/orderComplainedByDriverPDP | 强                                         | p2                                             | 无法查询订单完单后司机是否上报pdp          | 4          | 1         | 1.238            |                                                              |                  |                |           |
| disf!kproxy_vipcenter-kproxy                                 | redis                                                        | 强                                         | p0                                             | 所有缓存依赖                               |            |           |                  |                                                              |                  |                |           |
| disf!map-point-sys-global_poi_proxy                          | /mapapi/gethomeandcompany                                    | 弱（路线查不到不影响主流程，继续后续流程） | p2                                             | 无法预测路线，获取指定类型路线             | 2131.5     | 178.733   | 1268.252         |                                                              |                  |                |           |
| /mapapi/updateCommon                                         | 强                                                           | p2                                         | 无法提交通勤路线起点，路线地址更新(仅用户纬度) | 169.27                                     | 6          | 97.629    |                  |                                                              |                  |                |           |
| disf!sailing-growth-services-benefit_growth                  | /benefitGrowth/getValidMembership                            | 强                                         | p1                                             | 无法查询用户是否已购买会员                 | 14687.417  | 802.083   | 8073.452         |                                                              | 192              | 5.759          | 18214.711 |
| /benefitGrowth/getPurchasableMembership                      | 强                                                           | p1                                         | 无法查询用户可购买的会员                       | 13986.833                                  | 776.25     | 7735.186  |                  | 252.38                                                       | 70.459           | 17432.735      |           |
| /benefitGrowth/getSaleProductByID                            | 强                                                           | p2                                         | 会员无法查询售卖商品详情                       | 13                                         | 1          | 2.29      |                  |                                                              |                  |                |           |
| /benefitGrowth/benefitBuyTry                                 | 强                                                           | p1                                         | 会员权益无法锁定购买                           | 7                                          | 1          | 1.898     |                  | 97                                                           | 36               | 20             |           |
| /benefitGrowth/preSendBenefit                                | 强                                                           | p1                                         | 会员无法预发第一张出行券                       | 7                                          | 1          | 2.256     |                  | 111                                                          | 47               | 20             |           |
| /benefitGrowth/benefitBuyConfirm                             | 强                                                           | p1                                         | 会员无法确认购买                               | 5                                          | 1          | 1.456     |                  | 176                                                          | 76               | 14             |           |
| /benefitGrowth/benefitBuyCancel                              | 强                                                           | p1                                         | 无法取消购买会员权益                           | 4                                          | 1          | 1.435     |                  | 28                                                           | 17.75            | 11             |           |
| /benefitGrowth/withdrawPreSendBenefit                        | 强                                                           | p1                                         | 无法撤回会员预发送的权益                       | 3                                          | 1          | 1.275     |                  | 78                                                           | 78               | 10             |           |
| /benefitGrowth/getValidMemberBenefit                         | 强                                                           | p1                                         | 无法查询已购买的会员权益                       | 2504.444                                   | 201.8      | 1433.949  |                  | 134.38                                                       | 5.064            | 3351.639       |           |
| disf!sec-risk-global-gateway                                 | /sec/risk-gateway/common/global_ride_pass_rights_display     | 弱（风控判定失败时,正常推荐套餐）          | p0                                             | 风控判定                                   | 50524.708  | 2433.475  | 22843.649        |                                                              | 104.7            | 6.571          | 59011.661 |
| /sec/risk-gateway/common/global_ride_pass_h5_create_order    | 弱                                                           | p0                                         | 风控判定                                       | 50                                         | 2          | 25.592    |                  | 78                                                           | 9.102            | 68.842         |           |
| disf!ue_driver_reward                                        | mysql                                                        | 强                                         | p0                                             | 数据库                                     |            |           |                  |                                                              |                  |                |           |
| disf!ue_gs_driver_reward                                     | mysql                                                        | 强                                         | p0                                             | 数据库                                     |            |           |                  |                                                              |                  |                |           |



##  



## 7. 新增uri列表



## atom-api

| **添加时间** | **服务等级** | **接口**       | **功能** | **Max（10s）** | **Min** | **Avg** | **p99** | **平均耗时** | **QPS** |
| ------------ | ------------ | -------------- | -------- | -------------- | ------- | ------- | ------- | ------------ | ------- |
| 2025-12-09   | p1           | /new_dxgy/info |          |                |         |         |         |              |         |
| 2025-12-09   | p1           | /new_dxgy/bind |          |                |         |         |         |              |         |



### atom-app

| **添加时间** | **服务等级** | **接口**                     | **功能**           | **Max（10s）** | **Min** | **Avg** | **p99** | **平均耗时** | **QPS** |
| ------------ | ------------ | ---------------------------- | ------------------ | -------------- | ------- | ------- | ------- | ------------ | ------- |
| 2025-12-09   | p1           | /new_dxgy/api/taskCard       | dxgy任务卡片       |                |         |         |         |              |         |
| 2025-12-09   | p1           | /new_dxgy/api/resource/judge | 新版dxgy资源位接口 |                |         |         |         |              |         |

‍



###  



## 8. 报警组、报警群整理

> 目前将error级别报警也设置为p3故障，主要是为了防止一直发短信



error级别报警：二级报警，显示（P2故障）

warn级别报警：三级报警，显示（P3故障）

| **报警指标**                  | **报警组**                    |
| :---------------------------- | :---------------------------- |
| 错误数量/错误率               | atom错误率和数量报警组_稳定性 |
|                               |                               |
| p99                           | atom-app项目p99报警组_稳定性  |
| atom-api项目p99报警组_稳定性  |                               |
|                               |                               |
| 平均耗时                      | atom-app平均耗时报警组_稳定性 |
| atom-api平均耗时报警组_稳定性 |                               |
|                               |                               |
| QPS                           | atom-app项目QPS报警组_稳定性  |
| atom-api项目QPS报警组_稳定性  |                               |





## 9. 附录



### 9.1 dirpc的Errno含义

| **错误码** | **错误码含义**                                         |
| :--------- | :----------------------------------------------------- |
| 10001      | DiRPC 初始化出现问题                                   |
| 10002      | DiRPC 内部调用后，HTTP 协议的返回 Response 为空        |
| 10003      | DiRPC 内部调用，调用出现异常                           |
| 10004      | 暂时无用                                               |
| 10005      | DiRPC 没能获取到下游的 Client，不再进行重试            |
| 10006      | Thrift调用出现异常                                     |
| 10015      | 连接超时 （>=1.6.0）                                   |
| 10017      | 读写超时 （>=1.6.0）                                   |
| 11012      | DiRPC 发生熔断，具体熔断类型看异常信息                 |
| 11013      | DiRPC 开启 Mock 指定下游后，触发 Mock 将会返回该错误码 |

590 ~ 598

| 59x  | 限流返回 | 限流 | router/inrouter 限流模块设定流量状态dirpc限流使用状态码httpdns goroutine限流598是接入层自保限流596是业务在接入层配置的限流595是mesh返回的限流 |
| ---- | -------- | ---- | ------------------------------------------------------------ |
|      |          |      |                                                              |