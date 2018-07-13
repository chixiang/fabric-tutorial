# 创建第一个网络

我们接下来要创建的第一个网络包含两个组织机构，每一个组织机构有两个 peer 节点，和一个“solo”排序节点。

我们进入前边准备好的目录：

```bash
$ cd $HOME/go/fabric-samples/first-network
```

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
