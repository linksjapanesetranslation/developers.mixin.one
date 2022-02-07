---
title: アセットアイコンの提出
sidebar_position: 17
---

# アセットアイコンの提出

ERC-20、TRC-10、TRC-20、EOSの全てのトークンはMixinネットワークに入金後、直接使用することができますが、一部のコインにはアイコンがありません。開発者はMixinチームに連絡し、アイコンを提出することができます。その場合は、以下の手順でお願いします。

### トークンアイコンのフォーマット
  
- 520x520 ピクセル、 png。
- 円形である必要があります。背景が透明、白のものは不可です。
- 明確で確かなブランド認知度

### アセットIDの取得

- まずトークンをMixinウォレットに入金し、ウォレットから入金記録を探して、トランザクションIDをコピーします。
- Mixinブロックエクスプローラー https://mixin.one/snapshots を開き、トランザクション ID を入力し、検索をクリックします。
- 送金詳細の下部にあるトークン名のリンクをコピーし、リンクの末尾にある文字列を傍受します。例えば、`https://mixin.one/snapshots/c94ac88f-4671-3976-b60a-09064f1811e8`と出た場合、`c94ac88f-4671-3976-b60a-09064f1811e8`がアセットIDとなります。

### アイコンの提出

- GitHubのアカウントにログインし、プロジェクト`asset-profile`のコードをクローンします:

   `git clone git@github.com:MixinNetwork/asset-profile.git`

- `asset-profile` ディレクトリに移動し、トークンブランチを作成します。

- 提出すべきドキュメント
  
  `assets` ディレクトリに新しいディレクトリを作成します。ディレクトリ名は現在のアセットIDです。アイコンをディレクトリにコピーし、"icon.png"に名前を変更し、新しいindex.jsonを作成します。ファイル形式は以下の通りです:

  ```
  {
    "asset_id": "7b55bf4e-3f37-3cfa-bca5-3d30d72ab820",
    "chain_id": "43d61dcd-e413-450d-80b8-101d5e903357",
    "cmc_id": ""
  }
  ```

  `chain_id` はパブリックチェーンのIDで、Ethereum ERC 20 tokenであれば常に`43d61dcd-e413-450d-80b8-101d5e903357`に設定し、TRON TRC-10もしくはTRC-20 tokenならば常に`25dabac5-056a-48ff -b9f9-f67395dc407c`、EOS tokensならば`6cfe566e-4aad-470b-8c9a-2fd35b49c68d`、ほかのトークンであれば[document](../concepts/chain)を参照してください。

  `cmc_id` あなたのアセットがcoinmarketcapに含まれていない場合、ここは空欄にしてください。

- 申請

  用意したアイコンと設定ファイルに簡単な紹介を添えて、 `Create pull request` をクリックします。Mixin開発チームが事前審査を行った後、ノードに渡して審査してもらいます。問題がなければ、そのアイコンはデプロイされます。

---
詳細は、https://github.com/MixinNetwork/asset-profile プロジェクトのホームページ紹介をご覧ください、またそれ以前に投稿された [Record](https://github.com/MixinNetwork/asset-profile/commit/37c50161cbb0d9cdfd2387b1adb5837a601260a6 )も参照ください。

