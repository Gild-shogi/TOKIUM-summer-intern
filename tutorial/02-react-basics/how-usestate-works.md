# useStateの内部動作とstate管理

## useState とは何か？

**useState** = Reactコンポーネントに「記憶」を持たせるための仕組み

Railsでは、インスタンス変数やセッションでデータを保持しますが、Reactコンポーネントでは`useState`を使って状態を管理します。

## 基本的な使い方の復習

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  //     ↑       ↑           ↑
  //   現在の値  更新関数    初期値

  return (
    <div>
      <p>カウント: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

## useStateの内部で何が起きているのか？

### 1. 初回レンダリング時

```jsx
// 初回実行時
const [count, setCount] = useState(0);

// Reactの内部では...
// 1. count = 0 を保存
// 2. setCount関数を作成
// 3. [0, setCount関数] を返す
```

**React内部のイメージ:**
```javascript
// React内部の仮想的なコード
const componentState = {
  value: 0,  // 初期値を保存
  setValue: function(newValue) {
    this.value = newValue;
    // コンポーネントを再レンダリング！
  }
};
```

### 2. 状態更新時

```jsx
// ボタンクリック時
setCount(count + 1);

// Reactの内部では...
// 1. 新しい値（1）を受け取る
// 2. 内部の状態を更新
// 3. コンポーネントを再レンダリング
// 4. 再レンダリング時、countは新しい値（1）になる
```

### 3. 再レンダリング時

```jsx
// 2回目のレンダリング
const [count, setCount] = useState(0);

// Reactの内部では...
// 1. 「あ、このコンポーネントは既に状態を持ってるな」
// 2. 初期値（0）は無視
// 3. 保存されている値（1）を返す
// 4. [1, setCount関数] を返す
```

## 身近な例で理解する

### 例1：ノートに書いた数字

```
あなた = Reactコンポーネント
ノート = useState の内部状態
鉛筆 = setCount 関数

1. 最初にノートに「0」と書く
2. 「+1」ボタンを押す = 鉛筆で「1」に書き換える
3. 書き換わったノートを見る = 新しい値が表示される
```

### 例2：銀行口座

```
useState(1000) = 口座開設（初期残高1000円）

setMoney(1500) = 入金操作（残高を1500円に更新）
↓
残高が1500円に更新され、画面に反映される
```

## 複数のstateを持つ場合

```jsx
function UserProfile() {
  const [name, setName] = useState('');
  const [age, setAge] = useState(0);
  const [email, setEmail] = useState('');

  // Reactは各stateを別々に管理している
}
```

**React内部のイメージ:**
```javascript
const componentStates = [
  { value: '', setValue: setName },    // 1番目のuseState
  { value: 0, setValue: setAge },      // 2番目のuseState  
  { value: '', setValue: setEmail }    // 3番目のuseState
];
```

## stateの更新が「非同期」である理由

### 即座に反映されない例

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    console.log('クリック前:', count);  // 0
    setCount(count + 1);
    console.log('クリック後:', count);  // まだ 0！
  };

  console.log('レンダリング時:', count);  // 再レンダリング時に 1

  return <button onClick={handleClick}>+1</button>;
}
```

### なぜこうなるのか？

```
1. setCount(count + 1) 実行
   ↓
2. Reactが「更新予約」を受け取る
   ↓
3. 現在の処理が全て終わる
   ↓
4. Reactが再レンダリングを実行
   ↓
5. 新しい値が反映される
```

**身近な例:**
```
レストランでの注文
1. 「ハンバーガーください」（setCount実行）
2. 店員「承りました」（更新予約）
3. あなたは席で待つ（現在の処理継続）
4. 料理が完成（再レンダリング）
5. 「お待たせしました」（新しい値が反映）
```

## オブジェクトのstateを更新する場合

### ❌ 間違った更新方法

```jsx
const [user, setUser] = useState({ name: '', age: 0 });

// これは動かない！
const updateName = (newName) => {
  user.name = newName;  // stateを直接変更している
  setUser(user);        // 同じオブジェクトなので変更が検出されない
};
```

### ✅ 正しい更新方法

```jsx
const [user, setUser] = useState({ name: '', age: 0 });

// 新しいオブジェクトを作成
const updateName = (newName) => {
  setUser({
    ...user,        // 既存の値をコピー
    name: newName   // 変更したい部分だけ上書き
  });
};
```

## 配列のstateを更新する場合

### ❌ 間違った更新方法

```jsx
const [todos, setTodos] = useState([]);

// これは動かない！
const addTodo = (newTodo) => {
  todos.push(newTodo);  // 配列を直接変更
  setTodos(todos);      // 同じ配列なので変更が検出されない
};
```

### ✅ 正しい更新方法

```jsx
const [todos, setTodos] = useState([]);

// 新しい配列を作成
const addTodo = (newTodo) => {
  setTodos([...todos, newTodo]);  // 既存の配列をコピーして新しい要素を追加
};

// 削除の場合
const removeTodo = (todoId) => {
  setTodos(todos.filter(todo => todo.id !== todoId));  // 新しい配列を作成
};
```

## 関数型更新

### 現在の値に基づいて更新する場合

```jsx
const [count, setCount] = useState(0);

// ❌ 同時に複数回実行されると期待通りにならない
const increment = () => {
  setCount(count + 1);
  setCount(count + 1);  // これは count + 1 になる（count + 2 ではない）
};

// ✅ 関数型更新を使用
const increment = () => {
  setCount(prev => prev + 1);  // 前の値に基づいて計算
  setCount(prev => prev + 1);  // 前の値（既に+1された値）に基づいて計算
};
```

## パフォーマンスと再レンダリング

### stateが変わると何が起きるか

```jsx
function App() {
  const [count, setCount] = useState(0);
  
  console.log('App コンポーネントがレンダリングされました');
  
  return (
    <div>
      <p>{count}</p>
      <ChildComponent />  {/* 親が再レンダリングされると子も再レンダリング */}
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}

function ChildComponent() {
  console.log('Child コンポーネントがレンダリングされました');
  return <div>子コンポーネント</div>;
}
```

**結果:**
```
ボタンクリック時
↓
1. App コンポーネントがレンダリングされました
2. Child コンポーネントがレンダリングされました
```

## よくある間違いと対処法

### 1. stateの直接変更

```jsx
// ❌ 間違い
const [items, setItems] = useState([1, 2, 3]);
items[0] = 999;  // 直接変更
```

**何が起こっているか:**
```
1. items[0] = 999 実行
   ↓
2. 配列の中身は変わるが、配列自体は同じオブジェクト
   ↓
3. React「あれ？同じ配列だな。変更されてないな」
   ↓
4. 再レンダリングが発生しない
   ↓
5. 画面は更新されない（内部的には値は変わっているが表示されない）
```

```jsx
// ✅ 正しい
setItems(items.map((item, index) => index === 0 ? 999 : item));
```

**何が起こっているか:**
```
1. items.map() で新しい配列を作成
   ↓
2. 新しい配列には変更された値が入っている
   ↓
3. setItems(新しい配列) 実行
   ↓
4. React「おっ！新しい配列だ。変更されたな」
   ↓
5. 再レンダリングが発生
   ↓
6. 画面が更新される
```

### 2. 同期的な更新を期待する

```jsx
// ❌ 間違い
const handleSubmit = () => {
  setName('田中太郎');
  console.log(name);  // まだ古い値
  
  // APIに送信する場合
  api.updateUser({ name: name });  // 古い値が送信される
};
```

**何が起こっているか:**
```
1. setName('田中太郎') 実行
   ↓
2. React「更新予約を受け付けました」
   ↓
3. console.log(name) 実行 ← まだ更新されていない！
   ↓
4. api.updateUser({ name: name }) 実行 ← 古い値を送信！
   ↓
5. 関数の処理が全て終了
   ↓
6. Reactが再レンダリングを実行
   ↓
7. やっと name が '田中太郎' に更新される
```

```jsx
// ✅ 正しい
const handleSubmit = () => {
  const newName = '田中太郎';
  setName(newName);
  
  // 新しい値を使用
  api.updateUser({ name: newName });
};
```

**何が起こっているか:**
```
1. const newName = '田中太郎' ← 新しい値を変数に保存
   ↓
2. setName(newName) 実行 ← 更新予約
   ↓
3. api.updateUser({ name: newName }) 実行 ← 正しい新しい値を送信！
   ↓
4. 関数の処理が終了
   ↓
5. Reactが再レンダリングを実行
   ↓
6. name が '田中太郎' に更新される
```

### 3. オブジェクトの一部だけ更新

```jsx
// ❌ 間違い
const [user, setUser] = useState({ name: '田中', email: 'tanaka@example.com' });

const updateName = () => {
  setUser({ name: '佐藤' });  // emailが消える！
};
```

**何が起こっているか:**
```
1. setUser({ name: '佐藤' }) 実行
   ↓
2. React「新しいオブジェクトですね」
   ↓
3. user の値が { name: '佐藤' } に完全に置き換わる
   ↓
4. email プロパティが消失！
   ↓
5. 再レンダリング後、user = { name: '佐藤' } になる
```

```jsx
// ✅ 正しい
const updateName = () => {
  setUser({ ...user, name: '佐藤' });
};
```

**何が起こっているか:**
```
1. { ...user, name: '佐藤' } を実行
   ↓
2. まず ...user で既存の値を展開: { name: '田中', email: 'tanaka@example.com' }
   ↓
3. 次に name: '佐藤' で name を上書き: { name: '佐藤', email: 'tanaka@example.com' }
   ↓
4. 新しいオブジェクトが作成される
   ↓
5. setUser(新しいオブジェクト) 実行
   ↓
6. React「新しいオブジェクトですね」
   ↓
7. user の値が { name: '佐藤', email: 'tanaka@example.com' } に更新される
```

## まとめ

### useStateの仕組み
- Reactが内部で状態を管理している
- 状態が変わると自動的に再レンダリングされる
- 更新は非同期で行われる

### 重要なルール
1. **stateを直接変更してはいけない**
2. **オブジェクトや配列は新しいものを作成する**
3. **更新は非同期なので即座に反映されない**
4. **関数型更新を使って安全に更新する**

### Rails との違い

| Rails | React useState |
|-------|----------------|
| インスタンス変数 | state |
| 直接代入可能 | setter関数が必要 |
| ページ遷移で状態リセット | コンポーネント削除まで保持 |

**💡 覚え方**: 「useStateは魔法の箱。中身を変えたい時は、必ず専用の鍵（setter関数）を使う」と考えましょう！