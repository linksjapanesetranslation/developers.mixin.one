---
title: ラップドトークン
sidebar_position: 14
---

# ラップドトークン

The market for wrapped tokens(tokenized blockchain coins anchoring with coins on another blockchain) is now huge. The market value of BTC locked by WBTC, tBTC, and RenBTC has exceeded 1 billion U.S. dollars. Using MTG to issue, host wrapped tokens, and develop supporting Dapps is safe, stable, and efficient.


### 優位性

- 分散化とセキュリティ
  
  Protected by Mixin mainnet, assets under custody are well protected through the cold and hot wallet isolation and co-managed by up to 255 party multi-signature. The Dapps can be deployed on 255 node servers.

- 安定性
  
  The Dapps can be developed using more stable and mature languages, such as Go, Java, PHP, etc. The redeem service is stable and smooth, without network congestion and high gas problems. 

- 効率性
  
  The Dapps can be deployed on high-performance node servers to provide users with low-latency and efficient subscription and redemption services.

- 強靭性

  It naturally supports a variety of cross-chain assets. Transactions are free and instant with scalability to serve a large number of users.

### ステップ

- 分散型マルチシグネチャ組織

  マルチシグネチャ組織の仕事は、主にDappプログラムのアセット共同管理、レビュー、展開、ガバナンスなどに参加することです。複数の著名なチームや企業を束ねて分散型マルチシグネチャ組織を形成したり、暗号資産を発行して投票による分散型の自律組織を形成したりすることができます。例えば、5つのノードがある資産を共同管理する場合、資産を動かすには3つのノードの署名が必要である等の合意形成の設定などが可能です。合意形成の閾値は、`3/5`、`4/7`...`171/255`といったように、一般的にノードの3分の2を設定します。

- ラップドトークンの発行

  Wrapped tokens can be issued based on ERC20, TRC20, EOS, and other platforms that support token issuance. The token issuance contract can be set with a fixed limit or not, then deposit into Mixin network, and transfer to the asset co-management account. The tokens can also be withdrawn to other DeFi platforms that support them for trading.

- Dappの開発

  1. 開発環境: Go、Java、PHPなどのプログラミング言語と、MySQL、PostgreSQL、MongoDBなどのデータベースシステムを扱える必要があります。

  2. トランザクション処理: Nodes need to synchronize related UTXO transactions continuously. Subscription and redemptiond operations are processed separately according to the `Memo` of the transaction. All operations need to be verified and the data must be signed by enough nodes to be recorded in the database. All operations need to initiate a Mixin mainnet multi-signature, with Memo information including initiator, operation type, amount and other information, the format is as follows (recommended to use MessagePack + base64 to compress the data):

  ```golang
  memo = base64.StdEncoding.EncodeToString(msgpack(OrderAction{
    A:"3596ab64-a575-39ad-964e-43b37f44e8cb",  // Asset ID
    S:"43d61dcd-e413-450d-80b8-101d5e903357",  // Initiator
    M:"10",                                    // Amout
    T:"subscribe"                              // Operation Type
  }))
  ```

  3. 資産管理: All assets are managed by the node multi-signature. Collateral for subscription will be saved to the multi-signature address. Redemptions take effect after most nodes have verified the wallet.

- セキュリティ対策

  1. マルチシグネチャによる資産管理やコールドウォレットとホットウォレットの分離により、リスクへの対応力をさらに高めることができます。このような対策を行うことで、予測不可能なリスクによる損失規模を抑えることが可能です。

  2. すべてのノードは、潜在リスクを減らすために、独立してDappコードをレビューする必要があります。また、すべてのノードには資産管理に対する権利と義務があります。

  3. ノードを運営するチームは密に連絡を取り合う必要があります。問題が見つかれば、その時点でサービスを停止し、新しいコードをデプロイして迅速に問題を修正することができます。

- ガバナンス

  分散型マルチシグネチャ組織では、担保の種類や手数料などの案を投票によって決定しています。

---
MTGリファレンスコード: https://github.com/MixinNetwork/trusted-group . テクニカルサポートへのお問い合わせは、[Mixin Messenger](https://w3c.group/c/1609251387450619)でMixin ID：`762532`を検索してください。
