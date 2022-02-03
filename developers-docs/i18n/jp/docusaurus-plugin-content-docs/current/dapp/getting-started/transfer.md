---
title: トランスファー
---


## トランスファーを送信

Mixin Networkでは、私たちが送ることのできるトランザクションは2種類あります。通常のMixinアカウント同士（通常のユーザーやアプリケーションなど）の通常トランザクションと、生のKernelアドレス同士の「生」トランザクションです。もちろん、どちらも無料ですぐに利用できます。

通常のトランザクションは [`/トランスファー`](/docs/api/transfer/transfer) API によって作成されます。以下はGolangでの簡単な例です。

```go
type TransferInput struct {
	AssetId     string
	RecipientId string
	Amount      number.Decimal
	TraceId     string
	Memo        string
	OpponentKey string
}

func CreateTransfer(ctx context.Context, in *TransferInput, uid, sid, sessionKey, pin, pinToken string) error {
	if in.Amount.Exhausted() {
		return fmt.Errorf("Amount exhausted")
	}

	encryptedPIN, err := EncryptPIN(ctx, pin, pinToken, sid, sessionKey, uint64(time.Now().UnixNano()))
	if err != nil {
		return err
	}
	data, err := json.Marshal(map[string]interface{}{
		"asset_id":    in.AssetId,
		"opponent_id": in.RecipientId,
		"amount":      in.Amount.Persist(),
		"trace_id":    in.TraceId,
		"memo":        in.Memo,
		"pin":         encryptedPIN,
	})
	if err != nil {
		return err
	}

	path := "/transfers"
	token, err := SignAuthenticationToken(uid, sid, sessionKey, "POST", path, string(data))
	if err != nil {
		return err
	}
	body, err := Request(ctx, "POST", path, data, token)
	if err != nil {
		return err
	}

	var resp struct {
		Error Error `json:"error"`
	}
	err = json.Unmarshal(body, &resp)
	if err != nil {
		return err
	}
	if resp.Error.Code > 0 {
		return resp.Error
	}
	return nil
}
```

:::注意
フィールド `trace_id` は UUID であるべきですが、ほとんどの場合、ランダムな UUID を使用すると問題が発生する可能性があります。詳しくはこちらの記事[Mixin開発者が犯す最も一般的な間違いトップ10](https://gitpress.io/@lyric/top-10-most-common-mistakes-that-mixin-developers-make) をご覧ください。
:::


## 受信トランザクションの処理

Mixin APIは、Kernelがトランザクションを確認するときに正しい量の暗号資産を返却ことが常に期待されるため、実際には暗号資産を「受け取る」必要はない。

しかし、受信したトランザクションを処理する必要がある場合がある。たとえば、トランザクションが受信されたときにユーザーに通知を送信したい場合があります。

これを行うには、2つの方法があります。

1.  [SYSTEM_ACCOUNT_SNAPSHOT](/docs/api/messages/category#transfers) メッセージをメッセージループで処理します。
2. バックグラウンドジョブで[関連するすべてのスナップショットを同期](../guide/sync-snapshots)を実行する。

今回は、1の方法を使用します。

<!-- @TODO -->
...

最初の条件は、メッセージのカテゴリです。2つ目は、message.UserIDとbotのclientIDの比較です（Client IDはbotのユーザIDで、キーストアファイルで確認できます）。

```go
// @TODO an example
// ...
if msg.Category == mixin.MessageCategorySystemAccountSnapshot {
  // if the message is a transfer message
  // and it is sent by other users, then handle it
  if msg.UserID != client.ClientID {
    return handleTransfer(ctx, msg)
  }
  // or just drop it
  return nil
} else if ...
```

2つ目の条件の理由は、Mixin Messengerでは、このボットに関連するすべての送受信は、それが他人からのものであっても、ボット自身が送信したものであっても、メッセージングループで受信されるからです。そこで、メッセージのuserIDを比較し、botが送信した転送を無視するようにしています。

<!-- @TODO -->
...
