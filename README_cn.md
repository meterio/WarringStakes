# Meter.io战国争雄测试网参与指南
请填写[验证节点申请表](https://metervalidators.typeform.com/to/yVVUDw) 填写表格的时候请准备好Meter钱包地址，和联系方式，我们会给您转测试币。另外节点设置完毕后请在Github的Issue里填写一下节点的信息，我们会根据对应的信息检查和纪录积分。

# Meter.io概要

Meter是具有双链结构的 PoW + PoS 的混合共识链。所有账户信息和交易都记录在PoS链上，而PoW链仅用来进行挖矿产生 Meter 生态的稳定币 MTR。

Epoch
Meter 以 Epoch （时段）的方式运行，这些“Epoch”由 k 区块来签名（常规区块称为m区块）。在每一个“Epoch“结束时，委员会节点们对最长的POW链进行投票，并将挖矿奖励分配给所有PoW矿工，同时还将信息传输给PoW链，所有PoW矿工将会继续开始出具备时间戳的 PoW 链的块。

PoW 链通常需要拥有超过 60 个的区块时，才能创建一个 k 区块。由于 PoW 链的平均出块时间为1分钟，每个 Epoch 大约为1小时（现在 Epoch 的时间概念由PoW链的时间概念决定，但 Epoch 的时间概念在之后会进行调整）。所有与系统经济相关的活动（例如，奖励分配，进入和退出委托节点池）只会在 k 区块里发生。

另外，你需要在同一物理机或虚拟机上同时运行PoS和PoW的流程来确保系统安全。

在 Meter 网络中的3中类型的全节点
1.全节点

它们针对每个区块进行同步，并可以支持和钱包之间的交互
2.委托节点

这些节点是委员会节点的候选者，并权利提议和对区块进行签名。前 N 个（N是协议里的参数）已经进行过 Staking 的全节点（包括通过自己进行 Staking 或者从其他 staker 那里获得投票的全节点）是委托节点。
3.委员会节点

每个 Epoch 都会从委托节点中选出一些节点。这些节点构成委员会法庭并执行共识。委员会节点轮流产生区块（K区块）。如果产生的新区块收到委员会 2/3 以上节点的认可签名，哪些这些签名就会成为 QC（委员会认证）。每个新产生的区块都会带有前一个区块的QC。一旦新区块获得了 QC，之前的区块就会被认定为被认证过的，所有该区块的交易都已经最终确认。

在测试网和主网的第一次发布时，代理节点的数量将与委员会节点的数量相同。未来可能会由一部分有更好性能和网络连接的节点来专门做委员会节点的。

成为委托/委员会节点的要求：

为了运行 Meter 网络的全部性能，建议的硬件配置是：
8个以上虚拟CPU
16GB内存
100GB SSD（AWS c5.2xlarge实例或更高版本）
建议使用数据中心级别为1Gbps到10Gbps的网络连接

最低要求配置：
最低要求是2个vCPU和4GB内存。

Meter 的 PoW 里的最大区块大小约为1.3MB。Meter 的共识协议以能够在一定程度上根据交易的负荷大小、网络和节点处理速度，在2秒到30秒之间自动调整出块速度。

# 配置全节点

当前节点软件是用 docker 镜像来提供的。请参考《 Ubuntu Docker安装指南》。

1.从 GitHub 库中获取 <delegates.json>文件到您的主目录（以下说明假定delegates.json位于/home/ubuntu目录）。
```
git clone https://github.com/meterio/warringstakes
cp ./warringstakes/delegates.json .
```

该文件包含 <验证创世区块的节点> 以及<备用委托节点（在没有足够的 staking 委托节点的情况下组成委员会>

2.启动 Meter container
```
sudo docker pull dfinlab/meter-all-in-one; sudo docker run -e DISCO_SERVER="enode://3011a0740181881c7d4033a83a60f69b68f9aedb0faa784133da84394120ffe9a1686b2af212ffad16fbba88d0ff302f8edb05c99380bd904cbbb96ee4ca8cfb@35.160.75.220:55555" -e DISCO_TOPIC="shoal" -e POW_LEADER="35.160.75.220" -e COMMITTEE_SIZE="21" -e DELEGATE_SIZE="21" -v /home/ubuntu/delegates.json:/pos/delegates.json --network host --name metertest -d dfinlab/meter-all-in-one:latest
```
在上面的命令中：

DISCO_SERVER PoS网络中发现其他节点的服务器。 POW_LEADER 指向可以为PoW 链提供节点信息的服务器。这两个IP地址应该相同。

委员会和委托节点数量应该根据测试网的说明进行适当配置。它将下载最新的container并启动 Meter 全节点。

对Docker有用的一些命令：

```
sudo docker container ls -l

```

输出将如下所示：

```
CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS              PORTS               NAMES
260bbd571d1a        dfinlab/meter-all-in-one   "/usr/bin/supervisord"   23 hours ago        Up 23 hours                             metertest
```
```
sudo docker container stop metertest              //stop the container
sudo docker container start metertest             //start the container
sudo docker container rm metertest                //remove the container
sudo docker image ls
sudo docker image rm [image ID]                   //remove the container image, will trigger redownloading the image at the next docker run, it is recommended to do this every time we upgrade the testnet
sudo docker container exec -it metertest bash     //launch a bash in the container
```
日志文件可以在 container 的/var/log/supervisor 目录下找到。如果您发现了任何错误，请记得在Github的Issue中附加PoS日志（stderr和stdout）。下面命令可以把Docker Container里的日志文件拷贝的宿主机的当前目录：
```
sudo docker cp metertest:/var/log/supervisor/[LogFileNameHere]     //replace with the log file name
```

在通过日志确认节点正常运行之后，您可以将桌面钱包连接到您自己的完整节点。

3.下载 Meter 桌面钱包并通过钱包设置连接到您自己的全节点

在钱包中，您可以通过“Settings”添加 http://IPaddrOfYourNode:8669 设置自己的全节点，并在地址栏选择连接自己的全节点。如果一切正常，地址栏左侧的图标应该变为绿色。您可以使用钱包内的浏览器查看生成区块的状态。您还应该创建一个帐户。请确保将助记词保存在安全的位置，当我们在测试网之间切换时，您将需要它们来找回您的帐户，并且它也可以在将来的主网上使用。

![添加全节点](./addnode.png)
![连接全节点](./connectnode.png)

请联系团队成员以获取MTRG和MTR测试代币。


# 配置委托节点

成为委托节点需要抵押 MTRG 。您必须同时拥有 MTR 和 MTRG 才能执行交易。

1.为您的节点配置网络端口。

如果要成为委托节点并为入站 TCP 连接打开以下端口，建议使用公网 IP 地址端口。

| Port Range           | Functions        |
|----------------------|------------------|
| 9209                 | PoW P2P          |
| 8332                 | PoW API          |
| 8669                 | Wallet REST API  |
| 8670-8671            | PoW/PoS Messages |
| 55555                | Discovery Server |
| 11235                | PoS P2P          |
| 9100                 | node explorers   |

2.通过桌面钱包成为委托节点的候选人。

在桌面钱包的"Candidate"选项下，您可以通过质押至少 2个MTRG 代币并输入节点所需的所有信息来自行选择成为委托节点的候选人 （当前端口暂时不要改动，代码会使用端口8670，用于P2P通信和消息传递），"pub key"一栏的内容并不是账户的公钥，而是在Docker Container pos目录中public.key里的内容，这个public.key和master.key两个文件是节点运维的公钥和私钥，请将他们备份好。这两个文件是在container启动的时候自动生成的，如果未来升级container，需要用这两个文件覆盖系统自动生成的文件。

候选人设置结束后您可以通过 http://IPaddrOfYourNode:8669/staking/candidates 马上检查自己是否已经变成候选人。

您也可以让其他帐户将他们的MTRG token 投票给您，来增加成为委托节点的机会。变成候选人的交易在系统里会被立即记录并生效，并且节点可以开始接受投票。但是，选票统计会到下一个 k区块才做，您的节点如果到那时候有足够票数，就会成为委托节点。您可以通过 http://IPaddrOfYourNode:8669/staking/delegates 检查委托节点列表，但是这个列表也是要等到当前epoch结束才会根据所有候选人的票数更新。

3.成为委托节点。

如果候选人获得足够的票数并排在前N个候选节点中，它将成为委托节点。不久我们会添加更多 API 信息，来解释与此有关的更多信息。

4.成为委员会节点。

因为当前的委员会人数与委托节点数相同。在下一个 Epoch，如果新的委员会产生前，委托节点在线，则委托节点将自动加入委员会。在委员会内部，每个委员会成员将轮流产生 m 区块(普通PoS区块）。我们仍在添加 RPC 接口来显示展示委员会的信息。但是，如果您搜索日志，则会找到“I am in committee”。


# 委托节点密钥备份和升级

1. 备份密钥
```
sudo docker cp metertest:/pos /home/ubuntu/meter-data
```
在meter-data目录里, 您需要留意的是public.key，master.key和delegates.json文件，您可以发现一个consensus.key文件，这是该节点参与的最后一个epoch中共识的BLS签名密钥，我们建议每次重新启动的时候删除这个文件。另外有一些其它的文件和目录，在测试网上也建议重新启动的时候删除，这样可以让节点重新同步区块，保证信息的正确和完整性。

2. 停止并删除当前的节点container容器
```
sudo docker rm -f metertest
```

3. 获取最新的Meter Doker容器文件镜像
```
sudo docker pull dfinlab/meter-all-in-one:latest
```
3. 强制重新同步区块，并刷新BLS密钥（强烈建议做这一步）
```
sudo rm -rf /home/ubuntu/meter-data/instance-9eeef4f05bf08063
sudo rm -rf consensus.key
```

4. 重启Doker容器，并把备份密钥目录映射到Docker容器内的/pos目录 （-v /home/ubuntu/meter-data:/pos）
```
sudo docker run -e DISCO_SERVER="enode://3011a0740181881c7d4033a83a60f69b68f9aedb0faa784133da84394120ffe9a1686b2af212ffad16fbba88d0ff302f8edb05c99380bd904cbbb96ee4ca8cfb@35.160.75.220:55555" -e DISCO_TOPIC="shoal" -e POW_LEADER="35.160.75.220" -e COMMITTEE_SIZE="21" -e DELEGATE_SIZE="21" -v /home/ubuntu/meter-data:/pos --network host --name metertest -d dfinlab/meter-all-in-one:latest
```
后续升级只需重复2到4步即可
