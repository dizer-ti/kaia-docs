# ストレージの最適化

カイア・ブロックチェーンが成長するにつれ、チェーン・データを保存するのに必要なストレージも増加する。 カイアは、この増大するストレージ要件を管理するために、主に2つのテクニックを実装している：

## ステート・バッチ・プルーニング（ステート・マイグレーション）

状態移行は、実行中のノードを中断することなく、既存のノードに適用できるバッチ刈り込み機能である。

### モチベーション

ブロックステート（StateDB）は、オンチェーンアカウントとコントラクトをトライデータ構造に格納する。 Trieデータ構造は、古い状態と最近の状態の両方を保存するように設計されているので、Merkleハッシュを使って検証することができる。 トランザクションが状態変更を実行すると、状態トライは無限に成長する。 本稿執筆時点（2024年8月）では、カイア・メインネットのアーカイブ・ノード・サイズは20TBを超え、フル・ノードでも10TBを超えている。

### コンセプト

ステート・マイグレーションは、新しいブロックの処理に必要のない古いブロック・ステートを削除する。 それは、"古い "から "新しい "へのステートトライをコピーする。 すべてのトライノードがコピーされるわけではない。 選択ブロックの状態根から到達可能なものがコピーされる。 コピー後、古いディレクトリは削除されるので、選択したブロックの状態だけが残る。

技術的な詳細については、以下のブログ記事をお読みください：
[ステート・マイグレーション：ノード・ストレージの節約](https://medium.com/klaytn/klaytn-v1-5-0-state-migration-saving-node-storage-1358d87e4a7a)、
[カイア・ステート・マイグレーション：ブロックチェーン・データを削減する効率的な方法](https://medium.com/klaytn/klaytn-state-migration-an-efficient-way-to-reduce-blockchain-data-6615a3b36523)

Batch Pruningの実行方法については、[State Migration Guide](../../misc/operation/node-pruning.md#how-to-perform-batch-pruning)を参照してください。

## ステート・ライブ・プルーニング

ステート・ライブ・プルーニングは、増大するステート・データベースのサイズ問題に対する新しいソリューションである。 バッチ・プルーニング（ステート・マイグレーション）とは異なり、ライブ・プルーニングは、ノードのプロセスがブロックされると、古いステートを少しずつ自動的に削除する。

### モチベーション

ブロックステート（StateDB）は、オンチェーンアカウントとコントラクトをトライデータ構造に格納する。 Trieデータ構造は、古い状態と最近の状態の両方を保存するように設計されているため、Merkleハッシュを使って検証することができる。 トランザクションが状態変更を実行すると、状態トライは無限に成長する。 本稿執筆時点（2024年8月）では、カイア・メインネットのアーカイブ・ノード・サイズは20TBを超え、フル・ノードでも10TBを超えている。

これまでのステート移行では、最近の状態を選択的にコピーして古い状態を削除し、残りを削除することでこの問題を軽減していた。 これにより、ノード全体のサイズを5TB未満に抑えることができる。

とはいえ、ステート移行には欠点もある。 数日かかる可能性のあるステートトライ全体をトラバースするという高いオーバーヘッドに悩まされる。 また、状態移行は手動でトリガーしなければならない。 これらの制限を克服するために、ライブ・プルーニング技術が導入された。

### コンセプト

トライのプルーニングは、トライのノードが古いかどうかが不明なので難しい。 元のステートトライ構造では、トライノードは、それぞれが異なるブロックを構成する複数のトライの一部になることができる。 トライノード（口座残高など）が別の値に更新されても、他の親ノードがまだ必要としている可能性があるため、トライノードを削除することはできない。 この問題はハッシュ重複問題と呼ばれている。

ライブ・プルーニングは、同じ内容のトライノードを意図的に重複させる。 ライブ・プルーニングでは、トライノードはハッシュで参照されるのではなく、ExtHashで参照される。 ExtHashは、コンテンツの32バイトのハッシュと7バイトのシリアル・インデックスです。 直列インデックスは単調増加なので、すべてのトライノードは一意である。

```
Hash: 32-byte Keccak256
ExtHash: 32-byte Keccak256 + 7-byte シリアルインデックス
```

こうすることで、トライノードのコンテンツが変更されるたびに、そのトライノードは廃止されたと仮定しても問題ない。 Merkleハッシュは、シリアルインデックスを無視するだけで、同じように計算することができ、コンセンサスの点で非ライブ・プルーニング・ノードと互換性がある。

技術的な詳細については、このブログ記事をお読みください：[StateDB Live Pruningによるブロックチェーンデータ容量の効率的管理](https://medium.com/klaytn/strong-efficient-management-of-blockchain-data-capacity-with-statedb-live-pruning-strong-6aaa09b05f91).

ライブ・プルーニングを有効にする方法については、[ライブ・プルーニング・ガイド](../../misc/operation/node-pruning.md#how-to-perform-live-pruning)を参照してください。