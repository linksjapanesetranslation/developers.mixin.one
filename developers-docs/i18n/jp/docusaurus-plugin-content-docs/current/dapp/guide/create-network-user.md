---
title: ネットワークユーザーの作成
sidebar_position: 2
---

ネットワークユーザーは、アプリケーションユーザーの秘密鍵によって作成されるユーザーです。ネットワークユーザのキーストアは、アプリケーションによって作成され、Mixin Networkに登録されます。

次の例では、Go SDKを使用してウォレットユーザーのコードを生成します。

```go
import (
  "context"
  "crypto/rand"
  "crypto/rsa"
  "crypto/x509"
  "encoding/base64"
  "encoding/pem"

  "github.com/MixinNetwork/bot-api-go-client"
)

const (
  clientId   = ""
  sessionId  = ""
  privateKey = ""
)

func main() {
  ctx := context.Background()

  // Generate Ed25519 private key pair
  publicKey, privateKey, err := ed25519.GenerateKey(rand.Reader)
  if err != nil {
    println(err)
    return
  }
  sessionSecret := base64.RawURLEncoding.EncodeToString(publicKey[:])

  // register in Mixin network
  user, err := bot.CreateUser(ctx, sessionSecret, "fullname", clientId, sessionId, privateKey)
  if err != nil {
    println(err)
    return
  }
}
```

`clientId`, `sessionId`, `privateKey` are obtained from the keystore of the current wallet application.

その他の言語のSDKについては、[document](/docs/resources/sdk)を参照してください。
