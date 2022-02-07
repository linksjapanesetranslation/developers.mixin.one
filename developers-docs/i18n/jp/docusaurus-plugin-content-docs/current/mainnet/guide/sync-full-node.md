---
title: フルノードからのデータ同期
sidebar_position: 16
---

# フルノードからのデータ同期

### データノード、マイニングノードになるには

まず、MixinNetworkのフルノードのデータを同期させる必要があります。

1. https://github.com/MixinNetwork/mixin/releases から最新のリリースを取得します。
2. すべてのデータを格納するディレクトリを作成する`mkdir ~/.mixin`。
3. ダウンロードしたリリースを展開、`cp genesis.json nodes.json config.example.toml ~/.mixin`。
4.  `./mixin creataddress -public` を実行しMixinのメインネットアドレスを作成します。
5. config.example.toml を config.tomlに名前を変更し、以下の修正を行います:
   - signer-key に先のステップで作成した `spend key` を設定します。
   - listener には現在のマシンのIP:7239を設定します。
6. ファイアウォールがUDP port 7239 と TCP port 8239へのアクセスを許可していることを確認します。
8. `./mixin kernel -d ~/.mixin`　を実行します。

### FAQ

- 'badger LOG Compact FAILED with error: too many open files'を修正するにはどうすればいいですか?

  開くことのできるファイル数を65535にふやします。`ulimit -n`を確かめてください。

- 'failed to sufficiently increase receive buffer size (was: 208 kiB, wanted: 2048 kiB, got: 416 kiB). See https://github.com/lucas-clemente/quic-go/wiki/UDP-Receive-Buffer-Size for details.'を修正するにはどうすればいいですか?

  `sudo sysctl -w net.core.rmem_max=8388608`

  `sudo sysctl -w net.core.wmem_max=8388608`
  
- ポートが開いているかどうかを確認する方法は？

  `nc -zuv mixin-node 7239`

  `nc -zv mixin-node 8239`
