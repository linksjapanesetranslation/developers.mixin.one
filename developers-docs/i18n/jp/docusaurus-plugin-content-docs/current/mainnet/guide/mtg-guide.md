---
title: MTG開発ガイド
---

[mtg](https://developers.mixin.one/document/mainnet/mtg/overview)  は、Mixinが提供するスマートコントラクトソリューション群であり、独立した複数のノードが共同でマルチシグネチャウォレットを維持することで、資産の分散管理を実現するものです。

mtg開発の核となるポイントをまとめると、以下のようになります:

:::info
独立性を確保する前提により、すべてのノードがutxoの転送と消費の選択を常に同じに保ちます。
:::

## utxoの同期


Mixinは、マルチシグネチャグループのすべてのuxtoの更新を取得するAPI[GET /multisigs/outputs](https://developers.mixin.one/document/wallet/api/multisigs/outputs) を提供しています。utxoのリストは、utxoの**updated_at**で昇順に並べられます。utxo の状態が変わるたびに **updated_at** が更新されるので、```GET /multisigs/outputs```はべき乗ではないことになります。同じ ```offset``` パラメータを使用していても、異なる時刻にこの API を呼び出すと、異なる utxo リストになります。

マルチシグネチャグループのノードが同じ順番でutxoリストを取得できるように、utxoは2つのグループ ```[pending, commited]``` に分けることができます。まず ```/multisigs/outputs``` から取得した utxo を ```pending``` グループに入れ、 ```/multisigs/outputs``` が返す utxo の数が **limit** (最新の状態に同期された) 以下になるまで保留し、 ```pending``` グループ内の utxo を ```(created_at, transaction_hash, output_index)``` 順に ```commited``` に移します。

実装は [pando/syncer](https://github.com/fox-one/pando/blob/main/worker/syncer/syncer.go) を参照してください。

:::tip
スナップショットとは異なり、utxoには`unspent -> signed -> spent`の３つ状態があります。
:::

## utxoの処理

通常のdapp開発におけるスナップショットの順次処理と同様に、mtgプログラムも ```commited``` グループ内のutxoを順次処理します。

### チャート

| snapshot (amount > 0) | utxo | description|
|---|----|-----|
| opponent_id | sender  |Paying user|
| trace_id | trace_id  |Uniquely identifies|
| memo | memo  |Transfer notes|
| asset_id | asset_id  |Currency|
| amount | amount  |Number of transfers|

Snapshotとutxoの処理は基本的に同じで、転送メモ処理のビジネスロジックに従って外部転送を生成します。例えば、4swapでは、ユーザーがpUSDを支払ってBTCを購入するとします。 購入が成功した場合、BTCをユーザーに送金する必要があります。購入が失敗した場合、ユーザーにUSDTを返金する必要があります。

## 転送処理

マルチサインアウト転送を行うには、[Multisig Request](https://developers.mixin.one/document/wallet/api/multisigs/request)を起動する必要があります。同じ ```raw``` パラメータは同じマルチシグリクエストを返し、すべてのノードが同じ転送を検証して署名できるようにします。署名の数が ```threshold``` に達すると、十分な署名の入った ```Raw Transaction``` を取得し、メインネットワークノードに送信して転送を完了させることができます。

実装は[pando/assigner](https://github.com/fox-one/pando/blob/main/worker/assigner/assigner.go)、[pando/cashier](https://github.com/fox-one/pando/blob/main/worker/cashier/cashier.go)、[pando/spentsync](https://github.com/fox-one/pando/blob/main/worker/spentsync/spentsync.go)を参照することができます。

### utxoの選び方

In the ```commited utxo``` group, select the least utxo with the same asset_id and the sum of amount greater than or equal to transfer.amount in order.Because the number of utxo consumed at one time is limited, if utxo is too fragmented, it needs to be merged first, that is, transfer the first n utxo combined consumption to oneself, n suggestion 32.
```commited utxo```グループで、同じ asset_id と金額の合計が transfer.mount 以上である最小の utxo を順に選択します。一度に消費するutxoの数が限られているので、utxoがあまりにも断片的であれば、それは最初にマージする必要があります、つまり、自分自身に最初のn個のutxoを結合し転送します。nには32がサジェストされます。

使用済みのutxoは、usedとマークする必要があります。このステータスは、utxoのspentステータスとは別であることに注意してください。usedはローカルステータス、spentはリモートステータスです。例えば、あるutxoが他のノードによって署名され提出された後、ステータスはspentになりますが、処理が遅く、それ自身では使用されていないため、まだunusedの状態です。この場合、utxoを選ぶ際には、ローカルステータスがunspentのutxoから選択する必要があります。

### Rawトランザクションの送信先ノード

The proxy api provided by mixin can randomly submit Raw Transaction to any mainnet node, but in order to ensure that the submission is truly successful, after submission, you need to use the transaction hash to read the same node several times to ensure that the transaction is read and Snapshot is generated, otherwise change to another node and continue to submit.Therefore, it is not recommended to directly use this set of APIs provided by mixin, but to configure some nodes by yourself, submit and confirm point-to-point.
mixinが提供するproxy apiは、任意のメインネットノードに対してランダムにRaw Transactionを投入することができますが、本当に投入が成功したかどうかを確認するためには、投入後にトランザクションハッシュを使って同じノードを何度か読み込んでトランザクションが読み込まれてSnapshotが生成されているかどうか確かめる必要があります、さもなければ別のノードに変更して投入を継続する必要があります。したがって、mixinが提供するこのAPI群を直接使うことはお勧めせず、自分でいくつかのノードを設定して投入と確認をpoint-to-pointで行うことをお勧めします

### スナップショットカードの送信

マルチシグネチャ転送では、ユーザは転送カードを受け取らないため、ユーザ体験を最適化するために、ノードは転送完了後に ```AppCard``` を使用してユーザにカードを送信し、到着メッセージをシミュレートすることにします。また、ユーザーは十分なノード署名をもらい、メインネットを正常に送信した後に転送を受け取るため、ノードは消費したutxoのステータスがspentになった後にユーザーにメッセージを送る必要があります。カードアクションは ```mixin://snapshots?trace={trace_id}``` で、trace_idは消費したutxo transaction_hash と output_index で生成されています。
 [pando/notifier](https://github.com/fox-one/pando/blob/main/notifier/utils.go#L15)を参照してください。

## その他

### ビジネスパラメーターの変更方法

ビジネス・パラメーターは転送によって変更する必要があり、すべてのノードが同じ状態で変更できるようにすることで一貫性を持たせることができます。中央集権的なガバナンスの問題を回避するために、ノード投票機構を導入することができる。つまり、誰でも送金して提案を開始することができ、ノード管理者が投票し、十分な投票が集まれば修正が実施されることになります。

:::info
例えば、4swap取引ペアの取扱手数料率は、整合性を保つために全ノードが一律に変更する必要があります。
:::

### スケジュールされたタスクの実行方法

タイミングタスクのトリガーロジックをmtgの外部に転送する。タイミングタスクがトリガされると、転送によってノードに通知されるため、すべてのノードが同じ状態でこのタスクを実行できる。

:::info
例えば、4swapの流動性インジェクションタイムアウト時に、10分以内に2回のインジェクションが完了しなかった場合、ユーザーに返金されます。
:::
