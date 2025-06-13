# React Hooks 一覧

## Hooksとは何か？

**Hooks（フック）** = Reactの機能を「引っ掛けて」使うための仕組み

関数コンポーネントで、状態管理やライフサイクルなどのReactの機能を使えるようにする特別な関数です。

## Hooksの基本ルール

1. **関数コンポーネントの中でのみ使用可能**
2. **必ずコンポーネントのトップレベルで呼び出す**（if文やfor文の中ではダメ）
3. **名前が`use`で始まる**

## よく使用するHooks

### 1. useState - 状態管理

**用途**: コンポーネントに状態（記憶）を持たせる

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);  // 状態の定義

  return (
    <div>
      <p>カウント: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

**いつ使うか:**
- ユーザーの入力値を保存したい時
- 表示/非表示の状態を管理したい時
- カウンターやタイマーの値を保持したい時

**身近な例**: 電卓の表示画面（入力した数字を覚えている）

### 2. useCallback - 関数の安定化とAPI通信

**用途**: 関数の再作成を防いで安定した関数を提供する + API通信の処理

```jsx
import { useState, useCallback } from 'react';

function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);

  // API通信の関数をuseCallbackで安定化
  const fetchUsers = useCallback(async () => {
    try {
      setLoading(true);
      const response = await fetch('/api/users');
      const data = await response.json();
      setUsers(data);
    } catch (error) {
      console.error('データの取得に失敗しました', error);
    } finally {
      setLoading(false);
    }
  }, []);

  const addUser = useCallback(async (userData) => {
    try {
      const response = await fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData)
      });
      const newUser = await response.json();
      setUsers(prev => [...prev, newUser]);
    } catch (error) {
      console.error('ユーザーの追加に失敗しました', error);
    }
  }, []);

  return (
    <div>
      <button onClick={fetchUsers} disabled={loading}>
        {loading ? '読み込み中...' : 'ユーザー一覧を取得'}
      </button>
      
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
      
      <button onClick={() => addUser({ name: '新しいユーザー' })}>
        ユーザーを追加
      </button>
    </div>
  );
}
```

**API通信でuseCallbackを使う理由:**
- 関数の再作成を防ぐ（パフォーマンス向上）
- 子コンポーネントに渡す時の無駄な再レンダリングを防ぐ
- useEffectで使用する際の依存配列の問題を回避

**いつ使うか:**
- API通信の処理
- 子コンポーネントに関数を渡す時
- 関数を安定させたい時

**身近な例**: 専用の道具箱（毎回新しい道具を作らず、同じ道具を使い続ける）

### 3. useEffect - 副作用の処理

**⚠️ 注意**: useEffectはAPI通信には使用せず、以下の場面でのみ使用を推奨します。

**用途**: DOM操作、タイマー、イベントリスナーの設定など

```jsx
import { useState, useEffect, useCallback } from 'react';

function DocumentTitle() {
  const [count, setCount] = useState(0);

  // ページタイトルの更新（DOM操作）
  useEffect(() => {
    document.title = `カウント: ${count}`;
  }, [count]);

  return (
    <div>
      <p>カウント: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}

function Timer() {
  const [time, setTime] = useState(0);

  // タイマーの設定と解除
  useEffect(() => {
    const interval = setInterval(() => {
      setTime(prev => prev + 1);
    }, 1000);

    // クリーンアップ関数（コンポーネントが削除される時に実行）
    return () => {
      clearInterval(interval);
    };
  }, []);

  return <div>経過時間: {time}秒</div>;
}
```

**適切な使用例:**
- ページタイトルの変更
- タイマーやイベントリスナーの設定/解除
- DOM要素への直接操作

**避けるべき使用例:**
- API通信（useCallbackを使用）
- 単純な値の計算（通常の関数で十分）
- 他のstateに基づく値の更新（無限ループの原因）

**身近な例**: お店の電気の管理（開店時に点灯、閉店時に消灯）

### 4. useContext - データの共有

**用途**: 深い階層のコンポーネント間でデータを共有する

```jsx
import { createContext, useContext, useState, useCallback } from 'react';

// 1. コンテキストを作成
const UserContext = createContext();

// 2. プロバイダーコンポーネント
function App() {
  const [user, setUser] = useState({ name: '田中太郎', role: 'admin' });

  // API通信もコンテキストで共有
  const updateUser = useCallback(async (userData) => {
    try {
      const response = await fetch('/api/user', {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData)
      });
      const updatedUser = await response.json();
      setUser(updatedUser);
    } catch (error) {
      console.error('ユーザー更新に失敗しました', error);
    }
  }, []);

  return (
    <UserContext.Provider value={{ user, setUser, updateUser }}>
      <Header />
      <MainContent />
      <Footer />
    </UserContext.Provider>
  );
}

// 3. データを使いたいコンポーネント
function Header() {
  const { user } = useContext(UserContext);

  return (
    <header>
      <h1>ようこそ、{user.name}さん</h1>
    </header>
  );
}

function UserProfile() {
  const { user, updateUser } = useContext(UserContext);

  const handleRoleChange = () => {
    updateUser({ ...user, role: user.role === 'admin' ? 'user' : 'admin' });
  };

  return (
    <div>
      <p>役割: {user.role}</p>
      <button onClick={handleRoleChange}>
        役割を変更
      </button>
    </div>
  );
}
```

**いつ使うか:**
- ログインユーザー情報の共有
- テーマ（ダーク/ライトモード）の管理
- 言語設定の共有
- 深い階層の複数コンポーネントで同じデータが必要な時

**使わない方が良い場面:**
- 単純な親子関係（propsで十分）
- 頻繁に変更されるデータ（パフォーマンス低下）

**身近な例**: 会社の共有フォルダ（どの部署からでもアクセスできる）

## API通信のベストプラクティス

### useCallbackを使った推奨パターン

```jsx
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [loading, setLoading] = useState(false);

  // データ取得
  const fetchTodos = useCallback(async () => {
    try {
      setLoading(true);
      const response = await fetch('/api/todos');
      const data = await response.json();
      setTodos(data);
    } catch (error) {
      console.error('取得エラー:', error);
    } finally {
      setLoading(false);
    }
  }, []);

  // データ追加
  const addTodo = useCallback(async (todoData) => {
    try {
      const response = await fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(todoData)
      });
      const newTodo = await response.json();
      setTodos(prev => [...prev, newTodo]);
    } catch (error) {
      console.error('追加エラー:', error);
    }
  }, []);

  // データ削除
  const deleteTodo = useCallback(async (todoId) => {
    try {
      await fetch(`/api/todos/${todoId}`, { method: 'DELETE' });
      setTodos(prev => prev.filter(todo => todo.id !== todoId));
    } catch (error) {
      console.error('削除エラー:', error);
    }
  }, []);

  return (
    <div>
      <button onClick={fetchTodos} disabled={loading}>
        {loading ? '読み込み中...' : 'データを取得'}
      </button>
      
      <TodoList 
        todos={todos} 
        onAdd={addTodo} 
        onDelete={deleteTodo} 
      />
    </div>
  );
}
```

## Hooksの使い分け

| Hook | 使用場面 | API通信 | 主な目的 |
|------|----------|---------|----------|
| **useState** | コンポーネント内の状態管理 | × | 状態の保持 |
| **useCallback** | 関数の安定化・API通信 | ✅ 推奨 | 関数の再作成防止 |
| **useEffect** | DOM操作・タイマー | ❌ 非推奨 | 副作用の処理 |
| **useContext** | 深い階層でのデータ共有 | × | データの共有 |

## よくある間違い

### 1. useEffectでのAPI通信

```jsx
// ❌ 推奨されない
useEffect(() => {
  fetch('/api/users')
    .then(response => response.json())
    .then(data => setUsers(data));
}, []);

// ✅ 推奨される
const fetchUsers = useCallback(async () => {
  const response = await fetch('/api/users');
  const data = await response.json();
  setUsers(data);
}, []);

// 初回読み込みが必要な場合
useEffect(() => {
  fetchUsers();
}, [fetchUsers]);
```

### 2. useCallbackの依存配列

```jsx
// ❌ 間違い
const fetchUser = useCallback(async () => {
  const response = await fetch(`/api/users/${userId}`);
  // ...
}, []);  // userIdが依存配列にない

// ✅ 正しい
const fetchUser = useCallback(async () => {
  const response = await fetch(`/api/users/${userId}`);
  // ...
}, [userId]);  // userIdを依存配列に含める
```

## まとめ

### 学習の優先順位

1. **useState** - 必ず覚える（状態管理の基本）
2. **useCallback** - API通信と関数の安定化
3. **useContext** - アプリが大きくなったら覚える
4. **useEffect** - DOM操作とタイマーのみ、API通信には使わない

### 重要なポイント

- **useState**: コンポーネントの記憶装置
- **useCallback**: 関数の安定化 + API通信の推奨方法
- **useContext**: 遠い場所への情報共有
- **useEffect**: DOM操作とタイマーのみ（API通信は避ける）

### useCallbackとuseMemoの違い

- **useCallback**: 関数の再作成を防ぐ
- **useMemo**: 値の再計算を防ぐ（このチュートリアルでは扱いません）

**💡 覚え方**: 「API通信はuseCallback、DOM操作はuseEffect」と覚えましょう！

**⚠️ 注意**: 初心者の方は、まずuseStateをしっかり理解してから他のHooksに進むことをお勧めします。