---
title: PINのアップデート
sidebar_position: 4
---

6桁の暗証番号は、ユーザーが資産を譲渡する際に必要となるもので、秘密鍵のように機能し、紛失しても復元することはできません。

:::注意
開発者は、送金や暗証番号の変更に十分な注意を払う必要があります。

暗証番号の復旧やリセットはできませんのでご注意ください。間違った送金は元に戻せません。本番環境とテスト環境は分離し、十分にテストする必要があります。
:::

## PINの暗号化

PINを安全に送信するためには、PINを暗号化する必要があります。In Go language：

```go
func EncryptPIN(ctx context.Context, pin, pinToken, sessionId, privateKey string, iterator uint64) (string, error) {
	privateBytes, err := base64.RawURLEncoding.DecodeString(privateKey)
	if err != nil {
		return "", err
	}

	private := ed25519.PrivateKey(privateBytes)
	public, err := base64.RawURLEncoding.DecodeString(pinToken)
	if err != nil {
		return "", err
	}
	var dst, curve, pub [32]byte
	PrivateKeyToCurve25519(&curve, private)
	copy(pub[:], public[:])
	curve25519.ScalarMult(&dst, &curve, &pub)

	pinByte := []byte(pin)
	timeBytes := make([]byte, 8)
	binary.LittleEndian.PutUint64(timeBytes, uint64(time.Now().Unix()))
	pinByte = append(pinByte, timeBytes...)
	iteratorBytes := make([]byte, 8)
	binary.LittleEndian.PutUint64(iteratorBytes, iterator)
	pinByte = append(pinByte, iteratorBytes...)
	padding := aes.BlockSize - len(pinByte)%aes.BlockSize
	padtext := bytes.Repeat([]byte{byte(padding)}, padding)
	pinByte = append(pinByte, padtext...)
	block, err := aes.NewCipher(dst[:])
	if err != nil {
		return "", err
	}
	ciphertext := make([]byte, aes.BlockSize+len(pinByte))
	iv := ciphertext[:aes.BlockSize]
	_, err = io.ReadFull(rand.Reader, iv)
	if err != nil {
		return "", err
	}
	mode := cipher.NewCBCEncrypter(block, iv)
	mode.CryptBlocks(ciphertext[aes.BlockSize:], pinByte)
	return base64.RawURLEncoding.EncodeToString(ciphertext), nil
}
```

その他の言語のSDKについては、[ドキュメント](/docs/resources/sdk)をご参照ください。

## PINの設定

```go
const (
	userId     = ""
	pinToken   = ""
	sessionId  = ""
	privateKey = ""
)

func main() {
	ctx := context.Background()

	// Encrypt PIN
	encryptedPIN, err := bot.EncryptEd25519PIN(ctx, "123456", pinToken, sessionId, privateKey, uint64(time.Now().UnixNano()))
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(encryptedPIN)
	// Set initial code
	err = bot.UpdatePin(ctx, "", encryptedPIN, userId, sessionId, privateKey)
	if err != nil {
		fmt.Println(err)
		return
	}
}
```

:::インフォ

- パラメータ `iterator` はインクリメンタルで 0 より大きい値でなければなりません。 一般に、現在のシステムナノ時間を使用することが推奨されますが、自分で数字を選んで、呼び出しのたびにインクリメントすることもできます。
- 暗号化された暗証番号は1回しか使用できず、パスワード変更時に2回生成する必要があり、再利用はできません。
- PINのエラーにはタイムロックがかかっています。1日に5回失敗した場合、5回目以降はPINが正しくてもエラーが返されますので、再試行しないでください。それ以上の回数を繰り返すと、ロック時間が長くなります。試行したPINをメモしておき、翌日に再試行することをお勧めします。
- PINは一度紛失すると、二度と元に戻せません。開発者は、各ユーザーに定期的に入力させ、暗証番号を覚えさせることをお勧めします。初期設定時に3回以上入力させ、紛失すると取り返しがつかないことを念押しする。
- 資産保全の観点から、「123456」や「111222」のような単純すぎる暗証番号やよくある組み合わせを設定しないよう、ユーザーに注意喚起することをお勧めします。
:::
