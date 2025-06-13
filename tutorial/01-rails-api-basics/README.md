# Rails APIモード基礎編

Rails経験者向けに、APIモードの基本概念と従来のRailsとの違いを学習します。

> **💡 学習のコツ**
> 
> この章では、新しい概念がたくさん出てきます。すべてを完璧に理解する必要はありません！
> - 分からない部分は一旦飛ばして、後で戻ってきても大丈夫です
> - 詰まったときは、メンターやChatGPT/Claude等のLLMに質問してみましょう
> - 実際に手を動かしながら学ぶことで理解が深まります

## 補足資料

理解が難しい概念については、以下の補足資料を参照してください：

- 📖 **[JSONとは何か？](./what-is-json.md)** - API で使用するデータ形式について
- 🔒 **[CORSとは何か？](./what-is-cors.md)** - フロントエンドとバックエンド間の通信設定について  
- 🏛️ **[StatelessとStatefulの違い](./stateless-vs-stateful.md)** - APIの設計思想について

## 学習目標

- Rails APIモードの特徴を理解する
- 従来のRailsアプリケーションとの違いを把握する
- JSON APIの設計原則を学ぶ
- 基本的なAPIエンドポイントを作成できるようになる

## 1. Rails APIモードとは？

### 従来のRails（フルスタック）との違い

| 項目 | 従来のRails | Rails APIモード |
|------|-------------|-----------------|
| **View層** | ERBテンプレート | [JSON レスポンスのみ](./what-is-json.md) |
| **セッション** | Cookie-based | [Stateless（状態なし）](./stateless-vs-stateful.md) |
| **ミドルウェア** | フルセット | API用に最適化 |
| **アセット管理** | あり | なし |
| **用途** | Webアプリ全体 | バックエンドAPI |

### APIモードの利点

- **軽量**: 不要な機能が除外されている
- **高速**: レスポンス時間が短縮
- **分離**: フロントエンドと完全に分離
- **スケーラブル**: マイクロサービス化しやすい

## 2. 現在のプロジェクト構成を理解する

### ディレクトリ構造

```
app/
├── controllers/
│   ├── application_controller.rb    # ベースコントローラ
│   ├── api/v1/                     # API v1 namespace
│   │   └── hello_controller.rb     # サンプルコントローラ
│   └── home_controller.rb          # (通常は不要)
├── models/                         # データモデル
└── jobs/                          # バックグラウンドジョブ
```

### 重要な設定ファイル

```
config/
├── application.rb                  # API設定
├── routes.rb                      # ルーティング
└── initializers/
    └── cors.rb                    # CORS設定
```

## 3. 実際のコードを確認してみよう

### ApplicationController の確認

```bash
# コントローラの内容を確認
cat app/controllers/application_controller.rb
```

**解説**: APIモードでは `ActionController::API` を継承します。これにより、View関連の機能が除外されます。

### サンプルAPIエンドポイントの確認

```bash
# Hello API の確認
cat app/controllers/api/v1/hello_controller.rb
```

### ルーティングの確認

```bash
# ルーティング確認
cat config/routes.rb
```

### API バージョニング

このプロジェクトでは `/api/v1/` という名前空間を使用しています：

- **利点**: 後方互換性を保ちながらAPIを更新可能
- **構造**: `api/v1/hello_controller.rb` → `/api/v1/hello`

## 4. APIを実際に動かしてみよう

### 開発サーバーの起動

```bash
# Docker環境の場合（推奨）
docker compose up -d

# 直接Railsコマンドを実行したい場合
docker compose exec app bundle exec rails s -p 3000 -b '0.0.0.0'
```

### API テスト

```bash
# Hello APIをテスト
curl http://localhost:3000/api/v1/hello

# レスポンス例:
# {\"message\":\"Hello from Rails API!\"}
```

## 5. 新しいAPIエンドポイントを作成してみよう

### Step 1: Users コントローラを作成

**重要**: ファイルの作成はVSCodeで行うことを推奨します（Dockerでファイルを作成するとパーミッションエラーが発生することがあります）

```bash
# コントローラ生成（Docker経由）
docker compose exec app bundle exec rails generate controller api/v1/users index show

# または、VSCodeで手動でファイルを作成
# app/controllers/api/v1/users_controller.rb
```

### Step 2: ルーティングを追加

VSCodeで `config/routes.rb` を編集して追加:

```ruby
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      get '/hello', to: 'hello#index'
      resources :users, only: [:index, :show]  # 追加
    end
  end
end
```

### Step 3: コントローラを実装

VSCodeで `app/controllers/api/v1/users_controller.rb` を作成または編集:

```ruby
class Api::V1::UsersController < ApplicationController
  def index
    users = [
      { id: 1, name: "田中太郎", email: "tanaka@example.com" },
      { id: 2, name: "佐藤花子", email: "sato@example.com" }
    ]
    render json: { users: users }
  end

  def show
    user = { 
      id: params[:id], 
      name: "ユーザー#{params[:id]}", 
      email: "user#{params[:id]}@example.com" 
    }
    render json: { user: user }
  end
end
```

### Step 4: APIテスト

```bash
# ユーザー一覧
curl http://localhost:3000/api/v1/users

# 特定ユーザー
curl http://localhost:3000/api/v1/users/1
```

## 6. JSON レスポンスの設計

> **📖 補足**: JSONについて詳しく知りたい場合は **[JSONとは何か？](./what-is-json.md)** を参照してください。

### 良いAPI設計の原則

1. **一貫性のある構造**
```json
{
  "data": { ... },
  "meta": { "count": 10 },
  "errors": []
}
```

2. **適切なHTTPステータスコード**
- 200: 成功
- 201: 作成成功
- 400: リクエストエラー
- 404: 見つからない
- 500: サーバーエラー

3. **エラーハンドリング**
```ruby
def show
  user = User.find(params[:id])
  render json: { user: user }
rescue ActiveRecord::RecordNotFound
  render json: { error: "User not found" }, status: :not_found
end
```

## 7. CORS設定について

> **🔒 補足**: CORSについて詳しく知りたい場合は **[CORSとは何か？](./what-is-cors.md)** を参照してください。

### CORS（Cross-Origin Resource Sharing）とは？

フロントエンド（localhost:8080）からバックエンドAPI（localhost:3000）にアクセスするために必要な設定です。

### 設定確認

```bash
# CORS設定を確認
cat config/initializers/cors.rb
```

この設定により、React アプリからAPI呼び出しが可能になります。

## 8. まとめ

### 学習した内容

- ✅ Rails APIモードの特徴
- ✅ 従来のRailsとの違い  
- ✅ 基本的なAPIエンドポイントの作成
- ✅ JSON レスポンスの設計
- ✅ CORS設定の理解

### 次のステップ

次は **[React基礎編](../02-react-basics/README.md)** でフロントエンド側の学習を進めましょう。

### 参考資料

- [Rails API Documentation](https://guides.rubyonrails.org/api_app.html)
- [REST API Design Best Practices](https://restfulapi.net/)

## 練習問題

以下の練習問題に挑戦してみましょう。分からない場合は、メンターやLLMに相談してください。

1. `/api/v1/posts` エンドポイントを作成し、ブログ記事の一覧を返すAPIを実装してみてください
2. エラーハンドリングを追加して、適切なステータスコードを返すようにしてみてください
3. レスポンスに `meta` 情報（件数など）を含めるように改善してみてください

> **💡 ヒント**: ファイルの作成・編集はVSCodeで行い、Dockerコマンドは `docker compose exec app` を前に付けて実行しましょう。