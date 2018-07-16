# 创建第一个网络

我们接下来要创建的第一个网络包含两个组织机构，每一个组织机构有两个 peer 节点，和一个“solo”排序节点。

我们进入前边准备好的目录：

```bash
$ cd $HOME/go/fabric-samples/first-network
```

## 一键执行

目录下有一个一键式脚本 `byfn.sh`，可以方便的帮我们执行所有操作。我们首先运行这个脚本体验一下，接下来将分步骤手工执行，学习每个步骤完成的细节。

### 生成 Network Artifacts

```bash
$ ./byfn.sh generate
```

### 启动网络

```bash
$ ./byfn.sh up
```

因为后边我们会执行 chaincode，默认是 Go 语言版本，如果你想使用 Nodejs 版本的 chaincode，则这里需要这样启动网络：

```bash
$ ./byfn.sh up -l node
```

### 关闭网络

```bash
$ ./byfn.sh down
```

## 分步执行

### crypto 生成

- 配置文件 `crypto-config.yaml`

```bash
$ cryptogen generate --config=./crypto-config.yaml
```

生成的文件在 `crypto-config` 目录下，此外还需要设置一个环境变量，下一步会用到：

```bash
$ export FABRIC_CFG_PATH=$PWD
```

### Configuration Transaction Generator

- 配置文件 `configtx.yaml`

```bash
$ configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
```

这个步骤生成：

- orderer `genesis block`
- channel `configuration transaction`
- and two `anchor peer transactions` - one for each Peer Org.

### 创建 Channel Configuration Transaction

```bash
$ export CHANNEL_NAME=mychannel
$ configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
```

在 `channel` 上为两个机构定义 `anchor peer`

```bash
$ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
$ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP
```

### 启动网络

```bash
$ docker-compose -f docker-compose-cli.yaml up -d
```

### 环境变量

接下来我们需要进入 docker 容器中去执行命令，通过设置环境变量来指定我们需要进入的机器：

```bash
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
CORE_PEER_LOCALMSPID="Org1MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

### 进入容器

```bash
$ docker exec -it cli bash
```

### 创建并加入 channel

```bash
export CHANNEL_NAME=mychannel
peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
peer channel join -b mychannel.block
```

下面我们可以将其他 peer 也加入 channel，需要重设环境变量，也可以重新开一个终端操作：

```bash
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
CORE_PEER_ADDRESS=peer0.org2.example.com:7051
CORE_PEER_LOCALMSPID="Org2MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer channel join -b mychannel.block
```

### 更新 anchor peers

```bash
peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

### 安装和实例化 chaincode

#### Golang

```bash
peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
```

#### Node.js

```bash
# this installs the Node.js chaincode
# make note of the -l flag; we use this to specify the language
peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node/
peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -l node -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
```

#### Query

初始化后可以查询 `a` 的值：

```bash
peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
```

#### 调用

从 `a` 向 `b` 转移 10

```bash
peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'
```

调用成功后再次执行查询，`a` 将变成 90