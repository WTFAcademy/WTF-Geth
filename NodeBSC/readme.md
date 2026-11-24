# bsc 全节点搭建教程

> 本文保留 2022 年 Geth 1.1.12 的部署记录。部署当前版本，请遵循 [BNB Chain 全节点文档](https://docs.bnbchain.org/bnb-smart-chain/developers/node_operators/full_node/)，并使用同一发行版的二进制和配置文件。

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

    apt install wget curl git screen gcc automake autoconf libtool make unzip liblz4-tool aria2

## 三、节点安装部署

### 安装历史版本

以下命令使用 [v1.1.12 发布页](https://github.com/bnb-chain/bsc/releases/tag/v1.1.12) 上的 Linux 二进制和同一版本的主网配置。

```sh
mkdir -p /root/bsc
cd /root/bsc
wget https://github.com/bnb-chain/bsc/releases/download/v1.1.12/geth_linux -O geth
wget https://github.com/bnb-chain/bsc/releases/download/v1.1.12/mainnet.zip
chmod +x geth
./geth version
unzip mainnet.zip
./geth --datadir node init genesis.json
```

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

对于新节点，按 [BSC 快照仓库](https://github.com/bnb-chain/bsc-snapshots) 的说明将解压文件放到对应的数据目录。

## 四、运行节点

### 修改geth配置文件
    
    vi /root/bsc/config.toml

> HTTPHost：HTTP-RPC 的监听地址。本机访问可使用 `127.0.0.1`。
>
> HTTPPort：http协议rpc端口

> WSHost：WebSocket 的监听地址。本机访问可使用 `127.0.0.1`。
>
>WSPort：websocket协议rpc端口

### 运行geth

    screen -S bsc	#创建bsc节点启动窗口
    cd /root/bsc
    ./geth --config ./config.toml --datadir ./node --syncmode full --cache 56000 --txlookuplimit 0
退出窗口

## 五、查看节点状态

启动几分钟后，进入控制台查看节点状态

    /root/bsc/geth attach http://127.0.0.1:8545 #按照自己端口来

    >eth.syncing #查看区块同步情况

    >false #返回false则同步完成


