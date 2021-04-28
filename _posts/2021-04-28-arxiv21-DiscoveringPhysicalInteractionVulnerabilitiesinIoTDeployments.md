---
layout:     post
title:      Discovering Physical Interaction Vulnerabilities in IoT Deployments
subtitle:   
date:       2021-02-15
author:     bbq
header-img: img/smarthome-apps.jpg
catalog: true
tags:
    - research
    - IoT
    - attacks
---
	
>authors: Muslum Ozgur Ozmen,Xuansong Li*,Andrew Chun-An Chu,Z. Berkay Celik,Bardh Hoxha,Xiangyu Zhang


>Abstract: Internet of Things (IoT) applications drive the behavior of IoT deployments according to installed sensors and actuators. It has recently been shown that IoT deployments are vulnerable to physical interactions, caused by design flaws or malicious intent, that can have severe physical consequences. Yet, extant approaches to securing IoT do not translate the app source code into its physical behavior to evaluate physical interactions. Thus, IoT consumers and markets do not possess the capability to assess the safety and security risks these interactions present. In this paper, we introduce the IoTSeer security service for IoT deployments, which uncovers undesired states caused by physical interactions. IoTSeer operates in four phases (1) translation of each actuation command and sensor event in an app source code into a hybrid I/O automaton that defines an app&#39;s physical behavior, (2) combining apps in a novel composite automaton that represents the joint physical behavior of interacting apps, (3) applying grid-based testing and falsification to validate whether an IoT deployment conforms to desired physical interaction policies, and (4) identification of the root cause of policy violations and proposing patches that guide users to prevent them. We use IoTSeer in an actual house with 13 actuators and six sensors with 37 apps and demonstrate its effectiveness and performance.物联网(IoT)应用根据安装的传感器和执行器来驱动物联网部署的行为。最近已经表明，物联网部署很容易受到由设计缺陷或恶意意图引起的物理交互的影响，这可能会产生严重的物理后果。然而，现有的保护物联网安全的方法并没有将应用源代码转化为其物理行为，以评估物理交互。因此，物联网消费者和市场不具备评估这些交互所带来的安全和安全风险的能力。在本文中，我们介绍了用于物联网部署的IoTSeer安全服务，它可以发现物理交互引起的不希望状态。IoTSeer的操作分为四个阶段（1）将应用源代码中的每个执行命令和传感器事件翻译成一个混合I/O自动机，定义应用的物理行为；（2）将应用组合在一个新颖的复合自动机中，表示交互应用的联合物理行为；（3）应用基于网格的测试和伪造来验证物联网部署是否符合期望的物理交互策略；（4）识别策略违反的根本原因，并提出补丁，指导用户防止这些行为。我们在一个有13个执行器和6个传感器与37个应用的实际房屋中使用IoTSeer，并展示了其有效性和性能。

![](RackMultipart20210428-4-q8se7e_html_d0b576b4647b9bfa.png)

# INTRODUCTION

随着物联网(IoT)的不断扩散，在日益自动化的家庭中诊断不正确的行为变得相当困难。设备和应用程序可能会以触发动作规则的长序列链在一起，以至于从一个可观察到的症状（例如，一个未上锁的门）可能无法确定远处的根本原因（例如，一个恶意应用程序）。这是因为，目前，物联网审计日志是孤立在单个设备上的，因此无法用于重建复杂工作流程的因果关系。在这项工作中，我们提出了ProvThings，一种以平台为中心的物联网集中审计方法。ProvThings对物联网应用和设备API进行高效的自动化仪表，以便生成数据出处，为系统活动（包括恶意行为）提供整体解释。我们对三星SmartThings平台的ProvThings进行了原型设计，并针对26个物联网攻击的语料库对我们的方法的有效性进行了基准测试。通过引入选择性的代码工具优化，我们在评估中证明了ProvThings在物理物联网设备上只需要5%的开销，同时可以实现系统行为的实时查询，并进一步考虑如何利用ProvThings来满足物联网生态系统中各种利益相关者的需求。

一般的研究都关注软件层面的逻辑链，比如xxx使灯的状态转为亮，灯的状态为亮使关门指令被下达。但是这个研究关注到了物理上的联系，从声音、光、温度等产生类似规则链的东西。比如炉子点火造成温度升高，可能会导致其它以温度为输入的app产生预期之外的反应。

# BACKGROUND

主要定义了几种产生错误的方式：

- 单个app干扰其它传感器、多个app联合效果干扰传感器而引发事件、多个app对一个传感器同时造成冲突的干扰
- 还定义了intend/unintended channel
- PHYSICAL INTERACTION VULNERA BILITIES
- 具体展示了上述几种错误造成的非预期指令

# IOSEER系统的实现：

## 从app代码中提取传感器和命令：

由于开发应用的软件已经是很高层的平台，提供的大多是接口，以及if-then式的编程，所以这个比较容易实现，以及有现有工具。

## 用代码描述物理过程：

主要定义了几个部分：1模仿操纵装置的自动机，输入用户设定的参数以及开始工作时间，结束时间等，具有工作/关闭的状态等。2模仿物理介质的自动机，如根据物理方程来模仿温度随时间、距离的变化。3模仿传感器的自动机，模拟收到物理介质传来的信息，比如温度超过阈值则发出信号。

## System Identification (SI) for Tuning Automata Parameters

这个部分的功能是调节模仿操纵装置的自动机与物理介质之间的交互情况的。比如，炉子对温度的影响随时间、空间变化的情况就由这个部分决定。过去的方法是在实际场景中调试设备来获取这个温度情况，而作者改进的方法是预先对不同的机器大致设定一个曲线，只对具体的每个设备进行曲线的参数的调整。（细节不完全明白）

## 计算每个app的行为：

对每个app的每个指令，用这些自动机模拟的传感器等来定义这些指令。即把这些自动机通过指令连接起来，形成整体。主要的算法还是遍历和全排列这样的枚举算法。

**添加环境因素：**

把环境因素视作新的自动机，比如人的活动可能影响温度、产生声音等、和一个炉子加一个风扇的效果是一样的所以也可以用自动机来模拟。而这个自动机也存在工作和不工作两个状态。

**定义规则：**

定义了五种规则，但主要的内容就是不能让预期之外的传感器收到信息而引发app事件。

Security Analysis of an IoT Deployment

安全检查就是检查规则有没有被侵犯。算法有暴力法和优化算法。优化算法没太明白原理。暴力算法就是枚举不同的事件和命令，即让不同组合的自动机工作，然后看是否引发了非预期的传感器事件

# Limitation

现实情况下环境情况几乎不会干扰：如环境噪音、家具等的影响。而人的活动等被额外的自动机很好的模拟了也完全合乎实际。而自动机参数调整的部分不是太明白。