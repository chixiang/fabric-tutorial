## Hyperledger Fabric 概述

Hyperledger Fabric 是由 IBM 公司主导开发的一个面向企业级客户的开源项目。与比特币和以太坊这类公有链不同，Hyperledger Fabric 网络中的节点必须经过授权认证后才能加入，从而避免了POW资源开销，大幅提高了交易处理效率，满足企业级应用对处理性能的诉求。同时，为了满足灵活多变的应用场景，Hyperledger Fabric 采用了高度模块化的系统设计理念，将权限认证模块（MSP）、共识服务模块（Ordering Service）、背书模块（Endorsing peers）、区块提交模块（committing peers）等进行分离部署，使开发者可以根据具体的业务场景替换模块，实现了模块的插件式管理（plug-in/plug-out）。所以，Hyperledger Fabric是一个私有链／联盟链的开发框架，而且系统的运行不需要token支持。

### 关键组件

* **Channel：** 是一种数据隔离机制，保证交易信息只有交易参与方可见，每个channel是一个独立的区块链，这使得多个用户可以共用同一个区块链系统而不用担心信息泄露问题。
* **Chaincode：** 也叫智能合约，将资产定义和资产处理逻辑封装成接口，当其被用户调用的时候，改变账本的状态。
* **Ledger：** 区块链账本，保存交易信息和智能合约代码。
* **Network：**  交易处理节点之间的P2P网络，用于维持区块链账本的一致性。
* **Ordering service：** 利用kafka、SBTF等共识算法对所有交易信息进行排序并打包成区块，发给committing peers节点，写入区块链中。
* **World state：** 显示当前资产数据的状态，底层通过LevelDB和CouchDB数据库将区块链中的资产信息组织起来，提供高效的数据访问接口。
* **Membership service provider（MSP）：** 管理认证信息，为client和peers提供授权服务。

### Hyperledger Fabric Network 中的角色

在 Hyperledger 中，有三种类型的角色

* **Client：** 应用客户端，用于将终端用户的交易请求发送到区块链网络；
* **Peers：** 负责维护区块链账本，分为 endoring peers 和 committing peers，其中，endorser 为交易做背书（验证交易并对交易签名），committer 接收打包好的区块，然后写入区块链中。Peers 节点是一个逻辑的概念，endorser 和 committer 可以同时部署在一台物理机上。
* **Ordering Service：** 接收交易信息，并将其排序后打包成区块，放入区块链，最后将结果返回给 committer peers。

### Hyperledger 交易流程

1. 客户端通过 SDK 接口，向 endorsing peer 节点发送交易信息：

![](https://ws3.sinaimg.cn/large/0069RVTdgy1fusatij3yej30bj03w3yg.jpg)

2. 每个 endorsing peer 节点模拟处理交易，此时并不会将交易信息写入账本。然后，endorser peer 会验证交易信息的合法性，并对交易信息签名后，返回给 client。此时的交易信息只是在 client 和单个 endorser peer 之间达成共识，并没有完成全网共识，各个client的交易顺序没有确定，可能存在双花问题，所以还不能算是一个“有效的交易”。同时，client 需要收到“大多数” endorser peer 的验证回复后，才算验证成功，具体的背书策略由智能合约代码控制，可以由开发者自由配置。

![](https://ws2.sinaimg.cn/large/0069RVTdgy1fusauf9wwrj30bj03zjrd.jpg)

3. client将签名后的交易信息发送给order service集群进行交易排序和打包。Order service集群通过共识算法，对所有交易信息进行排序，然后打包成区块。Order service的共识算法是以组件化形态插入Hyperledger系统的，也就是说开发者可以自由选择合适的共识算法。

![](https://ws2.sinaimg.cn/large/0069RVTdgy1fusavcrihjj30bj04uglk.jpg)

4. ordering service将排序打包后的区块广播发送给committing peers，由其做最后的交易验证，并写入区块链。ordering service只是决定交易处理的顺序，并不对交易的合法性进行校验，也不负责维护账本信息。只有committing peers才有账本写入权限。

![](https://ws4.sinaimg.cn/large/0069RVTdgy1fusaw1z80qj30bj0520sr.jpg)

### 交易流程总结

区块链的账本由peer节点维护，并不是由ordering service集群维护，所以，只有peer节点上可以找到完整的区块链信息，而order service集群只负责对交易进行排序，只保留处理过程中的一部分区块链信息。Hyperledger Fabric系统中的节点是一个逻辑的概念，并不一定是一个台物理设备，但是对于生产环境的设计者来说，peer节点不能和order节点部署在一台机器上，而enduring peers和committing peers可以部署在同一台机器上，这种设计主要是为了系统架构的解耦，提高扩展性，以及通过主机隔离提高安全性。 Endorsing peer校验客户端的签名，然后执行智能合约代码模拟交易。交易处理完成后，对交易信息签名，返回给客户端。客户端收到签名后的交易信息后，发给order节点排序。Order节点将交易信息排序打包成区块后，广播发给committing peers，写入区块链中。一个完整的交易处理流程如下图所示：

![](https://ws3.sinaimg.cn/large/006tNbRwgy1furswjf7ebj30wu0h5q3o.jpg)

### Hyperledger Fabric Network的共识算法

在所有peers中，交易信息必须按照一致的顺序写入账本（区块链的基本原则）。例如，比特币通过POW机制，由最先完成数学难题的节点决定本次区块中的信息顺序，并广播给全网所有节点，以此来达成账本的共识。而Hyperledger Fabric采用了更加灵活、高效的共识算法，以适应企业场景下，对高TPS的要求。目前，Hyperledger Fabric有三种交易排序算法可以选择。

* **SOLO：** 只有一个order服务节点负责接收交易信息并排序，这是最简单的一种排序算法，一般用在实验室测试环境中。Sole属于中心化的处理方式。
* **Kafka：** 是Apache的一个开源项目，主要提供分布式的消息处理／分发服务，每个kafka集群由多个服务节点组成。Hyperledger Fabric利用kafka对交易信息进行排序处理，提供高吞吐、低延时的处理能力，并且在集群内部支持节点故障容错。
* **SBFT：** 简单拜占庭算法，相比于kafka，提供更加可靠的排序算法，包括容忍节点故障以及一定数量的恶意节点。目前，Hyperledger Fabric社区正在开发该算法。

### Channel的概念

Channels能够让上层不同的用户业务共享同一个区块链系统资源，主要包括网络、计算、存储资源。从本质上来说，channels是通过不同的区块链账本来为上层业务服务，而且，这些区块链统一部署在peers节点上，统一通过ordering service进行交易排序和打包区块。Channels之间通过权限隔离控制，不同channel内的成员，无法访问对方的交易信息，只能访问所属channel的交易信息。

channel可以理解为系统资源的逻辑单元，每个channel都包含peers资源、order资源、网络资源等等，而且这些资源有可能是和其它channel所共享。

### State Database

状态数据库保存了账本所有资产的最新状态（例如，账户A拥有某种资产的总量），同时，为智能合约提供了丰富的资产查询语义。所有的资产信息最终以文件形式记录在区块链账本中，而数据库是区块链账本的视图表现形式，能够让智能合约更加高效的和账本信息进行交互。数据库自动从底层区块链账本中更新或者恢复数据，默认的状态数据库是LevelDB，也可以替换为CouchDB。

* **LevelDB：** Hyperledger Fabric的默认数据库，简单的存储键值对信息；
* **CouchDB：** 提供更加丰富的查询语义，可以保存JSON对象，以及范围key的查询。

![](https://ws3.sinaimg.cn/large/0069RVTdgy1fusb1oe59ij30bj06a3yf.jpg)

