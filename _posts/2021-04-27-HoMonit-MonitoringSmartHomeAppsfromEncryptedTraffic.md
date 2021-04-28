---
layout:     post
title:      HoMonit:Monitoring Smart Home Apps from Encrypted Traffic
subtitle:   CCS’18, October 15-19, 2018, Toronto, ON, Canada
date:       2020-02-15
author:     bbq
header-img: img/smarthome-apps.jpg
catalog: true
tags:
    - research
    - IoT
    - attacks
---
	

>authors:Wei Zhang*,Yan Meng*,Yugeng Liu,Xiaokuan Zhang,Yinqian Zhang,Haojin Zhu†


![](RackMultipart20210428-4-1skfu6u_html_26c1520e2ffdba81.png)

>Abstract-智能家居是一种将大量的智能传感器和设备智能连接起来，以促进家电、照明、供暖和制冷系统以及安防和安全系统的自动化的新兴技术。我们的研究围绕三星SmartThings展开，这是目前智能家居平台中应用数量最多的智能家居平台。此前的研究已经揭示了SmartThings设计中的一些安全缺陷，这些缺陷使得恶意的智能家居应用（或SmartApps）可以拥有比设计更多的权限，并窃听或欺骗SmartThings平台中的事件。为了解决这些问题，本文利用侧信道推理能力设计并开发了一个被称为HoMonit的系统，从加密的无线流量中监控SmartApps。为了检测异常，HoMonit将从加密流量中推断出的SmartApps活动与其源代码或UI界面中规定的预期行为进行比较。为了评估HoMonit的有效性，我们分析了181个官方SmartApps，并对60个恶意SmartApps进行了评估，这些恶意SmartApps要么对智能设备进行超权限访问，要么进行事件欺骗攻击。评估结果表明，HoMonit能够有效验证SmartApps的工作逻辑，在检测SmartApp的不当行为方面达到了较高的准确性。

# Introduction

之前的工作中，对于物联网平台的安全和隐私保护主要有三种方式：

| 1
 2
 3 | 1、修改物联网平台应用，添加信息流控制的代码
 2、基于上下文的权限管理系统，帮助用户分配权限
 3、对智能设备源码进行分析后进行基于上下文的权限管理 |
| --- | --- |

而本文旨在提供一种第三方的保护措施，通过加密信道来监控智能设备。

HoMonit是一个能够从加密的无线通信信道中对智能设备进行监控的系统。其核心是DFA匹配算法。

**核心思想是，** 每个智能设备的行为都遵循一种确定的DFA模型，模型中的状态能够表示app运行的状态。

首先，从智能设备的网站上获取源码和文档结合app UI建立一个DFA模型。

然后，通过对无线网络通信的侧信道分析，对加密的无线通信信道进行监控，从而能够对DFA模型中的状态转换进行分析。

而侧信道中，对加密数据的监控，基于对数据包大小和间隔的观察。

一旦智能设备的运行出现了DFA模型中不包含的状态。即可以判断该设备出现了异常行为，检测出这是一个恶意的应用。

在对该系统的评估中，本文对60款开源应用进行修改，使其模拟恶意行为。结果显示，该系统能够有效的检测出几种恶意行为。

# 主要贡献：

1、新的技术：能够通过对源码、应用的UI分析构建出DFA模型； 侧信道分析无线通信信道的方法

2、新的系统：HoMonit（不需要与厂商合作）能够在第三方视角分析SmartThings平台的智能设备恶意行为。

3、开源数据集：在对系统评估时使用的60个修改过的恶意应用已经公开

# **MOTIVATIONS AND INSIGHTS**

本文中设计的系统较之前的工作，更加通用，它设计时以整个平台的角度出发。而不需要对特定应用进行修改。

HoMonit的设计，基于两个规律。一是，为了降低能耗智能设备的洗一大多设计为低传输速率，并且尽量减少数据冗余。二是，中继器和设备之间的通信，大多存在一些自定义的标记字段。

![](RackMultipart20210428-4-1skfu6u_html_ee4a9262b89b25c6.png)

## DFA BUILDING VIA SMARTAPP ANALYSIS

选择DFA来描绘智能应用的原因有两点：

1、智能应用管理有限数量的设备

2、智能设备会在满足条件的情况下由智能应用驱动产生状态变化，这与DFA的有限状态和转移条件的特性相吻合。

## DFA Builing for Open-source Apps

选择DFA来描绘智能应用得原因有两点：

1、智能应用管理有限数量的设备

2、智能设备会在满足条件得情况下由智能应用驱动产生状态变化，这与DFA的有限状态和转移条件的特性相吻合。

**DFA Builing for Open-source Apps**

对于开源应用，通过进行对源码的静态分析来建立DFA模型。

首先将源代码转换为AST抽象语法树。

再将AST转换为DFA。

**1**** 、建立状态集合**

| 1
 2
 3
 4 | 即找出应用具备的功能。
首先，通过阅读开发者文档作为参考依据；
其次，通过一些input（Groovy语法,类似获取数据并进行条件判断）函数方法来确定。
尤其是一些条件转移的代码。 |
| --- | --- |

**2**** 、确定转移条件**

| 1
 2 | 通过subscribe和handler方法（Groovy语法，
类似开启线程执行某个特定的与设备交互的功能）来确定转移条件。 |
| --- | --- |

使用该方法对181个应用进行了测试，其中150个成功建立了DFA模型（82.9%）

![](RackMultipart20210428-4-1skfu6u_html_304babe3d7951d01.png)

对于闭源应用，则通过在应用UI中的语义分析来构建DFA模型。在用户首次安装应用进行授权时，所分配的权限能够反映出一些有效信息。所以我们使用adb命令中的uiautomator来将UI hierarchy转换成xml获取应用的权限信息。并且可以从String.xml中得到一些有用的信息。例如一些应用中涉及场景的字符串。在应用添加设备时，我们认为可选的设备都具备该应用能够执行的某种功能。可以通过这种方式缩小范围。另外，应用的功能往往与某个名词有关联（例如water 与water sensor有联系）

当通过各种方式确定了应用具有的功能之后，接下来需要确定每个功能对应的参数和指令。这些信息可以先从官方文档中得到，并通过语义分析将动词和指令或参数匹配。（例如when water is sensed能够与water的wet参数匹配）

## **DETECTING APP MISBEHAVIORS BASED ON WIRELESS TRAFFIC FINGERPRINT**

核心思想是，总结出各个状态时，发送和接收的数据包的个数和时间间隔。以此来标记相应的状态和转移条件。 **（其他内容多涉及硬件，见原文）**

# COMMENT

该系统易于移植，适用范围广（相较于之前的工作中需要修改特定应用）。并且利用侧信道的方式，能够以第三方视角，巧妙地监测智能设备行为。具有较高的准确率。

对于一些恶意行为能够高效地辨别。

另外，本文中还提到了。这种测信道方式可能被应用于一些泄露用户隐私地攻击，提供了研究方向。

但是本文由于选择的平台为三星SmartThings，大多数设备通信使用ZigBee和Z-wave，而国内许多设备则通过BLE方式通信，而此通信协议在本文中不涉及，还有待进一步研究。