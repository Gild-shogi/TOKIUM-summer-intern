# Promiseとは何か？

## Promiseの基本概念

**Promise（プロミス）** = 「将来的に値が得られることを約束するオブジェクト」

JavaScriptで非同期処理を扱うための仕組みです。API通信のように「時間がかかる処理」を効率的に扱えます。

## 身近な例で理解する

### 例1：レストランでの注文

```
あなた: 「ハンバーガーセットください」
店員: 「承りました。番号札をお持ちください」 ← Promise作成
      「出来上がったらお呼びします」

番号札 = Promise
・まだ料理は完成していない
・でも将来的に料理がもらえることが約束されている
・他のことをしながら待てる
```

### 例2：宅配便の注文

```
あなた: 「商品を注文」
店舗: 「追跡番号をお送りします」 ← Promise作成

追跡番号 = Promise
・商品はまだ手元にない
・でも将来的に商品が届くことが約束されている
・追跡番号で状況を確認できる
```

## Promiseの3つの状態

### 1. Pending（待機中）

```javascript
const orderPromise = fetch('/api/order');
console.log(orderPromise); // Promise { <pending> }

// 状況: 注文は受け付けたが、まだ料理は作られていない
```

### 2. Fulfilled（成功）

```javascript
fetch('/api/order')
  .then(response => {
    console.log('注文が完了しました！', response);
  });

// 状況: 料理が完成して、お客さんに渡された
```

### 3. Rejected（失敗）

```javascript
fetch('/api/order')
  .catch(error => {
    console.log('注文に失敗しました', error);
  });

// 状況: 材料切れで料理が作れなかった
```

## 従来のコールバック vs Promise

### 従来のコールバック方式（複雑）

```javascript
// ❌ コールバック地獄
getUser(userId, function(user) {
  getProfile(user.id, function(profile) {
    getSettings(profile.id, function(settings) {
      updateDisplay(settings, function(result) {
        console.log('全て完了');
      });
    });
  });
});
```

### Promise方式（分かりやすい）

```javascript
// ✅ 見やすい連鎖
getUser(userId)
  .then(user => getProfile(user.id))
  .then(profile => getSettings(profile.id))
  .then(settings => updateDisplay(settings))
  .then(result => console.log('全て完了'))
  .catch(error => console.log('エラー:', error));
```

## Promiseの基本的な使い方

### 1. Promiseを作成する

```javascript
const myPromise = new Promise((resolve, reject) => {
  // 時間のかかる処理をシミュレート
  setTimeout(() => {
    const success = Math.random() > 0.5;
    
    if (success) {
      resolve('成功しました！'); // 成功時
    } else {
      reject('失敗しました...'); // 失敗時
    }
  }, 2000);
});
```

### 2. Promiseを使用する

```javascript
myPromise
  .then(result => {
    console.log('成功:', result); // '成功しました！'
  })
  .catch(error => {
    console.log('失敗:', error); // '失敗しました...'
  })
  .finally(() => {
    console.log('処理が完了しました'); // 成功・失敗関係なく実行
  });
```

## API通信でのPromise

### fetch APIの戻り値はPromise

```javascript
// fetchはPromiseを返す
const apiPromise = fetch('/api/users');
console.log(apiPromise); // Promise { <pending> }

// 成功時の処理
apiPromise
  .then(response => response.json()) // レスポンスをJSONに変換
  .then(data => {
    console.log('ユーザーデータ:', data);
  })
  .catch(error => {
    console.log('エラー:', error);
  });
```

### より実践的な例

```javascript
function fetchUsers() {
  return fetch('/api/users')
    .then(response => {
      // HTTPステータスをチェック
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      return response.json();
    })
    .then(data => {
      console.log('取得したユーザー:', data);
      return data; // 次のthenに渡される
    })
    .catch(error => {
      console.error('ユーザーの取得に失敗:', error);
      throw error; // エラーを再度投げる
    });
}

// 使用方法
fetchUsers()
  .then(users => {
    console.log('ユーザー数:', users.length);
  })
  .catch(error => {
    console.log('最終的なエラーハンドリング:', error);
  });
```

## Promiseの連鎖（チェーン）

### 複数のAPI呼び出しを順番に実行

```javascript
// ユーザー情報 → プロフィール → 設定 の順で取得
fetch('/api/user/1')
  .then(response => response.json())
  .then(user => {
    console.log('ユーザー取得完了:', user.name);
    return fetch(`/api/profile/${user.id}`); // 次のPromiseを返す
  })
  .then(response => response.json())
  .then(profile => {
    console.log('プロフィール取得完了:', profile.bio);
    return fetch(`/api/settings/${profile.id}`);
  })
  .then(response => response.json())
  .then(settings => {
    console.log('設定取得完了:', settings);
  })
  .catch(error => {
    console.log('どこかでエラーが発生:', error);
  });
```

## Promise.all()で並列実行

### 複数のAPIを同時に呼び出す

```javascript
// 全て同時に開始
const userPromise = fetch('/api/users');
const postPromise = fetch('/api/posts');
const commentPromise = fetch('/api/comments');

// 全て完了するまで待つ
Promise.all([userPromise, postPromise, commentPromise])
  .then(responses => {
    // 全てのレスポンスをJSONに変換
    return Promise.all(responses.map(response => response.json()));
  })
  .then(([users, posts, comments]) => {
    console.log('全てのデータが取得できました');
    console.log('ユーザー:', users);
    console.log('投稿:', posts);
    console.log('コメント:', comments);
  })
  .catch(error => {
    console.log('いずれかの取得に失敗:', error);
  });
```

**身近な例**: 複数の料理を同時に注文して、全部揃ったら配膳する

## よくある間違いとトラブルシューティング

### 1. thenでPromiseを返し忘れ

```javascript
// ❌ 間違い
fetch('/api/user')
  .then(response => {
    response.json(); // Promiseを返していない
  })
  .then(data => {
    console.log(data); // undefined になる
  });

// ✅ 正しい
fetch('/api/user')
  .then(response => {
    return response.json(); // Promiseを返す
  })
  .then(data => {
    console.log(data); // 正しいデータが表示される
  });
```

### 2. エラーハンドリングの位置

```javascript
// ❌ 間違い（各thenでエラーハンドリング）
fetch('/api/user')
  .then(response => response.json())
  .catch(error => console.log('JSON変換エラー', error))
  .then(data => console.log(data))
  .catch(error => console.log('データ処理エラー', error));

// ✅ 正しい（最後にまとめてエラーハンドリング）
fetch('/api/user')
  .then(response => {
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return response.json();
  })
  .then(data => {
    console.log(data);
  })
  .catch(error => {
    console.log('全体のエラーハンドリング:', error);
  });
```

### 3. Promiseの入れ子

```javascript
// ❌ 間違い（Promise地獄）
fetch('/api/user')
  .then(response => {
    return response.json()
      .then(user => {
        return fetch(`/api/profile/${user.id}`)
          .then(response => {
            return response.json();
          });
      });
  });

// ✅ 正しい（フラットなチェーン）
fetch('/api/user')
  .then(response => response.json())
  .then(user => fetch(`/api/profile/${user.id}`))
  .then(response => response.json())
  .then(profile => console.log(profile));
```

## ReactでのPromise使用例

```javascript
import { useState, useCallback } from 'react';

function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchUsers = useCallback(() => {
    setLoading(true);
    setError(null);

    fetch('/api/users')
      .then(response => {
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        return response.json();
      })
      .then(data => {
        setUsers(data);
      })
      .catch(error => {
        setError(error.message);
      })
      .finally(() => {
        setLoading(false);
      });
  }, []);

  return (
    <div>
      <button onClick={fetchUsers} disabled={loading}>
        {loading ? '読み込み中...' : 'ユーザーを取得'}
      </button>
      
      {error && <p style={{color: 'red'}}>エラー: {error}</p>}
      
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

## まとめ

### Promiseとは
- **将来の値を表すオブジェクト**
- 非同期処理を分かりやすく書ける
- 3つの状態：待機中、成功、失敗

### 主な使用場面
- API通信（fetch）
- ファイル読み込み
- タイマー処理
- データベースアクセス

### 重要なポイント
1. **then()で成功時の処理**を書く
2. **catch()でエラー処理**を書く
3. **finally()で共通処理**を書く
4. **チェーンで連続処理**ができる
5. **Promise.all()で並列処理**ができる

### Rails との違い

| Rails | JavaScript Promise |
|-------|-------------------|
| 同期的処理 | 非同期処理 |
| 順番に実行 | 並列実行可能 |
| サーバー側 | ブラウザ側 |

**💡 覚え方**: 「Promiseは将来の約束手形。いつか値がもらえることが保証されている」と考えましょう！

**🔗 次のステップ**: Promiseをより簡潔に書けるasync/awaitについて学習しましょう。