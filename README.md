# 🌟 Dgraphデモプロジェクト

このプロジェクトは、グラフデータベースDgraphの基本的な使い方を学ぶためのデモ環境を提供します。

## 📂 プロジェクト構造

```
.
├── docker-compose.yml    # Dockerコンテナの設定
├── schema.rdf           # Dgraphスキーマ定義
├── docs/
│   └── tutorial.md     # 詳細なチュートリアル
└── examples/
    ├── mutation.json   # データ挿入例
    └── queries.graphql # クエリ例
```

## 🚀 クイックスタート

1. 環境起動:
```bash
docker-compose up -d
```

2. サービスの確認:
```bash
docker-compose ps
```

3. スキーマの登録:
```bash
curl -X POST localhost:8080/alter --data-binary '@schema.rdf'
```

4. アクセス:
- Ratel UI (Web管理画面): http://localhost:8000
- Dgraph API: http://localhost:8080
- gRPC: localhost:9080

## 📚 チュートリアル

詳細な使い方は [チュートリアル](docs/tutorial.md) を参照してください。

### 主な機能

1. スキーマ定義
- Person型とCompany型の定義
- インデックスの設定
- リレーションの定義

2. データ操作
- ノードの作成
- リレーションの設定
- データの検索
- フィルタリング

### APIを使用したデータ操作

1. スキーマの設定
```bash
# スキーマの登録
curl -X POST localhost:8080/alter --data-binary '@schema.rdf'
```

2. データの挿入
```bash
# JSONデータの挿入
curl -X POST localhost:8080/mutate -H "Content-Type: application/json" --data-binary '@examples/mutation.json'
```

3. クエリの実行
```bash
# クエリの実行
curl -X POST localhost:8080/query -H "Content-Type: application/graphql" --data-binary '@examples/queries.graphql'
```

## 🛑 環境の停止

```bash
# コンテナの停止
docker-compose down

# データも完全に削除する場合
docker-compose down -v
```

## 📝 注意事項

- すべてのコマンドはプロジェクトのルートディレクトリで実行してください
- データの永続化はDocker volumeで行われます
- スキーマの変更は `/alter` エンドポイントを使用します
- データの操作は `/mutate` エンドポイントを使用します
- クエリの実行は `/query` エンドポイントを使用します
