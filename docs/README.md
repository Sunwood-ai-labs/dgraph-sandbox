# 🚀 Dgraph チュートリアル

## 🛠 環境セットアップ

### 前提条件
- Docker
- Docker Compose
- curl（コマンドラインツール）

### 起動手順

1. Dockerコンテナを起動:
```bash
docker-compose up -d
```

2. 起動確認:
```bash
docker-compose ps
```

すべてのサービスが「Up」状態であることを確認します。

## 🌐 基本的な操作

Dgraphは以下の主要なエンドポイントを提供します：

- `/alter` - スキーマの変更
- `/mutate` - データの挿入・更新
- `/query` - データの検索

また、Ratel UI ( http://localhost:8000 ) を使用して視覚的に操作することもできます。

## 📐 スキーマの設定

### コマンドラインから設定

```bash
# スキーマファイルを使用して設定
curl -X POST localhost:8080/alter --data-binary '@schema.rdf'
```

スキーマの内容（`schema.rdf`）:
```graphql
name: string @index(term) .
age: int .
email: string @index(exact) .
industry: string @index(term) .
location: string .
friend: [uid] @reverse .
works_for: uid @reverse .
type Person { name age email friend works_for }
type Company { name industry location }
```

### Ratel UIから設定

1. http://localhost:8000 にアクセス
2. 「Schema」タブを選択
3. 上記のスキーマを貼り付け
4. 「Update Schema」ボタンをクリック

### スキーマの説明
- `@index(term)`: 文字列の全文検索用インデックス
- `@index(exact)`: 完全一致検索用インデックス
- `@reverse`: 逆方向の関係を自動で作成
- `[uid]`: 他のノードへの参照（複数の関連を持てる）

## 💾 データの挿入

### コマンドラインから挿入

```bash
# JSONファイルを使用してデータを挿入
curl -X POST localhost:8080/mutate \
     -H "Content-Type: application/json" \
     --data-binary '@examples/mutation.json'
```

データの内容（`examples/mutation.json`）:
```json
{
  "set": [
    {
      "uid": "_:alice",
      "dgraph.type": "Person",
      "name": "Alice",
      "age": 30,
      "email": "alice@example.com"
    },
    {
      "uid": "_:bob",
      "dgraph.type": "Person",
      "name": "Bob",
      "age": 25,
      "email": "bob@example.com"
    },
    {
      "uid": "_:dgraph",
      "dgraph.type": "Company",
      "name": "Dgraph Labs",
      "industry": "Technology",
      "location": "San Francisco"
    },
    {
      "uid": "_:alice",
      "works_for": {
        "uid": "_:dgraph"
      }
    },
    {
      "uid": "_:alice",
      "friend": {
        "uid": "_:bob"
      }
    }
  ]
}
```

### Ratel UIから挿入

1. 「Mutate」タブを選択
2. 上記のJSONを貼り付け
3. 「Run」ボタンをクリック

## 🔍 クエリの実行

### コマンドラインからクエリ

```bash
# クエリファイルを使用してデータを検索
curl -X POST localhost:8080/query \
     -H "Content-Type: application/graphql" \
     --data-binary '@examples/queries.graphql'
```

クエリ例（`examples/queries.graphql`）:

### 全ての人を取得
```graphql
{
  people(func: type(Person)) {
    uid
    name
    age
    email
    friend {
      name
      age
    }
    works_for {
      name
      industry
    }
  }
}
```

### 特定の名前で検索
```graphql
{
  findPerson(func: eq(name, "Alice")) {
    uid
    name
    age
    email
    friend {
      name
      age
    }
    works_for {
      name
      industry
      location
    }
  }
}
```

### 年齢でフィルタリング
```graphql
{
  adults(func: type(Person)) @filter(ge(age, 25)) {
    name
    age
    email
  }
}
```

### 会社と従業員を取得
```graphql
{
  companies(func: type(Company)) {
    name
    industry
    location
    ~works_for @filter(ge(age, 25)) {
      name
      age
      email
    }
  }
}
```

### クエリの説明
- `func: type(Person)`: Person型のノードを検索
- `@filter(ge(age, 25))`: age >= 25 でフィルタリング
- `~works_for`: 逆方向の関係を参照（会社から従業員を取得）

## 🔧 トラブルシューティング

### コンテナが起動しない場合
```bash
# ログの確認
docker-compose logs

# コンテナの再起動
docker-compose down
docker-compose up -d
```

### データが表示されない場合
1. スキーマが正しく設定されているか確認
   ```bash
   curl localhost:8080/query -H "Content-Type: application/graphql" -d '{schema{}}'
   ```

2. Mutationが正常に実行されたか確認
   - レスポンスのJSONに "code": "Success" が含まれているか確認

3. クエリの構文が正しいか確認
   - GraphQL構文に従っているか
   - 参照しているフィールドがスキーマに定義されているか

### 接続エラーの場合
1. すべてのコンテナが起動しているか確認
   ```bash
   docker-compose ps
   ```

2. ポートの競合がないか確認
   ```bash
   netstat -an | findstr "8000 8080 9080"
   ```

3. APIエンドポイントの疎通確認
   ```bash
   curl -I localhost:8080/health
