# Rails API + React連携編

これまでに学んだRails APIとReactを組み合わせて、フロントエンドとバックエンドを連携させる方法を学習します。

> **💡 学習のコツ**
> 
> この章では非同期処理という新しい概念が出てきます。最初は混乱するかもしれませんが、大丈夫です！
> - 分からない部分は補足資料を読んで理解を深めましょう
> - 詰まったときは、メンターやChatGPT/Claude等のLLMに質問してみましょう
> - まずは動かしてみて、後から仕組みを理解するのもOKです

## 補足資料

非同期処理について理解を深めたい場合は、以下の補足資料を参照してください：

- 🤝 **[Promiseとは何か？](./understanding-promises.md)** - 非同期処理の基本概念について
- ⚡ **[同期処理と非同期処理の違い](./sync-vs-async.md)** - async/awaitの使い方について

## 学習目標

- axiosを使ったHTTP通信の基本を理解する
- Rails APIからデータを取得してReactで表示する
- CRUD操作（作成、読み取り、更新、削除）を実装する
- エラーハンドリングの方法を学ぶ
- 非同期処理（async/await）を理解する

## 1. HTTP通信の基本概念

### Rails の場合（従来）

```ruby
# コントローラで直接データを取得
class UsersController < ApplicationController
  def index
    @users = User.all
  end
end
```

### React + API の場合

```jsx
// コンポーネント内でAPIを呼び出し
function UserList() {
  const [users, setUsers] = useState([]);
  
  useEffect(() => {
    fetch('/api/v1/users')
      .then(response => response.json())
      .then(data => setUsers(data.users));
  }, []);
  
  return (
    // JSX
  );
}
```

## 2. axiosによるHTTP通信

> **🤝 補足**: 非同期処理が初めての方は **[Promiseとは何か？](./understanding-promises.md)** を先に読むことをお勧めします。

### axiosとは？

- JavaScriptのHTTPクライアントライブラリ
- Promise ベースで非同期処理を扱いやすい
- リクエスト・レスポンスのインターセプト機能
- 自動的なJSONデータ変換

### 基本的な使い方

> **⚡ 補足**: async/awaitについて詳しく知りたい場合は **[同期処理と非同期処理の違い](./sync-vs-async.md)** を参照してください。

```jsx
import axios from 'axios';

// GET リクエスト
const response = await axios.get('/api/v1/users');
console.log(response.data);

// POST リクエスト
const newUser = { name: '新規ユーザー', email: 'new@example.com' };
const response = await axios.post('/api/v1/users', newUser);

// PUT リクエスト
const updatedUser = { name: '更新済みユーザー' };
await axios.put('/api/v1/users/1', updatedUser);

// DELETE リクエスト
await axios.delete('/api/v1/users/1');
```

## 3. API設定の確認

### 現在の設定を確認

```bash
# CORS設定を確認
cat config/initializers/cors.rb

# 現在のAPIエンドポイントを確認
cat app/controllers/api/v1/hello_controller.rb
```

### axiosのベースURL設定

`front/src/lib/api.ts` を作成:

```typescript
import axios from 'axios';

// APIのベースURLを設定
const API_BASE_URL = 'http://localhost:3000/api/v1';

export const api = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Content-Type': 'application/json',
  },
});

// レスポンスインターセプター（エラーハンドリング）
api.interceptors.response.use(
  (response) => response,
  (error) => {
    console.error('API Error:', error.response?.data || error.message);
    return Promise.reject(error);
  }
);
```

## 4. 実際にAPIを呼び出してみよう

### Step 1: API接続テスト

`front/src/components/ApiTest.tsx` を作成:

```typescript
import { useState, useEffect } from 'react';
import { api } from '../lib/api';

function ApiTest() {
  const [message, setMessage] = useState<string>('');
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await api.get('/hello');
        setMessage(response.data.message);
      } catch (err) {
        setError('APIの呼び出しに失敗しました');
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, []);

  if (loading) return <div>読み込み中...</div>;
  if (error) return <div style={{ color: 'red' }}>エラー: {error}</div>;

  return (
    <div>
      <h2>API接続テスト</h2>
      <p>メッセージ: {message}</p>
    </div>
  );
}

export default ApiTest;
```

### Step 2: App.tsx に追加

```typescript
import ApiTest from './components/ApiTest';

function App() {
  return (
    <div className="App">
      <ApiTest />
    </div>
  );
}
```

### Step 3: 動作確認

```bash
# Docker環境起動（フロントエンド・バックエンド両方）
docker compose up -d

# ブラウザで http://localhost:8080 を確認
# ホットリロードが効くので、コード変更は自動で反映されます
```

## 5. ユーザー管理システムの実装

### Rails側: Users APIの実装

`app/controllers/api/v1/users_controller.rb`:

```ruby
class Api::V1::UsersController < ApplicationController
  def index
    users = [
      { id: 1, name: "田中太郎", email: "tanaka@example.com", created_at: "2024-01-15" },
      { id: 2, name: "佐藤花子", email: "sato@example.com", created_at: "2024-01-20" },
      { id: 3, name: "鈴木次郎", email: "suzuki@example.com", created_at: "2024-01-25" }
    ]
    render json: { users: users, count: users.length }
  end

  def show
    user = { 
      id: params[:id].to_i, 
      name: "ユーザー#{params[:id]}", 
      email: "user#{params[:id]}@example.com",
      created_at: "2024-01-#{params[:id].to_i + 10}"
    }
    render json: { user: user }
  end

  def create
    # 簡単な検証
    if params[:name].blank? || params[:email].blank?
      render json: { error: "名前とメールアドレスは必須です" }, status: :bad_request
      return
    end

    user = {
      id: rand(100..999),
      name: params[:name],
      email: params[:email],
      created_at: Time.current.strftime("%Y-%m-%d")
    }
    
    render json: { user: user, message: "ユーザーが作成されました" }, status: :created
  end

  def update
    user = {
      id: params[:id].to_i,
      name: params[:name] || "更新されたユーザー#{params[:id]}",
      email: params[:email] || "updated#{params[:id]}@example.com",
      created_at: "2024-01-#{params[:id].to_i + 10}"
    }
    
    render json: { user: user, message: "ユーザーが更新されました" }
  end

  def destroy
    render json: { message: "ユーザーID#{params[:id]}が削除されました" }
  end
end
```

### ルーティング追加

`config/routes.rb`:

```ruby
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      get '/hello', to: 'hello#index'
      resources :users # これで全てのRESTfulルートが作成される
    end
  end
end
```

### React側: Users APIクライアント

`front/src/services/userService.ts`:

```typescript
import { api } from '../lib/api';

export interface User {
  id: number;
  name: string;
  email: string;
  created_at: string;
}

export interface CreateUserData {
  name: string;
  email: string;
}

export interface UpdateUserData {
  name?: string;
  email?: string;
}

export const userService = {
  // ユーザー一覧取得
  async getUsers(): Promise<User[]> {
    const response = await api.get('/users');
    return response.data.users;
  },

  // 単一ユーザー取得
  async getUser(id: number): Promise<User> {
    const response = await api.get(`/users/${id}`);
    return response.data.user;
  },

  // ユーザー作成
  async createUser(userData: CreateUserData): Promise<User> {
    const response = await api.post('/users', userData);
    return response.data.user;
  },

  // ユーザー更新
  async updateUser(id: number, userData: UpdateUserData): Promise<User> {
    const response = await api.put(`/users/${id}`, userData);
    return response.data.user;
  },

  // ユーザー削除
  async deleteUser(id: number): Promise<void> {
    await api.delete(`/users/${id}`);
  }
};
```

## 6. CRUD操作の実装

### ユーザー一覧コンポーネント

`front/src/components/UserList.tsx`:

```typescript
import { useState, useEffect } from 'react';
import { userService, User } from '../services/userService';

function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  // ユーザー一覧を取得
  const fetchUsers = async () => {
    try {
      setLoading(true);
      const userData = await userService.getUsers();
      setUsers(userData);
    } catch (err) {
      setError('ユーザーの取得に失敗しました');
    } finally {
      setLoading(false);
    }
  };

  // ユーザー削除
  const handleDelete = async (id: number) => {
    if (!confirm('本当に削除しますか？')) return;
    
    try {
      await userService.deleteUser(id);
      setUsers(users.filter(user => user.id !== id));
      alert('ユーザーが削除されました');
    } catch (err) {
      alert('削除に失敗しました');
    }
  };

  useEffect(() => {
    fetchUsers();
  }, []);

  if (loading) return <div>読み込み中...</div>;
  if (error) return <div style={{ color: 'red' }}>エラー: {error}</div>;

  return (
    <div>
      <h2>ユーザー一覧</h2>
      <button onClick={fetchUsers} style={{ marginBottom: '16px' }}>
        再読み込み
      </button>
      
      {users.length === 0 ? (
        <p>ユーザーがいません</p>
      ) : (
        <div>
          {users.map(user => (
            <div key={user.id} style={{ 
              border: '1px solid #ccc', 
              padding: '16px', 
              margin: '8px 0',
              borderRadius: '4px'
            }}>
              <h3>{user.name}</h3>
              <p>メール: {user.email}</p>
              <p>作成日: {user.created_at}</p>
              <button 
                onClick={() => handleDelete(user.id)}
                style={{ 
                  backgroundColor: '#dc3545',
                  color: 'white',
                  border: 'none',
                  padding: '8px 16px',
                  borderRadius: '4px',
                  cursor: 'pointer'
                }}
              >
                削除
              </button>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}

export default UserList;
```

### ユーザー作成フォーム

`front/src/components/UserForm.tsx`:

```typescript
import { useState } from 'react';
import { userService, CreateUserData } from '../services/userService';

interface UserFormProps {
  onUserCreated?: () => void;
}

function UserForm({ onUserCreated }: UserFormProps) {
  const [formData, setFormData] = useState<CreateUserData>({
    name: '',
    email: ''
  });
  const [loading, setLoading] = useState<boolean>(false);
  const [error, setError] = useState<string | null>(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!formData.name || !formData.email) {
      setError('名前とメールアドレスを入力してください');
      return;
    }

    try {
      setLoading(true);
      setError(null);
      
      await userService.createUser(formData);
      
      // フォームをリセット
      setFormData({ name: '', email: '' });
      
      alert('ユーザーが作成されました');
      
      // 親コンポーネントに通知
      if (onUserCreated) {
        onUserCreated();
      }
    } catch (err) {
      setError('ユーザーの作成に失敗しました');
    } finally {
      setLoading(false);
    }
  };

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };

  return (
    <div>
      <h2>新規ユーザー作成</h2>
      {error && <div style={{ color: 'red' }}>{error}</div>}
      
      <form onSubmit={handleSubmit}>
        <div style={{ marginBottom: '16px' }}>
          <label>
            名前:
            <input
              type="text"
              name="name"
              value={formData.name}
              onChange={handleInputChange}
              style={{ marginLeft: '8px', padding: '4px' }}
            />
          </label>
        </div>
        
        <div style={{ marginBottom: '16px' }}>
          <label>
            メールアドレス:
            <input
              type="email"
              name="email"
              value={formData.email}
              onChange={handleInputChange}
              style={{ marginLeft: '8px', padding: '4px' }}
            />
          </label>
        </div>
        
        <button 
          type="submit" 
          disabled={loading}
          style={{
            backgroundColor: '#007bff',
            color: 'white',
            border: 'none',
            padding: '8px 16px',
            borderRadius: '4px',
            cursor: loading ? 'not-allowed' : 'pointer'
          }}
        >
          {loading ? '作成中...' : '作成'}
        </button>
      </form>
    </div>
  );
}

export default UserForm;
```

### 統合コンポーネント

`front/src/components/UserManagement.tsx`:

```typescript
import { useState } from 'react';
import UserList from './UserList';
import UserForm from './UserForm';

function UserManagement() {
  const [refreshKey, setRefreshKey] = useState<number>(0);

  // ユーザー作成後にリストを更新
  const handleUserCreated = () => {
    setRefreshKey(prev => prev + 1);
  };

  return (
    <div>
      <h1>ユーザー管理システム</h1>
      
      <div style={{ display: 'flex', gap: '32px' }}>
        <div style={{ flex: 1 }}>
          <UserForm onUserCreated={handleUserCreated} />
        </div>
        
        <div style={{ flex: 1 }}>
          <UserList key={refreshKey} />
        </div>
      </div>
    </div>
  );
}

export default UserManagement;
```

## 7. エラーハンドリングとローディング状態

### 包括的なエラーハンドリング

```typescript
// APIエラーの型定義
interface ApiError {
  message: string;
  status?: number;
}

// エラーハンドリング関数
const handleApiError = (error: any): ApiError => {
  if (error.response) {
    // サーバーからのレスポンスがある場合
    return {
      message: error.response.data?.error || 'サーバーエラーが発生しました',
      status: error.response.status
    };
  } else if (error.request) {
    // リクエストが送信されたが応答がない場合
    return {
      message: 'サーバーに接続できません'
    };
  } else {
    // その他のエラー
    return {
      message: '予期しないエラーが発生しました'
    };
  }
};
```

### ローディング状態の管理

```typescript
// カスタムフックでローディング状態を管理
import { useState } from 'react';

function useLoading() {
  const [loading, setLoading] = useState<boolean>(false);

  const withLoading = async <T>(asyncFunction: () => Promise<T>): Promise<T> => {
    setLoading(true);
    try {
      return await asyncFunction();
    } finally {
      setLoading(false);
    }
  };

  return { loading, withLoading };
}
```

## 8. 実装の動作確認

### Step 1: サーバー起動

```bash
# Docker環境起動（バックエンド・フロントエンド両方）
docker compose up -d

# ログ確認（必要に応じて）
docker compose logs -f
```

### Step 2: 機能テスト

1. **ユーザー一覧表示**: http://localhost:8080 でユーザー一覧が表示される
2. **ユーザー作成**: フォームからユーザーを作成できる
3. **ユーザー削除**: 削除ボタンでユーザーを削除できる
4. **エラーハンドリング**: バックエンドを停止してエラー表示を確認

### Step 3: ネットワークタブで通信確認

ブラウザのデベロッパーツールで：
1. Networkタブを開く
2. ユーザー作成・削除を実行
3. HTTP通信の詳細を確認（リクエスト・レスポンス）

## 9. まとめ

### 学習した内容

- ✅ axiosを使ったHTTP通信
- ✅ RESTful APIの実装（CRUD操作）
- ✅ 非同期処理（async/await）
- ✅ エラーハンドリング
- ✅ フォーム送信とstate管理
- ✅ コンポーネント間の連携

### Rails vs React+APIの比較

| 処理 | 従来のRails | React + API |
|------|-------------|-------------|
| **データ取得** | コントローラで取得 | API呼び出し |
| **フォーム送信** | form_with | axios.post |
| **ページ更新** | redirect_to | state更新 |
| **エラー表示** | flash | state管理 |

### 次のステップ

次は **[実践編：ToDoアプリ作成](../04-practical-todo-app/README.md)** で、より実践的なアプリケーションを作成しましょう。

### 参考資料

- [Axios Documentation](https://axios-http.com/docs/intro)
- [React Query](https://tanstack.com/query/latest) - より高度なデータフェッチ
- [SWR](https://swr.vercel.app/) - データフェッチライブラリ

## 練習問題

以下の練習問題に挑戦してみましょう。分からない場合は、メンターやLLMに相談してください。

1. ユーザー編集機能を追加してください（PUT リクエスト）
2. ユーザー検索機能を実装してください

> **💡 ヒント**: 
> - ファイルの作成・編集はVSCodeで行いましょう
> - `docker compose up -d`でサーバーを起動してテストしましょう
> - 非同期処理で困ったら補足資料を参照してください