---
layout: page
title: 开源调研
date: 2019-05-13
categories: 调研
tags: open-source
---



**了解各大厂开源趋势、开源目的及逻辑。部分解答了系统平台如何做商业化。**



# 1. 开源的逻辑

没有哪家公司是为了开源而开源，背后都有其商业思考。本篇通过一系列案例来阐述开源的逻辑。



### Q：对Google而言，Chrome的战略地位是什么？

web时代，桌面应用正在逐步进化成Gmail和谷歌地图等网页应用，浏览器几乎可以取代操作系统担当起存储、浏览、编撰、聊天、电子商务等等各种任务。（除了大象、微信、IDE、office办公软件，我几乎不使用mac上的app）

周鸿祎对Chrome战略价值的分析：在我看来，Chrome对于谷歌来说，绝不是一个浏览器那么简单，而是一个**打着“浏览器”旗号的“网络开发平台”**。其真正的目的是要打破基于Windows API的微软‘生态系统’的平衡，从而**使开发者和普通用户逐渐不再依赖微软操作系统**，最终实现颠覆微软的长期目标。

Google官方公开的数字：Chrome 在桌面和移动端的**活跃装机总量超过了 20 亿**。**市场占有率60%+**（[netmarketshare](https://www.netmarketshare.com/welcome)  2019/02数据 desktop/laptop 66.89%；Mobile 63.32%）



### Q：这么重要的产品，Chrome为什么要开源？

开源项目叫Chromium（https://github.com/chromium/chromium）

**（1）借助社区力量，让项目快速发展&普及**

- **收获更多技术创新idea：**Chromium上有诸如 DNS Prefetch、SPDY、QUIC、预渲染、多进程架构、PPAPI、v8 JavaScript 引擎等等高门槛的技术创新，并非全都来自google内部。 Chrome之上的扩展生态就更加活跃了，idea层出不穷。
- **迭代速度更快、bugfix更及时：**Chromium浏览器代码规模有2400万行（Windows操作系统大约5000万行左右）。只靠google工程师闭源迭代速度要慢不少，且不能覆盖更多bug场景，修复速度也会慢不少。（侧面数据印证：国内360、UC、QQ、搜狗这四家双核浏览器普遍能做到一年至少两次升核，而Google三个月发布一个大版本，半年就能做一次全网升级。）
- 节省研发成本(也许)**：**最多时候google调动超过1000个RD开发Chromium，一年的研发成本将近3亿美金。

开源确实让Chromium越来越好。

**（2）有更大话语权来影响行业标准**

浏览器需要遵守W3C、CABForm、RFC等规范。用户广、市场大的公司更有话语权，也更有能力让标准真正生效落地。

为什么成为行业标准很重要？仍旧是个故事：微软07年向ISO提交了自己的相关标准Office Open XML即OOXML 并力推其成为新的国际文档，遭到了来自中国软件产业界的联合抵制。原因是一旦OOXML成为标准，标准国产Office软件就会被迫去兼容封闭的文档格式，兼容的研发成本巨大，更重要的是这些软件的“迁移成本”护城河不见了，中国办公软件的市场空间将会面临空前压力。

**（3）开源项目也可以用来给自己的服务引流**

Chromium一直处于 Google 的影响之下，许多功能都包括了 Google 特有的服务。比如说，Chromium 预配置的搜索引擎是 Google、使用 Google 的 Safe Browsing API 来扫描每个你访问的 URL 的安全性，而且加载了一大堆的 Google 的二进制程序来提供各种内部服务。这意味着，许多用户数据仍然会被发往 Google 的服务器。



### Q：为什么Chrome开源出去了，但仍能占领市场，没被其他公司踩在chromium肩膀上抢占市场份额？？

国内的双核浏览器都是无一例外地基于 Chromium 开源项目做二次开发，国外的也是多得一匹，参考[《9 Alternative Chromium Browsers That Beat Chrome at Its Own Game》](https://www.makeuseof.com/tag/alternative-chromium-browsers/)

Chrome就不怕这些浏览器抢占Chrome的市场份额么？ 当核心功能和体验都很一样，抢占市场份额的关键因素是什么？ （体量？渠道导流能力？）

尤其像微软都放弃了自己的闭源浏览器Edge转而投向Chromium的怀抱，按微软的toC操作系统87.4%的占有率，预装自家浏览器会不会很快就能抢占掉Chrome的市场份额了？（虽然2018年全球智能手机出货量14亿部，全球个人电脑出货量2.6亿台）



### Q：看个反面故事，Google对宣称开源的Android掌控力很强

2007 年 6 月末，第一代 iPhone 发布。2007 年 11 月，Google 将 Android 开源。坊间传闻是为了对抗苹果。此后市场占有率一路走高。

18年7月欧盟对Google发起垄断诉讼，处以43亿欧元罚款，创下欧盟史上反垄断罚款纪录。

开源项目怎么能做到垄断，这是教程：

**（1）服务大礼包**

Android 是一个开源的操作系统，理论上手机厂商完全可以不安装 Google 的服务，但事实并非如此。

以GMS(Google 移动服务)为例，大量的系统级接口被放在它的 Play Service 里面，比如游戏对战、地图、语音控制。如果手机厂商不装它，第三方 Android 应用就可能完全无法使用。

且这20+的服务之间彼此依赖，部分安装是不可能的，比如为了用地图、商店，就不能拿掉 YouTube 或者搜索。

这些服务确保了手机上的软件、游戏和广告收入都掌握在 Google 手中。（中国市场特殊）

**（2）控制手机厂商**

Google 推出了一个 OHA（开放手机联盟），加入 OHA 的手机厂商将得到 Google App 更多的授权，但必须得签署一份协议：禁止构建非 Google 认证的设备（禁止和模仿 Android 操作系统的企业合作）。

2012 年时，宏碁想在中国生产运行阿里巴巴的 Aliyun OS 的设备时，宏碁就收到谷歌的通知，若新产品上搭载阿里云操作系统，Google 将会解除与其 Android 产品的合作和相关技术授权，后来手机发布会也被迫取消了。

**（3）部分开源**

Android分为两个部分：开放的部分是 Android 开源项目AOSP，它是 Android 的基础；封闭的部分是 Google 旗下的应用程序，参考GMS，这也是系统接口的一部分。



### **Q：特斯拉为啥开放专利？促进新兴市场快速崛起的新玩法**

虽然不是传统意义上的开源，背后的逻辑也很值得聊聊。

2014年6马斯克宣布将免费公开特斯拉所有专利（“将不会对那些善意使用我们技术的人提起专利诉讼”），目的是推动电动汽车技术的进步（“通过特斯拉，其他电动汽车制造商以及整个世界都将受益于一个共同而快速发展的技术平台）。

**（1）做大市场**

全球每年增长接近1亿辆新车，电动车或不使用燃油的汽车占比达不到**1%**。达不到规模效应，已有玩家成本居高不下，新玩家不愿入行，规模更难增大。

**（2）行业标准**

目前电动车市场上，电池以及充电方面尚未形成国际统一标准。特斯拉推崇使用小型电池组以及建设超级充电站为电动车充电，但是在电动车制造和使用的核心环节上，特斯拉的做法和标准并不是行业主流，这也是特斯拉在中国遭遇充电网络推进困难的主要原因。

在没有统一标准的背景下，开放自己的标准吸引更多的企业共享，一旦形成规模，越来越多的企业加入到特斯拉所谓的“共同而快速发展的平台”，也就大大提高了特斯拉标准的通用性。

（据Elon Musk本人说，博客发表不到一周的时间，他已经与宝马公司的高管进行了会面，讨论了潜在合作方式，其中一个就是在超级充电站上的合作。有海外媒体报道，日产也有意与特斯拉进行谈判，合作建设充电网络。）

专利能打造技术壁垒、保护大公司的地位，在新兴市场用处不大，反而会制约市场发展。



### **Q：蚂蚁金服开源出来的项目并不触及其业务核心，开源目的是否有些不同？**

Android和Chrome都是核心平台，能各自承载一个生态，是google版图的重要组成部分。

比较下来，蚂蚁金服的核心竞争力并不在前端、ServiceMesh、容器化（很重要但不是核心），那蚂蚁开源的原因是否会有些不同。

不能脱离公司大背景看开源，先看下蚂蚁的盈利模式：蚂蚁有9个业务板块，据说核心金融业务面临的监管压力日益增加，公司在往技术服务方向加大投入，涉及到帮助银行和其他机构提供在线风险管理和欺诈预防等服务。 据说2017年技术服务已经占到蚂蚁金服收入的34%，五年内要做到65%。 参考[《想让技术服务贡献六成营收，蚂蚁金服为何如此看重技术输出？》](https://www.huxiu.com/article/247305.html?f=member_article)

**（1）金融服务的特殊要求**

所服务的金融机构对监管和自主可控的要求更多

**（2）市场还在成长，通过开源触达覆盖更多金融服务场景**

依然存在蚂蚁没有覆盖到的金融服务场景，将技术开源可以供更多的客户应用到其自身的场景下，补充了蚂蚁金服的技术应用面。

- 不怕其他公司利用其开源项目**抢占市场份额**么？？至少目前开源的不是业务核心，而且金融云等门槛也高，蚂蚁有规模优势。

- 能起到促进市场成长，甚至**影响行业标准**的作用么？不好说。目前开出来的项目虽然是在金融场景下锤炼出来的，但不是金融服务场景下的核心系统，不太能影响到金融服务的行业标准（类比octo开源，能有利于O2O行业标准的制定么？）

**（3）其他常规好处**

- 若其组件已在线下被各个公司广泛采用，则迁移到蚂蚁金融云将变得非常容易。

- 借助社区力量快速迭代。越是复杂的大项目，靠一家公司闭源搞定就越困难。

- 在开源项目对应的领域有制定标准的话语权。

  

### **Q：除了站在公司角度分析开源，其他参与方的收益与成本？**

- **有助于技术招聘**。顶级人才很看重技术品牌。开源基金会CNCF技术监督委员会的首位华人委员会李响："在国外做云的、做底层架构的华人工程师，如果要考虑加入国内公司，首选阿里。" 
- **有利于研发人才培养**。RD提高技术能力，结交朋友，获得成就感，调度积极性等等。像代码规范什么的，参与两次开源项目自然而然就学会了，比反复强调还有用。
- **需投入额外人力。**蚂蚁总监杨冰：“将现有场景的代码开源，至少需要在已经运行稳定、结构清晰的现有代码基础上多付出 30% 的技术投入，才能做到初步的代码开源；而持续维护代码、多版本代码保持同步、维护开源社区等，多付出的技术成本还远远不止这些。”； 其实对公司而言，闭源项目在满足业务需求后确实没有继续迭代的必要了，开源项目需要持续投入。
- **有安全隐患**。梳理代码、安全扫描等手段无法检查出所有的敏感信息和安全漏洞，正在使用的核心系统开源出去，有被攻击的风险。长期看是有收益的，能有更多人帮忙捉虫并贡献bugfix，但有短期风险。
- ……



### Finally：一个不走心的总结

**后起者为了快速追赶而开源核心；**

**领先者开源非核心项目广泛收集小白鼠和临时工；**

**成熟市场抢占制定标准的话语权；**

**蓝海市场的高能玩家通过开源推进市场快速成熟。**



# 2. 各大厂开源现状

## 2.1. 阿里

阿里研发投入在中国企业中名列第二，2018年总投入29.14欧元（研发占销售额比例9.1%）。在开源方面也是国内标杆，阿里团队的活跃开源项目有150+，github star数全球排名第12位。

### 2.1.1. 开源运作

**开源委员会**

2011年阿里成立开源委员会，专门管理项目的开源流程。（据说成立的时候只有章文嵩带队的十几人，也要回答能给公司带来什么好处，给出的也是提升项目质量+招聘等常规答案。当时公司提出的唯一要求是不影响公司的核心竞争力即可，管理很宽松）

将公司的一个产品开源，需直接主管同意，要BU总监同意，提报开源委员会。开源委员会会对许可证的选择、代码测试的程度、文档的程度给出建议并进行备案。在对外开放的过程中，还有法务团队的审核，有安全团队负责审核代码开源后是否会被他人滥用，对现有系统发起攻击等。

**开源的目的（官方说法）**

- 任何一项技术产品，如果能得到全球化社区里诸多场景的验证和贡献，通过社会化开发来演进，都是这项技术能够快速发展和普及的关键推手。而开源社区极强的互动性、复用性，一方面有效避免了技术被锁死，另一方面提高了知识的效用，这种“众创”的方式，更容易带来业务和技术上的价值与创新，这也正是阿里希望通过开源达到的核心目的。
- 建立技术影响力，汇聚更多人才。

**开源的四个阶段**

本部分内容整理自：[阿里开源：思考，演进和发展](https://yq.aliyun.com/articles/59141)。唐容，2011年加入阿里，2016年开始负责阿里开源相关的工作。



### 2.1.2. 阿里开源项目

项目汇总：[阿里巴巴重要开源项目汇总](https://yq.aliyun.com/articles/676140)

| 项目                                                         | star  | fork  | commits | contributor |                                                              |
| ------------------------------------------------------------ | ----- | ----- | ------- | ----------- | ------------------------------------------------------------ |
| [**incubator-dubbo**](https://github.com/apache/incubator-dubbo) | 25541 | 17040 | 3305    | 188         |                                                              |
| [**druid**](https://github.com/alibaba/druid)                | 16007 | 5765  | 5941    | 105         | 数据库连接池                                                 |
| [**fastjson**](https://github.com/alibaba/fastjson)          | 17214 | 4787  | 3065    | 82          | Java 的 JSON 解析器和生成器                                  |
| [**p3c**](https://github.com/alibaba/p3c)                    | 14686 | 3274  | 226     | 25          | 代码标准                                                     |
| [**arthas**](https://github.com/alibaba/arthas)              | 11752 | 2263  | 607     | 47          | Java诊断工具                                                 |
| [**rocketmq**](https://github.com/apache/rocketmq)           | 7340  | 3664  | 1021    | 140         | a distributed messaging and streaming platform with low latency, high performance and reliability, trillion-level capacity and flexible scalability |
| [**openmessaging**](https://github.com/openmessaging)        | 577   | 169   | 314     | 8           | A vendor-neutral open standard for distributed messaging and streaming |
| [**pouch**](https://github.com/alibaba/pouch)                | 3811  | 896   | 2860    | 101         | An Efficient Enterprise-class Rich Container Engine          |
| [**sofa-boot**](https://github.com/alipay/sofa-boot)         | 2716  | 701   | 202     | 12          | 蚂蚁金服基于 Spring Boot 的研发框架，它在 Spring Boot 的基础上，提供了诸如 Readiness Check，类隔离，日志空间隔离等等能力。其他SOFA全家桶：基于多维度 Metrics 的系统度量和监控中间件 SOFALookout基于 Spring Boot 的研发框架 SOFABoot轻量级 Java 类隔离容器 SOFAArk分布式链路追踪中间件 SOFATracer高性能 Java RPC 框架 SOFARPC基于 Netty 的网络通信框架 SOFABolt |
| [**spring-cloud-alibaba**](https://github.com/spring-cloud-incubator/spring-cloud-alibaba) | 4855  | 1187  | 799     | 26          | Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。  |
| [**tair**](https://github.com/alibaba/tair)                  | 1156  | 462   | 289     | 2           | A distributed key-value storage system                       |
| [**tengine**](https://github.com/alibaba/tengine)            | 8220  | 1983  | 1390    | 67          | based on the [Nginx](http://nginx.org/) HTTP server and has many advanced features. |
| [**ant-design**](https://github.com/ant-design/ant-design)   | 44639 | 15588 | 14394   | 796         | 一套企业级的 UI 设计语言和 React 实现                        |
| [**incubator-weex**](https://github.com/apache/incubator-weex) | 12124 | 1592  | 11248   | 182         | A framework for building Mobile cross-platform UI            |
| [**ice**](https://github.com/alibaba/ice)                    | 12071 | 1319  | 2377    | 34          | 海量高质量物料；iceworks桌面工具；                           |
| [**egg**](https://github.com/eggjs/egg)                      | 12167 | 1193  | 838     | 139         | 企业级 Node.js 框架                                          |

[﻿](https://github.com/alibaba/fastjson)[﻿](https://github.com/alibaba/dubbo)

### 2.1.3. RocketMQ & OpenMessaging案例

2011年开始自主研发RocketMQ消息中间件，支撑阿里内部3000+应用，双十一当天达到万亿级消息量，峰值TPS几千万。17年9月捐献给社区，成为 Apache 顶级项目。

#### 如何发挥开源和商业的协同效应？

以开源为核，商业为辅的形式。开源分布式消息所有核心特性，借助社区力量把核心建扎实；在商业层面，侧重云支持、运维管控、安全授权、深度培训等。

#### **国际标准+“囊括一切”的庞大生态**

[解读OpenMessaging开源项目，阿里巴巴发起首个分布式消息领域的国际标准](http://jm.taobao.org/2017/10/18/20171018/)

这套标准已经进入 Linux 基金会，接下来会进入到 CNCF，希望成为云计算的事实标准。

制定标准需要有诸多考虑。负载均衡，在拉模式和推模式下策略会有所不同，消息本身的 sharding 也会因业务场景不同而不同。分布式追踪，主要是考虑 Linux CNCF 中的 opentracing。协议桥接，主要是考虑如何和现有的标准，如 JMS 进行无缝桥接。流计算，通过引入流计算算子，在消息投递过程中针对内容进行计算。Benchmark，类似 SPECJMS，把所有 Messaging Engine 拉到同一基调上做性能测试。）

围绕着 OpenMessaging 标准而努力打造的生态体系：



#### 开源改造、社区运营，是个大工程

开源改造：

- 代码及文档改造：重新设计编排了文档；全面拥抱 UTF-8；重写 API JavaDoc；清理代码，优化代码结构……
- 规范：设计了新的代码管理分支模型，以满足开源、商业化、集团内部和公有云、私有云共用一个 RocketMQ 内核的需求；通过 PR Checklist, checkstyle 和持续集成来自动化校验……

社区运营：可参考 [捐赠历程](http://jm.taobao.org/2016/12/08/little-known-stories-about-apache-rocketmq/) [阿里RocketMQ是怎样孵化成Apache顶级项目的？](http://jm.taobao.org/2017/11/03/‘20171103’/)  

- 社区不会凭空产生，不是把项目往 Github 上一丢就是开源，需要长期经营。（RocketMQ 社区的第一个国际贡献者，团队跟进、鼓励了近 2 个月）

- 面向社区举办编程马拉松，PMC 成员全程跟进，帮助参赛选手评审设计、Review 代码

- 在云栖社区，CSDN，InfoQ 中文站、国际站发表了多篇技术文章，撰写国际论文，在 ApacheCon、LinuxCon 等国际顶级开源峰会发表主题演讲，全球图文同步直播 Meetup

  

## 2.2. Google

成立于1998年9月4日。Alphabet 2018 全年总营收 1368.19 亿美元，净利 263.21 亿美元（近90%来自于广告）；研发支出 214.19 亿美元；现员工10 万人，2018 年新增近 2 万人，主要投入在云业务。

Chris DiBona于2004年加入谷歌任开源总监，创建了谷歌**OSPO（Open Source Program Office）**并开始推动谷歌完善开源战略，带领一个团队专门负责帮助谷歌员工如何发布开源项目，贡献代码，以及确保所有团队和产品满足开源许可协议的要求。（https://opensource.google.com/）

Google的开源目的有些多源：并非纯技术驱动，能看到很多商业策略在里面，如第1部分介绍的Android、以及下面案例中的利用开源做云；也并非纯商业驱动，《How Google Works》里描述的“不依赖商业战略，而依赖技术洞见”真实存在着。



### 2.2.1. 谷歌开源项目

|                                                              |                                                         |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------- | ------------------------------------------------------------ |
| [**Android**](https://source.android.com/index.html)         | 代码没有托管在github上，对应topic下有 **50,699 个repo** | a software stack for mobile devices that includes an operating system, middleware and key applications. |
| [**Chromium**](https://www.chromium.org/)                    | 只在github上建了mirror                                  | a project encompassing [**Chromium**](http://www.chromium.org/Home), the software behind Google Chrome, and [**Chromium OS**](http://www.chromium.org/chromium-os), the software behind Google Chrome OSdc devices. |
| [**TensorFlow**](https://www.tensorflow.org/)                |                                                         | a library for numerical computation using data flow graphics with support for scalable machine learning across platforms from data centers to embedded devices. |
| [**Kubernetes**](http://kubernetes.io/)                      |                                                         | a system for automating deployment, operations and scaling of containerized applications. |
| [**go**](https://github.com/golang/go)                       |                                                         | github上的是mirror https://go.googlesource.com/goGo is an open source programming language that makes it easy to build simple, reliable, and efficient software. |
| [**istio**](https://github.com/istio/istio)                  |                                                         | I由谷歌、IBM 与 Lyft 共同开发的开源项目An open platform to connect, manage, and secure microservices. |
| [**Angular**](https://angular.io/)                           |                                                         | a web application framework for JavaScript and Dart focused on developer productivity, speed and testability. |
| [**material-design-icons**](https://github.com/google/material-design-icons) |                                                         | Material design icons is the official [icon set](https://www.google.com/design/spec/style/icons.html#icons-system-icons) from Google |
| [**material-design-lite**](https://github.com/google/material-design-lite) |                                                         | 网站前端开发工具组。MDL 可以优化跨设备的使用体验，可以在旧版的浏览器进行平滑的切换，提供非常快速的访问体验。 |
| [**guava**](https://github.com/google/guava)                 |                                                         | Guava is a set of core libraries that includes new collection types, immutable collections, hashing, primitives, reflection, string processing, and much more! |



### 2.2.2. Kubernetes & CNCF 案例

#### **在云计算市场上，Google属于后起追赶者**

g

上图为云计算市场份额，AWS 34%、MS Azure 15%、Google 7%、IBM 7%、Ali 4%

云服务的特点是，当开始大量使用的某个云服务商的话，如果要迁移到另一个，就会有巨大的成本。因为公司内部已经写了大量的代码，工具，以及培训了大量员工，去适配云本身。正常公司不可能大规模切换，最多使用混合云的思路。

谷歌云能拼的是增量，也就是新的公司，新的领域（如机器学习，谷歌靠tensorflow、TPU、AI Service），已有领域的新技术方向（如容器）。



#### **为什么做云需要开源**

（此段内容来自Kubernetes负责人Dawn Chen）

很多开源项目都建立在谷歌的论文上，但是却又跟Google本身的系统不兼容。谷歌觉得这一点很难办，如果只是发表论文，而不去做一个真实的系统的话，问题是很难解决的。就算开源API，但是没有实现，甚至没有示例实现，那么API就是不成立的。综上所述，我们觉得谷歌应该重新检视下关于是否开源的决定。并不是说谷歌不愿意开源，否则它也不会去发表论文，最重要的问题在于开源需要太多的人力和物力了。
但是反过来说，如果真的要在云上面做文章，我们是没有办法不开源的。容器技术如果不开源的话，我们就应该做到让用户完全信任容器技术，能够不担心安全问题。但是我们做不到，用户会很担心：程序到底在哪里运行？他们会不会偷了我的东西？因为容器技术还没有得到大家的广泛认可，尤其在安全性能方面。



#### **Kubernetes**

Kubernetes已经成为被最广泛使用的编排工具。

- Kubernetes深度借鉴了Brog。Borg经累积了15年的深耕细作的发展和生产实践，Gmail、YouTobe、Google Search和其他流行的谷歌服务都是用此工具作为基础框架管理。

- Kubernetes从一开始就是以一个生态系统为目的而设计的。

- Kubernetes社区增长迅猛，1000+贡献者，34000+提交，远远超过了其他像Mesos竞争对手的五倍之多。

项目运营更是google擅长的：

- Kubernetes 快速与各大厂商建立起了合作；
- 借助CNCF，正推进成为行业标准；
- 全家桶战略，一系列开源软件强绑定，构建庞大生态。



#### **云原生基金会CNCF 是不是Google推标准的新手段？**

为了**统一云计算接口和相关标准**，2015 年 7 月隶属于 Linux 基金会的云原生计算基金会（CNCF，Cloud Native Computing Foundation）应运而生。 

概括地讲 CNCF 的使命包括以下三点，这也是CNCF 最初对云原生特征的定义：容器化包装；通过中心编排系统的动态资源管理；面向微服务。

CNCF托管了众多著名项目，彼此之间互相补充，覆盖了云原生的各个领域。

- – **Kubernetes** ：跨主机集群的自动部署、扩展以及运行应用程序容器的平台
- – **Prometheus** ：系统监控报警框架。启发于 Google 的 borgmon，由工作在 SoundCloud 的 google 前员工在 2012 年创建。
- – **OpenTracing**：中立的（厂商无关、平台无关）分布式追踪的 API 规范
- – **Fluentd**：数据收集器，可以连接各种数据源和数据输出组件。
- – **Linkerd**：为微服务提供可靠性支持、自动化负载均衡、服务发现和运行时可恢复性的Service Mesh
- – **gRPC**：高性能远程调用框架；
- – **CoreDNS**：快速灵活的构建 DNS 服务。Our goal is to make CoreDNS the cloud-native DNS server and service discovery solution.
- – **containerd**：将容器运行时及其管理功能从 Docker Daemon 剥离的镜像管理和容器执行技术；
- – **rkt**：帮助开发者打包应用和依赖包，简化搭环境等部署工作，提高容器安全性和易用性的容器引擎。

CNCF会员列表：https://github.com/cncf/landscape/blob/master/members.yml，白金会员有： alibaba-cloud、amazon-web-services、cisco、dell-emc、fujitsu、google、huawei、ibm、intel、jd-com、microsoft、oracle、pivotal、red-hat、samsung-sds、sap、vmware

CNCF全景图见：https://github.com/cncf/landscape 。图中还有很多非CNCF项目。StrumDocker Swarm、Mesos、Nomad和亚马逊EC2 Container Service都在，看起来非常公正。

**Google 在所有 CNCF 项目中的所有贡献占近 53%，而第二大贡献者的红帽仅占 7.4%。CNCF主页能明显看出来k8s有多重要：**https://www.cncf.io/



## 2.3. Microsoft

2011年成立了 微软开放技术 (MS Open Technologies) 子公司，算是开始了开源尝试。

2014年开始系统性运作，成立**开源计划办公室**，决定让开源在整个公司中普及, 并将开源融入主要的工程团队。 

（2014年也是納德拉上任的第一年，调整了公司战略。1、windows不再是微软的核心增长引擎，而是作为免费平台培养生态； 2、战略调整为 “mobile first, cloud first”，AWS面向不太复杂的中小型企业，而Azure面向大型企业提供混合云服务。   from《刷新》）

微软并没有一个中央开源战略或一个中央审批机构。取而代之的是，开源计划办公室促进了整个公司的讨论和决策。各个团队仍需要对其开源参与进行审查, 但大多数是在该团队里进行。

- 给开源团队充分授权：owner才充分理解在生态系统方面向什么方向驱动,希望技术互动如何运作
- 有中央政策（如如何管理 IP、安全要求等）并有工具和流程做保障

开源项目管理工具：从GHTorrent fork出GHCrawler来追踪项目（[The Open Source Program at Microsoft: How Open Source Thrives](https://github.com/todogroup/guides/blob/master/casestudies/microsoft.md)）

### 2.3.1. 微软开源项目

2016 年，根据 [GitHub 公布的数据](https://link.zhihu.com/?target=https%3A//techcrunch.com/2016/02/17/microsoft-brings-red-hat-enterprise-linux-to-azure/)，微软一共贡献了 16419 个开源项目，超过 Facebook 成为该开源社区中最大的贡献者。



# 3. 开源商业化

## 3.1. 盈利模式

**模式看下来差不多都是：**

- **开源软件+收费硬件**

- **开源软件+服务和咨询（培训、证书、如补丁和SLA支持等维护）**

- **开源社区版+收费企业版（包含定制化解决方案etc）**

- **开源软件+云（提供云上的版本，很多功能是企业级的）**

盈利能力数据找到的偏少，但看起来除了RedHat，其他公司规模和收入都不够高。

当然还有另一个“盈利模式”：被买。

## 3.2. 构建在开源项目之上的营利性公司

### 3.2.1. Red Hat

员工数1W人，年收入24亿美元。链接：https://www.redhat.com/en

红帽在1993年成立，1999年IPO，但是其实一直在盈利模式上没有很好的突破。直到2003年将自己的商业版和社区版的品牌分离。

2003年，红帽停止发行RedHat Linux 9，而是拆出了两个新项目：企业收费版 RedHat Enterprise Linux 2.1 ，和社区发行版Fedora。

目前的版本是Fedora 25，主要的版本有三个：工作站、云平台、服务器。Fedora 的logo和归属权是属于红帽公司的，但Fedora有着非常成熟的社区化运营和治理，不仅就操作系统而言有明确但项目分工，而且还有特别兴趣小组（SIG，Kubernetes 社区也模仿了此举措）这样的创新团队，成为RHEL重要的软件创新源头。

（在2003的红帽发行版重构品牌之时，也有不服的人站出来直接发展出项目CentOS。后来在2014年红帽将CentOS收购）

### 3.2.2. Docker

Docker第一个版本发布于2013年，发展迅速，很快成为了Linux容器的生态事实上的标准。很多公司利用Docker赚了钱，如国内很多基于容器的云公司，如红帽的OpenShiftV3的PaaS平台（Redhat自己宣传Docker+kubernetes=Openshift），如Mesosphere，以及公有云AWS、Azure、GCE等。然而Docker公司自己不赚钱。  历史故事这个文章讲得很全：https://juejin.im/entry/5c0615b06fb9a04a102f0cab

2017年Docker公司把原Docker项目改名成Moby，并推出了两个产品 Docker CE(社区免费版)和Docker EE(企业收费版)，对这一做法褒贬不一，这个文章有很具体客观的阐述：https://infoq.cn/article/2017/05/Moby-LinuxKit-Docker

### 3.2.3. Cloudera

是Hadoop生态下体量最大、估值最高的公司。2014年，Intel斥7.4亿美金巨资收购Cloudera约18%股份，使得Cloudera估值达到41亿美金，成为估值第二高的大数据公司，仅次于Palantir。2017年4月Cloudera登陆纳斯达克市场，IPO价格只有19亿美金。

Cloudera的Hadoop发行版CDH是完全开源的。核心产品完全开放，Cloudera如何盈利呢？

- 产品安装部署仅仅是开始，使用产品过程中会碰到各式各样的问题，需要有专门人员帮助其解决问题，Cloudera的收入来源就是帮助这些企业客户解决他们日常使用产品过程中的问题，收费方式是以**订阅形式，按年付费**。

- 除了提供Hadoop发行版CDH外，还开发了一款名为Cloudera Manager的**付费产品**。这个产品的作用是帮助大型企业管理其Hadoop集群，提升运维能力。

还有另一家类似的hadoop生态下的公司 Hortonworks

### 3.2.4. Black Duck Software

员工超过320人。链接：[https://www.blackducksoftware.com](https://www.blackducksoftware.com/)

**即使开源项目不在自己手里，也能用来赚钱的公司。**

其付费产品能让开源软件更容易被使用。它提供三大项目：Hub，用于识别和管理环境中使用的开源软件；Protex，用于确保符合开源许可证和公司政策；以及Security Checker，用于识别开源软件中的安全漏洞。它有2000多个客户，包括英特尔、任天堂、SAP和三星。

### 3.2.5. GitHub

GitHub创立于2008年，被Forrester称作“**面向开发者的Facebook**”。

GitHub被收购前，融资总额已达到了3.5亿美元，最后一轮融资后GitHub的估值约为20亿美元。2018年6月，被微软以75亿美元全面收购。

盈利模式有面向个人和企业的增值服务，以及商业版本Github Enterprise （暴雪、Paypal 都在用）

individuals  $7 per month

teams $9 per user per month

微软收购github的原因（非官方）：

- 微软将自己定位为开放源代码拥护者的一系列举措中的最新一项。近年来，微软已开始在其云平台Azure上支持、甚至使用Linux。所有这些都是为了帮助吸引开发者到Azure，收购GitHub的决定正好符合这种模式。
- GitHub拥有大量关于开发人员及其项目的数据，也许这些数据可以与微软2016年收购的LinkedIn整合，以帮助LinkedIn的招聘客户发掘编程人才。

### 3.2.6. Canonical

伦敦公司，员工数量超过550人，年收入1亿美元。链接：[https://www.canonical.com](https://www.canonical.com/)  老板的主页：https://www.markshuttleworth.com/

这家公司开发的Ubuntu是全球最流行的Linux发行版之一。该公司声称Ubuntu是“全球公共云和OpenStack云上最流行的操作系统。”

**据说收入主要来自云服务**，openstack上运行最多的系统就是Ubuntu服务器版。其他方面的盈利较少，包括技术支持、商业服务。

（2008年，Canonical CEO 兼 Ubuntu创始人Mark Shuttleworth说过：“I don’t think anyone can make money from the Linux desktop.”）

### 3.2.7. Nginx

员工不到250人。链接：[https://www.nginx.com](https://www.nginx.com/)

据官方网站称，它支撑“全球一半最忙碌的网站和应用。” 该公司提供支持版的开源软件，客户包括迪士尼、AT&T、时代华纳有线电视、美国银行、职业棒球大联盟、WordPress和韦里逊。

### 3.2.8. CloudBees

规模不到500人；链接：[https://www.cloudbees.com](https://www.cloudbees.com/)

基于开源软件Jenkins的开发运营平台CloudBees于18年完成了6200万美元C轮[融资](http://www.kejilie.com/channel/rongzi.html)，这其中包含了Delta-v Capital领投的3700万美元股权融资和来自Golub Capital旗下Late Stage Lending的2500万美元发展融资。[经纬创投](http://www.kejilie.com/channel/jingweichuangtou.html)、光速创投、Unusual Ventures和Verizon Ventures等现有[投资人](http://www.kejilie.com/channel/touziren.html)亦有参投。 

CloudBees的客户包含了46家财富100强的企业，其中前10强有3家。 

开源的Jenkins自动服务器组成了CloudBees产品线的核心（同时它还会为Jenkins提供训练和验证）。和其它开源公司类似，CloudBees也一直在通过新的**企业功能**来丰富Jenkins，并将这些功能捆绑到不同的产品中。最近收购了CodeShip，如今公司推出了供新的持续集成及交付平台，该平台可以提供托管服务，这和Jenkins没有太密切的关系。 

### 3.2.9. Databricks

员工数量估计不到200人。链接：[https://databricks.com](https://databricks.com/)

Databricks支持数据流项目：Apache Spark。创建该项目的开发人员在2013年创办了Databricks，为该项目提供商业支持。知名的Databricks客户包括NBC环球、惠普、壳牌、思科及3M等。云上版本也是收费的。

### 3.2.10. DataStax

员工数量超过400人。链接：[https://www.datastax.com](https://www.datastax.com/)

出售基于Cassandra的托管云解决方案，还提供商业支持版的Apache Cassandra NoSQL数据库。它声称拥有50多个国家的500多个客户。使用其产品的知名公司包括：奈飞、西夫韦、Adobe、Intuit和eBay。



## 3.3. 收购

被买，也是一种盈利。

| 2019-03-12 | F5 收购开源服务器 Nginx                                      | 6.7 亿美元 |
| ---------- | ------------------------------------------------------------ | ---------- |
| 2018-10-29 | IBM收购Red Hat                                               | 340亿美元  |
| 2018-07-02 | 瑞典私募股权公司EQT收购SUSE Linux                            | 25亿美元   |
| 2018-06-04 | 微软收购GitHub                                               | 75 亿美元  |
| 2016       | 微软欲收购Docker，未果                                       | 40亿美元   |
| ……         |                                                              |            |
| 2009       | VMware收购Spring Source                                      | 4.2亿美元  |
| 2008       | Sun 收购MySQL                                                | 10亿美元   |
| 1999       | Red Hat 收购Cygnus Solutions                                 | 6.75亿美元 |
| 1999       | 比尔盖茨曾如此评价 Linux 的:“确实我们承认在学生和爱好者当中我们不如 Linux,但是我们从多个角度考察过,并不认为它能在商业市场上有何作为。” |            |



### Q：RedHat为啥值340亿美元？

1995 年红帽公司的营收达到 48 万美元，第二年就翻了近一倍，第三年再次翻倍。1999 年上市时公司年营收已经超过 1000 万美元。

随后红帽通过不断的并购继续巩固了市场规模，收购了至少 30 多家公司。

据redhat2019财年第三季度财报（指的是2018年9月1日到2018年11月30日），已连续67个季度营收增长。季度总营收8.47亿美元，比去年同期增长13%，按固定汇率计算增长15%；季末递延收入总额为25亿美元，比去年同期增长20%，按固定汇率计算增长23%。

**RedHat股价在2018年6月达到高点176，而后一路走跌到117美元，10月29日IBM给出了每股190美元收购价，比Red Hat上周五的收盘价溢价63%。**

![image.png]()



### Q：微软欲收购Docker，为啥出价是40亿美元？

2014年1月完成了 1500 万美元的 B 轮融资。该轮融资由 Greylock Partners 领投，交易后估值为 10 亿美元。

2016年**，**微软出价40亿美元欲收购Docker未果。

2017年年底C轮，在总目标为7500万美元的融资计划中筹得6000万美元，当时的投资参与方包括AME Cloud Ventures、Benchmark、Coatue Management、高盛投资以及Greylock Partners，交易后估值13亿美元。

在2018年10月总额达1.92亿美元的融资计划中成功筹得9200万美元。Docker公司创始人兼前任首席执行官Solomon Hykes离开Docker公司时有说：利用这笔资金，Docker公司很可能会增加其销售与营销人员规模，并为可能于2019年开始的首轮公开募股（简称IPO）提供良好的营收数字。

有篇文章提到，Docker 在当时的估值已超过了 10 亿美元，投资商对它的期望出售价格应该不会低于 35 亿美元。为啥不会低于35亿美元？？



### **Q：估值是怎么估的？收购价是怎么估的？**

**（1）怎么估值？**

估值分两种大方案：相关估值（Relative Valuation）和绝对估值（Absolute Valuation）

相对估值就是跟别的同行业公司比，主要有两种方法：

- 可比公司分析（Trading Comps）：对比目标是已上市公司，通过价格除以盈利指标（比如销售额、运营收入、净利润等）可以得出一个“倍数”

- 先例交易分析（Deal Comps）：对比目标是没上市公司，通过过往交易（比如股权融资、出售老股等）的定价来算“倍数”

绝对估值的两种常用方法：

- 现金流折现分析（Discounted Cash FlowAnalysis，DCF）：是将估值对象的未来现金流折现，但由于主观因素对假设影响较大，很多时候主要作为参考（算出来的是每股股价）

- 杠杆收购分析（Leveraged Buy-Out Analysis，LBO）：主要关注估值公司未来产生的回报，常用于私募基金投资（算出来的是IRR回报率）

此处有个具体例子：https://mp.weixin.qq.com/s/SByRL7NFwAdFNC5NbG0KtA

**（2）谁来估值？**

聘请投行或财务顾问。投行会把所有适用的估值方法都粗略测算一遍，然后再根据交易本身最适用的方法进行调试，谈到最后主要就是一个价格倍数。

估值永远不会是一个单一的数字，而是一个估值区间，用一个类似美式橄榄球体育场的图形展示，俗称为“Foot Ball Chart”

​          ![92bbd79b0b99b11280a2db344ec6e818_hd.jpg]()

**（3）为什么收购价会高于估值这么多？**

收购价主要来源于两大块价值的评估：被并购企业的独立价值（standalone case）和 协同效应（synergy）

- **独立价值：**对初创企业和成熟企业的估值是有很大区别的。成熟企业能大致估算它的行业趋势，发展前景，也能根据过往的同类并购给一个大致的合理估值，一般不会溢价太高。初创企业大致是基于可预见的突破性的商业模式进入某一广阔市场，估算其未来X年内能占有的市场份额等，最后估值圈外人会觉得很夸张，但是圈内人，能洞察行业发展远景的人实际上是基于对该目标公司的行业远景和战略蓝图来判断的。
- **协同效应**：分为收入协同和成本协同。比如说共享渠道这种就是收入协同，采购集中这种就是成本协同。举个例子，优酷收购土豆之后在市场上的谈判力能增强，可以压低版权采购成本，比土豆单独经营更能盈利，就可以出价高一些。对于不同的买方，能跟目标公司实现的协同是不一样的，最终能给出的报价也会差很多。

**（4）在Docker案例中，与10亿估值比，控股收购 溢价300%，为啥会溢价溢出这么多？**

收购的原因新闻有很多，但40亿美元的定价逻辑，没能成功解释 (*T_T*) 。也许评估了未来云市场规模、docker的战略地位，觉得值这么多钱吧……



# 4. 开源数据分析

几乎所有大厂也已经把开源项目托管到了github上，github数据基本能覆盖所有开源项目。

## 4.1. github年度盘点报告

*（以下数据来自于GitHub于2018年10月16 日发布的 Octoverse 报告*https://octoverse.github.com/*）*

2018年，github上有3100 万+，9600万Repo，210万Organization，覆盖200+国家。

有 800 万名新开发者加入 GitHub，比 GitHub 成立后前 6 年的用户量总和还要多。新增长的用户主要来自美国、中国和印度（下图左）。

按contributors数量排序，美国、中国、印度排在前三位（下午右）

在按照贡献者数量评比的开源项目榜单中，VS Code，React 和 Tensorflow 再次登榜，下面是排名前 10 的开源项目：

每年有数以百万计的开发者支撑起 GitHub 上的开源开发，他们来自各大公司和院校。2018 年**拥有开源贡献者数量最多的前 10 家机构全部位于美国**，它们是：

与此同时，GitHub 也公布了过去一年平台上使用数量最多的 10 门编程语言，**JavaScript、Java 和 Python 继续稳居前三**。下图是过去 4 年 GitHub 上使用量最多的 10 门语言及其变化状况：

topics排名：

## **4.2. 中国项目的Grank排名，衡量项目的活跃度和社区化程度**

- **活跃度**：考量项目的提交数、 拉取请求数和贡献者数（其它数据如代码行数、文件数、issue 数、 fork 数、star 数等考核权重较低）。
- **社区化程度**：根据项目贡献者的身份信息、邮件后缀等依优先级来判断其所属身份，以其聚类结果的离散程度来评估项目的社区化程度。

采集了500个项目，来自于如下公司：阿里巴巴(Ant Design、蚂蚁金服、Angular Developers、淘宝、天猫前端、eggjs)、华为(华为 Hadoop)、腾讯(AlloyTeam、tarscloud)、百度(FEX、EFE、前端)、饿了么(前端)、网易、搜狐、奇虎 360(360 企业安全)、唯品会、豆瓣、大众点评、小米、美团(美团点评)、美丽、豌豆荚、当当、有赞、深度、DNSPod、新浪微博、今日头条、滴滴出行、eBay。

| **#**  | **企业** | **组织**   | **仓库**                                                     | **活跃度** | **#**  | **企业** | **组织**   | **仓库**           | **活跃度** |
| ------ | -------- | ---------- | ------------------------------------------------------------ | ---------- | ------ | -------- | ---------- | ------------------ | ---------- |
| **1**  | 阿里     | ant-design | ant-design（2018年有 2298 个提交，最多的一周有 122 个提交，一年内新增了 350 位贡献者和 1057 个 PR） | 49.13      | **26** | 阿里     | ant-design | ant-design-mobile  | 9.6        |
| **2**  | 阿里     | alibaba    | pouch                                                        | 42.49      | **27** | 滴滴     | didi       | mand-mobile        | 9.47       |
| **3**  | 华为     | apache     | CarbonData（华为捐献给 Apache 基金会的）                     | 35.71      | **28** | 阿里     | alibaba    | terraform-provider | 9.19       |
| **4**  | 阿里     | alibaba    | ice                                                          | 30.76      | **29** | eBay     | ebay       | skin               | 9.1        |
| **5**  | 饿了么   | elemefe    | element                                                      | 30.28      | **30** | eBay     | apache     | incubator-griffin  | 9.07       |
| **6**  | 个人     | apache     | incubator-skywalking                                         | 27.77      | **31** | 阿里     | antvis     | f2                 | 8.96       |
| **7**  | 阿里     | ant-design | ant-design-pro                                               | 23.28      | **32** | 阿里     | antvis     | g                  | 8.89       |
| **8**  | 有赞     | youzan     | vant                                                         | 22.82      | **33** | 腾讯     | tencent    | ncnn               | 8.34       |
| **9**  | 阿里     | apache     | incubator-weex                                               | 22.57      | **34** | 百度     | apache     | incubator-ECharts  | 8.27       |
| **10** | 有赞     | youzan     | zent                                                         | 22.16      | **35** | 腾讯     | tencent    | TSW                | 8.05       |
| **11** | 唯品会   | vipshop    | Saturn                                                       | 21.05      | **36** | 阿里     | alibaba    | BizCharts          | 7.57       |
| **12** | 腾讯     | tencent    | bk-cmdb                                                      | 20.73      | **37** | 阿里     | alibaba    | beidou             | 7.4        |
| **13** | 阿里     | apache     | incubator-dubbo                                              | 19.62      | **38** | 腾讯     | tarscloud  | Tars               | 7.09       |
| **14** | 百度     | baidu      | openrasp                                                     | 17.23      | **39** | 阿里     | alibaba    | druid              | 7.07       |
| **15** | 滴滴     | didi       | cube-ui                                                      | 15.17      | **40** | 小米     | xiaomi     | pegasus            | 6.96       |
| **16** | 阿里     | ng-zorro   | ng-zorro-antd                                                | 13.73      | **41** | 百度     | baidu      | san                | 6.85       |
| **17** | 有赞     | youzan     | vant-weapp                                                   | 13.57      | **42** | 阿里     | antvis     | g6                 | 6.49       |
| **18** | 阿里     | antvis     | g2                                                           | 13.21      | **43** | 唯品会   | vipshop    | vjtools            | 6.4        |
| **19** | 阿里     | alibaba    | rax                                                          | 12.88      | **44** | 腾讯     | tencent    | wcdb               | 6.18       |
| **20** | 阿里     | alibaba    | AliOS-Things                                                 | 12.07      | **45** | 阿里     | alibaba    | dawn               | 6.04       |
| **21** | eBay     | apache     | kylin                                                        | 11         | **46** | 百度     | tencent    | xLua               | 5.92       |
| **22** | 腾讯     | tencent    | wepy                                                         | 10.54      | **47** | 饿了么   | eleme      | bigkeeper          | 5.85       |
| **23** | 阿里     | ant-design | ant-design-pro-site                                          | 10.28      | **48** | 饿了么   | elemefe    | v-charts           | 5.71       |
| **24** | 阿里     | eggjs      | egg                                                          | 10.05      | **49** | 阿里     | alibaba    | atlas              | 5.64       |
| **25** | 阿里     | alibaba    | weex-ui                                                      | 9.76       | **50** | 阿里     | alibaba    | canal              | 5.6        |





# 5. 资料[﻿](https://yq.aliyun.com/articles/692793?spm=a2c4e.11157919.spm-cont-list.338.146c27aeCDHNDv)

| 章文嵩谈LVS开源 | https://www.linuxidc.com/Linux/2013-06/86666.htm |
|  |  |
| 谷歌与开源那些事儿 | [【转】开源纵横谈：谷歌与开源那些事儿](https://km.sankuai.com/page/141764333) |
| 蚂蚁金服总监杨冰：金融科技公司为什么要拥抱开源？ | https://tech.antfin.com/articles/315 |
| 文章 | [how-google-turned-open-source-into-a-key-differentiator-for-its-cloud-platform](https://www.forbes.com/sites/janakirammsv/2017/07/09/how-google-turned-open-source-into-a-key-differentiator-for-its-cloud-platform/#3326cd9d646f) |
| 运营开源项目-1 | [How to Spread The Word About Your Code](https://hacks.mozilla.org/2013/05/how-to-spread-the-word-about-your-code/?utm_source=statuscode&utm_medium=email) |
| 运营开源项目-2 | [FIRST WEEK OF LAUNCHING VUE.JS](http://blog.evanyou.me/2014/02/11/first-week-of-launching-an-oss-project/) |
| 2018中国开源年会（COSCon）资料下载 | http://www.kaiyuanshe.cn/thread/170815.mhtml |
| 2018中国开源年度报告 | 2018中国开源年度报告.pdf3.10MB |
| 中国云计算开源发展调查报告(信通院) | 中国云计算开源发展调查报告(信通院).pdf1.27MB |
| 2018年欧盟工业研发投资排名 | The 2018 EU Industrial R&D Investment Scoreboard (1).pdf1.95MB |

