---
title: Go
sidebar_position: 11
---

# Go SDK

### インストール

```
go get github.com/MixinNetwork/bot-api-go-client
```

### クイックスタート

```golang
package main

import (
	"context"
	"crypto/rand"
	"crypto/rsa"
	"crypto/x509"
	"encoding/base64"
	"encoding/pem"
	"fmt"
	"time"

	"github.com/MixinNetwork/bot-api-go-client"
)

const (
	appId         = ""
	appSessionId  = ""
	appPrivateKey = ``
)

func main() {
	ctx := context.Background()
	// Generate Ed25519 key pair.
	privateKey, err := rsa.GenerateKey(rand.Reader, 1024)
	if err != nil {
		fmt.Println(err)
		return
	}
	publicKeyBytes, err := x509.MarshalPKIXPublicKey(privateKey.Public())
	if err != nil {
		fmt.Println(err)
		return
	}
	sessionSecret := base64.StdEncoding.EncodeToString(publicKeyBytes)
	// Rigster the user on the Mixin network
	user, err := bot.CreateUser(ctx, sessionSecret, "fullname", appId, appSessionId, appPrivateKey)
	if err != nil {
		fmt.Println(err)
		return
	}
	userSessionKey := string(pem.EncodeToMemory(&pem.Block{
		Type:  "RSA PRIVATE KEY",
		Bytes: x509.MarshalPKCS1PrivateKey(privateKey),
	}))

	// encrypt PIN
	encryptedPIN, err := bot.EncryptPIN(ctx, "123456", user.PinToken, user.SessionId, userSessionKey, uint64(time.Now().UnixNano()))
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(encryptedPIN)
	// Set initial code.
	err = bot.UpdatePin(ctx, "", encryptedPIN, user.UserId, user.SessionId, userSessionKey)
	if err != nil {
		fmt.Println(err)
		return
	}
	//Sign authentication token.
	authenticationToken, err := bot.SignAuthenticationToken(user.UserId, user.SessionId, userSessionKey, "GET", "/assets", "")
	if err != nil {
		fmt.Println(err)
		return
	}
	// Read asset list
	assets, err := bot.AssetList(ctx, authenticationToken)
	if err != nil {
		fmt.Println(err)
		return
	}
	for _, a := range assets {
		fmt.Println(a.AssetId)
	}
}

```

その他の例は、[examples](https://github.com/MixinNetwork/bot-api-go-client/blob/master/examples/wallet.go)をご覧ください。

---
本SDKはMixinチームによって開発されています。テクニカルサポートに連絡するには、Mixin Messenger で 493230, 31911 を検索してください。Foxチームが提供しているGo SDK https://github.com/fox-one/mixin-sdk-go も非常に良く、より完全なコード例を提供しています。
