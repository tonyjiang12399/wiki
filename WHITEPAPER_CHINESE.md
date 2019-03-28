# Livepeer 白皮书

**去中心化视频流媒体网络-协议和经济激励**

Doug Petkanics <doug@livepeer.org>  
Eric Tang <eric@livepeer.org>

## 摘要 ###########################################

Livepeer项目旨在提供完全去中心化，高度可扩展，和加密令牌激励的实时视频流网络协议，并产生可作为分散式开发（web3）堆栈中的实时媒体层的解决方案。 此外，Livepeer旨在为任何现有的广播公司提供经济高效的集中广播解决替代方案。 在本文档中，我们描述了Livepeer协议 - 一种基于权利的委托协议，用于以游戏理论上安全的方式激励实时视频广播网络中的参与者。 我们提出了去中心化工作的可扩展验证解决方案，以及防止无用工作以试图在通货膨胀系统中对令牌分配进行游戏。


## 目录 ###########################################

* [介绍和背景](#introduction-and-background)
    * [实时视频堆栈](#the-live-video-stack)
* [Livepeer协议](#livepeer-protocol)
    * [视频片段](#video-segments)
    * [Livepeer令牌](#livepeer-token)
    * [协议角色](#protocol-roles)
    * [共识](#consensus)
    * [绑定+授权](#bonding--delegation)
    * [转码器() 事务](#transcoder-transaction)
    * [广播+转码工作](#broadcast--transcoding-job)
        * [预处理](#preprocessing)
        * [工作](#the-job)
        * [结束工作](#end-job)
    * [工作验证](#verification-of-work)
        * [关于Truebit的注释](#a-note-on-truebit)
    * [令牌生成](#token-generation)
    * [削减](#slashing)
    * [令牌分布](#token-distribution)
    * [治理](#governance)
* [攻击](#attacks)
    * [共识攻击](#consensus-attacks)
    * [DDoS](#ddos)
    * [无用或自我处理转码器](#useless-or-self-dealing-transcoder)
    * [转码器恶意破坏](#transcoder-griefing)
    * [连锁重组](#chain-reorg)
* [实时视频分发](#live-video-distribution)
* [用例](#use-cases)
    * [即用即付内容消费](#pay-as-you-go-content-consumption)
    * [自动扩展社交视频服务](#auto-scaling-social-video-services)
    * [不知不觉的现场新闻](#uncensorable-live-journalism)
    * [视频启用DApps](#video-enabled-dapps)
* [总结](#summary)
* [附录](#appendix)
    * [Livepeer协议参数参考](#livepeer-protocol-parameter-reference)
    * [Livepeer协议事务类型](#livepeer-protocol-transaction-types)
* [参考](#references)

*注意：本文最初发布于2017年4月.2018年12月提出了一个称为“Streamflow”的扩展提案，其中概述了下面提出的一些想法的一些迭代和增强。 阅读[Streamflow Proposal here](https://github.com/livepeer/wiki/blob/master/STREAMFLOW.md).* 

## 介绍和背景 ###########################################

在过去几年中，随着[以太坊](http://ethereum.org)等网络的出现，分布式网络的愿景开始实现，以实现无信任计算，[Swarm](http://swarm-gateways.net/bzz:/theswarm.eth/)和[IPFS / Filecoin](http://ipfs.io)启用去中心化存储和内容分发，比特币和各种令牌项目，以促进p2p价值转移，以及分散名称注册 比如[Blockstack](http://blockstack.org)和[ENS](http://ens.readthedocs.io/en/latest/introduction.html)，为内容和身份提供人类可访问的名称。 这些元素构成了去中心化应用程序（DApps）的基础，这些应用程序以大部分静态或不经常更新的Web或移动内容的形式构建，但目前DApps仍然缺乏以开放和去中心化方在对流媒体和数据的能力上。 Livepeer项目的目标是通过互联网去中心化实时视频广播。

[Livepeer Project Overview](https://github.com/livepeer/wiki/wiki/Project-Overview)提供了对互联网上现场视频当前状态的精彩介绍。 本白皮书将主要关注Livepeer的加密经济协议细节，而不是业务案例，但总的来说，概述描述了实时流式传输的快速增长，集中化和昂贵的现状。 另一方面，完全分散的P2P解决方案，其中节点贡献他们自己的计算和用于流直播视频的带宽将更加开放和可扩展，因为可以服务的连接数量没有限制。

这种技术肯定在某种程度上可用，但到目前为止，还没有动力让用户运行提供此功能的节点，也没有适当的资金来开发可以通过以下方式开发协议：使整个互联网受益，而不是一个集中的公司。然而，随着最近出现的加密令牌供电协议[[2, 3](#参考)]，现在有机会同时激励用户为实时视频广播贡献计算和带宽，其方式与融资方式一致。开发一种开放式媒体服务器解决方案，能够根据达到所有设备所需的所有最新标准和格式提供实时流式视频。此外，传统上被认为是令牌供电协议的结果的经济行为表明，为了使用Livepeer网络，广播公司的成本可能比任何集中式解决方案的成本更便宜。

随着Livepeer技术和协议的交付，它将使用户能够参与以下流程：

1. 在相机，手机，屏幕或网络摄像头上捕获视频，然后将其发送到Livepeer网络。
2. 在网络中运行的节点将其编码为所有必要的格式，以到达每个支持的设备。 运行这些节点的用户将通过ETH广播公司支付的费用进行激励，并有机会通过协议令牌建立声誉，以获得未来执行更多工作的权利。
3. 网络上的任何用户都可以请求查看流，它将自动近乎实时地分发给他们。

<img src="https://s3.amazonaws.com/livepeerorg/LPExample.png" alt="Livepeer Network Example" style="width: 750px">

### 实时视频堆栈

用于广播直播视频的技术栈已经发展了很多年并且包含许多层。 广播公司需要在源头捕获视频，与媒体服务器接口以处理视频并将其转码为多种不同格式，通过网络分发视频，然后允许最终消费者以高感知质量播放视频。 当人们通过这个堆栈思考时会引入经济问题，例如应该是广播公司还是应该为转移视频而支付带宽的消费者。

A typical live streaming platform today needs to support RTMP, HLS, Mpeg-Dash video formats in H.264 and VP8 codec. New codecs like H.265/HEVC, VP9, and AV1 will become more popular in the near future as consumers become more accustomed to higher video quality.  For HLS alone, [Apple suggests](https://developer.apple.com/library/content/documentation/General/Reference/HLSAuthoringSpec/Requirements.html#//apple_ref/doc/uid/TP40016596-CH2-SW1) bitrates from 145kb/s all the way up to 7800kb/s, in order to serve the different types of devices under different conditions. All of this adds a significant amount of complexity and cost to live video broadcasting.

The existing decentralized development stack (web3) contains solutions for some of the layers required for a live video platform, like file transfer and payments, but currently there are no solutions for the capture and interface, transcoding and processing, and serving layers of live video. For this, Livepeer introduces the [Livepeer Media Server (LPMS)](https://github.com/livepeer/wiki/wiki/Livepeer-Media-Server) - an open source implementation of a media server which provides all of the live video specific functionality necessary for DApp developers and existing broadcasters to build live functionality into their applications. [Read more about it here](https://github.com/livepeer/wiki/wiki/Livepeer-Media-Server).

As a standalone application, any developer could build a live application on top of the LPMS, but it would still be centralized and would need to be scaled through traditional means. However when every node on the Livepeer network is running the LPMS, and the protocol’s economic incentives ensure that those nodes will contribute their processing power and bandwidth in service of transcoding and distributing live video, **a self-scaling, pay-as-you-go network is made available to developers, who can simply send their live stream into the network, and have the implementation details of scaling, payment, and media hosting abstracted away**.

## Livepeer协议 ###########################################

Livepeer协议定义了直播流生态系统中的各个参与者如何以安全和经济合理的方式参与。 协议需要解决的两个主要领域是以高性能和可扩展的方式从源到大量消费者的实时视频的实际分发，以及在安全和游戏理论中鼓励参与网络的经济激励 方式。 虽然本白皮书将涉及与经济协议重叠的实时视频发布本身，但它将主要关注后者以展示安全性和经济一致性。 在最高级别，该协议旨在：

- 允许任何节点将实时视频发送到网络，并可选择付费以将其转码为各种格式和比特率。
- 允许任何节点从网络请求视频。
- 允许参与者为视频的转码和分发贡献其处理能力和带宽，并相应地进行补偿。

在去中心化的网络中，参与者按照他们贡献的工作量按比例获得奖励，确保安全性需要解决的两大挑战是：

- 是否可以验证节点所做的工作是否正确完成？
- 对于为网络贡献价值的实际工作，节点是否被授予，而不是为了不公平地获得令牌分配而进行的虚假工作？

Livepeer协议旨在解决工作验证和防止虚假工作的问题，同时还提供网络自动可扩展性的解决方案，并随着时间的推移在协议演变的治理中融入其中。

### 视频片段

Livepeer中媒体的核心单位是我们称之为`细分市场`。 段是时间长度为`t`的多路复用音频和视频的时间切片块。 Livepeer网络中的每个网段都是唯一的，并包含加密证据，以验证广播公司是否为此特定网段提供此特定数据。 每个流由许多连续的段组成，每个段包含标识其正确排序的序列号。 细分包含以下字段：

| 视频片段字段 | 描述 |
|--------|--------|
| **StreamID** | 标识此段所属的源节点和流。 |
| **SequenceNumber** | 此段属于原始流的顺序。 |
| **DataPayload** | 二进制元数据和表示此段中音频/视频的数据。 |
| **DataHash** | 数据有效负载的哈希值。 |
| **BroadcasterSignature** | 来自广告公司的`Priv（StreamID，SequenceNumber，hash（StreamID，SequenceNumber，DataHash））`的签名，可用于证明并验证广播公司声称这是该唯一段的真实数据。 |

Livepeer协议通常使用段作为转码，分发和支付的工作单元。

### Livepeer令牌

Livepeer Token（LPT）是Livepeer网络的协议令牌。 但它不是交换令牌的媒介。 广播公司使用以太坊的以太网（ETH）在网络上播放视频。 贡献处理和带宽的节点以广播公司的费用形式获得ETH。 LPT是一种权益令牌，参与者希望在网络上进行工作，以协调工作在网络上的分配方式，并提供安全性，确保工作能够诚实正确地完成。 LPT具有以下目的：

- 它作为委托证明利益系统中的绑定机制，其中利益被委托给参与协议的转码器（或验证器）以转码视频和验证工作。 由于协议违规而发生的令牌和潜在的削减是必要的，以便保护网络免受许多攻击。 更多下面。
- 它通过网络路由工作的比例与权益和委托令牌的数量成比例，基本上作为协调机制。
- 它是一个特定于Livepeer生态系统的帐户单元，它构成了SectorCoin概念的基础，适用于将来引入的其他功能[[4]（＃references）]。 DVR，隐藏式字幕，广告插入/货币化和分析等服务都可以插入Livepeer生态系统，并可能利用LPT提供的安全性。

将分发Livepeer令牌的初始分配，以便利益相关者可以在网络中履行各种角色并使用网络，然后将根据随时间推移的算法编程发布附加令牌。 请参阅[令牌分布]（＃令牌分发）部分。

遵循以太坊和许多流行的ERC20令牌[[16]（＃references）]的惯例，LPT将被10 ^ 18整除，较大的面额如LPT本身旨在用于诸如放样的用户级交易，以及 旨在用于协议会计的较小面额。

### 协议角色

在继续之前，让我们定义网络中的角色，以便讨论协议时有一个共同的词汇表。 Livepeer节点是运行Livepeer软件的任何计算机。

| 节点角色 | 描述 |
|--------|-----------|
| **Broadcaster** | Livepeer节点发布原始流 |
| **Transcoder** | Livepeer节点执行将流转码为另一种编解码器，比特率或打包格式的工作。 |
| **Relay Node** | Livepeer节点参与实时视频的分发和协议消息的传递，但不一定执行任何转码。 |
| **Consumer** | Livepeer节点请求stream，可能通过网关查看或通过其应用程序或DApp的用户提供服务。 |

除了运行Livepeer节点的用户所扮演的上述角色之外，该协议还将参考以下系统。 虽然我们使用某些特定系统来参考可能的实现，但如果它们提供类似的功能和加密经济保证，也可以交换替代系统：

| 系统的作用 | 描述 |
|-------|----------|
| **Swarm** | Content addressed storage platform. Data can be guaranteed to be available there temporarily during the verification process via SWEAR protocol [[7, 12](#references)]. *(Note in this document we refer to Swarm, but other content addressed storage platforms can be substituted if data availability can be guaranteed with high probability).* |
| **Livepeer Smart Contract** | Smart contract running on the Ethereum network [[1](#references)]. |
| **Truebit** | Blackbox verification protocol that guarantees correctness of computation placed on chain (at a hefty cost) [[6](#references)]. (<http://truebit.io>) |

下面是角色的可视化概述，以及它们在下面描述的工作验证过程中相互通信的方式。

<img src="https://livepeer-dev.s3.amazonaws.com/docs/lpprotocol.png" alt="Protocol Visual Overview" style="width: 750px">  

*片段从广播器流向译码器，最终流向消费者。译码员确保他们有签名和工作证明，以参与工作验证过程。*

**注释 代码转换器:** 转码器在Livepeer生态系统中扮演着最重要的角色。 他们正在采用输入流并将其及时转换为多种不同格式，以实现低延迟分发。 因此，它们受益于高可用性，高效，强大的硬件（可能具有GPU加速转码），高带宽连接和可靠的DevOps实践。 转码器应该比其他网络参与者流失得更少，因为当他们承担转码流的工作时，如果他们从网络中退出，它就不太理想了。 虽然网络可以扩展以支持许多参与者扮演转码器的角色（并获得必要的令牌分配），但这是从大多数网络参与者委派的特殊角色，以确保维持为广播公司提供价值的可靠网络。 以下是关于这个代表团的事。

### 共识

Livepeer有一个两层共识系统。 LPT分类账和交易由底层区块链（如以太坊）保护。 任何LPT令牌的转移或系统中的任何交易都可以被认为已经确认具有与基础工作证明或股权区块链证明相同的安全性。 然而，第二层决定了新生成的LPT的分布。 这由Livepeer智能合约管理，并由各方参与协议。 虽然每个发言都没有达成共识，但在接受和验证以前的版块方面，该协议规定了参与者的规则以及参与者因未能履行其职责而受到惩罚（削减）的条件。

管理新生成的令牌的第二级共识是基于委托的证明证明（DPOS），受Bitshares，Steem，Tendermint和Casper [[5,9,10,11]（＃references）]等系统的启发。 验证器在网络中的作用由转码器完成。 任何用户都可以将他们的利益委托给代码转换器，代码转换器然后需要在网络中执行转码作业，参与工作验证协议，并以特定的时间间隔在链上调用函数来验证这项工作。 该协议将分配费用和新生成的令牌，它将削减行为不端的演员的利益。 验证结果将在执行验证后通过Truebit在链上进行记录，因此广播公司和代码转换器之间不存在争议的余地。


### 绑定+授权

在Livepeer中，为了表明网络中的利益，节点必须绑定一定数量的LPT。 他们通过`Bond（）`交易来做到这一点，这将把他们在智能合约中的股份捆绑起来，直到他们`Unbond（）`，此时他们将进入一个无约束状态，这将持续“UnbondingPeriod”时间。 完成“UnbondingPeriod”后，他们可以撤回他们的LPT。

绑定金额用于将股权委托给Transcoder。 网络在任何时候都支持`N`个活动的代码转换器，这是一个可移动的网络参数。 任何节点都可以指示它希望成为具有`Transcoder（）`事务的Transcoder，并且协议将在每轮开始时选择具有最多累积权益（他们自己的+从其他节点委托）的`N`转码器。 ，以及来自等候名单的一个随机代码转换器。

Livepeer中新生成的令牌以与其绑定的工作量（减去费用）相对的比例分配给绑定节点，只要它们委派给根据协议行为的代码转换节点。 如果他们委派的节点不表现并违反其中一个削减条件，则可以削减债券（减少一定百分比）。 已经绑定并委托给代码转换器的节点也会收到代码转换器通过网络转码作业生成的部分费用。 从本质上讲，执行工作的节点可以获得广播公司为该工作支付的费用。

展望未来，当本文档使用术语“委托人”时，它指的是已将其权益委托给代码转换器候选者的绑定节点，而不是将其作为代码转换器委托给自己。

总之，参与者选择保留他们的股份，原因如下：

- 参与委派有效的代码转换器，为网络提供优质服务，确保其对广播公司的价值。
- 以分配令牌的形式按照股权比例建立声誉和未来工作分配。
- 赚取转码器产生的费用..
- 他们可能希望成为转码器。

### 转码器() 交易

节点通过提交`Transcoder（）`事务表明他们愿意成为代码转换器，该事务公开以下三个属性：

- `PricePerSegment`: 他们愿意接受转录视频片段的最低价格
- `BlockRewardCut`: 绑定节点将为转码服务支付的块奖励百分比。 （例2％。如果一个绑定节点在块奖励中接收100个LPT，那么2个LPT到转码器）。
- `FeeShare`: 代码转换器愿意与委托给它的绑定节点共享的广播作业的费用百分比。 （例25％。如果代码转换器要收取100 ETH的费用，它们将向绑定节点支付25 ETH）。

Transcoder可以在下一轮转码前的RoundLockAmount时间之前更新它们的可用性和信息。 这是一轮的百分比。 （示例10％= = 2.4小时。他们可以将此信息更改为下一个转码循环前的2.4小时，持续时间为`RoundLength`1天）。 这使得绑定节点有机会审查相对于其他代码转换器的费用分割和令牌奖励分割，以及基于它们收费和网络需求的费率的预期费用，并且如果他们愿意则移动他们的委托权益。 在转码循环开始时（通过调用`InitializeRound（）`事务触发），该轮的有效代码转换器是根据委托给每个代码转换器的总赌注确定的，赌注和利率被锁定持续时间 那一轮。

在`RoundLockPeriod`期间允许进行一项更改：任何候选代码转换器的最低提供价格/段都被锁定并且无法移动，但其他代码转换器候选者可以向下调整其价格/段。 这使他们能够匹配网络上的最低报价，以保证他们在网络上的利益加权份额。 在此期间，他们不得向上移动他们的报价。

以下是代理商在决定委派给谁时可以查看的代码转换器选项的示例状态。

| 转码器ID | 每段价格 | 阻止奖励削减 | 收费 |
|----|----|----|----|
| 1 | 22 wei | 1% | 25% |
| 2 | 30 wei | 2% | 40% |
| 3 | 10 wei | 4% | 1% |
| ... | ... | ... | ... |
| N | 14 wei | 0% | 2% |

*价格说明：在本文档中，我们列出价格/细分。 实际上，Livepeer计划使用一种gas会计启发模型，其中有一个段的某些工作参数所需的gas单位的概念，如比特率，编码，帧大小等。价格/段是一个支持，其中 激励措施是相同的，但实际上它们可能会传达价格/gas。*

### 广播+转码工作

Transcoders who are open for business on the network, throw their hat into the ring for transcoding work by submitting a `TranscodeAvailability()` transaction. This indicates their availability and places them into a pool of transcoders available to take a newly submitted job.

When a broadcaster submits their stream into the Livepeer network it is given a `StreamID`. This serves as both a unique identifier, and it also contains the origin node address so that nodes know how to request and route requests to consume this stream towards the origin. The stream contains many consecutive `Segments`, as described in the [Video Segments](#video-segments) section. If the broadcaster would like the network to take care of transcoding their stream into all the formats and bitrates necessary to reach every user on every device, then the first step is submitting a transcoding job transaction on chain. Jobs are given a unique ID as well, and the input data to job consists of:

`Job(StreamID, TranscodingOptions, PricePerSegment)`

The `TranscodingOptions` define the output bitrates, formats, encodings, etc, and the `PricePerSegment` lists the price that the broadcaster will offer.

As soon as this transaction is mined, the next blockhash will be used to pseudo-randomly determine the transcoder selected for this job. All transcoders with a price that’s lower than or equal to the price offered will be considered, and the blockhash modulus the number of candidate transcoders (weighted by their stakes) will determine the index of the selected transcoder.

At this point the broadcaster can begin streaming video segments towards the transcoder, and they’ll participate in the following protocol. The protocol also makes use of a persistent storage solution, for example Swarm, as part of the work verification process.

#### Preprocessing

1.  **Broadcaster**  -> **Livepeer Smart Contract**: submits a deposit on chain to cover the cost of the full transcoding job. This can be refilled later at any point, but the Transcoder may stop work if the deposit runs out as they gradually cash in for work done.

#### The Job

2. **Broadcaster** -> **Livepeer Smart Contract**: Job(streamID, options, price/segment)
    - Creates the job request on chain and places some ETH in escrow to pay for the work.
3. The protocol can use the next block hash to deterministically select the correct Transcoder for this job.
4. **Transcoder** -> **Broadcaster**: send output streamID and receipt that the job is accepted.
5. **Broadcaster** -> **Transcoder**: send stream segments, which contain signatures verifying the input data.
7. **Transcoder** performs transcoding and makes new output stream available on network
9. **Transcoder**: Store a transcode receipt for each segment of transcoding work. A transcode receipt has the following fields.

| Transcode Receipt Field | Description |
|-------|------------|
| **StreamID** | Identifies the origin node and stream that this segment belongs to. | 
| **Sequence Number** | The sequential order that this segment belongs in the original stream. |
| **Input Data hash** | The hash of the input segment data payload. |
| **Transcoded Data hash** | The hash of the output data after transcoding this segment. |
| **Broadcaster segment signature** | A signature from the broadcaster of Priv(StreamID, Seq#, Dhash) which can be used to attest and verify that the broadcaster claims this to be the true data for this unique segment. |
| **Transcoder segment signature** | A signature of all of the above fields from the transcoder attesting to the claim that this specific output transcoding was performed on this specific input. |

Whenever the transcoder observes that they are no longer receiving segments, they can call `ClaimWork()` to claim their work.

#### End Job

10. **Transcoder** -> **Livepeer Smart Contract**: Call `ClaimWork(JobID, StartSegmentSeq#, EndSegmentSeq#, MerkleRoot)`. Transcoder is claiming on chain they have performed work on the claimed segment range, with a merkle root of all of the transcode receipt data to commit to the content of these encoded segments.
11. Wait for this transaction to be mined, and observe the next blockhash. The protocol can then determine which segments will be verified based upon the `VerificationRate`.
12. **Transcoder** -> **Swarm**: Write input data payloads for the segments that will be challenged via verification, using SWEAR params to ensure the data will be there long enough for verification (`VerificationPeriod` time).
13. **Transcoder** -> **Livepeer Smart Contract**: Provide transcode claims on chain for each segment that needs to be verified, along with merkle proofs for the receipts for each segment in the transcode claims. The smart contract can verify the signatures from Broadcaster and **Transcoder** to ensure all data necessary is available to conduct verification, and can verify the merkle proofs against the committed merkle root from `ClaimWork()`.
14.  **Transcoder** -> **Truebit**: `Verify()`. This is an onchain call to the Truebit smart contract, where the Transcoder provides the Swarm input hash for the challenged segment. (More on verification in the following section)
15. **Truebit** -> **Livepeer Smart Contract**:  The result of the job is written on chain. This is compared to the transcoding claim result that the Transcoder provided.
16.  **Livepeer Smart Contract**: at this point the Livepeer smart contract has all the information it needs to determine if the Transcoder’s work is verified.
    - If verified correct, then use as input to token allocation algorithm and release of escrowed fees.
    - If incorrect, then Transcoder and its stakers get slashed `FailedVerificationSlashAmount` and the Broadcaster is refunded.

The Broadcaster can stop sending segments at any point, which effectively is an `EndJob()`.

At this point the transcoding has been performed, proof of the work has been claimed on the chain, and failure or success of the verification of the work has been reported. All the info is on chain to determine allocation of fees and token allocations to transcoders and delegators, or slashing in the case of failed verification. Let’s take a look at how work is actually verified.

### Verification of Work

In order to allocate fees to transcoders who claim that they have performed a transcoding job, it’s necessary that the protocol can determine that the job was actually performed correctly with high probability. For this, Livepeer extends the research of, and makes use of, the [Truebit Protocol](http://truebit.io) [[6](#references)].

Truebit works by having one participant (the solver) perform the actual work for the fee, in this case transcoding, and then having additional participants (verifiers) verify the work in order to detect mistakes, errors, or cheating. The task is broken down into very small steps, and the verifiers check the work of the solver to find the first step that differs from what they expected it to be. Then, only this one very small step needs to be played out on chain by a smart contract (judge), who can tell which party did the work correctly. The economic incentives, including forced errors to incentivize checking on the part of verifiers, ensure that it is not profitable to cheat or challenge incorrectly, but it is profitable to play the role of checking the work.

The downside of this protocol is that it costs between 5x-50x the cost of the original work in order to verify all work. Livepeer uses Truebit as a black box to verify segments, but it gets around having to pay this very high verification tax by only verifying a small percentage of segments randomly, and using slashing in the case of failed verifications. The `VerificationRate` set within Livepeer determines how frequently a specific segment is to be selected for challenge within Truebit, and the randomness of a future block hash after the work has been committed to the blockchain, determines which segments specifically are selected.

If work is committed via an `ClaimWork()` call in block `N`, then

If `Sha3(N, BlockHash(N), Seg#) % VerificationRate == 0` then the segment # must be verified.

The Transcoder provides Transcode Claims on chain for the candidate segments by invoking the `Verify()` transaction. The Livepeer Smart Contract can verify the authenticity of these claims using the internal signatures and provided merkle proofs, and then invoke a call to Truebit to verify only these segments.

Truebit solvers and verifiers access the input data for a segment from a persistent content addressed storage system, such as Swarm. The Transcoder is responsible for verifying that the segment data is available in Swarm, and can optionally look for receipts from the SWEAR protocol [[5](#references)] guaranteeing persistence for a certain period of time, which is long enough for Truebit to play out. Additionally, they can take it upon themselves to run a Swarm node ensuring that the data is available to Truebit verification. If they have reason to believe that data is not available in Swarm, they can provide it, or just call `ClaimWork()` on the previously available data.

Truebit will write the results of the computation (succeeded or failed) back to the Livepeer Smart Contract, which can then be used in the reward and slashing calculations within the protocol. A transcoding node can not predict in advance which segments will be verified, and the following penalties will be felt in the case of cheating or failing to transcode correctly:

- `FailedVerificationSlashAmount` will be slashed if they fail a verification from Truebit.
- `MissedVerificationSlashAmount` will be slashed if they fail to provide transcode claims and invoke Truebit on segments they were required to do so.
- Lost fee from the broadcaster.
- Not only will the Transcoder be slashed, but all their delegators will be slashed as well. They will take this account into their decision of who to delegate towards, and the Transcoder could lose the lucrative job they hold.

It is important that it be more profitable to simply stake LPT towards a valid, honestly performing transcoder, than it can be to cheat and take slashing penalties while still collecting fees and token allocations for dishonest work. Careful selection of the slashing params and verification rate can ensure this.

#### 关于Truebit的一个说明

*虽然协议使用Truebit来提供工作的完全不可信的验证，但是在实践中可能有必要使用可用的解决方案来提供验证，而不像Truebit在开发和测试期间所提供的不可信程度。根据不信任程度排序的选项包括:*

*1. 基于Livepeer API的Oracle -信任Livepeer来验证计算。非常集中，不适合测试之外的任何东西。*  
*2. Oraclize计算服务——信任一家提供计算证明的公司，这家公司的全部声誉依赖于将外部数据与未被篡改的证明放在chain上。*  
*3. 安全硬件飞地——像Intel SGX或TownCrier这样的服务提供可信的计算环境。相信他们的硬件实现是正确和安全的。这可以分散和审计。*


### 令牌生成

Livepeer的膨胀之处在于，随着时间的推移，新的令牌将根据[令牌分发](#令牌分发)中传递的时间表生成和分配。如果Livepeer中的所有角色都按照协议行事，那么新生成的令牌将按其绑定的股份(减去费用)的比例分配给用户。译码器的作用是调用` Reward()`函数，以触发新的令牌分配或削减，这可以从链上的所有可用数据计算出来。

每个转码器将被要求每轮调用`Reward()`一次。

-确保一个活跃的转码器正在调用`Reward()`。
-确保编译器在这轮还没有调用`Reward()`。
-根据`通货膨胀率`计算要铸造的令牌数目。铸造这许多代币。
-计算代码转换的削减基于他们的`块奖励削减`。
-分配到编码器的绑定桩。
-将剩余的分配到委派者奖励池中。
-将绑定的令牌数量更新到此转码器。

调用` Reward() `失败的直接后果是丢失了一部分令牌分配，并且在由委派者为该角色选出时，会对代码转换者的声誉造成损害。

### 削减

如前所述，削减的条件是：

- 验证失败
- 在需要时未能调用验证
- 根据委托的股权，不在平台内按比例分配所需的工作

在以太坊生态系统中构建的一个好处是，您可以在其他协议（如Truebit和Swarm / SWEAR）之上构建网络效果。 不幸的是，依赖于这些外部系统本身具有外部依赖性和激励性，其中一个协议的缺陷或弱点可能会导致Livepeer中的削减。

例如，如果Truebit验证作业在其队列中长时间没有任何解算器或验证者声明它，Livepeer将无法在调用`Reward（）`之前及时看到该验证的结果。 或者，如果Swarm网络遇到分区并且无法及时将文件传播到Truebit验证程序，那么这也可能会产生问题。

这些风险可以通过激励Livepeer协议中的参与者在内部播放这些角色来缓解，他们可能会发现它们最适合作为Truebit验证者或Swarm节点。 但是还有另一种方法是在削减参数上引入概率阈值的概念。 可以设置可选的协议变量，例如`VerificationFailureThreshold`，以指示只要节点通过99％的验证，它们就不会被削减。 这将是网络部署之前需要研究的另一个研究领域。

任何Livepeer协议参与者都可以检查和调用无法调用验证削减条件。 有一个`FinderFee`指定查找器将收到的斜线量的百分比，作为成功调用此削减条件的奖励。

根据协议的治理机制，剩余的大幅削减资金将进入`CommonPool`，可以被烧毁或分配给诸如进一步的生态系统发展等共同用途。

### 令牌分发

作为表示通过DPoS标记算法参与和执行网络工作的能力的标记，初始Livepeer标记分布将遵循需要广泛分布的创世状态的其他DPoS系统的模式。

令牌的初始分配将在创世纪和网络的早期阶段分发给社区。 收件人可以使用它来担任Transcoder或Delegator的角色。 一部分将分配给在起源之前为协议提供先前工作和资金的小组，并且一部分将被赋予核心项目的长期发展。

在网络启动时，令牌发布将根据通货膨胀计划继续进行，其中令牌在每轮`InflationRate`处生成，相对于令牌的未完成浮动。 由于令牌是按照议定书中所有参与者的股份比例发放的，因此可以激励积极参与。 由于获得了比例份额，参与者受到这种通货膨胀的“保护”。 只有不活跃的参与者才会坐在令牌上，而不会将其与参与者联系起来，他们会看到他们的比例网络所有权因这种通货膨胀而减少。


`InflationRate`的初始目标将被设置为其目的是激励LPT的大约`ParticipationRate`被绑定并积极参与[[19]（＃references））。 例如，如果`ParticipationRate`为50％，则存在激励以使一半的优秀代币被绑定。 通货膨胀率将在每轮算法上移动以提升参与目标。 较高的通货膨胀率会增加更多的保护标记，较低的税率将导致更多的人选择流动性而不是参与。 这是流动性偏好与网络所有权百分比权衡，由于网络中的许多经济因素，它应该找到均衡。

### 治理

Livepeer协议中的治理角色有三个方面：

1. 确定从行为不当的节点中削减的共同资金的消耗或挪用。
2. 调整网络参数以确保健康，蓬勃发展的网络，这对广播公司来说很有价值。
3. 以去中心化的方式调用提议的协议更新。

本文档中引用的许多网络参数，如`UnbondingPeriod`，`RoundLength`，`ParticipationRate`和`VerificationRate`都是可调整的。 可以提交对这些参数进行调整的提议，并且治理过程（包括代码转换器按其委托的比例进行投票）将确定在协议内自动采用这些更改。 治理的详细规范留给另一个文档。 [详见此处]（https://github.com/livepeer/wiki/wiki/Governance）。

## 攻击

本节包含对恶意行为者可能试图攻击Livepeer网络的各种方式的调查。 我们使用理性攻击者模型，攻击者根据自己的经济自身利益做出决策。 由于进行此类攻击无利可图，因此可以减轻许多攻击，但我们也努力确保在最坏的情况下，网络在持续无利可图的攻击中遭受效率降低，并且不会遭受失败。

### 共识攻击

如前所述，Livepeer生态系统的共识由底层区块链平台（例如以太坊）提供。 51％的攻击，Livepeer Token的双倍花费以及网络分支都需要与以太坊本身相同的资源和攻击成本。

Livepeer是一种基于赌注的协议，虽然转码器具有参与工作验证过程和令牌奖励分配过程的作用，但它们实际上不具有验证或接受其他转码器工作的作用。 没有链的概念，也没有先前块的验证。 简单地存在经济激励来验证自己的工作并在轮到它时分配一部分自己的令牌分配部分。 因此，在远程攻击，无任何问题和贿赂攻击等利益协议证明中看到的攻击不适用，因为没有机会尝试签署多个块或尝试创建 来自早期国家的较长链。 然而，人们应该意识到，当底层区块链迁移到股权证明时，如果在Livepeer上执行它们的好处超过对以太坊本身的攻击成本，这些攻击确实可能会破坏Livepeer。

虽然依赖底层区块链的安全性对于防止共识攻击很有帮助，但仍然存在一类可能损害Livepeer网络的质量和效率攻击。

### DDoS

Livepeer中的拒绝服务有两种方式：

1. 转码器可以尝试通过接受作业但拒绝转码来阻止或减慢广播公司将其编码流输出到网络。
2. 广播公司可以通过拒绝发送段来阻止转码器执行他们认为已分配的作业。

这两种攻击都有成本，可以减轻，轻微的烦恼。

在第一种情况下，Transcoder必须付费才能在链上声明其可用性。 如果他们因为不做这项工作而不会收取费用，那么他们就会把ETH扔掉。 广播公司可以重新提交作业并分配一个新的转码器。 可扩展性的一个潜在选择是协议可以按优先级顺序识别多个有效的代码转换器，而不仅仅是一个，这样，Broadcaster可以在没有其他链式事务的情况下继续前进。 此外，所有关于已接受作业的统计数据和转码/作业等的平均段数可以从链上数据计算，并且委托人将使用此作为他们决定委派给谁的决策的输入。 表现不佳将失去你的角色。

在广播公司阻止转码器工作的情况下，这仅仅是容量规划计算。 转码节点可以维护其并发作业容量的记录，作业处于活动/不活动状态的可能性，并确保它始终认为它将具有其声明的工作容量。 简单地忽略或在拒绝发送段的节点上调用`EndJob（）`几乎不会损坏转码器。

### 无用或自我处理转码器

如果Transcoder有足够的权益来维持他们的位置，他们理论上可以列出100％的`BlockRewardCut`，0％`FeeShare`，并收取高的`PricePerSegment`，这样他们就不会做任何工作，但可以收集他们的 令牌分配。 这可以通过`CompetitivenessTolerance`来阻止，这需要他们提供一些有效的工作。 另外，由于转码器参与协议所产生的交易成本，对于他们简单地将他们的令牌放到与他们共享费用的有效转码器上比将其作为无用转码器的人更有利可图。 不会收到任何费用。


输出无效输出的行为不端的转码器很快就会被削减到他们的权益降低到太低以至于无法真正保住工作并接受任何工作的程度。

### 转码器恶意破坏

如果广播公司想要使协议编码的协议非常昂贵，它可以发送代码转换器非连续的段号。 这是因为代码转换器可以在单个事务中声明连续范围的段号，但是必须使许多事务在随机段号范围内声明工作。 这可以通过以下选项进行辩护：

1. 转码器调用`EndJob（）`并且不打扰做工作或试图收取费用。
2. 协议实现链解析或更好的段索赔编码，以减少与在单个呼叫中声明非连续段相关联的费用。
3. 简单地忽略细分，永远不要求工作。

这种攻击对广播公司来说成本很高，因为他们必须有存款并在链上提交作业，以便首先分配给代码转换器。 它们能够使代码转换器生命烦恼并可能降低效率，但不会对网络造成损害。

### 连锁重组

当广播公司向Livepeer Smart Contract提交作业时，协议使用当前块哈希来确定将为哪个转码器分配作业。在这种情况下，底层区块链的重组可能会引起混淆。 虽然这不是直接“攻击”，但代码转换器将在一秒内有效，然后在重组时将不再有效。 当检测到重组时，广播公司可以将流重定向到新的有效代码转换器，或者协议可以检测主链中包含的叔块，并且如果给定阈值内的叔块具有该代码块，则认为代码转换器是有效的 使它们有效。

## 实时视频分发 ###########################################

本白皮书主要关注确保实时视频正确转码的经济激励和协议，这对于支持自适应比特率流并到达每个设备是必要的。 但同样重要的是在整个网络中分发视频，以便能够以高质量和低延迟消费。 分销经济依赖于Bittorrent推广的针锋相对的带宽计算，并通过SWAP [[13]（＃references）]等协议进行扩展。 作为简化，节点付费以请求一段视频，并且节点获得付费以服务一段视频。 如果一个节点已经有一个段并且可以将它提供给多个请求者，那么它是有利可图的。 我们将这种类型的节点称为中继节点。

当涉及到在网络中扮演不同角色的节点的带宽时，存在不同的激励。

* 消费者可能愿意交换上游带宽以将内容提供给其他消费者，以换取能够免费自己消费视频。 请参阅Webtorrent [[14]（＃references）]等系统。
* 广播公司作为原始节点，可能想要为视频的消费收费，或者可能想要补贴带宽成本，以便每个人都可以免费访问他们的视频。
* 转码器和中继节点愿意为分发视频提供带宽，只要它是有利可图的。 这类似于传统CDN的作用。



使用`Segments`作为流经网络的核心数据单元，可以使用ETH作为结算基础进行针锋相对的带宽计费。 我们从Swarm [[6]（＃references）]借用支票簿合同抽象作为一种通过链式结算传递的离线支付方法。 生态系统的未来发展，包括Raiden网络[[15]（＃references）]也可能允许支付渠道用于此目的。 由于令牌传输对于协议是本机的，因此还可以将与内容相关联的定价直接嵌入到协议中。 广播公司可以直接收取他们的时间或内容，节点将通过支付更高的价格/段来选择这种价值转移，这将转回广播公司。

需要注意的重要一点是，虽然可以使用带宽计费来运行中继节点，这些节点只是通过网络传输视频网段以增加容量，而不是CDN，但这些节点纯粹受到内容需求的激励， 不受新令牌分配的激励。 事实上，Livepeer的输出可以插入传统的CDN（如Amazon S3，Cloudflare等）或分散的CDN（如IPFS或Swarm）。 开发用于视频片段分发的这种点对点协议本身将是优化和改进性能的持续机会。

已经证明点对点CDN可以减少原始CDN服务器[[17]（＃references）]上80-98％的带宽需求，并且在分散网络中看到的令牌机制可以使利益相关者对齐开发和维护今天存在的专有P2P CDN的开放版本。 PPSPP协议[[18]（＃references）]是开放实施的可行候选者，专注于提供实时内容。

由于对Livepeer协议的加密经济学并不重要，因此本文不讨论细节，但是有兴趣的人可以随着开发(https://github.com/livepeer/go-livepeer)[跟随此处]，并寻找一个未来的纯视频分发协议文档。

## 用例 ###########################################

Livepeer项目关注的是去中心化一对多的直播视频广播（多播）。 这是最真实的媒体分发形式，因为它允许广播公司以第一手的方式直接与观众联系，不受改动，事后解释和旋转。 它为每个人提供了一个发声的平台。 现有的集中式解决方案可能遭受审查，第三方对用户数据/关系/货币化的控制，以及围绕服务支付的低效成本结构。 以下是在Livepeer之上构建应用程序和服务的一些逻辑用例。

### 即用即付内容消费

通过融合进协议中的价值转移交易，现在广播公司可以直接向观众收取其直播的消费，而无需通过集中平台对信用卡，账户或对用户身份的控制。 这适用于教育（付费参加在线课程），活动（付费观看音乐会或现场体育赛事），娱乐（付费观看游戏玩家或表演者的直播）以及许多其他用例 - 同时保留 观众的隐私，并允许他们只支付他们直接向广播公司消费的东西。

### 自动缩放社交视频服务

当今构建消费者视频服务的挑战之一是扩展基础设施，以支持随着新用户的增加而对越来越多的流和越来越多的消费者的需求。 一个服务层可以轻松地让开发人员开始在Livepeer网络之上构建他们的视频解决方案，该网络将自动扩展以支持任意数量的流和观看者，这将是基础设施开发人员的一个受欢迎的解决方案，否则他们将不得不继续配置服务器，媒体许可服务器，以及有效管理资源以处理峰值。

### 不知不觉的现场新闻

Twitter和Facebook等现有平台为大量受众提供了令人惊叹的实时视频解决方案，但它们也是第一个在各种政治冲突情况下被封锁或审查的平台。 使用像Livepeer这样的去中心化网络几乎不可能阻止实际上正在发生的事情。

### 视频启用的DApps

去中心化应用程序（DApps）开始出现，主要由以太坊生态系统驱动。 然而，到目前为止，还没有一种可行的解决方案，可以在不使用集中式解决方案的情况下，在DApp中嵌入实时视频，或者根据WebRTC的限制来限制消费客户端的数量。 通过将Livepeer引入堆栈，应用程序可以完全去中心化，但仍然包含大规模的实时视频，以及尽可能多的用户希望消费它。。

## 总结 ###########################################

总之，Livepeer协议激励节点将其处理和带宽贡献给网络，以便为转码和分发实时视频服务。 通过Truebit协议之上的可扩展扩展来解决工作验证，该扩展可以激励节点正确执行转码操作，以获得费用和令牌分配，并保留其作为代码转换器的价值收益角色。 网络的游戏化和虚假工作问题通过委托块奖励会计的委托证明的经济学来解决。 简单地将一个标记放入增值节点比在网络上支付费用以便在执行实际上没有实际需求的工作时分配给其他委托人变得更加经济合理。

最终结果是一个可扩展的，即用即付的网络，用于去中心化的实时视频广播 - 这是Livepeer试图填补的web3堆栈中的缺失层。

## 附录 ###########################################

### Livepeer协议参数引用

| 参数名称 | 描述 | 示例值 |
|----|------|---|
| `T` | 段长度，以秒为单位 | 2秒钟 |
| `N` | 活动代码转换器的数量 | 144 |
| `RoundLength` | 选举新一轮转码器之间的时间长度 | 1天 |
| `InflationRate` | 目前每轮LPT的目标通胀率。 （以算法方式移动）。 | .04% (equivalent to 15%/year) |
| `ParticipationRate` | 令牌绑定与流动的目标百分比。 | 50% |
| `RoundLockAmount` | 转码器在一轮结束时锁定一轮的这一百分比，以便委托人可以相应地审查和委派，而不必担心最后一刻的费率变化。 | 10% == 2.4小时 |
| `UnbondingPeriod` | 进入无约束状态和提取资金的能力之间的时间。 | 1个月 |
| `VerificationPeriod` | 提交工作索赔后验证工作索赔的截止日期。 这也是在去中心化存储解决方案中必须提供数据持久性接收的最短期限。 | 6小时 |
| `VerificationRate` | 将要验证的细分的百分比。 | 1/500 |
| `FailedVerificationSlashAmount` | 在验证失败的情况下减少百分比（超出潜在的允许故障阈值） | 5% |
| `MissedRewardSlashAmount` | 在缺少一个块奖励回合的情况下减少百分比（也许只在n次连续失误的情况下这样做） | 3% |
| `MissedVerificationSlashAmount` | 如果代码转换器没有调用验证，则减少百分比 | 10% |
| `CompetitivenessTolerance` | 如果所有代码转换器始终可用并设置相同的价格和费用，那么它们将按照其利益的比例接收工作。 此参数设置％必须在此目标工作％内才有资格进行令牌分配。 这可以防止代码转换器相对于它们的股份做很少的工作。 | 90％（极端的例子。有100个代码转换器和100,000个分段，这意味着如果我只做了100个分段（我应该做的1000分的10％），我就可以了）。 |
| `*SlashingThresholds` (TBD) | 占位符表示我们可能不会减少所有故障，只要它们超过故障率的某个阈值百分比。| |
| `VerificationFailureThreshold` | 您可以在不被削减的情况下失败的验证百分比。 有用，因为Swarm / Truebit等外部依赖可能会导致偶发故障。 | 1% |
| `FinderFee` | 发现者将收到的斜线数量的百分比作为补偿。 | 5% |
| `SlashingPeriod` | 在“验证期”完成后调用斜杠条件的截止日期。 | 1小时 |

### Livepeer协议事务类型

| 事务 | 描述 |
|----|------|
| `Bond()` | 绑定权益对转码器的影响。 |
| `Unbond()` | 输入固定的未绑定状态 `UnbondingPeriod`. |
| `Transcoder()` | 声明你的意图作为代码转换器。 |
| `ResignAsTranscoder()` | 将您的意图作为转码器辞职。 |
| `TranscodeAvailability()` | 此转码器目前可以接受另一项工作。 他们在池中随机分配新工作。 |
| `Job()` | 在链上提交转码作业。 |
| `EndJob()` | 结束工作以放弃转码责任。 |
| `Deposit()` | 提交链条上的存款，用于支付工作。 |
| `Withdraw()` | 退出存款和未绑定的权益。 |
| `ClaimWork()` | 结束转码工作并声明您可以证明您已经通过段范围和merkle root.take转码的段。 |
| `DistributeFees()` | Transcoder在验证后声明特定索赔的费用。 |
| `Reward()` | 链上的所有验证是否会削减或分配令牌分配。 只能由在当前轮次中活动的代码转换器调用，每轮一次。 |
| `Verify()` | 转码器提供段的转码声明，这些声明将与merkle证明一起进行验证，以便与merkle root进行比较 `ClaimWork()`。 明确调用Truebit执行验证。|
| `InitializeRound()` | 在新一轮的起始块之后需要调用此事务一次以初始化新的活动的转码转换器池。 |
| `UpdateDelegatorStake()` | 这允许委托人从前几轮申请他们的费用+代币分配。 它通过非绑定和绑定自动调用，但如果委托者想要在不改变状态的情况下更新它，它就可以作为故障保护。 |
| `*GovernanceTransactions()` | 待定  |

## 参考文献 ###########################################

1. 以太坊白皮书 - Vitalik Buterin - Ethereum Wiki - <https://github.com/ethereum/wiki/wiki/White-Paper>
2. 脂肪协议 - Joel Monegro - USV Blog - <http://www.usv.com/blog/fat-protocols>
3. 加密令牌和协议创新的未来时代 - Albert Wenger - <http://continuations.com/post/148098927445/crypto-tokens-and-the-coming-age-of-protocol>
4. SectorCoins案例 - Eric Tang - <https://medium.com/@ericxtang/case-for-sectorcoins-b70a7c820c2d#.7892n4a57>
5. 委托的证明共识 - Daniel Larimer - <https://bitshares.org/technology/delegated-proof-of-stake-consensus/>
6. Truebit - Jason Teutsch和Christian Reitweisner - <https://people.cs.uchicago.edu/~teutsch/papers/truebit.pdf>
7. 交换，发誓和诈骗，群体激励系统 - viktor trón, aron fischer, dániel a. nagy, zsolt felföldi, nick johnson - <http://swarm-gateways.net/bzz:/theswarm.eth/ethersphere/orange-papers/1/sw%5E3.pdf>
8. Kademlia: 基于XOR度量的点对点信息系统 - Petar Maymounkov和David Mazieres <https://pdos.csail.mit.edu/~petar/papers/maymounkov-kademlia-lncs.pdf>
9. Steem白皮书 - Daniel Larimer, Ned Scott, Valentine Zavgorodnev, Benjamin Johnson, James Calfee, Michael Vandeberg - <https://steem.io/SteemWhitePaper.pdf>
10. 介绍卡斯帕“友善的幽灵” - Vlad Zamfir - <https://blog.ethereum.org/2015/08/01/introducing-casper-friendly-ghost/>
11. Tendermint文档 - Jae Kwon和Ethan Buchman - <https://tendermint.com/docs>
12. Swarm - <http://swarm-gateways.net/bzz:/theswarm.eth/>
13. 在BitTorrent中通过激励措施建立稳健性 - Bram Cohen - <http://bittorrent.org/bittorrentecon.pdf>
14. WebTorrent - <https://webtorrent.io/>
15. 雷电网络 - <http://raiden.network/>
16. ERC20令牌标准 - <https://github.com/ethereum/EIPs/issues/20>
17. Peer5利用观众的设备实现流媒体视频的P2P方法 - <https://techcrunch.com/2017/01/26/peer5-y-combinator/>
18. 点对点流媒体对等协议 - <https://tools.ietf.org/html/rfc7574>
19. 基于股权协议的通货膨胀与参与 - Doug Petkanics - <https://medium.com/@petkanics/inflation-and-participation-in-stake-based-token-protocols-1593688612bf>
