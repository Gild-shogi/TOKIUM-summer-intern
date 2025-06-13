# CORSとは何か？

## CORSとは

**CORS（Cross-Origin Resource Sharing）** は、異なるドメイン間でのデータ通信を安全に行うためのWebブラウザのセキュリティ機能です。

## なぜCORSが必要なのか？

### 問題の状況
```
フロントエンド（React）: http://localhost:8080
バックエンド（Rails API）: http://localhost:3000
```

通常、ブラウザは**セキュリティ上の理由**で、異なるドメイン（オリジン）への通信を制限します。これを「**同一オリジンポリシー**」と呼びます。

### 同一オリジンポリシーとは

**オリジン = プロトコル + ドメイン + ポート**

| URL | プロトコル | ドメイン | ポート | オリジン |
|-----|-----------|----------|--------|----------|
| `http://localhost:8080` | http | localhost | 8080 | A |
| `http://localhost:3000` | http | localhost | 3000 | B |
| `https://example.com` | https | example.com | 443 | C |

オリジンAからオリジンBにアクセスしようとすると、ブラウザが「待った！」をかけます。

## CORSエラーの例

### よく見るエラーメッセージ
```
Access to fetch at 'http://localhost:3000/api/v1/users' 
from origin 'http://localhost:8080' has been blocked by CORS policy: 
No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

**翻訳**: 「localhost:8080からlocalhost:3000へのアクセスがCORSポリシーによってブロックされました」

## 身近な例で理解する

### 例：銀行のATM
```
あなた（React）が他の銀行（Rails API）のATMを使いたい場合

1. 普通は使えない（同一オリジンポリシー）
2. 提携している銀行なら使える（CORS許可設定）
3. 提携の証明書が必要（CORSヘッダー）
```

### 例：学校の図書館
```
A校の学生（React）がB校の図書館（Rails API）を使いたい場合

1. 普通は入れない（セキュリティ）
2. B校が「A校の学生OK」と許可すれば使える（CORS設定）
3. 学生証の確認が必要（CORSヘッダーチェック）
```

## Rails での CORS 設定

### 1. 設定ファイルの確認
```ruby
# config/initializers/cors.rb

Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins 'http://localhost:8080'  # Reactアプリのドメイン
    
    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end
```

### 2. 設定の意味

```ruby
origins 'http://localhost:8080'
# 「localhost:8080からのアクセスを許可する」

resource '*'
# 「すべてのURL（/api/v1/*など）へのアクセスを許可する」

methods: [:get, :post, ...]
# 「これらのHTTPメソッドを許可する」
```

### 3. 本番環境の設定例
```ruby
# 開発環境
origins 'http://localhost:8080'

# 本番環境  
origins 'https://my-react-app.vercel.app'

# 複数のドメインを許可
origins ['http://localhost:8080', 'https://my-app.com']
```

## CORSの動作の流れ

### 1. ブラウザが「事前確認」を送信
```
OPTIONS http://localhost:3000/api/v1/users
Origin: http://localhost:8080
```

### 2. サーバーが「許可情報」を返信
```
Access-Control-Allow-Origin: http://localhost:8080
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type
```

### 3. ブラウザが「OK、通信していいな」と判断
```
GET http://localhost:3000/api/v1/users
Origin: http://localhost:8080
```

### 4. 実際のデータを取得
```json
{
  "users": [...]
}
```

## よくあるCORSの問題と解決法

### 問題1: CORSエラーが出る
```
❌ CORS policy error
```

**解決法**: Rails の CORS 設定を確認
```ruby
# config/initializers/cors.rb が正しく設定されているか確認
origins 'http://localhost:8080'  # フロントエンドのURLと一致しているか
```

### 問題2: 本番環境でCORSエラー
```
❌ 開発環境では動くが本番で動かない
```

**解決法**: 本番のドメインを追加
```ruby
origins ['http://localhost:8080', 'https://your-app.herokuapp.com']
```

### 問題3: メソッドが許可されていない
```
❌ Method PUT is not allowed
```

**解決法**: 必要なメソッドを追加
```ruby
methods: [:get, :post, :put, :patch, :delete, :options, :head]
```

## CORSのセキュリティ

### なぜCORSが重要なのか？

**CORS設定なし**:
```
悪意のあるサイト -> あなたの銀行API -> 勝手に送金
```

**CORS設定あり**:
```
悪意のあるサイト -> あなたの銀行API -> ❌ ブロック！
信頼できるサイト -> あなたの銀行API -> ✅ 許可
```

### 適切な設定

```ruby
# ❌ 危険：すべてのドメインを許可
origins '*'

# ✅ 安全：特定のドメインのみ許可  
origins 'https://trusted-domain.com'

# ✅ 開発環境用
if Rails.env.development?
  origins 'http://localhost:8080'
end
```

## 実際の確認方法

### 1. ブラウザの開発者ツール
1. F12で開発者ツールを開く
2. Networkタブを選択
3. APIにアクセス
4. レスポンスヘッダーを確認

### 2. CORSヘッダーの確認
```
Response Headers:
Access-Control-Allow-Origin: http://localhost:8080
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
```

### 3. curlでの確認
```bash
curl -H "Origin: http://localhost:8080" \
     -H "Access-Control-Request-Method: GET" \
     -H "Access-Control-Request-Headers: X-Requested-With" \
     -X OPTIONS \
     http://localhost:3000/api/v1/users
```

## まとめ

- **CORS** = 異なるドメイン間の通信を安全に行うためのルール
- **目的** = セキュリティ向上（悪意のあるサイトからの攻撃を防ぐ）
- **設定** = Rails側で「どのドメインを信頼するか」を設定
- **重要性** = 設定しないとReactからAPIが呼べない

**💡 ポイント**: 「信頼できる相手とだけ通信する」ための仕組みと覚えましょう！

**🔧 実践**: プロジェクトの `config/initializers/cors.rb` を見て、設定を確認してみてください。