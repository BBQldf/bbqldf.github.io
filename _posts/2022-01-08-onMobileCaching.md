---
layout:     post
title:     On Mobile Edge Caching
subtitle:   IEEE Sur.&Tutorial19
date:       2022-01-08
author:     ldf
header-img: img/post-bg-mobilecaching01.png
catalog: true
tags:
    - EdgeComputing
    - DataPrivacy
---
# 

> Jingjing Yao , _Student Member, IEEE_, Tao Han , _Member, IEEE_, and Nirwan Ansari , _Fellow, IEEE_

**Abstract——** 随着各种移动应用的广泛采用，无线网络的流量正以指数级的速度增长，这给移动核心网络和回程链路带来了巨大的负担。移动边缘缓存，使移动边缘具有缓存存储，是缓解这一问题的一个有前途的解决方案。在本文中，我们旨在回顾移动边缘缓存的最新进展。我们首先介绍了移动边缘缓存的概况和它的优势。然后我们讨论了在网络中可以实现移动边缘缓存的位置。我们还分析了不同的缓存标准以及它们各自对缓存性能的影响。此外，我们比较了几种缓存方案，并讨论了它们的优点和缺点。我们进一步对缓存过程进行了详细而深入的讨论，它可以被划分为四个阶段，包括内容请求、探索、交付和更新。对于每个阶段，我们确定不同的问题，并回顾解决这些问题的相关工作。最后，我们提出了当前移动边缘缓存架构和技术所面临的一些挑战，以供进一步研究。

# 一、Introduction

解决这些限制（移动设备的资源受限）的一个突出的解决方案是将一些计算卸载到云端[8]。这种解决方案被称为移动云计算（MCC）[9]，其中术语 &quot;云 &quot;是指通常位于遥远的数据中心的服务器集合，为移动设备提供足够的 **计算、存储和网络** 资源。

此外，边缘服务器可以很容易地获得网络状态信息（例如，无线信道条件、流量模式和用户移动模式），这些信息可以被分析和利用，以提供更好的服务。例如，当无线信道条件不佳时，边缘服务器可以分配更多的计算资源，以减少服务器的服务处理时间，从而弥补较长的无线传输延迟。因此，总的服务响应时间（即无线传输时间加上服务处理时间）将不会增加，服务性能将得到保证。

**边缘缓存的作用** ——减少了网络中的移动流量，也减少了内容交付的延迟

## 1、为什么会有边缘缓存？

一些流行的内容可以被用户多次请求。因此，内容提供商必须重复发送相同的内容。这不仅会造成较长的服务延迟，而且还会注入更多的流量，从而有可能造成网络拥挤。移动边缘缓存利用支持缓存的边缘服务器来存储流行的内容，以便这些内容可以直接从缓存中传输，而不是从远程云端传输。因此，回程链路的流量负荷可以大大减少。

PS：由于用远程云的热门内容填充缓存会产生额外的网络流量，所以通常在非高峰期（如夜间）实现，而在高峰期（如白天）用缓存的内容服务请求。（这个是实际操作，其实真实的话，还要强调实时性，必须赶快缓存到边缘来）

## 2、无线边缘缓存关注的问题是？

缓存的概念在网络缓存[17]和以信息为中心的网络（ICN）[18], [19]中得到了充分的研究。Ali等人[17]讨论了ICN的现有网络缓存和预取方法。为了提高内容缓存的效率，许多研究工作都致力于 **优化路径选择**** [23] ****、服务器放置**** [24] ****和内容重复策略**** [25] ****（这几个都是相对于传统网络中的）** 。Borst等人[25]提出了在内容传递网络中最小化带宽成本的算法，其假设是内容流行度是给定的。

有线和无线网络中的缓存有很大的不同：

- 由于用户的流动性，无线缓存通常面临着动态流量负载的挑战。因此，网络流量与用户移动模式高度相关，因此很难预测网络流量。
- 此外，无线信道表现出更多的不确定因素，如有限的频谱资源和共信道干扰。因此，设计无线网络的缓存方案更具挑战性。

### 1). 无线边缘缓存的一些相关工作

- Han等人[20]对移动网络中的内容交付加速的现有解决方案进行了概述。然而，他们只关注了内容交付，这只是缓存过程的一个方面（包括内容放置、交付和更新）。

- Liu等人[21]讨论了无线边缘缓存的几个问题，包括内容放置和交付。然而，他们的工作并不完整，因为他们没有考虑缓存过程中的内容更新问题。他们也没有调查缓存的标准和方案。
- Wu等人[22]讨论了移动社交设备缓存（MSDC）的研究挑战，这是移动边缘缓存的另一个方面，因为它只解决了设备到设备（D2D）的缓存问题。
- Wang等人[10]讨论了一个关于移动边缘网络的广泛话题，包括移动边缘计算、缓存和通信。然而，他们的工作只涉及一般的缓存方案和应遵循的标准，没有讨论缓存过程的具体阶段。

我们还根据缓存位置、缓存标准和缓存方案提供了一个调查。特别是，本文还讨论了流量模式、用户社会关系、用户移动模式和内容流行对缓存策略的影响。我们还根据缓存位置、缓存标准和缓存方案提供了一个调查。

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220107113039.png)

# 二、MOBILE EDGE CACHING: AN OVERVIEW

## 1、Mobile Edge Caching

在移动边缘缓存中，由用户设备（UE）发出的内容请求由其中一个包含所请求内容的节点来回应。通常，域名系统（DNS）会将用户的请求重定向到最近的能够满足该内容的缓冲区。

移动边缘缓存有几个优点：

1. 首先，由于移动边缘缓存是在网络边缘进行的，它比远程互联网内容服务器更接近用户，它减少了获取用户 **请求内容的延迟** 。
2. 第二，移动边缘缓存避免了通过回程链路进行的数据传输，因此减少了回程流量。
3. 第三，移动边缘缓存有助于减少能源消耗。例如，当请求的数据被缓存在小蜂窝基站时，可以避免从宏基站传输数据的能量消耗。
4. 第四，通过移动边缘缓存可以提高频谱效率。例如，当多个用户请求相同的内容时，提供服务的BS可以通过组播来传输缓存的文件，这样可以 **共享相同的频谱** [11]。
5. 第五，移动边缘缓存可以利用移动边缘服务器收集的 **网络信息** （如用户偏好、文件流行度、用户移动信息、用户社会信息和信道状态信息）来提高缓存效率。 **例如，可以挖掘用户的社会关系，通过**** D2D ****通信来缓存和传播内容** 。

## 2、关于移动边缘缓存的问题

### 1). Where to caching

可以缓存在UEs（D2D情景中的用户侧），可以缓存在BSs（一般情景中的服务器侧）

### 2). How to caching

如何缓存涉及到选择缓存标准和设计缓存方案的问题。

1. 首先，缓存命中率要高。缓存命中率指的是用户请求的缓存文件数量与缓存中文件总数的比率。
2. ~~第二，~~ SE和EE是5G的主要性能指标，缓存方案的设计应改善这两个指标。
3. 第三，应考虑尽量减少内容检索延迟（本质上就是延迟，只不过不是内容交付的延迟，是一开始在服务器中检索内容的延迟），因为它直接关系到用户的体验质量（QoE）。
4. 第四，在边缘缓存流行的内容可以卸载回程链路的流量，因此最大化流量卸载（这个就是交付延迟）可以是缓存的标准之一。

有多种缓存方案：

在决定是在内容被请求之后还是之前进行缓存方面，提出了：

- 反应式缓存
- 主动式缓存。

根据缓存决策的位置，缓存方案可以分为：

- 集中式缓存。集中式缓存使用中央控制器来决定所有的内容放置方案
- 分布式缓存。分布式缓存只知道相邻UE或相邻BS的信息，并对本地缓存做出决定。

几种常见的解决方案：

由于边缘节点的缓存空间有限，为每个节点单独设计缓存策略可能会导致缓存的利用率不足。 **合作式缓存** 可以应对这个问题，因为不同的缓存节点可以互相分享内容。未被充分利用的缓存可以被其他节点使用，因此所有缓存节点的利用率可以得到提高。

**编码缓存** 利用了网络编码技术，将数据信息汇总（编码），转发到同一目的地，然后分离成不同的信息（解码）。这种技术可以通过减少传输次数来提高网络吞吐量，减少延迟。

**概率缓存处理** 的是用户位置的不确定性和移动问题。基于博弈论的缓存被用来分析不同服务提供商（SP）之间的合作和竞争。

### 3). What to caching

注意这里说的what并不是原始理解的缓存了什么（去解决缓存视频还是多媒体），而是要缓存哪些（即去解决要缓存哪几个视频，哪几个文件）

内容类型包括多媒体文件（如视频和文件）和物联网数据，它们表现出更多的维度和更短的生命期。为了获得特定内容被请求的概率，我们需要估计内容的流行度和用户的偏好。

# 三、缓存放置（Caching Location）

\&gt; 这里不是我关注的重点，略过！

## 1、五个位置：

1. **Caching at UEs**** （**D2D网络允许UE之间使用许可带（如LTE）或非许可带协议（如蓝牙和WiFi）进行直接通信。这些设备通常被组织成集群，由BS控制。一个用户的内容请求只能由同一集群内的其他用户满足。用户的物理位置在设计D2D缓存策略时起着重要的作用，这样用户就可以很容易地在某个相邻的用户处找到其请求的内容。）
2. **Caching at BSs** （通过在MBSs、SBSs、PBSs和FBSs安装缓存存储器，BSs的缓存可以减少回程流量，因为用户可以直接从BSs而不是从远程互联网内容服务器获得所需的内容。）
  1. **Caching at MBSs** （MBS通常比SBS有更大的缓存存储空间和覆盖区域，在MBS的缓存目标通常包括最小化回程流量、网络延迟和成功传输概率。）
  2. **Caching at SBSs** （通常采用合作式缓存（即不同的SBS可以相互分享内容）。缓存内容应根据动态内容的流行程度更频繁地更新。）
  3. **Caching at PBS s** （PBSs和FBSs是特殊的SBSs。PBS需要与MBS连接的高速回程链路，可能会产生很高的流量成本。）
  4. **Caching at FBSs** （FBSs的回程链路通常比PBSs的小，甚至可能有无线回程链路。因此，FBS的部署很灵活，成本效益高，但覆盖范围较小。）
3. **Caching at Relays** （中继通常被部署来扩大无线覆盖范围或作为城市热点。在为中继站设计缓存方案时，应考虑到中继站的能量消耗和它们的位置。）
4. **Joint Caching in HetNets** （HetNet通过密集地部署SBS来提高区域频谱效率。它们可以共享相同的频谱。）
5. **Caching in C-RAN** （C-RAN通过采用云计算技术将计算能力聚集到BBU池中，这为无线网络带来了灵活性和敏捷性。）

# 四、CACHING CRITERIA

## 1、Cache Hit Probability

缓存命中率指的是用户请求的缓存文件数量与缓存中文件总数的比率。较高的缓存命中率意味着更多的用户请求被缓存的内容所满足。增加缓存大小可以提高缓存命中率，从而降低所需的回程容量。

有如下几种方式：

- SBS可以形成一个集合，并相互分享内容，以提高缓存命中率。
- 最好用的地方是D2D，用户之间相互分享。当用户密度高时，D2D缓存的性能更好，因为更多的用户请求可以通过短距离的D2D通信同时得到服务。否则，SBS缓存更有利，因为SBS的缓存存储量通常更大，因此可以提高缓存命中率。
- 一般的场景是，一个用户向同一个区域内的多个运营商链接；多个用户按照预定的概率连接到不同的运营商。

## 2、Spectrum Efficiency

\&gt; 这一块同样不是关注重点，看一下它的核心思想！

SE是指在一个给定的频率带宽上支持的数据速率。为了改善区域SE，通过在宏蜂窝中部署更多的SBS来实现网络密集化，已经得到了利用。缓存也可以通过减少网络流量和提高网络吞吐量来改善SE。

## 3、Energy Efficiency

EE被定义为在给定能耗下支持的数据速率，是5G蜂窝网络的一个重要性能指标。当启用缓存时，EE会更高，因为它有助于减少回程流量，而回程流量会造成能量消耗。

Perabathini等人[87]讨论了如何优化BS的发射功率，以最大限度地提高缓存蜂窝网络的能源效率。Almomani等人[88]为叠加小单元内容缓存网络提出了一个启发式方案，以确定与每个UE连接的SBS，从而使所有UE的能量消耗最小。他们提出的方案是基于粒子群优化（PSO），其中利用一些粒子来迭代计算最优解。

## 4、Network Throughput

网络吞吐量被定义为网络能够提供的最大数据速率，并影响到网络性能。较大的网络吞吐量会导致较低的内容下载延迟。

Yang等人[69]对RAN缓存和D2D缓存共存的性能进行了建模和评估，目的是使网络吞吐量最大化。我们证明，与传统的非缓存方法相比，网络吞吐量可以增加57%。Khreishah等人[57]考虑了协调小蜂窝系统上视频传输的 **联合缓存、路由和信道分配问题** ，目标是使系统的吞吐量最大化。

## 5、Content Retrieving Delay

内容检索延迟，通常定义为用户获取内容的往返时间，与用户的QoS直接相关[94]。它通常由从BSs到UE的无线传输延迟和从BSs到移动核心网的回程延迟组成。

在没有中央协调人的情况下，Liu等人[91]提出了一种分布式算法来解决联合内容放置和传输问题，以最小化平均下载延迟。大多数工作没有考虑到请求转发问题。Dehghan等人[92]联合优化了内容放置和用户请求转发问题，以确定哪些内容应该被缓存在每个缓存中，以及如何转发用户请求以最小化平均内容访问延迟。

## 6、Traffic Offloading

\&gt; 这个也不是关注的重点，简单说一下。主要就是用户决定内容防止的问题。

移动边缘缓存可以卸载回程链路的流量。最大化的流量卸载可以带来更好的缓存性能。

在缓存存储能力和回程链路限制下，流量负载的预期总和最小化。流量负载发生在PBS和用户、PBS和PBS、MBS和用户以及移动核心网和MBS之间。设计了一个次优的贪婪算法来获得最佳的内容放置决策。

# 五、CACHING SCHEMES

## 1、Proactive Caching（主动缓存）

主动式缓存通过在非高峰期预先下载热门内容并为可预测的高峰期需求提供服务来提高缓存效率。与之相对的，反应式缓存策略，它决定了是否在某一特定内容被请求后进行缓存，无法有效地应对高峰流量的情况下。

主动式缓存策略根据对用户需求的预测，决定哪些内容应该在被请求之前被缓存[95]。主动式缓存通常利用对请求模式的估计（例如，用户移动模式、用户偏好和社会关系）来提高缓存性能并保证QoS要求。

Bastug等人[32]提出了一种主动网络范式，该范式利用社交网络和内容流行度分布，在满足请求的数量和卸载流量方面提高缓存性能。他们证明了主动式缓存比被动式缓存表现得更好。Tadrous等人[96]考虑了服务的受欢迎程度可以被预测的系统。缓存节点可以根据服务的受欢迎程度，在非高峰期主动缓存服务。他们通过考虑资源分配来探索主动缓存方案，以最大限度地减少成本，这与主动缓存产生的卸载流量有关。为了进一步提高主动缓存的性能，最好是在多个节点之间联合优化缓存。Hou等人[97]利用一种基于学习的主动缓存方法来最大化缓存命中率。在他们的系统模型中，不同的缓存可以共享信息和内容。他们首先通过学习方法估计了内容的受欢迎程度，然后设计了一个贪婪的算法来获得次优的内容分配方案。然而，缓存的性能高度依赖于预测的准确性是主动缓存的主要缺点。

## 2、Distributed Caching（分布式缓存）

集中式缓存能够通过最佳的缓存决策（例如，内容放置）实现最佳的缓存性能。然而，获得完整的网络信息是具有挑战性的，特别是在动态5G无线网络的背景下，预计将为越来越多的移动用户提供服务[99]。

在分布式缓存中，也被称为分散式缓存，缓存节点只根据其本地信息和相邻节点的信息进行决策（例如，内容放置和更新）。分布式缓存在[49]中得到了应用，相邻的BS被联合优化以提高缓存命中率。通过从多个相邻的缓存中获取内容，可以增加从用户那里看到的总缓存大小。相信传播（BP）方法已经被提出来作为一种有效的方法来分布式地解决无线网络中的资源分配问题。在BP中，复杂的全局优化问题通常被分解成多个子问题，这些子问题可以以并行和分布的方式有效解决。

## 3、Cooperative Caching

\&gt; 协作缓存可以看做是分布式缓存的加强！协作缓存不仅需要根据本地信息（如用户请求和CSI）和邻近的信息来制定相关缓存策略，还要求别的SBS共享其缓存资源。

由于BS的缓存空间相对较小，为每个BS独立设计缓存策略可能会导致缓存的利用率不足。当一些缓存被过度使用，而其他的缓存有很多空位时，就会发生这种情况。

在合作式缓存中，BSs能够相互分享缓存内容[98]。然而，从其他缓存中搜索和检索内容的延迟可能也很严重，因此应予以考虑。大多数关于合作式缓存的研究都假设了静态的流行度；对合作和学习时间变化的流行度的共同考虑仍然需要进一步研究。（协作缓存的一个问题就是最优解的求法，当流行度不断变化的时候，可能找不到最优解；还有一个问题就是隐私安全）

## 4、Coded Caching

\&gt; 这个不是关注的重点，略过！

在传统的交换网络中，网络节点一个接一个地转发数据包：两个数据包同时出现在节点中；两个数据包中的一个被转发，而另一个被排队，即使两个数据包都是去往同一个目的地。这种传统的数据包转发机制需要单独传输，因此降低了网络效率。网络编码是一种技术，它将两个独立的信息合并成一个编码信息，并将其转发到目的地。在收到编码信息后，网络节点将它们分离成两个原始信息。这种方案需要编码和解码过程，因此给网络节点带来了更多的处理开销。网络编码的复杂性可以通过有效的数据包传输来降低[104]。

## 5、Probabilistic Caching

与具有固定和已知拓扑结构的有线网络不同，由于用户位置不确定和用户请求的差异，无线网络面临着哪个用户将连接到哪个BS的不确定性。解决这个问题的方法是采用概率缓存策略，在这个策略中，内容可以根据一些随机分布放在缓存中。

Blaszczyszyn和Giovanidis[85]将用户位置建模为一个空间随机过程。他们优化了每个BS的每个内容被缓存的概率，目的是最大化缓存的命中率。他们还证明，广泛使用的贪婪算法，即缓存最受欢迎的文件，在一般的网络中不能总是保证优化，除非没有BS覆盖重叠。随机缓存策略，即用户对文件提出任意的请求，随着网络规模的增长，表现得更好。

## 6、Game Theory Based Caching

\&gt; 这个就是把缓存资源的价格考虑进行，进行一个博弈的分析。本质上还是传统的问题

例如，给BS带来更多的内容对用户有利，而由于额外的存储和电力消耗，会增加MNO的成本。由于每一方都只关心自己的利润，他们之间的竞争是不可避免的。为了有效地应对竞争并保证较高的整体用户体验，我们采用博弈论来分析这些各方之间的互动关系。

Hu等人[109]应用博弈论分析了不同方的自私性如何通过考虑不同方之间的关系和互动来影响整体缓存性能。他们考虑了两种情况，包括SBS缓存和D2D缓存。在前一种情况下，多个SP的目标是将自己的内容缓存到具有有限缓存存储空间的SBS中，并提出了一个拍卖游戏来解决这个问题。对于后者，他们采用了一个联盟博弈来分析如何形成一个合作小组来共同下载内容。在[12]中，他们通过引入缓存作为一种服务的概念扩展了他们的工作，他们利用了无线网络虚拟化技术，每个SP必须为MNO拥有的SBS缓存存储空间付费。他们提出了一种多对象拍卖机制来描述SP之间的竞争。由于所有的SP都倾向于缓存更多的内容以提高服务性能，他们打算作为竞标者，竞争有限的缓存存储空间。功用函数与平均内容下载文件有关。他们的机制是通过一系列的拍卖进行的，由市场匹配算法解决[110]。

# 六、CONTENT REQUEST ANALYSIS

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220107194313.png)

这一部分其实主要就是针对主动缓存这种缓存模式而言的，content types（针对不同的缓存内容，要求不一样——文件就是时间容忍的，主要是确保交付；视频就是要时间灵敏，不卡顿；物联网设备就是要注意能耗和容量）、request patterns（观察过去的请求到达情况是获得请求模式的一个可行的解决方案）没什么好说的。

## 1、Content Popularity

内容流行度被定义为对某一特定内容的请求数与用户请求总数的比率。它通常是在某一特定时期内为某一地区获得的。内容流行度的关键特征是，大多数人在一定时间内对少数流行的内容感兴趣，因此这些少数内容占了主要的流量负荷。Li等人[120]指出，在中国的移动网络中，优酷前5%的视频贡献了80%以上的内容。一般来说，内容流行度分布的变化速度相对较慢[49]。因此， **内容流行度的分布通常被认为是一个长时间的常量** （例如，电影是一个星期，新闻是两三个小时）[49]。此外，在一个大的区域如城市或国家的全球流行度往往与在一个小的区域如校园的本地流行度不同[95]。大多数作品利用Zipf分布来描述内容流行度。然而，Zipf仅被证明在视频下载中是准确的，可能不适合描述其他数据类型，例如物联网数据。

已经提出了一些时间序列模型，如自回归综合移动平均线[122]、回归模型[123]和分类模型[124]。这些基于模型的预测方案通常是通过机器学习方法进行的。

## 2、User Mobility Pattern

\&gt; 这个也不是特别重要，因为移动性只能作为模型设置时候的参变量，它能进一步影响社会关系，如果要从这个出发，相当于建立一个移动模型，问题会比较复杂。还不如直接让移动性作为参数，即用户移动模式一般被分为两类，包括空间和时间属性，分别反映了基于位置和与时间相关的特征[135]。

移动性是移动网络中需要考虑的一个重要因素，因为它随着时间的推移影响着移动网络的拓扑结构（例如，不同的用户-BS关联）[130]。移动网络的延迟可归因于移动性导致的不可预知的拓扑变化[131]。用户的移动性也包含许多有用的信息（如社会关系和交通模式），可以利用这些信息来提高缓存性能。

用户移动模式包括空间属性和时间属性。空间属性取决于物理位置。常见的模型是随机航点模型。时间属性反映了用户移动的时间相关特征，这些特征通常由接触时间和相互接触时间衡量。由于有强大社会关系的用户往往有更多相似的移动模式，通过分析社会关系来估计用户移动模式需要进一步研究。

Poularakis和Tassiulas[132]讨论了Femtocaching网络的存储分配问题。由于移动后的未来位置与当前位置高度相关，用户移动性可以用马尔可夫链来建模。由于具有更多类似移动模式的用户往往具有更强的社会关系，因此用户移动模式依赖于社会关系[133]。

# 七、CONTENT EXPLORATION

## 1、Content Query Problem

\&gt; 针对用户请求的问题也不是考虑的重点，它们主要在解决SBS之间的干扰、内容放置、用户关联的问题。

在收到内容请求后，将执行一连串的步骤来处理该请求。一般来说，UE首先在其本地缓存中搜索请求的内容。如果该内容没有被缓存在其本地存储中，它将在邻近的UE中搜索该内容。如果用户在其他UE中找不到所需的内容，该请求将被发送到小区域的BS（如FBS、PBS、SBS），然后再发送到MBS。如果在任何一个BS不能找到所请求的内容，它将被转发到移动核心网络。在最后的情况下，这个请求将通过互联网转给内容提供商。（一层层地向上提交请求）

oularakis等人[139]提出了一种近似算法，以联合优化SBS缓存网络中的用户关联和内容放置问题，目标是使SBS服务的请求最大化。该问题受到缓存存储和SBS回程能力的限制。他们在[140]中进一步研究了在支持缓存的SBS网络中视频传输的用户关联和内容放置的联合问题，以最小化平均用户体验延迟。然后设计了一个拉格朗日松弛算法来解决这个问题。然而，他们的工作都假设不同的SBS工作在正交的频段，因此忽略了SBS之间的干扰。考虑到干扰，Wang等人[68]联合优化了SBS缓存网络中的内容放置和用户关联问题，以最小化平均下载延迟。他们证明这个联合问题是NP-hard的。然后他们分离了耦合变量（即缓存变量和用户关联变量）以降低问题的复杂性。然后设计了一个拉格朗日松弛算法来解决这个转化的问题。

## 2、Content Placement Problem

内容放置问题始终是任何缓存方案中最重要的问题。它决定了 **哪些内容应该被放置在每个缓存中** 。它还解决了每个缓存的大小以及缓存在不同网络节点的位置。请注意，将内容分配到缓存节点会给网络带来流量；在设计内容放置策略时，应考虑额外的流量开销。

Golrezaei等人[54]研究了女性缓存网络中的缓存策略，以便在辅助缓存存储的约束下最小化平均文件下载延迟。用户可以从具有低速率回程但高存储容量的帮助器或拥有整个文件库的BS中获取内容。他们将平均下载延迟最小化问题转化为高速缓存命中率最大化问题。然而，他们的工作没有考虑无线信道条件，并假设用户和帮助者之间的延迟是不变的。Song等人[143]则考虑了女性高速缓存网络中的无线信道条件。他们将内容放置问题制定为一个ILP模型，并证明该模型的闭合形式解很难得到。然后他们设计了一个贪婪的算法来获得次优解。在他们的模拟中，他们证明了缓存最受欢迎的文件并不总是最好的选择，而且无线信道条件也会影响缓存的性能。另一方面，Peng等人[144]通过考虑回程限制，研究了在一个支持缓存的无线网络中的内容放置问题，其中BSs配备了缓存，如果文件没有被缓存，中央控制器可以将文件传输给用户。他们的目标是最小化文件传输延迟，包括回程链路和无线传输的延迟。他们首先将这个问题表述为一个混合整数非线性编程问题，然后设计了一个基于松弛的启发式算法来获得次优解。用户的社会关系会影响移动边缘缓存的内容放置策略。

他们还讨论了四种内容放置策略：

1）基于用户联系信息的全局内容放置；

2）基于各自社区的社会意识内容放置；

3）基于用户自身兴趣的个人内容放置；

4）随机内容放置。

用户流动性也是内容放置问题的一个关键因素。用户可以从一个地方移动到另一个地方，并在一个小区停留一定的时间（即小区停留时间）。在小区逗留时间内，用户与一个BS相关联。因此，在这段时间内，用户只能接收来自某些BS的数据。因此，小区停留时间会影响BS的内容放置策略。

由于放置决策涉及0-1的变量，大多数工作将这个问题制定为ILP模型，通常通过设计启发式算法获得次优解来解决 ~~。~~~~ D2D缓存中的内容放置问题通常考虑用户的社会关系，这影响到用户的接触时间和位置距离。 ~~然而，大多数工作忽略了缓存内容所造成的额外流量开销。仍然需要考虑~~ 开销的内容放置策略~~。

# 八、CONTENT DELIVERY

这一块的内容主要是解决怎么把内容发给用户，所有有很多计算机网络的信息，涉及到传输功率、信道频率、频谱利用、信道干扰等信息，比较硬核。

## 1、Multicast Transmission（多播、组播）

多播可以通过广播向多个目的地同时传输内容。通过组播，一个BS可以同时为要求相同内容的多个用户提供服务。因此，组播可以帮助减少重复传输和能源消耗。用户的请求可能是在不同时间发起的。

组播方案应该考虑不同的数据类型。对于小规模的网络，他们提出了一种贪婪的算法来提供内容交付。对于大规模网络，他们开发了一种多播感知的集群内合作缓存算法，其中不同的SBS可以相互分享内容。Zhou等人[147]没有利用组播来优化缓存策略，而是试图在由MBS和SBS组成的HetNets中优化给定内容分布下的组播内容交付策略，以最小化平均网络延迟和功率。MBS提供了网络的全面覆盖，并产生了较高的功耗。

\&lt;font size = &#39;5&#39; color=&#39;red&#39;\&gt;注意重点：\&lt;/font\&gt;这些用户在同一时间使用相同的频率资源在同一小区请求相同的内容。然而，请求不一定及时到达。因此，提 **供服务的**** BS ****必须延迟一些请求，在一定的时间窗口内收集多个请求，然后在同一时间为多个用户服务。因此，如何确定这个时间窗口是至关重要的，以便不损害用户的**** QoS ****。** 较长的时间窗口意味着在同一时间可以为更多的用户提供服务，从而提高频谱效率。相反，较短的时间窗口会导致较短的延迟，从而提高用户的服务质量。因此，在确定时间窗口时，要在频谱效率和用户服务质量之间进行权衡；

## 2、Relay Multihop（中继多跳）

\&gt; 这里就是针对D2D网络模式的。

D2D传输使一个UE能够通过D2D链接从附近的UE获取内容。较高的UE密度可以导致较高的概率，可以提供所需的内容。通过D2D中继的多跳传输允许邻近的UE作为中继进行内容传输。这种基于中继的机制允许更广泛的内容交付。此外，当请求的内容被缓存在多个UE中时，它们可以合作交付内容，以提供更高的传输率。Xia等人[149]研究了UE通过多跳中继合作下载内容的情况。具体来说，他们的方案将UE分组为多播组，这些组可以通过无线多播传输从BS下载文件。然后，每个组播组可以作为中继，向其他组播组传输文件。因此，内容传输是按组进行的。他们讨论了如何形成有效的群组以最小化所有UE的功耗的问题。他们通过模拟证明，在多跳D2D网络中，通过对UE进行分组，可以大大节省总功耗。

## 3、Joint Optimization of Content Placement and Content Delivery

在实践中，内容放置和内容交付会相互影响。一方面，内容放置决定了内容的分布，并影响到内容交付的路径。另一方面，在很长一段时间内，内容交付的统计数据可以被用来探索流行分布。 **缓存可以根据这些统计数据进行定期更新** 。

Maddah-Ali和Niesen[105]联合优化了缓存和编码组播传输，并证明缓存和传输的联合优化可以提高缓存性能。Cui和Jiang[61]联合考虑了由MBS层和PBS层组成的具有缓存功能的两层HetNet中的缓存放置和多播传送问题。他们考虑了MBS层中相同的缓存和PBS层中的随机缓存。在MBS层中，所有的MBS都缓存同一组文件。在PBS层中，所有的PBS随机缓存不同的文件，除了已经在MBS缓存的文件。

# 九、CONTENT UPDATE

使用过时的信息（如内容流行度、用户位置、网络流量负载等）的缓存策略可能会降低性能，因为它可能无法反映当前的网络状态。因此，每隔一段时间更新缓存是至关重要的。缓存更新过程一般发生在内容交付完成之后。内容替换问题是关于哪些内容应该从缓存中删除，何时删除以及如何缓存新内容。

已经提出了几种内容替换策略，如最不常用（LFU）、最近最少使用（LRU）以及它们的变体[150]。LRU更新每个缓存以保留最近请求的内容，而LFU保留最频繁请求的内容。

在请求到达时立即执行的方案（如LRU和LFU），被称为在线方案。然而，在线方案只考虑最近和当前的状态，可能无法获得全局最优解决方案。因此，未来的内容流行度也应该被考虑在内。Li等人[152]提出了一种新的缓存替换方法（PopCaching）来学习和估计内容流行度。然后他们确定哪些内容应该从缓存中驱逐。他们证明了PopCaching能快速收敛并接近最佳缓存命中率。

**注意：** 过于频繁地更新内容会给网络带来很大的流量。相反，如果长时间不更新缓存，缓存可能无法满足用户需求。因此，网络流量和内容更新频率之间的权衡问题还有待解决。

# 十、CHALLENGES AND FUTURE WORK

\&gt; 这里只着重分析一下用户隐私的问题。

为了充分挖掘移动边缘缓存的潜力，应该考虑无线网络中的独特挑战，例如，不确定的信道、干扰、用户移动性、有限的UE电池寿命和 **用户隐私** 。在本节中，我们将讨论移动边缘缓存所面临的这些挑战，并确定未来的研究方向。

**第一阶段的防御：**

为了防御攻击，应该设计安全协议。Mukherjee等人[157]调查了雾计算的安全和隐私问题。Kim等人[158]解决了名称数据网络中的内容中毒攻击问题。名称数据网络是一种新的网络范式，它使用户能够通过指定内容名称而不是IP地址来请求内容。Leguay等人[159]提出了一个安全协议，以实现内容交付网络中加密内容的缓存。

**第二阶段的防御：**

移动边缘所面临的无线信道和移动流量更加动态，由于无线网络的广播性质，传输容易受到各种恶意攻击，包括被动窃听的数据拦截和主动干扰的合法传输。窃听者可以轻易地偷听到无线通信会话。传统的方法是利用加密技术来防止攻击者截获数据。干扰攻击通过恶意产生干扰来破坏合法传输。

**第三阶段的防御：**

保护隐私的传统方式是匿名化，即对传输数据进行加密或删除个人身份信息。然而，大数据技术的进步使攻击者有能力将匿名数据与从用户那里收集的非匿名公共数据（如购物和阅读偏好、位置和照片）结合起来，以识别个人[162]。此外，通过观察和分析数据，用数据挖掘和机器学习技术，可以直接嗅到和提取敏感数据。能够防止攻击者提取敏感信息的缓存策略是人们迫切追求的。 **用户隐私与信息共享（如缓存内容、用户偏好、社会关系和用户移动模式）相冲突** 。当获得内容流行度以提高缓存性能时，应提取内容信息并分析其与用户隐私的关系。因此，用户隐私条例应该被很好地设计，以平衡用户隐私和缓存性能之间的权衡。