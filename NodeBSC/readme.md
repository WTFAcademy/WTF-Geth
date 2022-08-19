# bsc 全节点搭建教程

## 一、服务器配置
>系统：Linux(Ubuntu 20.04)
> 
>CPU：16核
> 
>内存：64GB内存
> 
>带宽：50M以上
> 
>硬盘：大于2T固态SSD可用空间数据盘
 
推荐使用下列hetzner服务器

[EX100](https://www.hetzner.com/dedicated-rootserver/ex100/configurator#/)
CPU:i9-12900K RAM:128 GB DRIVES:1.92TB NVMe SSD x2 Connection:1 GBit/s


[AX101](https://www.hetzner.com/dedicated-rootserver/ax101/configurator#/)
CPU:AMD 5950X RAM:32x4 GB DRIVES:3.84TB NVMe SSD x2 Connection:1 GBit/s

[AX161](https://www.hetzner.com/dedicated-rootserver/ax161/configurator/)
CPU:AMD 7502P RAM:按需配置 DRIVES:按需配置 Connection:1 GBit/s

购买完服务器磁盘阵列默认是raid1 , 需要手动改成raid0(io更快) , 更改教程可以参考 [此处](https://npchk.info/hetzner-raid0/)

## 二、环境准备

### 更新软件包

    apt-get update
    
### 安装环编译境及其他工具

    apt install wget git screen gcc automake autoconf libtool make unzip liblz4-tool aria2

### 安装golang(用于编译geth)

    apt install golang -y

使用`go version`确认安装正确

    root@Ubuntu-2004-focal-64-minimal ~ # go version
    go version go1.13.8 linux/amd64


## 三、节点安装部署

### 安装geth

    git clone https://github.com/bnb-chain/bsc
    cd bsc
    make geth

添加全局变量

    ln -s /usr/bin/geth /root/bsc/build/bin/geth

使用`geth version`确认安装正确

    root@Ubuntu-2004-focal-64-minimal ~ # geth version
    Geth
    Version: 1.1.12
    Git Commit: 723863ee564e6b10d5c0218aad56aa287bdfa2f9
    Git Commit Date: 20220808
    Architecture: amd64
    Go Version: go1.13.8
    Operating System: linux
    GOPATH=
    GOROOT=go

### 配置创世区块

    wget   $(curl -s https://api.github.com/repos/bnb-chain/bsc/releases/latest |grep browser_ |grep mainnet |cut -d\" -f4)
    unzip mainnet.zip
    geth --datadir node init genesis.json

### 下载BSC快照

创建一个用来下载快照的screen窗口

    screen -S download

**注意1：使用screen窗口期间可以退出或者关闭命令行对话框**

**注意2：退出当前窗口时用ctrl+ad(顺序按a和d字母即可), 绝对不要用exit或ctrl+d退出会话**

**注意2：退出会话后 , 可以用screen -r download , 这样可以保持在shell下运行，网络中断不会影响**

回到主目录

    cd 


前往 [bsc-snapshots](https://github.com/bnb-chain/bsc-snapshots) 下载快照

    aria2c -o geth.tar.lz4 -s14 -x14 -k100M https://download.bsc-snapshot.workers.dev/{filename} -o geth.tar.lz4

下载完成后解压
    
    tar -I lz4 -xvf geth.tar.lz4

解压完成后,将解压出的 chaindata 和 triecache 移动到 /bsc/node/geth/ 目录下
    
    rm -rf /root/bsc/node/geth/chaindata #删除多余文件夹
    mv /root/server/data-seed/geth/chaindata /root/bsc/node/geth
    mv /root/server/data-seed/geth/triecache /root/bsc/node/geth
    exit

删除快照

    rm -rf /root/geth.tar.lz4
    rm -rf /root/server

## 四、运行节点

### 修改geth配置文件
    
    vi /root/bsc/config.toml

> HTTPHost: HTTP-RPC服务连接白名单，此参数的值默认为 "localhost"，仅允许本地可访问，如果需要外网访问节点请设置为："0.0.0.0"
>
> HTTPPort：http协议rpc端口

>WSHost：websocket服务连接白名单，此参数的值默认为 "localhost"，仅允许本地可访问，可设置为："0.0.0.0"
>
>WSPort：websocket协议rpc端口

### 运行geth

    screen -S bsc	#创建bsc节点启动窗口
    geth --config ./config.toml --datadir ./node --diffsync --cache 56000 --rpc.allow-unprotected-txs --txlookuplimit 0
退出窗口

## 五、查看节点状态

启动几分钟后，进入控制台查看节点状态

    geth attach http://127.0.0.1:8545 #按照自己端口来

    >eth.syncing #查看区块同步情况

    >false #返回false则同步完成


