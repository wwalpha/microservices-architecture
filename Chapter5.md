# モノリスの分割

モノリスは時間とともに増大します。新機能やコードを驚くべき速さで取り込みます。すぐに組織にとって恐ろしく巨大な存在になり、手を出したり変更したりするのをためらうようになります。しかし、適切なツールがあれば、この怪物を倒すことができます。

## モノリスを分割する理由

もっとも恩恵が得られるの接合部から分解する

- 変化の速度
  - 自律のシステムは迅速に変更できる
- チームの構成
  - 開発チームの地域が異なる
  - 例えば、東京とハワイ
- セキュリティ
  - データ格納などセキュリティ要件が異なる
  - 例えば、監査
- 技術
  - サービスの使用技術が異なる
  - 例えば、Java と Go

## 分割の方法

- 外部キーの削除
  - API 経由のデータ公開
    - 性能の心配はあり
      - 性能テストあれば安心
      - 遅くなったとしても大丈夫な機能はある
- 共有静的データ
  - テーブルに保存する ×
  - ソースコード 〇
  - 参照データ ◎
- 共有データ
  - 新しいサービスとして提供する
- 共有テーブル
  - クエリ特性でテーブル分割

## データベースリファクタリング

- 階段的な分割
  - 1. 単一スキーマ
  - 2. スキーマ分割
  - 3. サービス分離
  - スキーマ分割でトランザクションの整合性は失う

## トランザクションの境界

- 結果整合性 **おすすめ、現在は標準となった**
  - キューに入れて、リトライできる
  - 結果整合性 (Eventual Consistency) を受け入れる
- 操作全体の中止
  - システムを一貫性の状態に戻る必要がある
  - 補正 (Compensating) トランザクションを発行する
    - 例えば、新規登録後に削除する
    - 補正失敗したら、どうする
      - リトライ
      - 失敗キューに入れる
- 分散トランザクション **絶対採用していけない**
  - トランザクションマネージャで管理
  - 仕組み的には、全員に「投票」し、全て「はい」の場合コミットする
  - 投票(調整)期間中、ロック状態となり、リソースに対するロック競合を招く
    - スケーリングが困難になる

## レポートデータベース

データベースのレプリカを採用する

## データポンプ

- 定期的に中央レポートデータベースにデータを送る
- 1 つデータベースは 1 つスキーマの原則を守る
- データレイクの作成
- イベントデータポンプ
  - サービスのデータベースが状態変更があった同時に、中央データベースにレコード投入
  - DynamoDB Stream
- バックアップデータポンプ
  - BigQuery Resource

## 変更のコスト

CRC (Class-Responsibility-Collaboration) の手法