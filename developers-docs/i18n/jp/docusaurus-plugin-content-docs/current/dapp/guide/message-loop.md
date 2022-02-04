---
title: メッセージループの処理
---

## ウェブソケットの取り扱い

Mixinのメッセージサービスでは、websocketを使ってメッセージを配信しています。次のGolangコード・スニペットはwebsocket接続を処理する方法を示しています。

```go
func ConnectMixinBlaze(uid, sid, key string) (*websocket.Conn, error) {
  token, err := SignAuthenticationToken(uid, sid, key, "GET", "/", "")
  if err != nil {
    return nil, err
  }
  header := make(http.Header)
  header.Add("Authorization", "Bearer "+token)
  u := url.URL{Scheme: "wss", Host: "blaze.mixin.one", Path: "/"}
  dialer := &websocket.Dialer{
    Subprotocols: []string{"Mixin-Blaze-1"},
  }
  conn, _, err := dialer.Dial(u.String(), header)
  if err != nil {
    return nil, err
  }
  return conn, nil
}
```

もちろん、SDKを利用することで作業を簡略化することも可能です。

## メッセージの受信

WebSocket 接続後、保留中のメッセージを受信するためには、まず LIST_PENDING_MESSAGES メッセージを送信する必要があります。

```json
{
  "id": "UUID",
  "action": "LIST_PENDING_MESSAGES"
}
```

データフォーマット:

```json
// On success:
{
  "id": "UUID",
  "action": "CREATE_MESSAGE",
  "data": {
    "conversation_id": "UUID",
    "user_id": "UUID",                      // Sender.
    "message_id": "UUID",
    "category": "PLAIN_TEXT",               // Message type.
    "status": "SENT",
    "data": "Base64 decoded data",
    "created_at": "2020-11-02T12:47:32.472333Z",
    "updated_at": "2020-11-02T12:47:32.472333Z",
    "source": "LIST_PENDING_MESSAGES",      // "LIST_PENDING_MESSAGES" or empty.
    "quote_message_id": "UUID",             // Optional, quoted message.
    "representative_id": "UUID"             // Optional, the user being replaced.
  },
}

// On failure.
{
  "id": "0623f846-aa86-4664-bb65-0e5559890a5c",
  "action": "CREATE_MESSAGE",
  "error": {
    "code": 20121,                                  // Error code.
    "desciption": "Authorization code expired."
  },
}
```

その他のメッセージタイプについては、ドキュメント[メッセージタイプ](../../api/messages/category)を参照してください。

## メッセージステータス

WebSocket接続が成功し、ボットがメッセージを受信して処理した場合、ボットはメッセージの状態をMessengerサーバーに送信し、サーバーがメッセージを受信したことを検知する必要があります。そうしないと、メッセージが繰り返しプッシュされることになります。メッセージステータスは、パフォーマンスを向上させるために、バッチで送信することができます。

```json
[
  {
    "message_id": "928c5c40-769c-3e97-8387-fb1ae0645311",
    "status":"READ"
  },
  ...
]
```

詳しくは、ドキュメント「ステータスの一括送信」(../../api/messages/send)を参照してください。
