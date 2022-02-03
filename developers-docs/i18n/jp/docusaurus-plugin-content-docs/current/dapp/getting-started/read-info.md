---
title: 情報の読み込み
---

[前回記事](./oauth)では、OAuth2のフローと、それを使ってアクセストークンを取得する方法について紹介しました。今回は、ユーザーの情報を読み取る方法を紹介します。


今後、SDKについては、以下の記事で紹介していきます。Mixinチームとコミュニティは、開発者が利用できるようにさまざまなSDKを提供しています。SDK は [SDK](/docs/resources/sdk) のページで確認することができます。

## ユーザープロファイルの読み込み

:::ヒント
ユーザーの基本個人情報を取得するには、「PROFILE:READ」パーミッションが必要です。
:::

ユーザーのプロファイルを読み取るには、[`/me`](/docs/api/users/profile)エンドポイントを使用します。`/me`は認証されたエンドポイントなので、最初にアクセストークンを取得してHTTPヘッダーに記述する必要があります。

リクエストを送信しましょう

```bash
GET -H "Authorization: Bearer $ACCESS_TOKEN" https://api.mixin.one/me
```

```json title="Response"
{
  "data": {
    "type":            "user",
    "user_id":         "773e5e77-4107-45c2-b648-8fc722ed77f5",
    "name":            "Team Mixin",
    "identity_number": "7000"
  }
}
```

:::インフォ
「user_id」フィールドは、Mixin Network全体の各アカウントやユーザーに対して一意なIDを指定します。

さらに、そのユーザーがMixin Messengerのユーザーであれば、[`POST /messages`](/docs/api/messages/send) で「user_id」フィールドを指定してメッセージを送信することができます。
:::

## ユーザー資産の読み込み

:::ヒント
ユーザーの資産残高を取得するには、「ASSETS:READ」パーミッションが必要です。
:::

`GET /assets` を呼び出すと、残高が0以上のアセットを返信します。残高が0の新規ユーザーが所有するトークンでこのAPIを呼び出すと、空のリストが返信されます。

```bash
GET -H "Authorization: Bearer $ACCESS_TOKEN" https://api.mixin.one/assets
```

import RespAssets from '../../_partials/_resp.assets.md'

<RespAssets />

:::インフォ
 `asset_id` フィールドは、Mixin Network 全体における各アセットのユニークなIDです。

https://mixin.one/snapshots から `btc` などのアセットコードを検索して入手することができます。また、Mixin Messenger のウォレットにアセットを入金し、ボット `7000103061` に呼びかけ、アセット情報を検索してコピーすることも可能です。
:::

## アプリケーションの資産の読み取り

Mixinアプリケーションは、特殊なユーザーです。また、通常のユーザーと同じようにウォレットを持ちます。デベロッパーダッシュボード - appを選択 - `Wallet` tabをクリック "にアクセスすることで、アプリケーションのアセットを検査することができます。


また、アプリケーションのアクセストークンを使って [`/アセット`](/docs/api/assets/assets) を呼び出すことで、プログラム的にアセットを読み込むことができます。アクセストークンを生成するアルゴリズムは[こちら](. /guide/generate-jwt-token) ですが、処理を簡単にするために [SDK](/docs/resources/sdk) を使用することも可能です。

ここでは、公式Go SDKを使用して、トークンの生成とアセットの読み取りを行う例を示します。

```go
import "github.com/MixinNetwork/bot-api-go-client"
import "fmt"

func main() {
  ctx := context.Background()

  // generate token
  token, err := bot.SignAuthenticationToken(
    CLIENT_ID,
    SESSION_ID,
    CLIENT_SECRET,
    "GET",
    "/assets",
    ""
  )
  if err != nil {
    fmt.Println(err)
    return
  }

  // query assets
  assets, err := bot.AssetList(ctx, token)
  if err != nil {
    fmt.Println(err)
    return
  }
  for _, a := range assets {
    fmt.Println(a.AssetId)
  }
}
```

