# StatelessとStatefulの違い

## 基本的な考え方

**State（状態）** = 「覚えている情報」のこと

- **Stateful** = 状態を覚えている
- **Stateless** = 状態を覚えていない

## 身近な例で理解する

### 例1：コンビニのレジ

**Stateful（状態を覚える）:**
```
店員: 「いらっしゃいませ」
あなた: 「弁当ください」
店員: 「かしこまりました。他にありますか？」  ←弁当を覚えている
あなた: 「お茶も」
店員: 「弁当とお茶ですね」  ←両方覚えている
```

**Stateless（状態を覚えない）:**
```
あなた: 「弁当とお茶ください」  ←毎回全部言う必要がある
店員: 「弁当とお茶ですね」
```

### 例2：電話での注文

**Stateful:**
```
1回目の電話: 「ピザのLサイズお願いします」
2回目の電話: 「さっきの注文にコーラも追加で」 ←前の注文を覚えている
```

**Stateless:**  
```
1回目の電話: 「ピザのLサイズお願いします」
2回目の電話: 「ピザのLサイズとコーラお願いします」 ←毎回全部言う
```

## 従来のRails（Stateful）

### セッションによる状態管理
```ruby
class UsersController < ApplicationController
  def show
    # ログイン状態をセッションで記憶
    if session[:user_id]
      @user = User.find(session[:user_id])  # 「あなたは田中さんですね」
    else
      redirect_to login_path  # 「あなたは誰ですか？」
    end
  end
end
```

### 特徴
- **サーバーが覚えている**: 「この人は田中さん」「ログイン済み」など
- **Cookie/Session**: ブラウザとサーバーで状態を共有
- **継続性**: ページを移動しても状態が保持される

```
ブラウザ ←→ Rails（Webアプリ）
   ↑
セッションで状態を共有
「あなたは田中さん、ログイン済み」
```

## Rails API（Stateless）

### トークンによる認証
```ruby
class Api::V1::UsersController < ApplicationController
  def show
    # 毎回トークンで身元確認
    token = request.headers['Authorization']  # 「身分証明書見せて」
    user = User.find_by_token(token)         # 「確認しました」
    
    if user
      render json: { user: user }
    else  
      render json: { error: 'Unauthorized' }, status: 401
    end
  end
end
```

### 特徴
- **サーバーは覚えていない**: 毎回「あなたは誰？」
- **毎回身元確認**: リクエストごとにトークンで確認
- **独立性**: 各リクエストが完全に独立

```
React ←→ Rails API
  ↑
毎回トークンを送信
「私は田中です（証明書付き）」
```

## なぜAPIはStatelessなのか？

### 1. スケーラビリティ（拡張性）

**Stateful（覚える方式）:**
```
サーバー1: 田中さんのセッション情報を保持
サーバー2: 佐藤さんのセッション情報を保持

田中さんがサーバー2にアクセス → 「あなたは誰？」（情報がない）
```

**Stateless（覚えない方式）:**
```
どのサーバーでも同じ
田中さん: 「私は田中です（トークン付き）」
サーバー1: 「確認しました」
サーバー2: 「確認しました」
```

### 2. 複数のクライアント対応

**従来のRails:**
```
Webブラウザのみ ←→ Rails
```

**Rails API:**
```
Reactアプリ ←→ Rails API
スマホアプリ ←→ Rails API  
他のシステム ←→ Rails API
```

### 3. 障害対応

**Stateful:**
```
サーバー再起動 → セッション情報消失 → 全ユーザーがログアウト
```

**Stateless:**
```
サーバー再起動 → トークンは残る → ユーザーはそのまま利用可能
```

## 実際のコード比較

### 従来のRails（Stateful）
```ruby
# コントローラ
class PostsController < ApplicationController
  before_action :authenticate_user!  # セッションでチェック
  
  def create
    @post = current_user.posts.create(post_params)  # current_userはセッションから
    redirect_to @post
  end
end

# ビュー
<%= form_with model: @post do |f| %>
  <%= f.text_field :title %>
  <%= f.submit %>
<% end %>
```

### Rails API（Stateless）
```ruby
# コントローラ
class Api::V1::PostsController < ApplicationController
  def create
    # 毎回トークンで認証
    token = request.headers['Authorization']
    user = authenticate_token(token)
    
    if user
      post = user.posts.create(post_params)
      render json: { post: post }
    else
      render json: { error: 'Unauthorized' }, status: 401
    end
  end
end
```

```javascript
// React側
const createPost = async (postData) => {
  const response = await axios.post('/api/v1/posts', postData, {
    headers: {
      'Authorization': `Bearer ${userToken}`  // 毎回トークンを送信
    }
  });
};
```

## まとめ

### Stateful（従来のRails）
- **サーバーが状態を覚えている**
- セッション/Cookieを使用
- ブラウザ専用
- スケールしにくい

### Stateless（Rails API）
- **サーバーは状態を覚えていない**
- 毎回認証情報を送信
- どんなクライアントでも対応可能
- スケールしやすい

**📝 覚え方:**
- **Stateful** = 「馴染みの店員さん（あなたを覚えている）」
- **Stateless** = 「セルフレジ（毎回全部説明が必要）」

**💡 重要:** 
APIを作る理由の一つが「いろんなアプリから使えるようにするため」です。そのためには、サーバーが特定の状態を覚えていない方が都合が良いのです。

**🤔 最初は難しく感じるかもしれませんが、「毎回身分証明書を見せる」と考えると理解しやすいです！**