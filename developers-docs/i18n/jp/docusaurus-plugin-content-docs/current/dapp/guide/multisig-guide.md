---
title: マルチシグネチャガイド
sidebar_position: 1
---

完全な2/3マルチシグネチャーのデモがあります: https://github.com/MixinNetwork/trusted-group, オンラインテストボットのID: 7000101488。

マルチシグネチャが使われるシナリオ：A、B、Cの3人のチームが所有するアセットがあります。この場合、2/3のマルチシグネチャが適切です。

A、B、Cは3つの異なるボットまたはXixinネットワークのユーザーである必要があります（プライベートキーとPINが必要）。

## 1. A、B、C の共通アカウントに資産を移動する POST /transactions

マルチシグネチャーの口座に資産を移管する場合、詳しくは以下の資料をご参照ください。
https://developers.mixin.one/docs/api/multisigs/request

## 2. 独自のutxo /multisigs/outputsを取得します。

注意すべきは、トランザクションにおいて、thresholdとmembersは同じメンバーにマッピングする必要があり、署名の数（2/3 の 2）です。
APIドキュメント：https://developers.mixin.one/docs/api/multisigs/outputs

継続的にapiサーバーにリクエストし、最新のutxoを取得し、保存する必要があります。

## 3. 資産の出金は、前述の通り、完了までに少なくとも2つのサインが必要ですが、その手続きは同じです。

マルチシグネチャに関連する操作は，署名，取消し，ロック解除の3種類です。それぞれの操作の前に、リクエストを作成する必要があります。APIドキュメント： https://developers.mixin.one/docs/api/multisigs/request

目的：Aは共通口座から100CNBを出金する必要がある。

1. Aはリクエスト(POST /multisigs/requests)を送信する。
2. Aは署名する。POST /multisigs/requests/:id/sign, in :idはリクエストIDである。
3. BとCも同じようなことをする必要があります。Cが受け取る出金では，状態が署名されているので，CはAによって署名されたtxを受け取って，ステップ1と2を繰り返します。
4. 署名済みtxを解析し、署名者の数が2人であるかどうかを確認し、2人以上であれば、トランザクションをメインネットに送信します。
5. Aは100CNBを獲得し、取引は完了します。

支払い後の払い戻しに署名するためのコード https://github.com/MixinNetwork/trusted-group/blob/master/sample/models/payment.go#L164

取引構築の過程で、AのGhostKeys（ワンタイムキー）が使用されます。このキーは、https://developers.mixin.one/docs/api/network/outputs で取得できます。

## 4. トランザクションの取り消し

1. リクエストを構築するプロセスは、トランザクションを構築する時のものと同じです。
2. キャンセルリクエストを送信します。POST /multisigs/requests/:id/cancel，:id はリクエストIDです。注意：未使用状態のTXのみキャンセル可能です。

## 5. トランザクションのアンロック

Txが署名されているが送信されていない場合、取り消しはTxのロックを解除することを意味し、そのプロセスは取り消しとよく似ている。

1. リクエストを構築する処理はトランザクションを構築するのと同じで ある。
2. リクエストを送信する。POST /multisigs/requests/:id/unlock，:id はリクエストID。

## 結論

多くの場合、ステップ1、2、3が重要になります。まず入金し、次に関連する出力に署名し、txをメインネットに送信して使用します。
