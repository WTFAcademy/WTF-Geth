# 搭建 BNB Smart Chain 全节点

本示例使用 BSC Geth [v1.8.2](https://github.com/bnb-chain/bsc/releases/tag/v1.8.2) 的 Linux 二进制与同版本 `mainnet.zip`。部署前核对[官方全节点文档](https://docs.bnbchain.org/bnb-smart-chain/developers/node_operators/full_node/)中的系统、磁盘和同步要求。

## 获取二进制和配置

```sh
mkdir -p "$HOME/bsc-node"
cd "$HOME/bsc-node"
release=https://github.com/bnb-chain/bsc/releases/download/v1.8.2
curl -fL "$release/geth_linux" -o geth
curl -fL "$release/mainnet.zip" -o mainnet.zip
chmod +x geth
./geth version
unzip mainnet.zip
```

ARM64 Linux 将发行资产 `geth_linux` 换成 `geth-linux-arm64`。按发布页校验下载文件。新节点优先按官方文档选择匹配版本的[快照](https://github.com/bnb-chain/bsc-snapshots)。若从创世块初始化，在全新的数据目录中运行 `./geth --datadir node init genesis.json`。

## 启动并检查同步状态

```sh
./geth --config config.toml --datadir node --history.transactions 0
```

仅供本机使用时，将配置里的 `HTTPHost` 和 `WSHost` 保持在 `127.0.0.1`。启动后可从另一个终端运行 `./geth attach http://127.0.0.1:8545`，再通过 `eth.syncing` 检查进度。升级现有节点时，按[官方升级步骤](https://docs.bnbchain.org/bnb-smart-chain/developers/node_operators/full_node/#d-upgrade-geth)保留数据目录并更新同一发行版的配置。
