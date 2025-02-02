---
title: 'IoT学习笔记'
cover: 'http://jinluzhang.github.io/assets/posts_img/2019-07-18-IoT_summary/image-20190718144444613.png'
date: 2019-07-18
categories: 调研
author: Beaug
tags: spec
---



我国物联网（Internet of Things，简称 IoT）产业规模已从2009年的1700亿元跃升至 2017年的11500 亿元，年复合增长率为26.9%，并预估将在2020年达到1.8万亿。2017年中国互联设备12.1亿，并预估将在2015年达到53.8亿。

据IDC估算，到2020年全球互联设备将达到300亿，产生$1.46 trillion价值。GSMA估计全球 Industrial IoT互联设备在2025年将达到138亿。



##IoT四层技术架构

将物联网的技术架构分为四层，分别为感知层、传输层、平台层和应用层。感知层主要涉及芯片、模组以及传感器等感知设备；传输层分为短距离即局域网传输（ WiFi、蓝牙和Zigbee等）和长距离即广域网传输（ NB-IoT、LoRa等）；平台层分为连接管理平台、设备管理平台、应用使能平台和业务分析平台；应用层包括十大领域，分别为物流、交通、安防、能源、医疗、建筑、制造、家居、零售和农业。

![image-20190718141802883](http://jinluzhang.github.io/assets/posts_img/2019-07-18-IoT_summary/image-20190718141802883.png)

**传输层：**

物联网的传输层主要负责传递和处理感知层获取的信息,分为有线传输和无线传输两大类,其中无线传输是物联网的主要应用。无线传 输技术按传输距离可划分为两类:一类是以Zigbee、WiFi、蓝牙等为代表的短距离传输技术,即局域网通信技术;另一类则是 LPWAN(low-power Wide-Area Network,低功耗广域网),即广域网通信技术。LPWAN又可分为两类:一类是工作于未授权频 谱的LoRa、Sigfox等技术;另一类是工作于授权频谱下,3GPP支持的2/3/4/5G蜂窝通信技术,比如eMTC(enhanced machine type of communication ,增强机器类通信)、NB-IoT( Narrow Band Internet of Things ,窄带物联网)。 

![image-20190718143343500](http://jinluzhang.github.io/assets/posts_img/2019-07-18-IoT_summary/image-20190718143343500.png)

![image-20190718151158592](http://jinluzhang.github.io/assets/posts_img/2019-07-18-IoT_summary/image-20190718151158592.png)

**平台层：**

* 连接管理平台CMP(Connectivity Management Platform)：应用于运营商网络上,通过连接物联网卡,该平台可以实现对物联网连接管理、故障管理、网络资源用量管理、资费管理、账单管理以及服务托管等

* 设备管理平台DMP(Device Management Platform)：对物联网终端进行远程监控、配置调整、软件升级、故障排查以及生命周期管理等功能，并通过提供开放的API调用接口帮助客户实现系统集成和增值开发等,所有设备的数据存储在云端

* 应用使能平台AEP(Application Enablement Platform)：该层是能够快速开发部署物联网应用的云平台,同时能够为客户提供完整、具有动态扩展、按需服务以及高可用性的物联网应用,是一个结合应用场景的系统开发平台

* 业务分析平台BAP(Business Analytics Platform)：该基础大数据服务和机器学习等数据分析服务。

**应用层：**

![image-20190718144444613](http://jinluzhang.github.io/assets/posts_img/2019-07-18-IoT_summary/image-20190718144444613.png)

