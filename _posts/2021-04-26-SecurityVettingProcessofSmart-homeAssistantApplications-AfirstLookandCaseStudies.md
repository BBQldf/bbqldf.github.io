---
layout:     post
title:      Security Vetting Process of Smart-home Assistant Applications:A first Look and Case Studies
subtitle:   Hang Hu,Limin Yang,Shihan Lin，Gang Wang
date:       2013-01-17
author:     Transliteration
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - IoT
    - attacks 
    - research 
---
	



# Abstract：

Amazon Alexa和Google Home目前的智能家庭物联系统里对应用商店中第三方应用的安全检查主要是自动检查和人工检查。作者说这些检查是不充分的，所以做了一个初步的调查和实验来研究存在的安全漏洞。这个实验主要关注Alexa/Google cloud和third-party application servers(i.e.end-points)之间的身份验证机制。作者表示，一方面，这两者之间的身份验证机制本身就有问题，另一方面，应用商店的安全检查也不能弥补这个问题。所以作者设计了一个验证实验，写了一个应用并尝试通过应用商店的安全检查然后上架，然后展示这个身份验证机制的问题。初步试验下，找到了219个存在该安全问题的真实存在的应用以及其服务器。

# Introduction:

在Amazon Alexa和Google Home目前的智能家庭物联系统里，用户只需要简单地下载应用然后使用。而应用开发者只需要让应用通过安全检查即可上架应用。

一方面，在这个研究中作者的希望通过自己写应用并上架，来了解Amazon Alexa和Google Home到底做了哪些方面的安全检查，然后研究这些安全检查能否提醒开发者自己的程序存在安全问题。

另一方面，作者具体展示的安全漏洞是：Amazon Alexa和Google Home的云端对每一个应用使用同一个签名来表明自己的身份，所以攻击者可以自己写一个应用来骗取一个带签名的信息并且转发给受害者应用的server，使受害者应用误以为该命令来自于云端，并且执行该命令。这个漏洞之所以可行是因为智能家庭物联的应用程序（文中称作skill）与手机app不同，这里的应用每一次执行命令都需要通过云端作为中转站来把信息传递给应用的服务器，所以通信中身份验证机制可能有问题。

# BACKGROUND &amp; MOTIVATION：

![](RackMultipart20210428-4-12l0uoj_html_3476e43d8a913fc.png)

图一展示了一个应用程序的命令执行过程。1是用户发出指令，2是指令由设备传给云端，3是云端通过HTTPS请求发送信息给endpoint也就是第三方应用的服务器。4是服务器回复5云端向设备下达指令6设备执行指令

然后作者介绍了这些步骤中的身份验证，别的验证都没啥问题就不提了，作者发现34有问题。

3、4通过HTTPS发送信息，endpoint识别cloud是通过识别cloud的签名，并检查内容里的APP-ID和TIME。但是一个应用程序开发者写程序的时候很可能忘了要检查APP-ID和TIME，那么这个时候应该是Amazon Alexa和Google Home的安全检查的责任来提醒他，并且不允许它上架。但事实上Alexa和Google并没有。所以作者通过实验证明了一下。

# AUTOMATED SKILL VETTING

![](RackMultipart20210428-4-12l0uoj_html_c4b721aa9e1f06a3.png)

这个表左侧一列是作者实验中开发的APP的服务器（endpoint）对cloud发来的信息的检查情况。valid表示检查云端签名，检查APP-ID，检查Time。然后依次往下分别是只不检查ID、只不检查Time、接受所有信息（云端的签名都不检查）、拒绝所有信息、离线。右边三列是HTTPS的证书，这个和研究内容无关，意思是endpoint有Standard和Wildcard的证书都是一样的而且不影响通过，没有证书或者不合法证书就过不了安全检查。

结果显示，应用商店的安全检测（无论是人工还是自动的）都不管endpoint是否检查APP-ID和TIME，只管它是否检查了cloud的身份签名，并且只在签名有效的情况下响应。不检查APP-ID就是说发给另一个APP的消息如果被发给了它，它也会响应。

所以问题来了，cloud发给另一个APP的endpoint的信息可以被另一个不检查APP-ID的APP的endpoint响应。

# SPOOFING THE CLOUD

![](RackMultipart20210428-4-12l0uoj_html_6459de0192fd428b.png)

所以可以假冒云端来发送信息，来模仿一个来自用户的指令。作者设计了一个恶意应用并上架，这个应用有和目标应用一样的指令和指令格式，攻击者下达该指令后，云端向恶意应用的endpoint发送带有云端签名的请求（并包含恶意应用的APP-ID）然后攻击者拿到这个请求后直接转发给目标应用的endpoint（假设已经知道url）。然后，如果这个目标应用不检查消息的APP-ID，那么它就会响应这个请求。在实验中，请求是一个简单的SQL注入，然后可以盗取endpoint数据库中的信息。

作者表示这攻击和被攻击应用都是他自己写的所以没有道德问题。

之后的一段作者表示Google的自动+人工安全检查和Alexa的自动一样，也不管应用有没有检查APP-ID

#ALEXA VULNERABLE ENDPOINTS

![](RackMultipart20210428-4-12l0uoj_html_45b7646ca3a85e5c.png)

做完对自己写的APP的攻击之后作者开始在现实中找是不是存在有这个漏洞的真实应用。他先用一个简单方法猜到一些应用的endpoint的host（怎么找的不重要就不写了）然后用刚才的方法假冒云端来发送指令，这里的指令是每个应用都有的默认指令（类似开关机暂停等等）。具体就是攻击者还是做一个恶意APP然后上架，这个恶意APP模仿目标应用的指令。使用时这个恶意APP时，恶意应用的endpoint收到cloud发来的请求后直接转发给目标endpoint。这样，如果目标endpoint响应了这个请求，攻击者就实现了对另一个应用的操作（实验中是开机关机暂停等基础操作）。然后如表2所示，找到的3,346,425个host中有122个存在漏洞，也就是响应了作者伪造的指令，然后作者又来了一次round2，又搞了一批host然后找到了100个有漏洞的应用。

后面还写了这些真实例子的分布地区和发布这些APP的公司，以及这些真实例子中的APP是用于哪些功能的（有银行的开车的智能家居的社交软件的），然后对应地讲了讲这个漏洞的危害性。

# DISCUSSION &amp; CONCLUSION

讲了1）这是Google和Alexa应用商店的责任来提醒开发者要检查APP-ID，不检查的不能让上架。2）整个通信过程的身份验证有问题可以XXX解决。