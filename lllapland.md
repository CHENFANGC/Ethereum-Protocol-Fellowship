---
timezone: Asia/Dubai
---

# lllapland

1. 自我介绍
   大家好我是 lllapland，爱好和饭碗都是写代码，智能合约萌新。

2. 你认为你会完成本次残酷学习吗？
   根据以往经验不会完成，但是我会尽力完成。我自控力比较差，一般被push的时候效果最好（逃）。

## Notes


<!-- Content_START -->

### 2025.02.06

https://epf.wiki/#/eps/week1

#### Prehistory and Philosophy

- Free Software movement
  - **UNIX** (1960s): Modular design, key to Ethereum's architecture; Bell Labs' open collaboration mirrors Ethereum's core development.  
  - **Free Software Movement**: Led by Richard Stallman, emphasizing software freedom and open source principles
  - **FOSS (Free & Open Source Software)**: Software that respects users' freedom and community
  - **GNU**: Foundation for open-source software, influencing Ethereum's development principles. GNU stands for "GNU's Not Unix"
- Cryptography
  | **Characteristic** | **Symmetric Encryption** | **Asymmetric Encryption** |
  |-------------------|-------------------------|--------------------------|
  | **Key Usage** | same key for encryption & decryption | public/private key pair |
  | **Security** | less secure if key is exposed | more secure; only private key must be kept secret |
  | **Speed** | faster (simpler computation) | slower (complex mathematical operations) |
  | **Key Distribution** | requires secure key exchange | public key can be shared openly |
  | **Common Algorithms** | AES, DES, RC4 | RSA, ECC, Diffie-Hellman |
  | **Use Cases** | encrypting files, database security | digital signatures, 🌟 **blockchain**, secure communication (TLS, PGP) |


#### Implementations and Development
- **Clients**: Implementations of the Execution Layer (EL) or Consensus Layer (CL).
  - EL clients: Geth, Nethermind, Besu
  - CL clients: Prysm, Lighthouse, Teku
- **Nodes**: Computers running both an EL and CL client, actively participating in the Ethereum network.
#### Ethereum Roadmap Phases  
- **The Merge** Transition from Proof of Work (PoW) to Proof of Stake (PoS).  
- **The Surge** Scalability improvements with **rollups** and **sharding**.  
- **The Scourge** Addressing censorship resistance and decentralization issues.  
- **The Verge** Introduction of **Verkle Trees** to optimize data storage.  
- **The Purge** Pruning old state data to reduce node storage requirements.  
- **The Splurge** Miscellaneous upgrades and optimizations.  

---
### 2025.02.07

https://epf.wiki/#/eps/week2

#### Execution Layer

##### Block Validation
```typescript
stf(parentBlock: Block, curBlock: Block, state: State): [State, Error]
```
- `stf` -> state transition function
- verify header
  - merkle roots
  - gas limit
  - timestamp
  - etc.
- verify and process a block
##### Block Building

```typescript
build(env: Environment, pool: TransactionPool, state: State): [Block, State, Error]
```
- construct a new block from environment, transaction pool, and current state


### 2025.02.09
https://epf.wiki/#/eps/week2



#### Execution Layer

##### Uncle Blocks
- valid blocks that were mined but not included in the canonical chain ( usually due to network delays or simultaneous block production by multiple miners)
  - Why exist?
    - for miners to include uncle blocks in new blocks and receive **additional rewards**
    - to improve security and reduce centralization by compensating **valid but orphaned blocks**


##### Modern Ethereum Execution Clients

| **Client**     | **Language** | **Why Chosen?**                                      |
|---------------|-------------|------------------------------------------------------|
| **Geth**      | Go          | Easy concurrency, large developer community, mature ecosystem |
| **Erigon**    | Go          | Optimized storage, improved sync speed 💡         |
| **Nethermind** | C#         | Suitable for .NET ecosystem, enterprise-friendly   |
| **Besu**      | Java        | Hyperledger compatibility, enterprise-level support |
| **Reth**      | Rust 💡       | High performance, memory safety                     |


##### Consensus Layer vs. Execution Layer

| **Comparison**      | **Execution Layer (EL)**            | **Consensus Layer (CL)**       |
|---------------------|----------------------------------|----------------------------------|
| **Main Function**   | Processes transactions, executes smart contracts | Validates blocks, runs **PoS** consensus |
| **Core Components** | EVM, Transaction Pool, State Database | Beacon Chain, Validators, Slot/Epoch Mechanism |
| **Runs PoS?**       | ❌ No | ✅ Yes |
| **Consensus Role**  | Executes transactions but does not determine consensus | Runs PoS rules, decides block validity |
| **Client Examples** | Geth, Erigon, Nethermind | Prysm, Lighthouse, Teku, Nimbus |
| **Stored Data**     | Account balances, smart contracts, transaction history | Validator registry, voting data, finality information |

##### Why Did Ethereum Split CL and EL?

Before **The Merge**, Ethereum used a **PoW (Proof of Work) + Execution Layer** model, where a single client handled both **consensus (mining) and transaction execution**, leading to several issues:

- High computational cost (electricity).  
- Slow blockchain synchronization (the continuously growing state increased sync time).  
- Inefficient consensus mechanism (PoW was difficult to optimize).  

After **The Merge**, Ethereum split into **Execution Layer (EL) + Consensus Layer (CL)**, offering key advantages:  
- Execution Clients focus on **transaction execution and performance optimization**
- Consensus Clients handle **PoS rules**, improving **security and decentralization**
- EL and CL can be upgraded **independently**, enhancing **flexibility** 🌟

##### How Do CL and EL Communicate?

via the **Engine API**, 

1. **CL** requests **EL** to provide a new block
2. **EL** executes the block and returns the execution result
3. **CL** decides whether to accept the new block through validator voting.  

##### EVM
![EVM](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*a4cYYJZLlObtCGMIDiQeKQ.png)

- arithmetic (add, mod, etc.)
- bitwise 
- environment (block context, etc.)
- system (call, create, storage, return, etc.)
- control flow (branch, jump, etc.)
- stack (push, pop, etc.)
- memory (load, store, etc.)

##### p2p
- Execution layer operates** on devp2p (ethereum's p2p protocol).
  - **devp2p** allows nodes to support different **sub-protocols**:
    - **eth/68, eth/69** – for transaction and state synchronization.
    - **snap** – for fast state synchronization.
      - optimizations:
          1. **quickly syncing new nodes** by fetching **flat state data** instead of reconstructing the Merkle Patricia Trie.  
          2. uses **batch requests** (`GetAccountRange`, `GetStorageRanges`) to efficiently retrieve account and contract storage data.  
          3. **reduces disk I/O** by eliminating deep MPT traversals, improving sync speed. 
      - outcome:
        - enables nodes to complete state synchronization in **3-6 hours instead of days**.      
    - **les** – for light clients.
  - features  
    - efficiently communicate and synchronize data.
    - support different types of clients (full nodes, light clients, etc.) 💡
- **Responsibility**
  1. historical data
    - `GetBlockHeader`
    - `GetBlockBodies`
    - `GetReceipts`
    - etc.
  2. pending transactions
    - `Transactions`
    - `NewPooledTransactionHashes`
      - notifies peers of new pending transactions by sharing only their transaction hashes, reducing bandwidth usage
    - `GetPooledTransactions`
    - etc.
  3. state

    


### 2025.02.10
https://epf.wiki/#/eps/week3
  
- how to make digital scarcity?
  - solution: don't have a single trusted operator
- how to remove single trusted operator?
  - solution: consensus via "state machine replication"
- BFT: byzantine fault tolerance
  - 2PC: two-phase commit protocol
      1. **prepare phase** (each participant submit a binary vote for **commit** or **abort**)
      2. **commit phase** (if all participants vote commit, then execute the command)

  - PBFT: practical byzantine fault tolerance
    - utilizes **message signatures** and **multiple rounds of voting** to reach consensus.


| **Feature**            | **PBFT (Practical Byzantine Fault Tolerance)** | **2PC (Two-Phase Commit)** |
|------------------------|-----------------------------------------------|----------------------------|
| **Objective**         | tolerates malicious nodes | ensures **all** databases either commit or rollback a transaction consistently |
| **Fault Tolerance Model** | **Byzantine Fault Tolerance (BFT)**, capable of handling malicious nodes | **Crash Fault Tolerance (CFT)**, only handles node crashes but cannot tolerate malicious behavior |
| **Applicable Scenarios** 🌟 | **Blockchain** (Hyperledger, Tendermint, HotStuff, etc.) | **Distributed Databases & Transaction Management** (MySQL, PostgreSQL, distributed storage) |
| **Voting Mechanism**   | **Voting + Signature Authentication**, consensus is reached if more than **2/3 agree** | **Unanimous Agreement Required**, if **any** participant votes `"abort"`, the transaction is rolled back |
| **Communication Complexity** | **O(n²)** -> multiple rounds of message exchanges, where each node broadcasts messages to all other nodes 🌟| **O(n)** |



### 2025.02.11
https://epf.wiki/#/eps/week3

- PoS
  - in-protocol signal allow for **penalties**, not just rewards (compared to PoW)
    - scenarios:
      - double signing
      - going offline
      - equivocation -> voting for multiple **competing blocks** at the same height [💡](https://youtu.be/5gfNUVmX3Es?si=z7ba-EZ48kbH4I5V)
    - how?
      - **slashing**: cutting validator's stake for malicious behavior (up to 32 ETH)
      - **inactivity leak**: gradual stake reduction for being offline to maintain network liveness
    - benefits:
      - reduce attack surface
      - less resource intensive
-  Bribe attacks -> attackers influence validator behavior through **economic incentives**
     - examples:
       - paying validators to vote for specific blocks
       - bribing validators to stay offline
     - mitigations:
       - slashing penalties make bribe acceptance **more costly than potential gains**
       - withdrawal delays increase the cost of malicious behavior 
- GHOST vs. Casper
  - ethereum 2.0's consensus mechanism uses a modified version of **GHOST** combined with **Casper FFG** (Finality Gadget)

| Mechanism | Purpose | Consensus Type | Key Features |
|-----------|---------|---------------|--------------|
| **GHOST** | **fork-choice rule** to determine the canonical chain | **PoW & PoS** | • usually considers the **entire tree of blocks** (special case: LMD-GHOST looks at validators' **latest messages**)<br>• chooses the **heaviest** subtree<br>• prevents short-chain attacks |
| **Casper FFG** | **finality gadget** for PoS consensus | **PoS** | • checkpoint-based finalization<br>• uses validator voting<br>• provides economic finality through slashing |



### 2025.02.12
https://epf.wiki/#/eps/week3

- Attestation: statement or proof of a certain fact
  - Ethereum's PoS consensus [tutorial](https://youtu.be/5gfNUVmX3Es?si=_fkMJ6mn3LQGAE4V&t=83)
    - epoch -> **a time unit** used to **organize validator duties** and **finalize** blocks efficiently
      - **1 epoch = 32 slots** (Each slot is **12 seconds**)
      - **1 slot** = opportunity to **propose a block**
    - How to?
      1. **a validator is selected** to propose a block in **each slot**
      2. **other validators attest** using **BLS signatures** (aggregating hundreds of signatures into a single signature)
         - validators are assigned to different **committees**
         - each committee is responsible for validating **specific shards**
      3. **attestations are aggregated** and included in the next block
      4. **epoch ends after 32 slots** (~6.4 min)
      5. **If 2/3 validators agree, finalization occurs**
  - EAS: Ethereum Attestation Service 🔥 [tutorial](https://youtu.be/DMGj5GNll0k?si=LFfMfwQp7LqfNyCV&t=967)
    - [usage cases / ideas](https://docs.attest.org/docs/category/example-use-cases)
      - identity
      - statements
      - etc.



 



<!-- Content_END -->
