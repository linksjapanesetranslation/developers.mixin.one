---
title: Kotlin
sidebar_position: 12
---

# Kotlin SDK

### 使い方

[Kotlin SDK](https://github.com/MixinNetwork/bot-api-kotlin-client)をインポート

依存関係を追加する
 [Mixin Dashboard](https://developers.mixin.one/dashboard)から取得します。
```
dependencies {
    implementation 'com.github.MixinNetwork:bot-api-kotlin-client:v0.1.0'
}
```

### コンフィギュレーション

```
object Config {
    const val pin = ""
    const val userId = ""
    const val sessionId = ""
    const val privateKey = ""
    const val pinTokenPem = ""
}
```

### クイックスタート

```kotlin
    val key = getEdDSAPrivateKeyFromString(Config.privateKey)
    val pinToken = decryASEKey(Config.pinTokenPem, key) ?: return@runBlocking
    val client = HttpClient.Builder().configEdDSA(Config.userId, Config.sessionId, key).build()

    val sessionKey = generateEd25519KeyPair()
    val publicKey = sessionKey.public as EdDSAPublicKey
    val sessionSecret = publicKey.abyte.base64Encode()

    // Create a user, and register him on the Mixin network.
    val user = client.userService.createUsers(
        AccountRequest(
            Random().nextInt(10).toString() + "User",
            sessionSecret
        )
    ).data
    user ?: return@runBlocking
    // The user signs the authorization token.
    client.setUserToken(
        SessionToken.EdDSA(
            user.userId, user.sessionId,
            (sessionKey.private as EdDSAPrivateKey).seed.base64Encode()
        )
    )

    // decrypt pin token
    val userAesKey: String
    val userPrivateKey = sessionKey.private as EdDSAPrivateKey
    userAesKey = calculateAgreement(user.pinToken.base64Decode(), userPrivateKey).base64Encode()

    // Create PIN for the user.
    val pinResponse = client.userService.createPin(
        PinRequest(requireNotNull(encryptPin(userAesKey, DEFAULT_PIN)))
    )
    if (pinResponse.isSuccess) {
        println("Create pin success ${pinResponse.data?.userId}")
    } else {
        println("Create pin fail")
    }

    // Get asset list
    val assetResponse  =  client.assetService.assets()
    if (assetResponse.isSuccess) {
        println("Get assets ${pinResponse.data}")
    } else {
        println("Get assets fail")
    }
```
その他のサンプルは、SDK [サンプル](https://github.com/MixinNetwork/bot-api-kotlin-client/blob/main/samples/src/main/java/jvmMain/kotlin/Sample.kt)をご覧ください。

---
このSDKはMixinチームによって開発されています。テクニカルサポートへのお問い合わせは、Mixinメッセンジャーで26832, 31911を検索してください。
