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

The vision of the decentralized web has begun to be realized over the past couple years with the emergence of networks like [Ethereum](http://ethereum.org) to enable trustless computing, [Swarm](http://swarm-gateways.net/bzz:/theswarm.eth/) and [IPFS/Filecoin](http://ipfs.io) to enable decentralized storage and content distribution, Bitcoin and various token projects to facilitate p2p transfer of value, and decentralized name registries like [Blockstack](http://blockstack.org) and [ENS](http://ens.readthedocs.io/en/latest/introduction.html) to provide human accessible names to content and identities. These elements form the basis for decentralized applications (DApps) to be built in the form of largely static or infrequently updated web or mobile content, but at the moment DApps still lack the ability to include streaming media and data in an open and decentralized way. The goal of the Livepeer project is to decentralize live video broadcast over the internet.

The [Livepeer Project Overview](https://github.com/livepeer/wiki/wiki/Project-Overview) provides a nice introduction to the current state of live video on the internet. This whitepaper will largely focus on the cryptoeconomic protocol details of Livepeer, rather than the business case, but in summary the overview describes the current state of live streaming as growing at a rapid pace, centralized, and expensive. On the other hand, a fully decentralized P2P solution, where nodes contributed their own computation and bandwidth in service of streaming live video would be more open and scalable, as there would be no limit to the number of connections that could be served.

This technology is certainly available to a certain extent, but to date there has been no incentive to get users to run nodes that provide this functionality, nor has there been proper funding for the development of an open protocol that can facilitate this in a way that benefits the entire internet rather than one centralized company. However, with the recent emergence of crypto token powered protocols [[2, 3](#references)], there is now an opportunity to simultaneously incentivize users to contribute computation and bandwidth towards live video broadcast, in a way that aligns with financing the development of an open media server solution capable of delivering live streamed video according to all the latest standards and formats required to reach the full range of devices. Additionally, the economic actions traditionally seen as a result of token powered protocols indicate that the cost to the broadcaster in order to use the Livepeer network could be cheaper than the cost of any centralized solution.

As the Livepeer technology and protocol are delivered, it will enable users to participate in the following flow:

1. Capture a video on your camera, phone, screen, or web cam and send it into the Livepeer network.
2. Nodes running within the network will encode it into all the necessary formats to reach every supported device. Users running these nodes will be incentivized via fees paid by the broadcaster in ETH, and the opportunity to build reputation through the protocol token to earn the right to perform more work in the future.
3. Any user on the network can request to view the stream, and it will automatically be distributed to them in near realtime.

<img src="https://s3.amazonaws.com/livepeerorg/LPExample.png" alt="Livepeer Network Example" style="width: 750px">

### The Live Video Stack

The technology stack for broadcasting live video has evolved over many years and contains many layers. Broadcasters need to capture video at the source, interface with a media server to process and transcode the video into many different formats, distribute the video across a network, and then allow the video to be played in high perceived quality by the end consumer. There are also economic questions that are introduced when one thinks through this stack, such as whether it should be the broadcaster or consumer who should be paying for the bandwidth to transfer the video.

A typical live streaming platform today needs to support RTMP, HLS, Mpeg-Dash video formats in H.264 and VP8 codec. New codecs like H.265/HEVC, VP9, and AV1 will become more popular in the near future as consumers become more accustomed to higher video quality.  For HLS alone, [Apple suggests](https://developer.apple.com/library/content/documentation/General/Reference/HLSAuthoringSpec/Requirements.html#//apple_ref/doc/uid/TP40016596-CH2-SW1) bitrates from 145kb/s all the way up to 7800kb/s, in order to serve the different types of devices under different conditions. All of this adds a significant amount of complexity and cost to live video broadcasting.

The existing decentralized development stack (web3) contains solutions for some of the layers required for a live video platform, like file transfer and payments, but currently there are no solutions for the capture and interface, transcoding and processing, and serving layers of live video. For this, Livepeer introduces the [Livepeer Media Server (LPMS)](https://github.com/livepeer/wiki/wiki/Livepeer-Media-Server) - an open source implementation of a media server which provides all of the live video specific functionality necessary for DApp developers and existing broadcasters to build live functionality into their applications. [Read more about it here](https://github.com/livepeer/wiki/wiki/Livepeer-Media-Server).

As a standalone application, any developer could build a live application on top of the LPMS, but it would still be centralized and would need to be scaled through traditional means. However when every node on the Livepeer network is running the LPMS, and the protocol’s economic incentives ensure that those nodes will contribute their processing power and bandwidth in service of transcoding and distributing live video, **a self-scaling, pay-as-you-go network is made available to developers, who can simply send their live stream into the network, and have the implementation details of scaling, payment, and media hosting abstracted away**.

## Livepeer Protocol ###########################################

The Livepeer Protocol defines how the various actors in a live streaming ecosystem participate in a secure and economically rational way. The two major areas that the protocol needs to address are the actual distribution of live video from the source to a large number of consumers in a performant and scalable way, and the economic incentives for encouraging participation in the network in a secure and game-theoretic manner. While this whitepaper will touch on the live video distribution itself where overlapping with the economic protocol, it will largely focus on the latter in order to demonstrate security and economic alignment. At the highest level, the protocol is designed to:

- Allow any node to send a live video into the network, and optionally pay to have it transcoded into various formats and bitrates.
- Allow any node to request the video from the network.
- Allow participants to contribute their processing power and bandwidth in service of transcoding and distribution of video, and to be compensated accordingly.

In a decentralized network where participants are rewarded in proportion to the amount of work that they contributed, the two big challenges that need to be addressed to ensure security are:

- Can it be verified that the work that the nodes did was done correctly?
- Are the nodes being awarded for real work that contributed value to the network, as opposed to fake work done in an attempt to gain token allocations unfairly?

The Livepeer protocol is designed to address both the verification of work and the prevention of fake work, while also offering solutions for automatic scalability of the network and baked in governance for protocol evolution over time.

### 视频片段

The core unit of media within Livepeer is what we will call a `segment`. A segment is a time sliced chunk of multiplexed audio and video of time length `t`. Every segment in the Livepeer network is unique, and contains the cryptographic evidence to verify that the broadcaster intended this specific data for this specific segment. Each stream is made up of many consecutive segments, each containing a sequence number identifying their proper ordering. A segment contains the following fields:

| Video Segment Field | Description |
|--------|--------|
| **StreamID** | Identifies the origin node and stream that this segment belongs to. |
| **SequenceNumber** | The sequential order that this segment belongs in the original stream. |
| **DataPayload** | The binary metadata and data representing the audio/video in this segment. |
| **DataHash** | The hash of the data payload. |
| **BroadcasterSignature** | A signature from the broadcaster of `Priv(StreamID, SequenceNumber, hash(StreamID, SequenceNumber, DataHash))` which can be used to attest and verify that the broadcaster claims this to be the true data for this unique segment. |

The Livepeer protocol generally uses segments as the unit of work for transcoding, distribution, and payments.

### Livepeer令牌

Livepeer Token（LPT）是Livepeer网络的协议令牌。 但它不是交换令牌的媒介。 广播公司使用以太坊的以太网（ETH）在网络上播放视频。 贡献处理和带宽的节点以广播公司的费用形式获得ETH。 LPT是一种权益令牌，参与者希望在网络上进行工作，以协调工作在网络上的分配方式，并提供安全性，确保工作能够诚实正确地完成。 LPT具有以下目的：

- 它作为委托证明利益系统中的绑定机制，其中利益被委托给参与协议的转码器（或验证器）以转码视频和验证工作。 由于协议违规而发生的令牌和潜在的削减是必要的，以便保护网络免受许多攻击。 更多下面。
- 它通过网络路由工作的比例与权益和委托令牌的数量成比例，基本上作为协调机制。
- 它是一个特定于Livepeer生态系统的帐户单元，它构成了SectorCoin概念的基础，适用于将来引入的其他功能[[4]（＃references）]。 DVR，隐藏式字幕，广告插入/货币化和分析等服务都可以插入Livepeer生态系统，并可能利用LPT提供的安全性。

将分发Livepeer令牌的初始分配，以便利益相关者可以在网络中履行各种角色并使用网络，然后将根据随时间推移的算法编程发布附加令牌。 请参阅[令牌分布]（＃令牌分发）部分。

遵循以太坊和许多流行的ERC20令牌[[16]（＃references）]的惯例，LPT将被10 ^ 18整除，较大的面额如LPT本身旨在用于诸如放样的用户级交易，以及 旨在用于协议会计的较小面额。

### Protocol Roles

Before going forward, let’s define the roles in the network so that there is a common vocabulary for discussing the protocol. A Livepeer node is any computer running the Livepeer software.

| Node Role | Description |
|--------|-----------|
| **Broadcaster** | Livepeer node publishing the original stream |
| **Transcoder** | Livepeer node performing the job of transcoding the stream into another codec, bitrate, or packaging format. |
| **Relay Node** | Livepeer node participating in the distribution of live video and passing of protocol messages, but not necessarily performing any transcoding. |
| **Consumer** | Livepeer node requesting the stream, likely to view it or serve it through a gateway to their app or DApp’s users. |

In addition to the above roles played by users running Livepeer nodes, the protocol also will refer to the following systems. While we use certain specific systems to make reference to a possible implementation, alternative systems can also be swapped in if they provide similar functionality and cryptoeconomic guarantees:

| System Role | Description |
|-------|----------|
| **Swarm** | Content addressed storage platform. Data can be guaranteed to be available there temporarily during the verification process via SWEAR protocol [[7, 12](#references)]. *(Note in this document we refer to Swarm, but other content addressed storage platforms can be substituted if data availability can be guaranteed with high probability).* |
| **Livepeer Smart Contract** | Smart contract running on the Ethereum network [[1](#references)]. |
| **Truebit** | Blackbox verification protocol that guarantees correctness of computation placed on chain (at a hefty cost) [[6](#references)]. (<http://truebit.io>) |

Here is a visual overview of the roles, and the ways in which they communicate with one another in the work verification process described below.

<img src="https://livepeer-dev.s3.amazonaws.com/docs/lpprotocol.png" alt="Protocol Visual Overview" style="width: 750px">  

*Segments flowing from the broadcaster to the transcoder and eventually to the consumer. The transcoder ensures they have signatures and proof of work to participate in the work verification procedure.*

**Note on Transcoders:** Transcoders play the most critical role in the Livepeer ecosystem. They are the ones who are taking an input stream and converting it into many different formats in a timely manner for low latency distribution. As such they benefit from high availability, efficient, powerful hardware (potentially with GPU accelerated transcoding), high bandwidth connections, and solid DevOps practices. Transcoders should churn far less than other network participants, as when they take on the job of transcoding a stream, it’s less than ideal if they drop off the network. While the network can scale to support many participants playing the role of transcoder (and earning the requisite token allocations), this is a special role that’s delegated from most network participants, in order to ensure that a reliable network that provides value to broadcasters is maintained. More below on this delegation.

### Consensus

Livepeer has a two layer consensus system. The LPT ledger and transactions are secured by the underlying blockchain, such as Ethereum. Any transfer of the LPT token or any transaction in the system can be considered to have been confirmed with the same security as the underlying proof of work or proof of stake blockchain. The second layer however, dictates the distribution of newly generated LPT. This is governed by the Livepeer Smart Contract, and participation in the protocol by various actors. While there is no consensus required per say, in terms of acceptance and validation of previous blocks, the protocol defines rules for participation and conditions upon which actors will be penalized (slashed) for failing to fulfill their role.

This second level of consensus governing the newly generated token is based upon Delegated Proof of Stake (DPOS), as inspired by systems like Bitshares, Steem, Tendermint, and Casper [[5, 9, 10, 11](#references)]. The role of validators in the network is played by Transcoders. Any user can delegate their stake towards a transcoder, who then needs to perform transcoding jobs in the network, participate in the work verification protocol, and invoke functions on chain at specific intervals to validate this work. The protocol will distribute fees and newly generated token, and it will slash the stake of badly behaved actors. The validation result will be recorded on-chain via Truebit after it performs the validation, so there will be no room for disputes between the broadcaster and the transcoder.

### Bonding + Delegation

In Livepeer, in order to indicate stake in the network, nodes must bond some amount of their LPT. They do this through the `Bond()` transaction, which will tie up their stake in the smart contract until they `Unbond()`, at which point they will enter an unbonding state which will last for `UnbondingPeriod` time. Upon completion of the `UnbondingPeriod` they can then withdraw their LPT.

The bonded amount is used to delegate stake towards a Transcoder. The network supports `N` active transcoders at any one time, which is a moveable network parameter. Any node can indicate that it wishes to be a Transcoder with a `Transcoder()` transaction, and the protocol will select the `N` transcoders with the most cumulative stake (their own + delegated from other nodes) at the start of each round, along with one random transcoder from the waitlist.

Newly generated token in Livepeer is distributed to bonded nodes in relative proportion to the amount of work that they have bonded (minus fees), as long as they’ve delegated towards transcoding nodes that behave according to the protocol. Bonds can be slashed (reduced by a certain percentage) if the nodes that they’ve delegated towards do not behave and violate one of the slashing conditions. Nodes who have bonded and delegated towards a Transcoder also receive a portion of the fees that the Transcoder generates through transcoding jobs on the network. In essence, nodes who perform work, earn the fees that broadcasters paid for that work.

Going forward, when this document uses the term "delegator", it is referring to bonded nodes who have delegated their stake towards a transcoder candidate, instead of delegating it towards themselves as a transcoder.

In summary, participants choose to bond their stake for the following reasons:

- Participate in delegating towards effective transcoders who will provide great service to the network, ensuring its value to broadcasters.
- Build reputation and future-work allocation in form of allocated token in proportion to stake.
- Earn fees generated from transcoders.
- They may wish to be a Transcoder.

### Transcoder() Transaction

A node indicates their willingness to be a transcoder by submitting a `Transcoder()` transaction, which publicizes the following three properties:

- `PricePerSegment`: the lowest price they are willing to accept to transcode a segment of video
- `BlockRewardCut`: The % of the block reward that bonded nodes will pay them for the service of transcoding. (Example 2%. If a bonded node were to receive 100 LPT in block reward, then 2 LPT to the transcoder).
- `FeeShare`: The % of the fees from broadcasting jobs that the transcoder is willing to share with the bonded nodes who delegate towards it. (Example 25%. If a transcoder were to receive 100 ETH in fees, they would pay 25 ETH to the bonded nodes).

The Transcoder can update their availability and information up until `RoundLockAmount` time before the next transcoding round. This is offered as a % of the round. (Example 10% == 2.4小时. They can change this information until 2.4小时 before the next transcoding round which lasts for `RoundLength` 1 day). This gives bonded nodes the chance to review the fee splits and token reward splits relative to other transcoders, as well as anticipated fees based upon the rate they're charging and network demand, and move their delegated stake if they wish. At the start of a transcoding round (triggered by a call to the `InitializeRound()` transaction), the active transcoders for that round are determined based upon the total stake delegated towards each transcoder, and stakes and rates are locked in for the duration of that round.

There is one change that is allowed during the `RoundLockPeriod`: The lowest offered price/segment for any of the candidate transcoders is locked in and can't be moved, but other transcoder candidates can adjust their price/segment downwards. This allows them to match the lowest offered price on the network if they wish in order to guarantee their stake-weighted share of work on the network. They are not allowed to move their offered price upwards during this period.

Here is an example state of Transcoder options that a delegator can review when deciding whom to delegate towards.

| Transcoder ID | PricePerSegment | BlockRewardCut | FeeShare |
|----|----|----|----|
| 1 | 22 wei | 1% | 25% |
| 2 | 30 wei | 2% | 40% |
| 3 | 10 wei | 4% | 1% |
| ... | ... | ... | ... |
| N | 14 wei | 0% | 2% |

*Note on price: In this document we list price/segment. In reality, Livepeer plans to use a gas accounting inspired model where there is a notion of units of gas required for certain job parameters of a segment such as bitrate, encoding, frame size, etc. Price/segment is a stand in, where the incentives are the same, but in reality they’ll likely be communicating price/gas.*

### Broadcast + Transcoding Job

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

#### A Note On Truebit

*While the protocol makes use of Truebit in order to provide fully trustless verification of work, it may be necessary in practice to use available solutions that provide verification without the degree of trustlessness that Truebit can offer while Truebit is still under development and testing. Some options, ordered by degree of trustlessness, include:*

*1. Livepeer API Based Oracle - Trust Livepeer to verify computation. Very centralized, not ideal for anything beyond testing.*  
*2. Oraclize Computation Service - Trust a company who provides proofs of computation and who's entire reputation relies upon putting external data on chain with proofs that it wasn't tampered with.*  
*3. Secure hardware enclaves - Services like Intel SGX or TownCrier provide trusted computing environments. Trust that their hardware implementation is correct and secure. This can be decentralized and audited.*


### Token Generation

Livepeer is inflationary in that new tokens will be generated and allocated over time according to the schedule communicated below in [Token Distribution](#token-distribution). If all roles in Livepeer behave according to the protocol, then newly generated tokens will be allocated to users in proportion to their bonded stake (minus fees). Transcoders have the role of calling the `Reward()` function in order to trigger the new token allocation or slashing which can be computed from all data available on chain.

Each transcoder will be required to call `Reward()` once per round.

- Ensure that an active Transcoder is calling `Reward()`.
- Ensure that the Transcoder has not called `Reward()` yet in this round.
- Compute the number of token to mint based upon the `InflationRate`. Mint this many token.
- Calculate the Transcoder's cut based upon their `BlockRewardCut`.
- Distribute this into the Transcoder's bonded stake.
- Distribute the remainder into the delegators reward pool.
- Update the bonded amount of token to this Transcoder.

Failure to invoke `Reward()` results in the direct consequence of losing a portion of token allocations, and showing up as a ding on one’s Transcoder reputation when it comes to being elected by Delegators for the role.

### Slashing

As previously mentioned, the conditions for slashing are:

- Failing a verification
- Failing to invoke verification when required to do so
- Not performing a proportional share of the required work within the platform based upon delegated stake

One of the benefits of building within the Ethereum ecosystem are the network effect benefits you receive from being able to build on top of other protocols such as Truebit and Swarm/SWEAR. Unfortunately, with reliance on these external systems, which themselves have external dependencies and incentives, it’s possible that a flaw or weakness in one of those protocols could result in slashing within Livepeer.

For example, if a Truebit verification job sat in their queue for a long period of time without any solver or verifier claiming it, Livepeer would fail to see the result of that verification in time before `Reward()` was called. Or if the Swarm network suffered a partition and couldn’t propagate the file to the Truebit verifier in time, then this could also create an issue.

These risks can be mitigated by incentivizing these roles to be played in house by participants in the Livepeer protocol, who may find it in their best interest to serve as Truebit verifiers or Swarm nodes. But there’s also another approach which is introducing the concept of probability thresholds on the slashing parameters. Optional protocol variables such as `VerificationFailureThreshold` could be set to indicate that as long as the node passes 99% of verifications they won’t be slashed for example. This will remain a further area of research to be worked out prior to network deployment.

The failure to invoke verification slashing condition can be checked and invoked by any Livepeer protocol participant. There is a `FinderFee` which specifies the percent of the slash amount which the finder will receive as a reward for successfully invoking this slashing condition.

The remainder of the slashed funds will enter the `CommonPool`, which can be burned or allocated to common uses such as further ecosystem development, according to the governance mechanism of the protocol.

### Token Distribution

As a token that represents the ability to participate and perform work in the network through a DPoS staking algorithm, the initial Livepeer token distribution will follow the patterns of other DPoS systems which require a widely distributed genesis state.

An initial allocation of the token will be distributed to the community at genesis and over the early stages of the network. Receipients can use it to stake into the role of Transcoder or Delegator. A portion will be allocated to groups who contributed prior work and money towards the protocol before the genesis, and a portion will be endowed for the long term development of the core project.

At the launch of the network, token issuance will continue according to an inflationary schedule with token being generated at `InflationRate` per round relative to the outstanding float of token. As token is issued in proportion to stake of all bonded participants in the protocol, it serves to incentivize active participation. Participants are "protected" from this inflation, due to earning their proportional share. It is only inactive participants who are sitting on token without bonding it for participation, who will see their proportional network ownership dilluted by this inflation.

The initial target for `InflationRate` will be set such that it aims to incentivize approximately `ParticipationRate` of the LPT to be bonded and actively participating [[19](#references)). For example, if `ParticipationRate` is 50% then incentives will exist to have half the oustanding token bonded. The inflation rate will move algorithmically each round to incent the participation target. A higher inflation rate would incent more token to be bonded, and a lower rate would lead to more people choosing liquidity rather than participation. It's this liquidity preference vs network ownership percentage tradeoff which should find equilibrium due to a number of economic factors in the network.

### 治理

Livepeer协议中的治理角色有三个方面：

1. Determine the burning or appropriation of common funds which were slashed from misbehaving nodes.
2. Adjust network parameters to ensure a healthy, thriving network which is valuable to broadcasters.
3. Invoke proposed protocol updates in a decentralized fashion.

Many of the network parameters referenced in this document such as `UnbondingPeriod`, `RoundLength`, `ParticipationRate`, and `VerificationRate` are adjustable. Proposals for adjustments to these parameters can be submitted, and the governance process, including voting by transcoders in proportion to their delegated stake, will determine adoption of these changes automatically within the protocol. The detailed spec for governance is left for another document. [See more here](https://github.com/livepeer/wiki/wiki/Governance). 

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



With `Segments` as the core unit of data flowing through the network, it is possible to do tit-for-tat bandwidth accounting using ETH as the basis for settlement. We borrow the Chequebook Contract abstraction from Swarm [[6](#references)] as a method of offchain payment passing with on chain settlement. Future developments in the ecosystem including the Raiden Network [[15](#references)] may allow of payment channels to be used for this purpose as well. Since token transfer is native to the protocol, it is also possible to embed pricing associated with content directly into the protocol. A broadcaster can charge for their time or content directly, and nodes will opt into this transfer of value by paying a higher price/segment which will flow back to the broadcaster.

What's important to note is that while bandwidth accounting can be used to make it profitable to run Relay Nodes which just pass video segments around the network to add capacity, a-la a CDN, these nodes are purely incentivized by demand for the content, and not incentivized by new token allocations. In fact, the output of Livepeer can be inserted into a traditional CDN (like Amazon S3, Cloudflare, etc) or decentralized CDN (like IPFS or Swarm). Development of this peer-to-peer protocol for video segment distribution itself will be an ongoing opportunity for optimization and improvement in performance.

Peer-to-peer CDNs have been shown to reduce 80-98% of bandwidth requirements on an origin CDN server [[17](#references)], and the token mechanics seen in decentralized networks can align stakeholders for the development and maintenance of an open version of the proprietary P2P CDNs that exist today. The PPSPP Protocol [[18](#references)] serves as a viable candidate for an open implementation that focuses on delivery of live content.

As non-critical to the cryptoeconomics of the Livepeer protocol, the details are spared from this document, but the interested can [follow along here](https://github.com/livepeer/go-livepeer) with the development, and look for a future document addressing purely the video distribution protocol.

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

1. Ethereum White Paper - Vitalik Buterin - Ethereum Wiki - <https://github.com/ethereum/wiki/wiki/White-Paper>
2. Fat Protocols - Joel Monegro - USV Blog - <http://www.usv.com/blog/fat-protocols>
3. Crypto Tokens and the Coming Age of Protocol Innovation - Albert Wenger - <http://continuations.com/post/148098927445/crypto-tokens-and-the-coming-age-of-protocol>
4. The Case For SectorCoins - Eric Tang - <https://medium.com/@ericxtang/case-for-sectorcoins-b70a7c820c2d#.7892n4a57>
5. Delegated Proof-of-Stake Consensus - Daniel Larimer - <https://bitshares.org/technology/delegated-proof-of-stake-consensus/>
6. Truebit - Jason Teutsch and Christian Reitweisner - <https://people.cs.uchicago.edu/~teutsch/papers/truebit.pdf>
7. swap, swear and swindle, incentive system for swarm - viktor trón, aron fischer, dániel a. nagy, zsolt felföldi, nick johnson - <http://swarm-gateways.net/bzz:/theswarm.eth/ethersphere/orange-papers/1/sw%5E3.pdf>
8. Kademlia: A Peer-to-peer Information System Based On The XOR Metric - Petar Maymounkov and David Mazieres <https://pdos.csail.mit.edu/~petar/papers/maymounkov-kademlia-lncs.pdf>
9. Steem Whitepaper - Daniel Larimer, Ned Scott, Valentine Zavgorodnev, Benjamin Johnson, James Calfee, Michael Vandeberg - <https://steem.io/SteemWhitePaper.pdf>
10. Introducing Casper "the Friendly Ghost" - Vlad Zamfir - <https://blog.ethereum.org/2015/08/01/introducing-casper-friendly-ghost/>
11. Tendermint Docs - Jae Kwon and Ethan Buchman - <https://tendermint.com/docs>
12. Swarm - <http://swarm-gateways.net/bzz:/theswarm.eth/>
13. Incentives Build Robustness in BitTorrent - Bram Cohen - <http://bittorrent.org/bittorrentecon.pdf>
14. WebTorrent - <https://webtorrent.io/>
15. Raiden Network - <http://raiden.network/>
16. ERC20 Token Standard - <https://github.com/ethereum/EIPs/issues/20>
17. Peer5 leverages viewers’ devices for a P2P approach to streaming video - <https://techcrunch.com/2017/01/26/peer5-y-combinator/>
18. Peer-to-Peer Streaming Peer Protocol - <https://tools.ietf.org/html/rfc7574>
19. Inflation and Participation in Stake Based Protocols - Doug Petkanics - <https://medium.com/@petkanics/inflation-and-participation-in-stake-based-token-protocols-1593688612bf>
