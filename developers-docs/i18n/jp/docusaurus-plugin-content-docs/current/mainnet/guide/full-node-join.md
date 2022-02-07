---
title: フルノードのデプロイ
sidebar_position: 15
---

# フルノードのデプロイ

Mixin Networkは、PoSコンセンサスに基づく分散型ネットワークです。フルノードはXINを担保として使用します。毎年担保にするXINは流通量の2％です。1年目は10,000XIN、2年目は11,000XINというようになります。十分な担保トークンがあるば、申請することで、承認なく誰でも匿名でMixinのメインネットに参加することができます。申請の手順は以下の通りです。

### メインネットの秘密鍵とアドレスの生成

```
$ git clone https://github.com/MixinNetwork/mixin.git
$ cd mixin
$ go build
```

コンパイルに成功したら、./mixin createaddress -public で2つの秘密鍵とアドレスを生成します。1つはsignerというメインネット取引のグループ化と署名に使用され、もう1つはpayeeというマイニング報酬の受け取りに使用されます。アドレス、ビューキー、スペンドキーはきちんとバックアップして保管しておいてください

```
$ ./mixin createaddress -public

address: XINRXkrW1CpocUznN5feEBGYtMLku3vRKTZDpT6wFoobYnPhtbdsKjiTp6DPCUHWm8VPrcyaRabGjbxjFR5rWFa9XU77tX6d
view key: 708a8db3011a806e788a518646414ca5750f6a9ca51c0b309e5e45cdf00b2100
spend key: c0619ce9c6fad9d2b6f7f4c0e676aed8f4d07b422e5fa55fee6154961954be0c
```


メインネットへの参加を開始するアドレスは、signer、payee、その他のアドレスのいずれでもかまいません。要件はありませんが、ノードの有効期限が切れたり、メインネットから退出したりすると、担保は受取人のアドレスに返却されます。

`$$\color{red}{ Note: The signer must be replaced with a new key each year. There is no such requirement for the payee key.}$$`


### 担保となるトークンの受け渡し

現在、Mixin Messengerで直接メインネットアドレスにトークンを送金することはできず、ボットを介してトークンの送金を行う必要があります。

- [Mixin Messenger](https://mixin-www.zeromesh.net/messenger)をダウンロードし、[Dashboard](//developer.mixin.one/dashboard)を開き、QRコードをスキャンしてログインしてください。

- 左側の "New Application"をクリックし、プロンプトに従ってボットを作成します。"Key "タブに切り替え、"App Session "をクリックしてキーストアをダウンロードし、"Wallet "タブに切り替え、キーストアに情報を入力してウォレットを有効にしてください。

- Mixinメッセンジャーのホームページを開き、その上で先ほど作成したボットのMixin IDを検索し、担保トークンをロボットに転送します。その後、ウォレットのインターフェイスを更新して送金を確認してください。


### メインネットアドレスへの担保トークンの預け入れ

トークンをメインネットアドレスに入金する方法は3つあります。まずはCNBでテストすることをお勧めします。

- ダッシュボードで出金する

  ウォレット資産一覧のXIN資産の出金ボタンをクリックし、メインネットアドレス、金額等を入力し、出金ボタンをクリックします。

- 送金API経由

  ボットの認証トークンを使って `POST /transactions` を呼び出し、メインネットアドレスに送金します。

- コマンドライン経由

  ```
  $ git clone https://github.com/MixinNetwork/bot-api-go-client.git
  $ cd bot-api-go-client/cli
  $ go build
  $ ./cli transaction  \
    -asset c94ac88f-4671-3976-b60a-09064f1811e8 \
    -k XINRXkrW1CpocUznN5feEBGYtMLku3vRKTZDpT6wFoobYnPhtbdsKjiTp6DPCUHWm8VPrcyaRabGjbxjFR5rWFa9XU77tX6d \
    -amount 10000 \
    -uid xxx \
    -sid xxx \
    -pin xxx \
    -pin_token xxx\
    -private "-----BEGIN RSA PRIVATE KEY-----
  xxx
  -----END RSA PRIVATE KEY-----"
  ```

  `-k`mainnetアドレス、その他のパラメータはKeystoreから取得し、mainnet transaction_hashは成功時に返されるので、次のコマンドで確認します。:

  ```
  ./mixin --node 35.234.74.25:8239 gettransaction --hash 95d09ebff27f988b276b32d4408ba7fc7f61e4d85af7c2267093f18486cb9a58
  ```

### メインネットへの参加申請

プレッジトランザクションは、メインネットへの参加を申請するための特別なトランザクションとなります。

- 署名入り誓約トランザクションの構築

   `./mixin signrawtransaction`を使い、署名入りの制約トランザクションを構築します。

  | パラメーター | 説明 |
  | :----- | :---- |
  | key | トランザクションの署名、view + spendキー文字列スプライシングを使用。ローカル署名でネットワークに送信されません。 |
  | extra | 拡張情報、署名者公開鍵+受取人公開鍵の文字列スプライシング。  `./mixin decodeaddress -a XINxxx` で署名者と受取人の両方の公開鍵を取り出せます。 |
  | asset | ここにはXINのアセットIDである`a99c2e0e2b1da4d648755ef19bd95139acbbe6564cfb06dec7cd34931ca72cdc`をセットしてください。 |
  | inputs | `[{"hash": "xxx", "index":0}]`, `hash` と `index` は、 `transaction_hash` と  `index` で、ボットからメインネットに渡されます。 |
  | outputs | `[{"type": 163, "amount":"11000"}]`, `type` 163 は、このトランザクションが誓約トランザクションであることを示しています。 |
  | amount | 担保に入れるXINの量です。 |

  ```
  ./mixin -n 35.234.74.25:8239 signrawtransaction -key VIEWSPEND  -raw '{"version":1,"asset":"a99c2e0e2b1da4d648755ef19bd95139acbbe6564cfb06dec7cd34931ca72cdc","inputs":[{"hash":"28926e30a9959fe7e7ce9c347d8fbacae2249425015d455208de70e4c678bd88","index":0}],"outputs":[{"type":163,"amount":"10000"}],"extra":"979939097dd50d0d6be42c47b3235c07108c28ce7cca150eed3b745283a9ef9639d749ab642df3e0e5573052b7031cd1e96c328e6f73d22851c475d96b7c5257"}'
  ```

  **`extra` が正しいことを確認してください！**

- 誓約トランザクションの送信

  署名が成功するとHexが生成されます。メインネットへの参加を申請する特別なトランザクションを送信するために `./mixin sendrawtransaction` を実行してください。

  ```
  ./mixin -n 35.234.74.25:8239 sendrawtransaction -raw 86a756657273696f6e01a54173736574c420a99c2e0e2b1da4d648755ef19bd95139acbbe6564cfb06dec7cd34931ca72cdca6496e707574739185a448617368c42020001842d6eff5129c11f7c053bf1209f0267bf223f1681c9cb9d19fc773a692a5496e6465780ba747656e65736973c0a74465706f736974c0a44d696e74c0a74f7574707574739185a45479706500a6416d6f756e74c7050002addc961da44b65797391c4202902d1f57276eb32882066de06c0d70fc17a3cd8dc801c1c50e6d4918e39344fa6536372697074c403fffe01a44d61736bc4209515d480e6f28b79228c32db33718e1ca03750bbe7748cd3968a5d450cd743d1a54578747261c400aa5369676e6174757265739190
  ```

### ハードウェア

- **メモリー** 64G
- **CPU** 16 コア
- **ハードディスク** 1T SSD (SSDは必須です)
- **帯域幅** 現在特別な要件はありません

### ノードの実行

- 最新のmixinをダウンロード https://github.com/MixinNetwork/mixin/releases
- config.jsonの`listener`に自分のパブリックIPまたはドメインを設定します。
- config.json で `signer`、`private`、`spend`、`key` を適切に設定します。
- Mixinをsystemd daemonとして実行します。 https://github.com/MixinNetwork/mixin/blob/master/config/systemd.service

---
**テクニカルサポートへのお問い合わせは、Mixin Messengerで31911、26596を検索してください。**
