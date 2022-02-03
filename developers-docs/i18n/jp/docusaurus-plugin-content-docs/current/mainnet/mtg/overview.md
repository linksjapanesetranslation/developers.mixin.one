---
title: 概要
sidebar_position: 8
---

# 概要

マルチシグネチャ技術とMixinネットワークの高速転送を組み合わせることで、Mixinネットワーク上に信頼できる分散アプリケーションを展開できるソリューション「MTG」（Mixin Trusted Group）を提案します。

![MTG](./overview-architecture.svg)

MTGはシャーディング技術の中でも最も効率の良い手段です。各MTGの実装は、複数の独立したノードからなるMixin Networkのパラチェーンであり、Mixinメインネットワークのマルチシグネチャ技術によりコンセンサスが得られます。

### 特徴

- セキュリティ

  Mixin mainnet Kernel PoS is combined with application consensus, achieving double security.
  

- 堅牢性

  Each node will strictly review the program code, rather than simply deploy and execute it. If program vulnerabilities are found, nodes will quickly respond and repair, and the cold and hot isolation of asset storage further enhances the security.
  
- マルチチェーン対応

  It supports all public chains supported by Mixin, cross-chain asset exchange can be achieved easily.


- 高性能

  It supports high concurrency and can be applied to commercial scenarios for a large number of users on a large scale.

- トランザクションの高速処理

  Transfers are free and instant, which meets the requirement of commercial scenarios such as micropayments and everyday payment.

- スケーラビリティ

  Being infinitely scalable, each MTG is an independent decentralized network that does not consume the mainnet system resources.

- 互換性

  Compatible with all blockchain networks and traditional centralized Internet technologies and ecosystem. For example, the Ethereum virtual machine can be deployed directly after simple modification of the MTG consensus. All Ethereum smart contracts can be used on Mixin, which is faster. There are no transfer fees.

- 柔軟性

   Flexible development language: Developers can freely use any language (Go, Java, PHP, Rust, etc.) to implement MTG.
  
   Flexible data storage: You can use blockchain technology to store data, or you can directly use traditional databases such as MySQL, PostgreSQL, MongoDB, SQL Server, etc. to store data. The scheme is similar to distributed databases but stored separately by different nodes.
  
   Flexible number of nodes: MTG supports hierarchical consensus, each node in the upper layer corresponds to 255 nodes in the lower layer, the top-level consensus supports up to 255 nodes, the second-layer consensus supports up to 255 * 255 nodes, and so on.

### ユースケース

Using the MTG to develop decentralized applications is very simple. It can be designed and developed like centralized projects. The only difference is that the data needs to be asynchronously waited for confirmation and signature by other nodes before being stored in the database. After the development is completed, we need to deploy the code to each node. It a good choice if you want to upgrade traditional projects to blockchain projects.

- [パブリックチェーン](./chains)

  Using the MTG solution to develop a PoS public chain can save the time and cost of modules such as consensus, network, and transactions. The new chain obtains Mixin mainnet security, high performance, free transfer within seconds, multi-chain support, and many other features while maintaining the independence of accounting. It is a parallel chain of Mixin.

- [ステーブルコイン](./stablecoin)

  Stable coins have a wide range of uses. In addition to serving as a safe-haven asset, there is a great demand in use cases like short-term loans, cross-border remittances, and fiat currency deposits. Using the MTG solution to issue, custody stable coins, and develop supporting Dapps is secure, stable, and efficient.

- [自動マーケットメイカー（AMM）](./amm)

  Compared with the traditional order book transactions, automated market makers makes the process automated and free of human interventions. Developing automated market makers with MTG is more efficient than Ethereum-based Uniswap, and has higher security, stability and fault tolerance, etc.

- [モーゲージローン](./lend)

  MTG-based decentralized mortgage loans are characterized by high performance, openness, data transparency, currencies richness, gas fee free and instant borrow and repay, etc.

- [分散型取引所](./exchange)

  MTG-based decentralized exchanges are characterized by high performance, asset isolation (separation of maker order assets and wallet assets), openness and transparent data on the chain, currencies richness, free of transfer fee for maker and taker orders, and autonomous currency listings.

- オラクル

   By collecting data on and off chain, the decentralized oracle service based on MTG collects can provide low-latency and cost-effective data services for other products in Mixin ecosystem.

- Atomic Swap

  Atomic swap can be implemented with MTG. Assets are safely locked in the node's multi-signature account, and the assets are either successfully exchanged or returned.

- Coin Mixing Technology

  Mixin transfers have a high degree of privacy. Nodes cannot know the identities of specific transaction parties. If you want to further enhance the privacy of both parties of the transaction, MTG coin mixing technology is your friend.

### Q&A

- How to identify whether it is a Dapp based on MTG?

  If transferring money to Dapp through Mixin Messenger is not a multi-signature transaction, then the Dapp must be centralized.

  ![Mutisig Trasaction](./overview-mutisig-transaction.png)

- Are there any MTG-based products?

  Search for 7000103117 in Mixin Messenger to try the MTG version of 4swap.


- How to find a trustworthy node team?

 Projects can attract third-party teams to participate through regular currency issuance and elections. They can also directly contact well-known teams such as exchanges, wallets, blockchain browsers, smart contract teams, and others to participate. A professional MTG service provider will be on Mixin in the future, providing services such as one-click deployment.

---
MTG provides extremely high theoretical and practical value for the popularization and large-scale application of blockchain. This is a very mature technical solution. Compared with other smart contract platforms, it has high performance, flexibility, and other advantages. Welcome to join the construction of MTG.
