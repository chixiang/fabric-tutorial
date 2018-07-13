# Hyperledger Fabric Tutorial

来源于对官方文档的学习，精简后记录在这里，方便后期复习回顾。

原教程地址：[http://hyperledger-fabric.readthedocs.io/en/latest/index.html](http://hyperledger-fabric.readthedocs.io/en/latest/index.html)

---

## 预备知识和先决条件

操作系统建议使用 MacOS 或者 Ubuntu，不推荐 Windows，会有很多麻烦

### cURL

### Docker and Docker Compose

- MacOSX, *nix, or Windows 10: Docker Docker version 17.06.2-ce or greater is required.

### Go Programming Language

- Go version 1.10.x is required.

必须将包含 Fabric 代码等的工作目录设置为 `GOPATH` 环境变量，再将其中的 `bin` 目录加入到 `PATH` 中，例如：

```bash
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```

### Node.js Runtime and NPM

- Node.js version 8.9.x or greater

### Python

- Python version 2.7