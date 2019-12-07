# Instructions for Participating in the Meter Test Net

# Overview
Meter is a hybrid PoW and PoS blockchain system with dual chain structure.  All the accounts and transactions are recorded on the PoS chain while PoW chain (currently a modified version of Bitcoin starting from the same genesis of Bitcoin) just maintains the crypto puzzles for mining. The PoW chain submits the solutions for the crypto puzzles to the PoS chain and the winning miners receive reward on their accounts on the PoS chain.  

Epoch:
Meter operates on epochs, which are signaled by k-blocks (regular blocks are called m-blocks).  At the end of epoch, the committee nodes vote on the longest PoW chain and distribute mining rewards to all the PoW miners, it also pass the information to the PoW chain and all the PoW miners will have to start mining for the stamped PoW block.  To create a k-block, the PoW chain typically has to have more than 60 blocks.  Since the average PoW block period is 1 minute, each epoch is therefore around 1 hour (currently the time for epoch is completely decided by PoW, but we will implement cross interactions for epoch adjustments in the future).  All system financial related activities like reward distribution, entering and exiting the delegate node pool only happens at k-blocks.     

It is also required to run both PoS and PoW processes on the same physical or virtual machine to ensure security.

In Meter, there are several types of full nodes in the network:
1. Regular full node: They sync for each block and can support interactions with wallets
2. Delegate nodes: These nodes are candidates for the committee nodes and have opportunity to propose and sign blocks.  To become a delegate node, the top N (N is a protocol parameter) staked full nodes(including both self staking and votes from other stakers) are the delegate nodes.
3. Committee nodes: A random subset of the delegate nodes are selected for every epoch.  These nodes form a committee quorum and perform consensus.  The committee nodes take turns to propose blocks.  If a proposed block receives endorsement signatures from 2/3 of nodes in the committee, the signatures form a QC (Quorum Certificate).  Each newly proposed block carries QC for the previous block.  Once the newly proposed block obtains a QC, the previous block is considered as confirmed and finalized.

In the test net and initial launch of the main net.  The number of Delegate Nodes will be the same as the number of the committee nodes.  In the future, there maybe a subset of nodes with better performance and network connectivity dedicated as the leaders in the committee nodes.


Requirements for running a delegate/committee node:
To achieve the full performance of the Meter network, the recommended hardware configuration is more than 8 compute optimized vCPU, 16GB of memory and 100GB of SSD (AWS c5.2xlarge instance or better).  The maximum block size in Meter is around 1.3MB. It is also recommended to have data center class 1Gbps to 10Gbps internet connection.  However the Meter consensus protocol is capable of adapting to transaction load, network and node processing speed to some extent by varying the block period from 2 sec to up to 30 sec.  The minimum requirement is 2 vCPU and 4GB of memory.  

# Setting up a full node
Node software is currently provided as docker images.  Please refer to [Ubuntu Docker Installation Guide](https://phoenixnap.com/kb/how-to-install-docker-on-ubuntu-18-04).

1. Obtain the delegates.json file from the GitHub repo to your home directory (the following instructions assume delegates.json is in /home/ubuntu).  This file contains the nodes that validate the genesis blocks and also serves as backup delegate nodes to run the committee in case there is not enough staked delegate nodes to form the committee

2. Launch the Meter container

```
sudo docker run -e DISCO_SERVER="enode://3011a0740181881c7d4033a83a60f69b68f9aedb0faa784133da84394120ffe9a1686b2af212ffad16fbba88d0ff302f8edb05c99380bd904cbbb96ee4ca8cfb@35.160.75.220:55555" -e POW_LEADER="35.160.75.220" -e COMMITTEE_SIZE="21" -e DELEGATE_SIZE="21" -v /home/ubuntu/delegates.json:/pos/delegates.json --network host --name metertest -d dfinlab/meter-all-in-one:latest
```
In the above command DISCO_SERVER points to the discovery server for other peers in the network. POW_LEADER points to node that could provide peer information for the PoW chain.  The two IP addresses should be the same.  The committee and delegate size should be configured properly based on the instructions of the test net.
It will download the latest container and launch a meter full node

Several useful commands for docker:

```
sudo docker container ls -l

```
The output will be like the following:
```
CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS              PORTS               NAMES
260bbd571d1a        dfinlab/meter-all-in-one   "/usr/bin/supervisord"   23 hours ago        Up 23 hours                             metertest
```
```
sudo docker container stop metertest              //stop the container
sudo docker container start metertest             //stop the container
sudo docker container rm metertest                //remove the container
sudo docker image ls
sudo docker image rm [image ID]                   //remove the container image, will trigger redownloading the image at the next docker run, it is recommended to do this every time we upgrade the testnet
sudo docker container exec -it metertest bash     //launch a bash in the container
```

The log files can be located inside the container, under /var/log/supervisor directory.  If you file any bugs, please remember to attach the logs for PoS (both the stderr and stdout) in the bug.  After confirming the node is running properly through the log, you could then connect the desktop wallet to your own full node.

3. Download [Meter desktop wallet](https://meter.io/developers) and connect to your own full node
In the settings of the wallet, under node, you could and connect add your own full node by adding http://IPaddrOfYourNode:8669 .  The icon in the left of the address bar should turn green if everything is running properly.  You could use the explorer inside the wallet to look at the status of the block productions. You should also create an account.  Please make sure you keep the mnemonics in a secure location, you will need them to retrieve your account when we switching between the test nets and it should also work on the future main net.  Please contact a team member to obtain MTRG and MTR test tokens.

# Become a delegate node
Becoming a delegate node requires staking MTRG tokens.  You will have to have both MTR and MTRG tokens in order to perform the transaction.

1. Configure network ports for your node.  It is recommended to have a public IP address if you want to become a delegate node and have the following ports open for inbound TCP connections

| Port Range           | Functions        |
|----------------------|------------------|
| 9209                 | PoW P2P          |
| 8332                 | PoW API          |
| 8669                 | Wallet REST API  |
| 8670-8671            | PoW/PoS Messages |
| 55555                | Discovery Server |
| 11235                | PoS P2P          |
| 9100                 | node explorers   |

2. Become a candidate
In the desktop wallet, under the "candidate" tab, you could self elect to be a candidate for delegate node by staking at least 2 MTRG tokens and input all the required information for your node (currently the port configuration is not used, the code will always port 8670 for P2P communications and messaging).  You could have other accounts delegate their votes to you as well to increase the chance of becoming a delegate node.  The candidate transaction is recorded immediately and the node could start to receive votes.  However, you won't become a delegate node until the next k-block even with enough votes.  You could check the list of delegate nodes through http://IPaddrOfYourNode:8669/staking/delegates

3. Become a delegate node
If a candidate receives enough votes and ranked in the top N candidate nodes, it will become a delegate node. more APIs will soon be added for additional information regarding to this.

4. Become a committee node
Since the current committee size is the same as the delegate size.  At the next epoch, if the delegate is online when the new committee starts, it will automatically join the committee.  Within the committee, each committee member will take turns to propose blocks.  We are still implementing the rpc interface for display committee information.  However, if you search the log, you will find "I am in committee"
