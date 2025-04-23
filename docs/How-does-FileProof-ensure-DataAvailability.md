# How FileProof Ensures the Availability of User Data

[FileProof Design Document](https://github.com/memoio/meeda-docs/blob/main/docs/developers/Contract.md)

---

1. What effect does file storage proof need to achieve?

2. Why must nodes store the user's data completely to respond to storage proofs and challenges?

3. What are the advantages of applying kzg polynomial commitment in file storage proof?

---

## 1. What effect does file storage proof need to achieve?

First, for question 1, what effect does file storage proof need to achieve? Let's consider when file storage proof is needed, which is derived from the scenario to determine the requirements.

Storage models have evolved from centralized storage to distributed storage and now to cloud storage. Centralized storage refers to data stored on one or multiple host machines that form a central node. Distributed storage refers to devices coordinating and mobilizing stored information through a topology network, a technology that stores data across multiple servers. Cloud storage is a technology that stores data in the cloud. These storage models do not require storage proof because their storage proof relies on the service provider's credibility. Centralized service providers control the storage devices, and whether data is correctly stored is entirely up to the service provider.

However, in the decentralized cloud storage model that has emerged in recent years, there is no centralized service provider, and data is randomly stored across storage nodes across the network. It is not enough to rely solely on the storage node's verbal assurance that the data is completely stored on the node. The system needs to automatically detect node failures in a timely manner to facilitate data recovery, thus ensuring the availability of user data.

Therefore, in the decentralized cloud storage model, file storage proof is needed. The purpose is to have storage nodes automatically and regularly prove the integrity of data storage to the system, enabling the system to detect node failures and perform data repair in a timely manner, thereby ensuring the availability of user data.

So, the primary and most important effect that file storage proof must achieve is to demonstrate that nodes indeed store the user's data completely. Secondly, since in decentralized cloud storage, blockchain's decentralized, tamper-proof, and automated execution characteristics are relied upon to verify file storage proof, and given the current limitations on chain transaction size and high costs (compared to no transaction fees) as well as limited chain TPS, file storage proof should also achieve low cost and high efficiency. Moreover, it is not possible to directly send the file to the verifier for verification, firstly, because the data volume is too large, and secondly, it would compromise the privacy of the user's data.

## 2. Why must nodes store the user's data completely?

Through the aforementioned [FileProof Design Document](https://github.com/memoio/meeda-docs/blob/main/docs/developers/Contract.md), it is known that the proof `proof` submitted by users regularly depends on the randomly generated number `rnd` each time. The user's data needs to be divided into 32-byte `atoms` as the coefficients of the polynomial to calculate the value of the polynomial at `rnd`, which is the `y` value. If the node does not save the user's complete data, it cannot obtain the polynomial coefficients and, therefore, cannot calculate the correct `y` value, and thus cannot provide the correct proof `proof`. In the optimistic verification phase, the node will not pass.

Of course, this is under optimistic conditions where the node cannot pass the verification. If the node is fraudulent and provides an incorrect aggregate commitment value `Cn`, making the incorrect proof `proof` pass verification (`proof` and `Cn` match successfully). In this case, fraud proof is needed. Any other node that finds the aggregate commitment value `Cn` to be problematic can challenge it on the chain. Since the user's data's true commitment value `Commitment` is saved on the chain, it can accurately determine that the node's submitted aggregate commitment value `Cn` is incorrect, and thus the node has not passed the storage proof.

In summary, nodes cannot pass the storage proof by only saving the user data's `Commitment` or only storing partial data. Nodes must store the user's data completely to pass the storage proof.

## 3. What are the advantages of kzg polynomial in file storage proof?

Similarly, through the [FileProof Design Document](https://github.com/memoio/meeda-docs/blob/main/docs/developers/Contract.md), it is known that kzg polynomial commitment ensures that during the storage proof process, only the storage node knows the user's data (of course, it is data that has been sliced and encrypted). The verifier and challenger do not need to know the user's data. This design concept fully protects the privacy of the user's data.

kzg polynomial commitment also supports batch proof. Whether it is verifying the storage proof of one file or tens of thousands of files, the node only needs to submit a fixed size storage proof `proof` once, and the storage proof data volume is small, only two G1 points. In the bls12_381 curve, a G1 point occupies 96 bytes (x and y each occupy 48 bytes), so the storage proof data volume is only 192 bytes. This greatly reduces the cost of storage proof and has good scalability.

> On-chain verification of storage proof requires one G1 multiplication, two G1 additions, one G2 multiplication, one G2 addition, and one pairing operation, which takes approximately 1.3544ms, negligible. According to the gas consumption calculation of geth pre-compiled contracts, on-chain verification of storage proof consumes approximately 253708 gas (this is only a rough calculation, further complete testing is needed). Based on the current GasPrice: 43.57537882Gwei, ETH Price: $2017.35 (as of 2023.11.20), it costs about $22.3. This cost is still relatively high. Later, it is necessary to find a chain with lower gas fees and less gas consumption for bls12_381 pre-compiled contracts to reduce the cost. For example, Arbitrum chain, based on the current GasPrice: 0.1Gwei, it costs about $0.05.

Nodes obtaining storage proof require running several G1 multiplications and additions. According to [Benchmarking pairing-friendly elliptic curves libraries - HackMD](https://hackmd.io/@gnark/eccbench), a single addition operation takes about 387ns, and a single multiplication operation takes about 245ns. The number of additions and multiplications the node runs depends on the size of the user data to be verified. If generating a storage proof for 32GB of data, it would take approximately 1 hour. This is quite efficient in the current storage proof industry.
