# 20260113-上游API反映atom-app服务超时

参考模板：https://cooper.didichuxing.com/docs2/document/2203821043705



# 1、问题背景

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768532274329%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768532428380%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()

0113晚22:27，上游API服务的RD同学找过来，反映atom-app侧边栏接口/pass/api/getSideInfo 10015、10017错误率飙升



# 2、时间轴（北京时间）



## **问题发生**

2025-01-13 22:27（此时上游报警，RD同学找过来，后续排查机器延迟实际发生的时间在21:10左右）



## **问题定位**

2026-01-14 00:42

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768533119785%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()





## **问题修复**

2026-01-14 00:45

对92号机器进行了变更IP漂移



## **问题善后（如需要）**

10.12.82.17 网卡硬件异常，维修中

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768533336798%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768534080529%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()



# 3、影响范围

所有打到atom-app-92号机器的请求都会出现10015、10017的错误返回，上游只有API侧找过来应该是因为侧边栏接口QPS高



## 3.1 订单影响



## 3.2 资损影响



## 3.3 用户体验影响

侧边栏不出会员&套餐（其他所有打到92号机的请求都会报错，影响面不太好评估）



## 3.4 其他影响



# 4、根因分析（需要梳理出架构和链路，错误代码需要贴出来）



# 5、问题反思

问题发生后，排查流程有问题

当时完整的排查流程如下：

1、第一反应：是不是改动引起的？

上线群里排查九点前后的相关上线、Apollo变更记录，发现跟出现问题的接口并不相关

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768535043330%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()

2、排查上游给到的trace

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768535240453%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()

发现atom-app返回了[http.Do](http://http.do/) fail, err=Post "http://10.134.248.98:18001/atom-app/pass/api/getSideInfo": dial tcp 10.134.248.98:18001: i/o timeout（这个时候应该意识到，连接超时导致上游接收到了10015的errno，这个errno不是atom-app服务会返回的，所以应该判断出来请求没走到我们的服务；又：atom-app没有日志也能判断出来，但当时以为是日志采集的问题）

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768535258247%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()

3、查看机器状态

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768535925944%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()

odin查看了机器状态，发现都是running，以为机器没问题，给出了「机器没问题」的结论（实际上就算是running状态也可能出现网络延迟，出现10017（net/http: timeout awaiting response headers，服务器响应超时） & 10015（dial tcp ...: i/o timeout，TCP连接超时）的问题）

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768535935982%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()

4、去查atom-app服务的返回

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768536209098%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()

查看服务的日志，没有发现侧边栏接口有什么异常情况，给出了「目前是正常的」结论（其实这一步已经错了，因为请求并没有打过来，查看服务日志没有意义）

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768536222584%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()

5、查看具体的10017返回有哪些

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768536329312%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768543526800%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()

查看下游服务的10017超时返回（这个时候已经是病急乱投医了，就算下游10017超时，我们atom-app服务返回的errno也应该是500，而不会把10017返回给上游）

6、odin查看接口返回错误率（继续乱投医中，请求都没到服务，所以服务本身的任何指标都无法正确反映出来问题）

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768544159990%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()

AGGR.request.error.counter: 服务的接口返回错误数

by_tags.rpc_dirpc_call.error.counter: 服务下游的接口返回错误数

by_tags.rpc_in.error.counter: rpc入口流量，由mesh上报



![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768545365024%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()

7、最后通过查看by_tags.rpc_in.error.counter指标，定位到是92号机器有问题

![img](https://didoc.didichuxing.com/api/uploadfile?b=didoc2-upload-file-prod&k=images%2F1768550086429%2Fimage.png&filename=image.png&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRJZCI6IjVnQTdCaXhpODk2MXZ4MW1ubFVNQlp1cEZOYXJZQW5KIiwidXNlcm5hbWUiOiJsdW9iaWFvX2kiLCJkb2NJZCI6IjcwMzY4NzQ0ODYwMTA1IiwiaWF0IjoxNzcxNDA1NDA1LCJleHAiOjE3NzIwMTAyMDV9.DSMga2eQw5gdRuZQX6kTLWeECCfD_whf0rsFC3JUu1Y&x-s3-process=image%2Fformat%2Cwebp)![img]()



# 6、待改进项