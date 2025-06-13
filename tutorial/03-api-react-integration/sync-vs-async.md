# 同期処理と非同期処理の違い（async/await）

## 同期処理と非同期処理とは？

### 同期処理（Synchronous）
**一つずつ順番に処理を実行する方式**

### 非同期処理（Asynchronous）  
**複数の処理を並行して実行できる方式**

## 身近な例で理解する

### 例1：料理の準備

**同期処理の場合:**
```
1. 米を洗う（5分）
2. 米を炊く（30分） ← 炊けるまで待つ
3. 野菜を切る（10分）
4. 肉を焼く（15分）

合計時間: 60分
```

**非同期処理の場合:**
```
1. 米を洗う（5分）
2. 米を炊き始める（30分かかる）
3. 炊いている間に野菜を切る（10分） ← 並行作業
4. さらに肉を焼く（15分） ← 並行作業

合計時間: 35分（25分短縮！）
```

### 例2：メールの送信

**同期処理:**
```
あなた: メール送信ボタンを押す
システム: メール送信中...（3秒）
あなた: 送信完了まで画面が固まって何もできない
システム: 送信完了！
あなた: やっと他の作業ができる
```

**非同期処理:**
```
あなた: メール送信ボタンを押す
システム: メール送信開始（バックグラウンド）
あなた: 送信中でも他の作業を続けられる
システム: 送信完了通知
```

## JavaScriptでの同期 vs 非同期

### 同期処理の例

```javascript
function syncExample() {
  console.log('1. 処理開始');
  
  // 重い処理をシミュレート（3秒ブロック）
  const start = Date.now();
  while (Date.now() - start < 3000) {
    // 3秒間何もしない（CPUを占有）
  }
  
  console.log('2. 重い処理完了');
  console.log('3. 次の処理');
}

syncExample();
// 実行結果:
// 1. 処理開始
// (3秒待つ)
// 2. 重い処理完了  
// 3. 次の処理
```

### 非同期処理の例

```javascript
function asyncExample() {
  console.log('1. 処理開始');
  
  // 非同期で重い処理を実行
  setTimeout(() => {
    console.log('2. 重い処理完了');
  }, 3000);
  
  console.log('3. 次の処理');
}

asyncExample();
// 実行結果:
// 1. 処理開始
// 3. 次の処理
// (3秒後)
// 2. 重い処理完了
```

## async/await の基本

### async/await とは？

**async/await** = Promiseをより分かりやすく書くための構文糖衣

- **async**: 関数を非同期関数として定義
- **await**: Promiseの完了を待つ

### 従来のPromise vs async/await

**従来のPromise書き方:**
```javascript
function fetchUserData() {
  return fetch('/api/user')
    .then(response => response.json())
    .then(user => {
      console.log('ユーザー:', user);
      return fetch(`/api/posts/${user.id}`);
    })
    .then(response => response.json())
    .then(posts => {
      console.log('投稿:', posts);
      return posts;
    })
    .catch(error => {
      console.error('エラー:', error);
    });
}
```

**async/await書き方:**
```javascript
async function fetchUserData() {
  try {
    const userResponse = await fetch('/api/user');
    const user = await userResponse.json();
    console.log('ユーザー:', user);
    
    const postsResponse = await fetch(`/api/posts/${user.id}`);
    const posts = await postsResponse.json();
    console.log('投稿:', posts);
    
    return posts;
  } catch (error) {
    console.error('エラー:', error);
  }
}
```

## async/await の基本的な使い方

### 1. async関数の定義

```javascript
// 関数宣言
async function getData() {
  return 'データ';
}

// アロー関数
const getData = async () => {
  return 'データ';
};

// async関数は常にPromiseを返す
console.log(getData()); // Promise { 'データ' }
```

### 2. await の使用

```javascript
async function example() {
  // Promiseの完了を待つ
  const response = await fetch('/api/data');
  const data = await response.json();
  
  console.log('データ取得完了:', data);
  return data;
}

// async関数を呼び出す場合
example()
  .then(result => console.log('結果:', result))
  .catch(error => console.error('エラー:', error));
```

### 3. エラーハンドリング

```javascript
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('データ取得に失敗:', error);
    throw error; // エラーを再度投げる
  }
}
```

## 実践的なAPI通信の例

### 1. 単一のAPI呼び出し

```javascript
async function getUsers() {
  try {
    const response = await fetch('/api/users');
    const users = await response.json();
    return users;
  } catch (error) {
    console.error('ユーザー取得エラー:', error);
    return [];
  }
}

// 使用方法
const users = await getUsers();
console.log('ユーザー一覧:', users);
```

### 2. 複数のAPI呼び出し（順次実行）

```javascript
async function getUserWithPosts(userId) {
  try {
    // まずユーザー情報を取得
    const userResponse = await fetch(`/api/users/${userId}`);
    const user = await userResponse.json();
    
    // 次にその人の投稿を取得
    const postsResponse = await fetch(`/api/users/${userId}/posts`);
    const posts = await postsResponse.json();
    
    return { user, posts };
  } catch (error) {
    console.error('データ取得エラー:', error);
    throw error;
  }
}
```

### 3. 複数のAPI呼び出し（並列実行）

```javascript
async function getAllData() {
  try {
    // 3つのAPIを同時に呼び出し
    const [usersResponse, postsResponse, commentsResponse] = await Promise.all([
      fetch('/api/users'),
      fetch('/api/posts'),
      fetch('/api/comments')
    ]);
    
    // 全てのレスポンスをJSONに変換
    const [users, posts, comments] = await Promise.all([
      usersResponse.json(),
      postsResponse.json(),
      commentsResponse.json()
    ]);
    
    return { users, posts, comments };
  } catch (error) {
    console.error('データ取得エラー:', error);
    throw error;
  }
}
```

## ReactでのAsync/Await使用例

### useCallbackでAPI通信

```javascript
import { useState, useCallback } from 'react';

function UserManagement() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  // ユーザー一覧取得
  const fetchUsers = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      
      const response = await fetch('/api/users');
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const userData = await response.json();
      setUsers(userData);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, []);

  // ユーザー作成
  const createUser = useCallback(async (userData) => {
    try {
      const response = await fetch('/api/users', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(userData),
      });
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const newUser = await response.json();
      setUsers(prev => [...prev, newUser]);
      
      return newUser;
    } catch (err) {
      console.error('ユーザー作成エラー:', err);
      throw err;
    }
  }, []);

  // ユーザー削除
  const deleteUser = useCallback(async (userId) => {
    try {
      const response = await fetch(`/api/users/${userId}`, {
        method: 'DELETE',
      });
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      setUsers(prev => prev.filter(user => user.id !== userId));
    } catch (err) {
      console.error('ユーザー削除エラー:', err);
      throw err;
    }
  }, []);

  return (
    <div>
      {error && <p style={{color: 'red'}}>エラー: {error}</p>}
      
      <button onClick={fetchUsers} disabled={loading}>
        {loading ? '読み込み中...' : 'ユーザー一覧を取得'}
      </button>
      
      <ul>
        {users.map(user => (
          <li key={user.id}>
            {user.name}
            <button onClick={() => deleteUser(user.id)}>削除</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## よくある間違いと対処法

### 1. await を忘れる

```javascript
// ❌ 間違い
async function badExample() {
  const response = fetch('/api/data'); // awaitを忘れている
  const data = response.json(); // responseはまだPromise
  console.log(data); // Promise { <pending> }
}

// ✅ 正しい
async function goodExample() {
  const response = await fetch('/api/data');
  const data = await response.json();
  console.log(data); // 実際のデータ
}
```

### 2. async関数の中でしかawaitを使えない

```javascript
// ❌ 間違い
function badExample() {
  const data = await fetch('/api/data'); // エラー！async関数の外
}

// ✅ 正しい
async function goodExample() {
  const data = await fetch('/api/data'); // OK
}
```

### 3. エラーハンドリングを忘れる

```javascript
// ❌ 間違い
async function badExample() {
  const response = await fetch('/api/data');
  const data = await response.json(); // エラーが起きたら例外が発生
  return data;
}

// ✅ 正しい
async function goodExample() {
  try {
    const response = await fetch('/api/data');
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('API呼び出しエラー:', error);
    throw error;
  }
}
```

### 4. 不要なawaitの使用

```javascript
// ❌ 非効率（順次実行）
async function badExample() {
  const user = await fetch('/api/user').then(r => r.json());
  const posts = await fetch('/api/posts').then(r => r.json());
  const comments = await fetch('/api/comments').then(r => r.json());
  
  return { user, posts, comments };
}

// ✅ 効率的（並列実行）
async function goodExample() {
  const [user, posts, comments] = await Promise.all([
    fetch('/api/user').then(r => r.json()),
    fetch('/api/posts').then(r => r.json()),
    fetch('/api/comments').then(r => r.json())
  ]);
  
  return { user, posts, comments };
}
```

## 同期 vs 非同期の使い分け

### 同期処理を使う場面

```javascript
// 簡単な計算
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// 配列の変換
function formatUsers(users) {
  return users.map(user => ({
    ...user,
    displayName: `${user.firstName} ${user.lastName}`
  }));
}
```

### 非同期処理を使う場面

```javascript
// API通信
async function fetchData() {
  const response = await fetch('/api/data');
  return response.json();
}

// ファイル操作（Node.js）
async function readFile(filename) {
  const fs = require('fs').promises;
  return fs.readFile(filename, 'utf8');
}

// タイマー
async function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

## まとめ

### 同期 vs 非同期

| 項目 | 同期処理 | 非同期処理 |
|------|----------|------------|
| **実行方式** | 順番に一つずつ | 並列実行可能 |
| **待機時間** | 完了まで待つ | 他の作業も可能 |
| **使用場面** | 計算、データ変換 | API通信、I/O操作 |
| **パフォーマンス** | 遅い（待機時間あり） | 速い（並列処理） |

### async/awaitのメリット

1. **読みやすい**: 同期処理のような書き方
2. **エラーハンドリング**: try/catchが使える
3. **デバッグしやすい**: スタックトレースが分かりやすい

### 重要なポイント

- **await は async関数内でのみ使用可能**
- **async関数は常にPromiseを返す**
- **エラーハンドリングは try/catch を使用**
- **並列処理には Promise.all() を活用**

### Rails との違い

| Rails | JavaScript async/await |
|-------|------------------------|
| 同期的なリクエスト処理 | 非同期処理 |
| サーバー側で順次実行 | ブラウザ側で並列実行可能 |
| レスポンス完了まで待機 | 他の処理も並行実行 |

**💡 覚え方**: 「async/awaitは非同期処理を同期処理のように書ける魔法の構文」と考えましょう！

**⚠️ 注意**: awaitを使いすぎると逆に遅くなることもあります。必要に応じてPromise.all()で並列処理しましょう。