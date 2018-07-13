# 安装示例、程序和 Docker 镜像

选定一个工作目录，比如 `$HOME/go`，执行[^1]：

```bash
$ cd $HOME/go
$ curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.0
```

*注意：经测试，在 MacOS 中加上 `-s` 会执行失败，执行的时候直接去掉*

以上脚本做了如下四件事：

- If needed, clone the hyperledger/fabric-samples repository
- Checkout the appropriate version tag
- Install the Hyperledger Fabric platform-specific binaries and config files for the version specified into the root of the fabric-samples repository
- Download the Hyperledger Fabric docker images for the version specified

[^1]:经测试，在 MacOS 中加上 `-s` 会执行失败，执行的时候直接去掉