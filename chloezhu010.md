---
timezone: Europe/Berlin
---

# Chloe Zhu

1. 自我介绍
    - Chloe，[ETHPanda](https://x.com/EthPanda_org) Core team，prev [EIP Fun](https://x.com/EIPfun) project lead
    - 去年参加了第一期 EPF study group，对以太坊底层协议研发开始上瘾，去年10周 study group 的笔记也可以作为参考：https://hackmd.io/@chloezhux/epfsg_notes ， 目前我对 protocol network/ light client 比较关注和感兴趣
    - 底层协议的信息量巨大，前沿领域也在不断发展，需要一遍遍不断学习，so here I am~
    - 我的 [Twitter](https://x.com/Chloe_zhuX) 和 [Telegram](https://t.me/chloe_zhu)
2. 你认为你会完成本次残酷学习吗？
    - 一定！

## Notes

<!-- Content_START -->

### 2025.02.06
- What's a P2P network
    - Definition
        - A decentralized communication model where nodes in the network can communicate directly with each other witout a central server
        - Unlike traditional client/ server model, where a centralized authority manage all connection & data transfer, p2p network distribute workload and data among participants
    - Key features of p2p network
        - decentralized: no central server, nodes share data directly
        - scalable: network grows as more nodes join
        - fault tolerance: no single point of failure
        - resource sharing: peers can share computing power, storage, or bandwidth
    - Type of p2p network
        - unstructured p2p
            - nodes randomly connect (eg. Gnutella, Kazaa)
        - structured p2p
            - use algo to route data (eg. DHT in BitTorrent, Kademila)
        - hybrid p2p
            - mix of decentralized peers and some centralized componenets
- What type of p2p is Ethereum and Bitcoin
    - Bitcoin: mostly unstructured p2p with a gossip protocol for tx & block propagation
        - Network structure
            - bitcoin nodes randomly connect to other nodes
            - tx and blocks are relayed to neighbours, which propagate them further
            - nodes discover & main peer lists dynamically
        - Data progagtion
            - Uses flooding (gossip protocol) where each node forwards data to its connected peers
        - Peer discovery
            - use DNS seed nodes, hardcoded bootstrap nodes, and peer exchanges
    - Ethereum: structured p2p with Kademlia DHT
        - Network structure
            - used a modified Kademlia DHT to structure peer discovery & routing
            - nodes are identified by unique IDs and stored in tree-like structure for efficient lookup
            - allow for faster peer discovery & data retrieval compared to Bitcoin
        - Data propagation
            - also use gossip protocol
            - has additional subnetworks (devp2p, libp2p) for different types of data, eg. state sync, block propagation, tx relaying
        - Peer discovery
            - use a Kademlia DHT for peer lookup
            - nodes maintain a routing table that organizes peers based on proximity in the DHT
      
    |         | Bitcoin | Ethereum |
    | -------- | ------- | ------- |
    | network type  | unstructred p2p    | structured p2p (kademlia DHT)    |
    | node discovery | random peer selection, DNS seed     | kademlia DHT for structured peer lookup   |
    | data porpagation    | gossip-based (flooding)    | gossip-based + DHT routing   |
    | efficiency    | redundant message forwarding    | more efficient lookup   |

### 2025.02.07
- What's DHT and Kademlia DHT
    - DHT (Distributed Hash Table)
        - a decentralized system for storing & retrieving key-value pairs in a distributed network
        - How it works
            - Each node in the network store a portion of the key-value pairs
            - Keys are hashed to produce a unique identifier -> determine which node is responsible for storing the cooresponding value
            - When a node wants to retrive a value, it uses the DHT to locate the node responsible for that key
    - Kademlia DHT
        - a specific implementation of a DHT that is widely used in P2P networks, including Ethereum, BitTorrent, and IPFS. It was introduced in 2002 by Petar Maymounkov and David Mazières and is known for its efficiency, simplicity, and robustness
        - Key features
            - use a binary tree-based routing algo to locate nodes & data in O(logN) steps
            - use XOR (excl. OR) to measure the distance btw nodes and keys
            - send queries to multiple nodes simultaneously
            - each node & key is assigned a unqiue 160-bit ID
            - each node maintains a routing table (k-bucket) that stores info about other nodes in the network
        - How it works
            - Node ID assignment: each node is assigned a unique 160-bit ID, usually generated by hashing its IP address or public key
            - Key-value storage: keys are also hashed to 160-bit ID; each key-value pair is stored on the node whose ID is closest to the key ID (bassed on XOR)
            - Lookup process: a node sends a lookup request to the nodes in its routing table that are closest to the key's ID; These nodes respond with information about even closer nodes, and the process repeats until the closest node (responsible for the key) is found
            - Routing table maintenance: nodes periodically update their routing tables by querying other nodes and exchanging information about peers
        - Application
            - Used in Ethereum, BitTorrent, IPFS
    - Other types of DHT
        - Chord (consistent hashing-based)
            - use ring structure: node ID and keys are arranged in a circular space
            - each node maintains a finger table pointing to nodes at exponentially increasing distance in the ring
            - require more maintenance when nodes join/ leave, whereas Kademlia’s XOR-based buckets provide better resilience
        - Pastry (prefiex-based routing)
            - use prefix-matching for routing. Nodes and keys have numerical IDs, and nodes forward requests to peers whose ID shares the longest prefix with the target
            - each node keeps a leaf set (close nodes) and a routing table for long-range hops
            - need more state per node (bigger routing table) than Kademlia
        - CAN (content addressable network)
            - use a d-dimensional coordinate space, where each node owns a zone
            - keys are mapped to coordinate points in this space, and nodes forward queries toward the target zone
            - lookup complexity is O(d N^(1/d)), scalable with more dimensions
            - less efficient than Kademlia for large networks because it requires more hops in high-dimensional spaces
        - Why Kademlia is a better choice
            - XOR-based distance, enables parallel lookups
            - better fault tolerance: node cache more peer info, more resilient to churn
            - efficient lookups: O(logN) hops with min maintenance overhead
- What's Ethereum Protocol design in high level
    - Design philo
        -  Simplicity, Universality, Modularity, Non-discrimination, Agility
    - Main component
        - EL: execution engine
            - handle user tx and all state (addr, contract data)
        - CL: implement pos mechanism
            - ensure security and fault tolerance
    - Implementation & development
        - Client: an implementation of the EL or CL
        - Node: a computer running this client & connecting to the network; a node is a pair of EL and CL clients actively participating in the network
        - Client diversity strategy 
    - Testing & security
        - Different testing tools for state transition testing, fuzzing, shadow forks, RPC tests, client unit tests and CI/CD, etc.
    - Coordination

### 2025.02.10
- Protocol architecture
    - Graph: https://epf.wiki/#/wiki/protocol/architecture
    - What's user APIs and beacon APIs
        - User API (aka JSON-RPC API)
            - primary interface for interacting with the EL
            - used by wallet, dapps etc.
            - Key features
                - JSON-RPC protocol: a lightweight remote procedure call (RPC) protocol, that allows clients to send & receive response in JSON format
                - Common use: send tx, query blockchain data (eg. balance, contract states), deploy & interact with smart conracts, listen for events (logs emitted by smart contract)
                - Endpoints: expose endpoints eg. eth_sendTransaction, eth_getBalance, eth_call, eth_getLogs
        - Beacon API
            - interface to interact with the beacon chain, which coordinate validators and achieve consensus
            - Key features
                - RESTful interface
                - Common use: query info about the beacon chain (block headers, validator status), submmit attestations & block proposals from validators, monitor the status of the beacon network
                - Endpoints: eg. /eth/v1/beacon/blocks (retrive beacon chain blocks), /eth/v1/validator/attestation (submit an attestation from a validator)
                - Staking pools & monitor tools use the api to track validator performance and network health
    - Issue with JSON RPC api
        - Centralization
            - rely on centralized infra provider (eg. infura, alchemy, quicknode) to access Ethereum nodes via json rpc. These service act as intermediaries, reducing the need for developers to run their own nodes
            - barrier to run full nodes: it requires significiant resources (storage, bandwidth, computation power) to run a full node, so many devs opt for centralized service instread
        - Scalability
            - high load on nodes: can lead to performance bottlenecks and increased cost for node operators
            - inefficient data retrieval: not optimized for querying large amounts of data, can result in slow response time and high latency
        - Security
            - json-rpc endpoints can expose sensitive info if not properly secured (eg. account balance, tx history)
            - public json-rpc endpoints are often targeted by DDoS attacks
            - by default json-rpc don't require authentication, making it easy for unauthorized user to access node data
        - Lack of modern features
            - No RESTful design
            - limited tool: lack support for features like filtering, sorting etc.
            - verbose and complex
        - Potential Alternative/ Solution
            - Decentralized node infra: eg. the Graph, EPNS
            - Light clients and stateless: reduce the resource required for running nodes
            - RESTful api: eg. Besu
            - Improved json-rpc: add support for batch request, better error handling etc.
### 2025.02.11
- Blockchain level protocol
    - reference link: https://epf.wiki/#/wiki/protocol/design-rationale
  - Accounts over UTXOs
      - UTXO (unspent tx output)
      - Account
      - What's the pros/ cons of account vs UTXO? Why Ethereum chooses account-based model?
  - Merkle patricia trie
      - a modified MPT
      - deterministic and cryptographically verifiable
  - Verkle tree
      - vector commitments allow for much smaller proofs (aka witness)
  - RLP (recursive length prefix)
  - SSZ (simple serialize)
  - Hunt for finality: Casper FFG + LMD GHOST
  - Discv5: the discovery protocol
      - a kademlia based DHT to store ENR records
      - ENR (ethereum node record) contain routing info to establish connections between peers
### 2025.02.12
- Pros Cons of Account vs UTXO
    - Account: How it works in high level
        - the blockchain maintains a global state composed of accounts
        - each account has a balance (in smart contract, storage and code)
        - tx directly modify the state of these accounts
    - UTXO: How it works in high level
        - the blockchain tracks unspent tx outputs
        - a UTXO represents a chunk of crypto that has been created as an output of a tx, but has not yet been spent
        - tx consume existing UTXO and create new ones
    - Comparison
  
    |         | Account-based | UTXO-based |
    | -------- | ------- | ------- |
    | state representation | global state of accounts & balances   | set of unspent tx outputs    |
    | tx logic  | directly modify account balance   | consume UTXOs and create new ones    |
    | complexity  | easier for dev, esp. for smart contractss   | more complex for devs, esp. for advanced logic   |
    | parallelizability  | limited, as txs modifying the same account must be processed sequentially   | high, as independent UTXOs can be processed in parallel    |

- Why MPT then Verkle tree
    - MPT
        - a data structure, that combines merkle tree (provide cryptographic proofs to verify the data integrity) and patricia trie (a compressed trie that stores key-value pairs)
    - Verkle tree
        - more advanced data structure to address the issue of MPT
        - vector commitment + merkle tree, it uses vector commitment (eg. polynomial commitments) to create smaller & more efficient proofs
    - Comparison
 
    |         | MPT | Verkle tree |
    | -------- | ------- | ------- |
    | proof size | large (scale with tree depth)   | small (constant or logarithmic)   |
    | efficiency | less efficient for deep trees   | more efficient for deep trees   |
    | stateless clients | inefficient due to large proof   | efficient due to compact proof   |
    | scalability | limited by proof size & depth   | better scalability for large states   |
    | cryptographic basis | merkle proof (hash-based)   | vector commitment (eg. polynomial)   |
- What's RLP? What's its purpose? Why want to convert to SSZ?
    - Recursive length prefix: a serialization format, to encode & decode data structure into compact byte array
    - Where is it used in Ethereum
        - transaction: txs are serialized using RLP before being broadcasted or stored in blocks
        - block: seralized for storage and transmission
        - state trie: stored in a MPT, where keys and values are RLP-encoded
        - p2p networking protocol: RLP is used to encode data for p2p networking
    - Issue
        - don't natively support some data type (Eg. int, float, boolean), but treat everything byte arrays
        - not optimized for merkleization
        - add overhead for small data structure due to length-prefixing scheme
        - not human-readable
        - variable-length coding makes it harder for light clients to parse and verify data
- What's SSZ? What's its purpose?
    - simple serialize is a serialization format designed specifically for eth2
    - encode & decode data structure in a more efficient, type-aware, optimized for merkleization style
### 2025.02.13
Update the notes on mindmap: https://ab9jvcjkej.feishu.cn/mindnotes/IfABbVTMfmg5IFnqinEcmcDqnFe#mindmap
### 2025.02.15
- 最近新出的关于 hash table 算法的研究和论文，感觉对 DHT 和以太坊会有不少影响，以下是 ds + gpt 老师的回复
    - 研究总结链接：https://mp.weixin.qq.com/s/3IvM0b9kHdO66KV18cv11w
    - 对以太坊的影响
        - 状态存储优化
            - 更高效的 MPT：可能优化 MPT 的结构，减少存储空间和查询时间
        - 节点发现和通信
            - discv5 协议改进：可能优化 Discv5 的路由表结构，使得节点发现更快、更可靠
            - 降低网络开销：通过改进哈希表算法，可以减少节点之间的通信开销，从而提高以太坊网络的整体效率
        - 可扩展性
            - 状态同步加速：更快的哈希表可以提升存储引擎查询效率，进而加快区块同步（fast sync/ snap sync）和轻客户端数据索引
        - smart contract 执行和 EVM 性能
            - 智能合约存储 & 计算涉及大量哈希运算（例如 SSTORE 操作存储合约变量），更快的哈希表可以优化存储访问，降低 gas 费用
            - 尤其是 MEV 交易、去中心化交易所（DEX）撮合、L2 Rollup 状态提交 等，都可能受益
    - 对 DHT 的影响
        - 加速 p2p 网络的节点发现
            - Kademlia DHT 依赖 XOR 距离度量进行路由查询，每个查询涉及多个哈希表操作（节点存储、索引、查询）
            - 更快的哈希表 可以优化 查找最近节点的速度，从而提升 网络连接稳定性 和 低延迟通信
        - 优化 DHT 结构的扩展性
            - 目前的 DHT 通常采用 树形（Trie）或分桶（Bucket）存储 进行哈希索引
            - 更快的哈希表算法可能允许 减少存储开销，或者提高节点维护的效率，让 DHT 可以扩展到更大规模的 P2P 网络
        - 改进分布式存储系统
            - DHT 作为去中心化存储（如 IPFS、Arweave、Filecoin）的核心组件，影响数据定位和检索速度
            - 更快的哈希表可以减少存储查询延迟，优化去中心化存储网络的性能
### 2025.02.16
Update notes on devp2p and discv5
https://ab9jvcjkej.feishu.cn/mindnotes/IfABbVTMfmg5IFnqinEcmcDqnFe#mindmap
### 2025.02.17
Update notes on 
- What problem does portal network solve for light clients
- Why portal needs DHT
### 2025.02.18
- Participated Portal implementer call and updated the call notes: https://hackmd.io/@chloezhux/H1x36pYIJg
### 2025.02.19
- Go through how CL and EL client work in pair, what's the workflow?
- Lookup what's block header, body, and receipt
- To finish: different sync modes for EL and CL
https://ab9jvcjkej.feishu.cn/mindnotes/IfABbVTMfmg5IFnqinEcmcDqnFe#mindmap
### 2025.02.20
- Followed the node setup workshop to setup the Geth & Lighthouse pair https://epf.wiki/#/eps/nodes_workshop
  - On Holesky, lighthouse always timeout when checkpoint syncing
  - On Sepolia, everything works well (why?)
### 2025.02.21
- Learned different sync modes of EL & CL
- Setup node notes
    - Always verify the downloaded package first
    - Geth 用的 pebbleDB 而不是 levelDB
    - Setup CL client 需要先 generate jwtsecret file 用于 EL CL 之间的沟通
    - Default sync mode
        - geth: snap sync
        - lighthouse: default mode 会显示 syncing from the genesis is insecure 推荐 checkpoint sync
            - checkpoint sync endpoitns: https://eth-clients.github.io/checkpoint-sync-endpoints/
    - Testnet: Holesky 在我本地总是 timeout，我再重新 run 一个看看问题
    - EL run 起来后
        - 会一直 looking for peers，没有 CL 给 header EL 无法 sync
    - CL run 起来后
        - syncing deposit contract block cache
        - synced
### 2025.02.23
- Rewatched the EL overview by lightclient
    - review the state transition function and the execution payload building function
    - https://ab9jvcjkej.feishu.cn/mindnotes/IfABbVTMfmg5IFnqinEcmcDqnFe#mindmap
### 2025.02.24
- Rerun the client pair with ephemery, had some problem with pairing geth and lighthouse
- Notes
- ## Setup the client pair
### EL: Geth
1. Download the geth's pre-built binary (in tar.gz) and the stable release (in .asc)
    - https://geth.ethereum.org/downloads

2. Download the signature
    - openPGP signature
3. Import the key
    ```markdown
    gpg --import macos_builder_key.asc 
    ```
    ```markdown
    cat macos_builder_key.asc 
    ```
    When cat, should return: 
    ```markdown
    -----BEGIN PGP PUBLIC KEY BLOCK-----
    Comment: Hostname: 
    Version: Hockeypuck 2.2

    xsFNBFggyxoBEAC299KoAS43p0FyJetAc7E0m1B/wnpyQesFycop/1csNQCjSGMy
    EvERt8Mv5VvbyZ696gTnzyLP/YHvx5+j/lKZhixw+7VkOng6JgPF3YgN3WrykIjK
    .....
    -----END PGP PUBLIC KEY BLOCK-----
    ```

4. Verify the downloaded binary with the key
    ```markdown
    gpg --verify Geth_Darwin_AMD64_1.15.2.asc Geth_Darwin_AMD64_1.15.2.tar.gz 
    ```

    Should return:
    ```markdown
    gpg: Signature made Lun 17 fév 13:16:21 2025 CET
    gpg:                using RSA key 558915E17B9E2481
    gpg: Good signature from "Go Ethereum macOS Builder <geth-ci@ethereum.org>" [unknown]
    gpg: WARNING: This key is not certified with a trusted signature!
    gpg:          There is no indication that the signature belongs to the owner.
    Primary key fingerprint: 6D1D AF5D 0534 DEA6 1AA7  7AD5 5589 15E1 7B9E 2481
    ```
5. Unpack the Geth binary
    ```markdown
    tar -xzf Geth_Darwin_AMD64_1.15.2.tar.gz 
    ```

### CL: Lighthouse
1. Download the lighthouse's pre-built binary (in tar.gz) and the stable release (in .asc)
    - https://github.com/sigp/lighthouse/releases
    - recommended downloading stable release > beta release
2. Obtain & importe the PGP key
    - lighthouse PGP key: 15E66D941F697E28F49381F426416DC3F30674B0 from its release page
    ```markdown
    gpg --keyserver keys.openpgp.org --search-keys 15E66D941F697E28F49381F426416DC3F30674B0
    gpg --recv-keys 15E66D941F697E28F49381F426416DC3F30674B0
    ```
    Should return:
    ```markdown
    gpg: data source: http://keys.openpgp.org:11371
    (1)	Sigma Prime <security@sigmaprime.io>
        4096 bit RSA key 26416DC3F30674B0, created: 2020-11-27
    Keys 1-1 of 1 for "15E66D941F697E28F49381F426416DC3F30674B0".  Enter number(s), N)ext, or Q)uit > 

    gpg: key 26416DC3F30674B0: public key "Sigma Prime <security@sigmaprime.io>" imported
    gpg: Total number processed: 1
    gpg:               imported: 1
    ```
3. Verify the downloaded binary with the key
    ```markdown
    gpg --verify Lighthouse_v6.0.1_macOS.tar.gz.asc Lighthouse_v6.0.1_macOS.tar.gz 
    ```
    Should return: 
    ```markdown
    gpg: Signature made Lun 16 déc 05:07:42 2024 CET
    gpg:                using RSA key 15E66D941F697E28F49381F426416DC3F30674B0
    gpg: Good signature from "Sigma Prime <security@sigmaprime.io>" [unknown]
    gpg: WARNING: This key is not certified with a trusted signature!
    gpg:          There is no indication that the signature belongs to the owner.
    Primary key fingerprint: 15E6 6D94 1F69 7E28 F493  81F4 2641 6DC3 F306 74B0
    ```
4. Unpack the lighthouse binary
    ```markdown
    tar -xzf Lighthouse_v6.0.1_macOS.tar.gz 
    ```
## Launch the client pair
1. Launch geth in default
    ```markdown
    ./geth
    ```
2. Create a separate folder to store data
    ```markdown
    mkdir geth_data lighthouse_data
    ```
3. See the comments/ flags
    ```markdown
    ./geth --help
    ./lighthouse --help
    ```
4. Launch geth with config
    ```markdown
    ./geth --holesky \
    --datadir geth_data \
    --syncmode snap \
    --http \
    --http.port 8545 \
    --authrpc.jwtsecret /tmp/jwt \
    --authrpc.port 8551
    ```
5. Create/ Cat a JWT secret file
    ```markdown
    cat /tmp/jwt
    ```
    Should return a 64-bit code
6. Launch lighthouse with config
    - launch with checkpoint sync
    - beacon chain checkpoint endpoints: https://eth-clients.github.io/checkpoint-sync-endpoints/

    ```markdown
    ./lighthouse beacon_node \
    --network holesky \
    --datadir lighthouse_data \
    --http \
    --checkpoint-sync-url https://checkpoint-sync.holesky.ethpandaops.io \
    --execution-endpoint http://127.0.0.1:8551 \
    --execution-jwt /tmp/jwt
    ```
    Mine always return error as below (if switch to Sepolia, everything works fine)
    ```markdown
    Feb 24 11:33:32.566 INFO Logging to file                         path: "lighthouse_data/beacon/logs/beacon.log"
    Feb 24 11:33:32.569 INFO Lighthouse started                      version: Lighthouse/v6.0.1-0d90135
    Feb 24 11:33:32.569 INFO Configured for network                  name: holesky
    Feb 24 11:33:32.578 INFO Data directory initialised              datadir: lighthouse_data
    Feb 24 11:33:32.585 INFO Deposit contract                        address: 0x4242424242424242424242424242424242424242, deploy_block: 0
    Feb 24 11:33:32.651 INFO Blob DB initialized                     oldest_data_column_slot: None, oldest_blob_slot: Some(Slot(950272)), path: "lighthouse_data/beacon/blobs_db", service: freezer_db
    Feb 24 11:33:37.180 INFO Starting checkpoint sync                remote_url: https://checkpoint-sync.holesky.ethpandaops.io/, service: beacon
    Feb 24 11:36:37.877 CRIT Failed to start beacon node             reason: Error loading checkpoint state from remote: HttpClient(, kind: timeout, detail: operation timed out)
    Feb 24 11:36:37.877 INFO Internal shutdown received              reason: Failed to start beacon node
    Feb 24 11:36:37.878 INFO Shutting down..                         reason: Failure("Failed to start beacon node")
    Failed to start beacon node
    ```
## Launch the client pair with ephemery
1. Download the testnet_all.tar.gz from the ephemery release
    - https://github.com/ephemery-testnet/ephemery-genesis/releases/
2. Init the ephemery on Geth
    ```markdown
    geth --datadir ephemery init ephemery_test_all/genesis.json
    source ephemery_test_all/nodevars_env.txt 
    ```
    Should return
    ```markdown
    INFO [02-24|12:50:57.200] Maximum peer count                       ETH=50 total=50
    INFO [02-24|12:50:57.204] Set global gas cap                       cap=50,000,000
    INFO [02-24|12:50:57.204] Initializing the KZG library             backend=gokzg
    INFO [02-24|12:50:57.226] Defaulting to pebble as the backing database
    INFO [02-24|12:50:57.227] Allocated cache and file handles         database=/Users/chloe/Documents/Github/node_setup_2/ephemery/geth/chaindata cache=16.00MiB handles=16
    INFO [02-24|12:50:57.284] Opened ancient database                  database=/Users/chloe/Documents/Github/node_setup_2/ephemery/geth/chaindata/ancient/chain readonly=false
    INFO [02-24|12:50:57.284] State schema set to default              scheme=path
    ERROR[02-24|12:50:57.284] Head block is not reachable
    INFO [02-24|12:50:57.321] Opened ancient database                  database=/Users/chloe/Documents/Github/node_setup_2/ephemery/geth/chaindata/ancient/state readonly=false
    INFO [02-24|12:50:57.321] Writing custom genesis block
    INFO [02-24|12:50:57.428] Successfully wrote genesis state         database=chaindata hash=a45355..e0c1d2
    INFO [02-24|12:50:57.428] Defaulting to pebble as the backing database
    INFO [02-24|12:50:57.428] Allocated cache and file handles         database=/Users/chloe/Documents/Github/node_setup_2/ephemery/geth/lightchaindata cache=16.00MiB handles=16
    INFO [02-24|12:50:57.481] Opened ancient database                  database=/Users/chloe/Documents/Github/node_setup_2/ephemery/geth/lightchaindata/ancient/chain readonly=false
    INFO [02-24|12:50:57.481] State schema set to default              scheme=path
    ERROR[02-24|12:50:57.481] Head block is not reachable
    INFO [02-24|12:50:57.520] Opened ancient database                  database=/Users/chloe/Documents/Github/node_setup_2/ephemery/geth/lightchaindata/ancient/state readonly=false
    INFO [02-24|12:50:57.520] Writing custom genesis block
    INFO [02-24|12:50:57.600] Successfully wrote genesis state         database=lightchaindata hash=a45355..e0c1d2
    ```
3. Launch geth with config
    ```markdown
    geth --datadir ephemery --authrpc.jwtsecret=/tmp/jwt --bootnodes $BOOTNODE_ENODE --networkid $CHAIN_ID  --http
    ```
4. Launch lighthouse with config
    ```markdown
    ./lighthouse beacon_node -t ephemery_test_all --execution-endpoint http://localhost:8551 --execution-jwt=/tmp/jwt --boot-nodes=$BOOTNODE_ENR_LIST
    ```
### 2025.02.25
- Downloaded the latest release fron Trin to play around
    - Trin book: https://ethereum.github.io/trin/introduction/quickstart.html
    - Release page: https://github.com/ethereum/trin/releases
- The architecture part is useful to understand Trin's codebase: https://ethereum.github.io/trin/developers/architecture/index.html
- Had some issue when querying data, need further investigation

### 2025.02.26
- 重新整理了一下 node workshop 的 notes，方便大家可以不用看视频，直接跟着 setup client pair
    - https://hackmd.io/QK_4OJQ7Sputq93-Urz_gQ

### 2025.02.27
- Pectra Holesky testnet 升级分叉回顾
  - 主要问题：三个主要执行层客户端（besu、nethermind、geth）出现 config issue
    - EL clients had an issue where they forgot to add the correct Deposit contract address, which matters for Pectra Requests Hash calculation
    - An invalid chain was justified (not finalized!), due to the majority of clients having the bug. 
    - https://x.com/parithosh_j/status/1894236676567785814
  - 目前进展
    - https://x.com/TimBeiko/status/1894472196464197917
    - EL config issue is fixed, releases out
    - CL clients struggle to sync, working on patches, some are being tested right now
    - We expect to save Holesky by building on the minority chain 
  - 一些核心开发者的反馈
    - https://x.com/TimBeiko/status/1894177342295244981
    - https://x.com/vdWijden/status/1894330068668756353
  - 今晚 ACD call 将讨论如何修复 Holesky 分叉
  - 针对 Pectra 升级的 $2,000,000 bug bounty 
    - https://x.com/parithosh_j/status/1894236703667491232
  - 此前针对 Pectra 升级协调的反思讨论
    - https://ethereum-magicians.org/t/pectra-retrospective/22637

### 2025.02.28
- 今天没有学习啥新知识，列一些准备做的 Todo
    - 梳理这一个月以来的知识，参考优秀残酷学友 zhouCode 的思维导图：https://github.com/IntensiveCoLearning/Ethereum-Protocol-Fellowship/blob/main/zhouCode.md
    - 研究去年 EPF cohort5 core dev 列的一些项目：https://github.com/eth-protocol-fellows/cohort-five/blob/main/projects/project-ideas.md
    - 看本周四 ACD 针对 Pectra 升级 holesky 分叉的讨论：
        - ACDE call 2.27: https://www.youtube.com/watch?v=tlezpGztpi8
        - Holesky Validator Incident Response Call: https://www.youtube.com/watch?v=ksr-6iuSvrg

### 2025.03.02
- Watched the video of ACDE call: https://www.youtube.com/watch?v=tlezpGztpi8
    - Pectra holesky fork update
        - altho new validators are coming online to support the correct head of holesky, most validators on holesky are still in the existing process (slashed for their incorrect block proposal or suffer an inactivity leak for not proposing to the correct chain)
        - block production has recovered to 44% success rate on holesky
    - Holesky next step
        - plan to coordinate a mass slashing for all validators attesting to the incorrect head of the chain to force holesky into a state of finalization
    - Sepolia fork timing
        - still scheduled for March 5th
    - Other holesky initiatives
        - upgrade CL clients so taht they can checkpoint sync from a socially coordinated epoch or block, rather than a finalized one
        - agree on a single EL genesis/ fork config format to avoid issue of misconfig system contract address
        - use a fork ID to coordinate client team on system contract config btw forks
    - EIP 4444 (didn't have time to be discussed on this call)
        - scheduled to be implemented on May 1st, 2025
        - allow for dropping of block bodies and receipts from pre-merge blocks by clients
        - prev. agreed by all major clients( Besu, EthereumJS, Erigon, Geth, Nethermind, Nimbus, Reth)

<!-- Content_END -->
