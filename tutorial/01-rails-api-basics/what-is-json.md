# JSONとは何か？

## JSONとは

**JSON（JavaScript Object Notation）** は、データを交換するためのテキスト形式です。名前にJavaScriptとありますが、現在はあらゆるプログラミング言語で使用されている標準的なデータ形式です。

## なぜJSONが使われるのか？

### Railsの従来の方法（HTML）
```erb
<!-- ブラウザ向けのHTML -->
<div class="user">
  <h3><%= @user.name %></h3>
  <p><%= @user.email %></p>
</div>
```

### API（JSON）
```json
{
  "user": {
    "id": 1,
    "name": "田中太郎",
    "email": "tanaka@example.com"
  }
}
```

**JSON の利点:**
- 人間が読みやすい
- 軽量（HTMLより小さい）
- あらゆるプログラミング言語で扱える
- スマホアプリ、Webアプリ、他のシステムなど、何でも使える

## JSONの基本構造

### 1. オブジェクト（{}で囲む）
```json
{
  "name": "田中太郎",
  "age": 25,
  "active": true
}
```

### 2. 配列（[]で囲む）
```json
[
  "りんご",
  "バナナ",
  "オレンジ"
]
```

### 3. 組み合わせ
```json
{
  "users": [
    {
      "id": 1,
      "name": "田中太郎",
      "hobbies": ["読書", "映画鑑賞"]
    },
    {
      "id": 2,
      "name": "佐藤花子", 
      "hobbies": ["料理", "旅行"]
    }
  ],
  "total": 2
}
```

## データ型

| 型 | 例 | 説明 |
|---|---|---|
| **文字列** | `"Hello"` | ダブルクォートで囲む |
| **数値** | `123`, `3.14` | そのまま書く |
| **真偽値** | `true`, `false` | そのまま書く |
| **null** | `null` | 何もない値 |
| **オブジェクト** | `{"key": "value"}` | 中カッコで囲む |
| **配列** | `[1, 2, 3]` | 角カッコで囲む |

## Railsでの実例

### コントローラでJSONを返す
```ruby
class Api::V1::UsersController < ApplicationController
  def index
    users = User.all
    
    # JSONとして返す
    render json: {
      users: users.map do |user|
        {
          id: user.id,
          name: user.name,
          email: user.email,
          created_at: user.created_at.strftime('%Y-%m-%d')
        }
      end,
      total: users.count
    }
  end
end
```

### 実際のレスポンス
```json
{
  "users": [
    {
      "id": 1,
      "name": "田中太郎",
      "email": "tanaka@example.com",
      "created_at": "2024-01-15"
    },
    {
      "id": 2,
      "name": "佐藤花子",
      "email": "sato@example.com", 
      "created_at": "2024-01-20"
    }
  ],
  "total": 2
}
```

## よくある間違い

### ❌ 間違い例
```json
{
  name: "田中太郎",           // キーにクォートがない
  'email': "tanaka@...",     // シングルクォート
  "age": 25,                 // 最後にカンマ
}
```

### ✅ 正しい例
```json
{
  "name": "田中太郎",
  "email": "tanaka@example.com",
  "age": 25
}
```

## JSONの確認方法

### 1. ブラウザで確認
```
http://localhost:3000/api/v1/users
```
ブラウザでAPIにアクセスすると、JSONが表示されます。

### 2. curl コマンドで確認
```bash
curl http://localhost:3000/api/v1/users
```

### 3. ブラウザの開発者ツール
1. F12で開発者ツールを開く
2. Networkタブを選択
3. ページをリロード
4. APIのリクエストをクリック
5. ResponseタブでJSONを確認

## JSON vs Ruby Hash の違い

### Ruby Hash
```ruby
{
  name: "田中太郎",
  age: 25,
  hobbies: ["読書", "映画鑑賞"]
}
```

### JSON
```json
{
  "name": "田中太郎",
  "age": 25,
  "hobbies": ["読書", "映画鑑賞"]
}
```

**主な違い:**
- JSONは**必ず**キーをダブルクォートで囲む
- JSONは文字列形式（テキスト）
- Ruby HashはRubyオブジェクト

## まとめ

- JSONはAPIでデータを送受信するための標準形式
- 人間が読みやすく、プログラムでも扱いやすい
- Railsでは `render json:` で簡単にJSONを返せる
- フロントエンド（React）がこのJSONを受け取って画面に表示する

**💡 ポイント**: まずは「データを交換するためのテキスト形式」と覚えておけば十分です！