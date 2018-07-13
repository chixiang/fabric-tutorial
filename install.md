# 安装示例、程序和 Docker 镜像

选定一个工作目录，比如 `$HOME/go`，执行：

```bash
$ cd $HOME/go
$ curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.0
```

*注意：经测试，在 MacOS 中加上 `-s` 会执行失败，执行的时候直接去掉*

以上脚本做了如下四件事：

- 将 `hyperledger/fabric-samples` 仓库克隆到本地
- checkout 合适的版本
- 下载 Hyperledger Fabric 平台运行相关的程序和配置文件到 `fabric-samples` 目录
- 下载指定版本的 Hyperledger Fabric docker 镜像

执行完成后，`$HOME/go/fabric-samples/bin` 目录内容如下：

![](https://ws2.sinaimg.cn/large/006tNc79gy1ft8fruw3qnj30jy02bt94.jpg)