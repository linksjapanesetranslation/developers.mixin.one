---
title: OAuth認証
---

アプリケーションでMessengerユーザーのプロファイル、アセット、スナップショットなどのデータを読み込む必要がある場合があるため、[OAuth 2.0](https://tools.ietf.org/html/rfc6749) プロトコルを使用してユーザー認証を行う必要があります。

### リクエストから認証まで
ボットがユーザーから認証されていないことを検知した場合、以下のURLにジャンプしてユーザーに認証を要求する必要があります：

```
https://mixin.one/oauth/authorize?client_id=CLIENT_ID&scope=SCOPE&response_type=code&return_to=
```

その中で、パラメータは:

**必須パラメータ**

- **client_id** - アプリケーションのclient_idは、前回の記事で紹介したキーストアから取得することができます。
- **scope** - 要求される権限に関して、詳しくは [このドキュメント](/docs/api/oauth/scope/)を参照してください。少なくとも 「PROFILE:READ」 パーミッションが含まれている必要があります。
- **response_type** - 認証コードを返すには 「code」 を使用します。

**オプションパラメータ**

- **state** - アプリケーションで生成されたランダムな文字列であり、後で検証する必要があります。
- **code_challenge** - アプリが生成したcode challengeはあなたのコードベリファイアのSHA256ハッシュです。詳細については、[こちら](https://www.oauth.com/oauth2-servers/pkce/authorization-request/)をご覧ください。
- **code_challenge_method** - コードチャレンジ方式は、「SHA256」に設定してください。

:::ヒント
ユーザーは認証時に特定のパーミッションのチェックを外すことができます。
開発者は必要な権限のみを申請し、権限がない場合は適切なガイダンスGUIを作成することを推奨します。
:::

## アクセストークンの認証コードの交換

認証に成功すると、ページは自動的にアプリケーションの 「OAuth URL」にジャンプし、認証コードとreturn_toパラメータが付加されます。

```
YOUR_APP_OAUTH_URL?code=AUTHORIZATION_CODE&return_to=YOUR_APP_RETURN_URL
```

開発者は 「AUTHORIZATION_CODE」 を読み込んで、アクセストークンを交換する必要があります。

```
POST https://api.mixin.one/oauth/token
```

```json title="Payload"
{
    "client_id":    "application's client_id from keystore",
    "code":         "AUTHORIZATION_CODE in the callback URL",
    "client_secret":"the `app secret` that generated in the previous article"
}
```

```json title="Response"
{
    "access_token": "user's authorization token",
    "scope":        "list of permissions that the user has given, e.g. 'PROFILE:READ ASSETS:READ'"
}
```

:::ヒント
開発者はアクセストークンをキャッシュし、その後APIを呼び出してアクセストークンを介してユーザーデータにアクセスし、ユーザーが認可したかどうかを判断することが推奨されます。
:::

