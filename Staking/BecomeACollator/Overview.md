# Overview

Collators are the most important members of the network as they not only maintain the state by running a Curio Parachain full node, but are also allowed to change it by building state transition proofs and sharing them with the Relay Chain validators. Generally speaking, the latter finalize the proposed block if and only if it represents a valid state transition. 

It is important to note  that elusive collators can never get invalid blocks finalized thanks to  the design security umbrella provided by the Relay Chain. Thus, the most harm dishonest collators can do is to slow down or halt the network. As long as at least one honest collator exists, the parachain is secured  and fully operative. However, the block time would be slower than with a full set of honest and functioning collator nodes.  

If you want to join the Curio Parachain network as a collator, you have to run a full node of the blockchain and  set up your session keys. You are also required to hold a minimum amount of  self-staked tokens to qualify for a collator seat. Once you have finished the mandatory steps described throughout the following sections, you can be added to the candidate pool. The candidate pool is sorted first by the total staking amount including delegations. If the pool is full and the new candidate has the exact same stake  amount as the last member of the pool (by total stake), the blockchain favors the candidate that has been in the pool longest. Thus, only the collators with the highest total stake are periodically selected to be eligible block authors.

## **Roadmap**

We will guide you through the steps to become a collator. First, we will discuss the hardware requirements and how you could test the performance of your node. Then, we go over a few configuration options and show you how to set up and start a Curio Parachain collator, including how to generate your sessions keys and join the pool of collator candidates.