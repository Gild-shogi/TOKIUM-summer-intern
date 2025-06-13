# コンポーネントとは何か？

## コンポーネントの基本概念

**コンポーネント** = 再利用可能なUI部品

Railsでいう部分テンプレート（partial）のより高機能版だと考えると理解しやすいです。

## 身近な例で理解する

### 例1：LEGO ブロック
```
LEGOブロック = コンポーネント
- 小さなパーツを組み合わせて大きな作品を作る
- 同じブロックを何度でも使える
- ブロックを変更すると、それを使っている全ての作品に反映される
```

### 例2：スマホアプリの画面
```
┌─────────────────────────────────┐
│ ヘッダーコンポーネント           │ ← 再利用可能
├─────────────────────────────────┤
│ ユーザーカードコンポーネント     │ ← 何個でも作れる
│ ユーザーカードコンポーネント     │ ← 同じ構造、違うデータ
│ ユーザーカードコンポーネント     │
├─────────────────────────────────┤
│ フッターコンポーネント           │ ← 再利用可能
└─────────────────────────────────┘
```

## Rails の部分テンプレートとの比較

### Rails の部分テンプレート
```erb
<!-- app/views/shared/_user_card.html.erb -->
<div class="user-card">
  <h3><%= user.name %></h3>
  <p><%= user.email %></p>
</div>

<!-- 使用側 -->
<%= render 'shared/user_card', user: @user %>
```

### React コンポーネント
```jsx
// UserCard.jsx
function UserCard({ user }) {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
}

// 使用側
<UserCard user={user} />
```

## Reactコンポーネントの特徴

### 1. 再利用性が高い

**同じコンポーネントを何度でも使える:**
```jsx
function App() {
  const users = [
    { id: 1, name: "田中太郎" },
    { id: 2, name: "佐藤花子" },
    { id: 3, name: "鈴木次郎" }
  ];

  return (
    <div>
      {users.map(user => (
        <UserCard key={user.id} user={user} />  // 同じコンポーネントを3回使用
      ))}
    </div>
  );
}
```

### 2. 独立性がある

**各コンポーネントは独立している:**
```jsx
function Counter() {
  const [count, setCount] = useState(0);  // このコンポーネント専用の状態

  return (
    <div>
      <p>カウント: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}

// 3つのCounterは別々にカウントを持つ
<Counter />  // カウント: 0
<Counter />  // カウント: 0  
<Counter />  // カウント: 0
```

### 3. データを受け取れる（props）

**親から子へデータを渡せる:**
```jsx
// 子コンポーネント
function Greeting({ name, age }) {
  return <h1>こんにちは、{age}歳の{name}さん！</h1>;
}

// 親コンポーネント
function App() {
  return (
    <div>
      <Greeting name="田中太郎" age={25} />
      <Greeting name="佐藤花子" age={30} />
    </div>
  );
}
```

## コンポーネントの種類

### 1. 関数コンポーネント（推奨）

```jsx
// シンプルな書き方
function Welcome(props) {
  return <h1>Hello, {props.name}!</h1>;
}

// 分割代入を使った書き方（推奨）
function Welcome({ name }) {
  return <h1>Hello, {name}!</h1>;
}

// アロー関数での書き方
const Welcome = ({ name }) => {
  return <h1>Hello, {name}!</h1>;
};
```

### 2. TypeScript の場合

```typescript
// インターフェースで型定義
interface WelcomeProps {
  name: string;
  age?: number;  // オプショナル
}

function Welcome({ name, age }: WelcomeProps) {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      {age && <p>年齢: {age}歳</p>}
    </div>
  );
}
```

## コンポーネントの構成要素

### 1. props（プロパティ）
```jsx
function UserCard({ user, showEmail = false }) {  // デフォルト値も設定可能
  return (
    <div>
      <h3>{user.name}</h3>
      {showEmail && <p>{user.email}</p>}
    </div>
  );
}

// 使用例
<UserCard user={user} showEmail={true} />
```

### 2. state（状態）
```jsx
function Counter() {
  const [count, setCount] = useState(0);  // コンポーネント内の状態

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

### 3. イベントハンドラ
```jsx
function Button() {
  const handleClick = () => {
    alert('ボタンがクリックされました！');
  };

  return <button onClick={handleClick}>クリック</button>;
}
```

## コンポーネント設計のベストプラクティス

### 1. 小さく、単一責任で

**❌ 悪い例（大きすぎる）:**
```jsx
function UserManagement() {
  // ユーザー一覧表示
  // ユーザー作成フォーム  
  // ユーザー編集フォーム
  // ユーザー削除機能
  // ... 100行以上のコード
}
```

**✅ 良い例（小さく分割）:**
```jsx
function UserManagement() {
  return (
    <div>
      <UserForm />           {/* ユーザー作成専用 */}
      <UserList />           {/* ユーザー一覧専用 */}
    </div>
  );
}

function UserList() {
  return (
    <div>
      {users.map(user => (
        <UserCard key={user.id} user={user} />  {/* ユーザーカード専用 */}
      ))}
    </div>
  );
}
```

### 2. 明確な命名

```jsx
// ✅ 良い命名（何をするコンポーネントか分かる）
function LoginForm() { ... }
function UserProfile() { ... }
function TodoItem() { ... }

// ❌ 悪い命名（何をするか分からない）
function Component1() { ... }
function MyComponent() { ... }
function Thing() { ... }
```

### 3. propsの型定義（TypeScript）

```typescript
// ✅ 明確な型定義
interface TodoItemProps {
  todo: {
    id: number;
    title: string;
    completed: boolean;
  };
  onToggle: (id: number) => void;
  onDelete: (id: number) => void;
}

function TodoItem({ todo, onToggle, onDelete }: TodoItemProps) {
  // ...
}
```

## コンポーネントの階層構造

### 親子関係の例

```jsx
function App() {                    // 祖父コンポーネント
  return (
    <div>
      <Header />                    // 叔父コンポーネント
      <UserManagement />            // 親コンポーネント
      <Footer />                    // 叔父コンポーネント
    </div>
  );
}

function UserManagement() {         // 親コンポーネント
  const [users, setUsers] = useState([]);
  
  return (
    <div>
      <UserForm onAdd={addUser} />  // 子コンポーネント
      <UserList users={users} />    // 子コンポーネント
    </div>
  );
}

function UserList({ users }) {      // 親コンポーネント
  return (
    <div>
      {users.map(user => (
        <UserCard key={user.id} user={user} />  // 子コンポーネント
      ))}
    </div>
  );
}
```

## まとめ

### コンポーネントとは
- **再利用可能なUI部品**
- LEGOブロックのように組み合わせてアプリを作る
- Railsの部分テンプレートの進化版

### 主な特徴
- **再利用性**: 同じコンポーネントを何度でも使える
- **独立性**: 各コンポーネントは独立した状態を持てる
- **データ受け渡し**: propsでデータを受け取れる

### 設計のポイント
- **小さく作る**: 一つのコンポーネントは一つの責任
- **明確な命名**: 何をするコンポーネントか分かる名前
- **型定義**: TypeScriptで明確な型を定義

### Rails との対応関係

| Rails | React |
|-------|-------|
| 部分テンプレート | コンポーネント |
| ローカル変数 | props |
| インスタンス変数 | state |
| Helper | カスタムフック |

**💡 覚え方**: 「再利用できるUI部品を作って、LEGOブロックのように組み合わせる」と考えましょう！