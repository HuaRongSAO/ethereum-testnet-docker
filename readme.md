# ethereum-testnet-docker 基于docker的以太坊集群的私有链开发环境

> 使用docker-machine进行对docker集群部署以太坊私有链

![BUILD](https://img.shields.io/docker/build/jrottenberg/ffmpeg.svg)   
源码: https://github.com/HuaRongSAO/ethereum-testnet-docker

作者环境:

- ubuntu 16.04

## 目录

- 使用本地搭建
  - docker 安装
  - docker-compose 安装
  - 搭建eth私有链
  - 简单测试
- 使用docker-machine搭建集群
  - docker 集群创建
  - docker-machine 安装

## <span id="docker-install">1.docker</span>

安装地址: https://docs.docker.com/install/

如果觉得安装步骤太麻烦,官方提供一件安装脚本:  
Linux下可以运行:

```shell
  # 下载脚本
  curl -fsSL get.docker.com -o get-docker.sh
  # 运行脚本
  sudo sh get-docker.sh
```

安装成功后:

- 启动服务:
  - ubuntu: systemctl start docker
  - centos: service docker start
  - window: 直接打开windowd的客户端

```shell
# 运行 docker version
$ docker version

Client:
  Version:	18.03.0-ce
  API version:	1.37
  Go version:	go1.9.4
  Git commit:	0520e24
  Built:	Wed Mar 21 23:10:01 2018
  OS/Arch:	linux/amd64
  Experimental:	false
  Orchestrator:	swarm

Server:
 Engine:
    Version:	18.03.0-ce
    API version:	1.37 (minimum version 1.12)
    Go version:	go1.9.4
    Git commit:	0520e24
    Built:	Wed Mar 21 23:08:31 2018
    OS/Arch:	linux/amd64
    Experimental:	false

```

## 2.docker-compose

> Docker Compose 是 Docker 官方编排（Orchestration）项目之一，负责快速的部署分布式应用。

文档: https://docs.docker.com/compose/

安装: Linux系统下:

```sh
$ sudo curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

- python环境可以使用pip安装:

```sh
sudo pip install -U docker-compose
```
其他系统请看
- (官网)[https://docs.docker.com/compose/install/]
- (Docker — 从入门到实践)[https://yeasy.gitbooks.io/docker_practice/content/compose/install.html]

运行:
```sh
$ docker-compose --version
docker-compose version 1.17.0, build ac53b73
```

## 3.搭建eth私有链
下载仓库: https://github.com/HuaRongSAO/ethereum-testnet-docker

```sh
git clone https://github.com/HuaRongSAO/ethereum-testnet-docker --depth 1
cd ethereum-testnet-docker 
```

文件目录结构:

```sh
.
├── docker-compose-standalone.yml # 单独启动一个eth节点
├── docker-compose.yml # 单独启动一个eth节点， 一个初始节点， 一个网络状态
├── eth-netstats # 网络状态
│   └── Dockerfile
├── genesis # 初始化配置
│   ├── genesis.json
│   ├── keystore # 初始化 配置账户
│   │   ├── UTC--2018-03-20T06-02-05.635648305Z--d6e2f555878f29ca190b8ef6bf7de334e1c47e51
├── geth-node # 以太坊节点+api
│   ├── app.json
│   ├── Dockerfile
│   └── start.sh
├── install-compose.sh
└── readme.md

```

运行:

```sh
# 进入ethereum-testnet-docker
cd ethereum-testnet-docker
# 在ethereum-testnet-docker目录下运行 启动以太坊
# 第一次运行可能会加载时间相对长,以后就快了
docker-compose up -d
# 查看运行状态
docker ps
# 运行状态如下
CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS              PORTS                                                                                             NAMES
59380c23b730        ethereumdocker_eth         "/root/start.sh '--d…"   5 seconds ago       Up 3 seconds        8545-8546/tcp, 30303/tcp, 30303-30304/udp                                                         ethereumdocker_eth_1
fbe34c9110b1        ethereumdocker_bootstrap   "/root/start.sh '--d…"   5 seconds ago       Up 4 seconds        0.0.0.0:8545->8545/tcp, 0.0.0.0:30303->30303/tcp, 8546/tcp, 0.0.0.0:30303->30303/udp, 30304/udp   bootstrap
4d77ae7e987e        ethereumdocker_netstats    "npm start"              6 seconds ago       Up 4 seconds        0.0.0.0:3000->3000/tcp                                                                            netstats
```

折就是运行成功了,接下来你可以直接打开浏览器访问: http://127.0.0.1:3000   
界面仓库:https://github.com/cubedro/eth-netstats  
你就可以查看到运行状态了
![netstats.png](https://upload-images.jianshu.io/upload_images/6167081-25a3e4587dd2e81e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在你就能看到两个节点:

- bootstrap  # 初始化节点
- e69fe54d216d # 新加入的节点

接下来添加多个节点

```sh
# docker-compose scale 作用是启动多个eth服务
docker-compose scale eth=3
```

出现了三个新的node节点:

- e69fe54d216d
- 55f0f83728db
- 1770aec8a167

![netstats.png](https://upload-images.jianshu.io/upload_images/6167081-d553072ba9fdce25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 使用geth

使用docker exec进行与geth的交互

```sh
# 格式 docker exec -it ethereumdocker_eth_{第几个节点} geth attach ipc://root/.ethereum/devchain/geth.ipc
docker exec -it ethereumdocker_eth_1 geth attach ipc://root/.ethereum/devchain/geth.ipc
```

进入docker内部的geth就可以正常的使用geth的功能了.   
阅读geth文档: 
- [geth文档](https://www.ethereum.org/cli)

```sh
Welcome to the Geth JavaScript console!

instance: Geth/v1.8.3-unstable-faed47b3/linux-amd64/go1.10
coinbase: 0xd6e2f555878f29ca190b8ef6bf7de334e1c47e51
at block: 0 (Thu, 01 Jan 1970 00:00:00 UTC)
 datadir: /root/.ethereum/devchain
 modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0

> admin.nodeInfo.enode
"enode://3732c434c5330205b12ef83e31e59dd36b6d04fc8392770057a05b678483524c2c5649932bc3cc8bced54d94befbd0fda6daae988243489ce0faecfa7a5b0914@[::]:30303"
> # geth cli
```

### 开始挖矿

节点是默认不挖矿的需要手动开启挖矿:

```sh
docker exec -it ethereumdocker_eth_1 geth attach ipc://root/.ethereum/devchain/geth.ipc
# 开始挖矿
miner.start()
```

等带一段时间后,就能重127.0.0.1:3000看到区块了.

```sh
# eth 查看用户
eth.accounts
```

```sh
# stop 停止挖矿
miner.stop()
```

到此以太坊的local本机就开发好了,但是会出现其他的问题

- 以太坊钱包冲突,因为以太坊钱包的端口也是30303,8545

解决方法:

- 你可以修改docker-compose里的端口映射

或者使用docker-machine进行docker隔离(强推)

## 使用docker-machine搭建私有链集群

> Docker Machine 可以在多种操作系统平台上安装，包括 Linux、macOS，以及 Windows。

### 1.安装docker-machine

- 安装包:[docker-machine](https://github.com/docker/machine/releases)
- 官网: https://docs.docker.com/machine/install-machine/

Linux下直接:

```sh
  base=https://github.com/docker/machine/releases/download/v0.14.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```

看到输出就是安装成功

```sh
$ docker-machine -v
docker-machine version 0.13.0, build 9ba6da9
```

### 2.使用docker-machine

创建一台eth的测试虚拟机:
linux 下:

```sh
docker-machine create -d virtualbox --engine-opt dns=114.114.114.114 --engine-registry-mirror https://registry.docker-cn.com --virtualbox-memory 2048 --virtualbox-cpu-count 2 eth
```

参数解析:

- --engine-opt dns=114.114.114.114 配置 Docker 的默认 DNS

- --engine-registry-mirror https://registry.docker-cn.com 配置 Docker 的仓库镜像

- --virtualbox-memory 2048 配置主机内存

- --virtualbox-cpu-count 2 配置主机 CPU

mac和window系统请阅读: [mac和window,其他系统](https://yeasy.gitbooks.io/docker_practice/content/machine/usage.html)

查看是否创建成功:

```sh

docker-machine ls
# 输出
NAME   ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
eth    -        virtualbox   Running   tcp://192.168.99.100:2376           v18.03.0-ce
```

### 搭建eth服务

#### 远程到你创建的虚拟机

```sh
# 远程到虚拟机
docker-machine ssh eth
# 为用户设置管理员密码
sudo passwd
# 设置密码
root
root
```

#### 拉取项目到你的虚拟机

```sh
# 拉取本项目的代码
git clone https://github.com/HuaRongSAO/ethereum-testnet-docker --depth 1
# 跳转目录
cd ethereum-testnet-docker
# 为脚本添加权限
sudo chmod +x ./install-compose.sh
# 运行脚本
./install-compose.sh
```

这里解释一下脚本: 只是用来安装docker-compose

#### 搭建私有链集群

```sh
# 跳转目录
cd ethereum-testnet-docker
# 启动 docker-以太坊 节点
docker-compose up -d
# 添加多节点
docker-compose scale eth=3
# 查看是否运行
docker ps
```

到这里虚拟机的私有链就安装成功了

#### 访问

在退出虚拟机的情况下(使用```exit```退出,)获取虚拟机的ip

```sh
docker-machine ip eth
# 输出ip 这就是你虚拟机的ip
192.168.99.100
```

打开浏览器:输入192.168.99.100:3000,就能看到挖矿节点的信息

#### geth交互

1. 先远程登入虚拟机

```sh
docker-machine ssh eth

```

2. 运行

```sh
# 格式 docker exec -it ethereumdocker_eth_{第几个节点} geth attach ipc://root/.ethereum/devchain/geth.ipc
docker exec -it ethereumdocker_eth_1 geth attach ipc://root/.ethereum/devchain/geth.ipc

# 其他操作
...

```

到这里集群就安装完毕,你的以太坊私有链的地址就不在是127.0.0.1:30303和8545就不会和本地的钱包发生冲突了,你就可以使用钱包链接到本地192.168.99.100的私有链了.

文献参考:

- [Docker — 从入门到实践](https://www.gitbook.com/book/yeasy/docker_practice/details)
- [go-ethereum](https://github.com/ethereum/go-ethereum)