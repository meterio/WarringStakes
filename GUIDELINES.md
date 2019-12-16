Warring Stakes
Overview

If you haven't done it please make sure you register through the [Validator Application](https://metervalidators.typeform.com/to/yVVUDw)

We are going to run the game in phases, participants will be given points based on their contributions during the process and the final reward pool of the mainnet MTRG and MTR tokens will be divided proportionally based on the points.  Since all main net MTR tokens will be created through mining, the MTR portion of the reward will be purchased directly the miners by us.  We will publish the leader boards in Github on a weekly basis.  At the end of the game

Phase 0: Node bring up and bug hunt

Phase 1: Stability and adversary challenge

Phase 2: Community challenge


The initial Warring Stakes test net has 21 committee nodes, to become a committee node, you will have to stake more than 2 MTRG tokens.  

Please try to keep your node up or uncandidate before going offline (if you "uncandidate" and stay till the end of the epoch, one of our nodes should gracefully take over the committee node position) as more than 7 committee nodes failing will break the basic assumption of consensus and stall the test net.  In addition, since all the nodes in the committee will take turns to propose blocks, if you node goes offline, it will cause everyone to slow down and wait for you during your turn.  No points will be given to validator who failed to "uncandidate" before going offline.

We are assign points for various activities for example bring up full node and connecting wallet, starting validation, filing bugs and even porting some Ethereum smart contracts.  The participants should file issues in the Github so we could reward you proper points.

We will tag the issues with different priorities and give different points based on their priorities

P3 - low severity of bug or test that you have run. Example: Bugs in our configuration or setup
P2 - medium severity bug or test. Example: Bugs that impacting the performance or stall the consensus
P1 - high severity bug or some major features on Meter. Example: Bugs that break the security of the network.  Porting a great EVM smart contract like Uniswap to Meter
