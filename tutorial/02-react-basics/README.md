# React基礎編

Rails経験者がReactを学ぶための基礎教材です。Railsの概念と対比しながら、Reactの基本を理解していきましょう。

> **💡 学習のコツ**
> 
> Reactは新しい概念がたくさん出てきます。完璧に理解しようとせず、まずは動かしてみることから始めましょう！
> - 分からない部分は一旦飛ばして、後で戻ってきても大丈夫です
> - 詰まったときは、メンターやChatGPT/Claude等のLLMに質問してみましょう
> - Railsとの違いを意識しながら学習すると理解しやすいです

## 補足資料

理解が難しい概念については、以下の補足資料を参照してください：

- 🧩 **[コンポーネントとは何か？](./what-is-component.md)** - Reactの基本構成要素について
- 🔄 **[useStateの内部動作](./how-usestate-works.md)** - state管理の仕組みについて
- 🪝 **[React Hooks 一覧](./react-hooks-overview.md)** - よく使用するHooksの説明

## 学習目標

- Reactの基本概念（コンポーネント、JSX、props、state）を理解する
- RailsのViewとReactコンポーネントの違いを把握する
- 基本的なReactアプリケーションを作成できるようになる
- イベントハンドリングとstate管理を理解する

## 1. React vs Rails View の比較

### Rails View（ERB）の場合

```erb
<!-- app/views/users/index.html.erb -->
<h1>ユーザー一覧</h1>
<% @users.each do |user| %>
  <div class="user-card">
    <h3><%= user.name %></h3>
    <p><%= user.email %></p>
  </div>
<% end %>
```

### React Component の場合

```jsx
// UserList.jsx
function UserList({ users }) {
  return (
    <div>
      <h1>ユーザー一覧</h1>
      {users.map(user => (
        <div key={user.id} className="user-card">
          <h3>{user.name}</h3>
          <p>{user.email}</p>
        </div>
      ))}
    </div>
  );
}
```

### 主な違い

| 項目 | Rails View | React Component |
|------|------------|-----------------|
| **言語** | ERB (HTML + Ruby) | JSX (HTML + JavaScript) |
| **データ** | インスタンス変数 (@users) | props として受け取り |
| **再利用性** | 低い（部分テンプレート必要） | 高い（そのまま再利用可能） |
| **動的更新** | ページリロード必要 | リアルタイム更新可能 |

## 2. 現在のReactプロジェクト構成を理解する

### ディレクトリ構造

```
front/
├── src/
│   ├── App.tsx              # メインアプリケーション
│   ├── main.tsx            # エントリーポイント
│   ├── App.css             # アプリケーションのスタイル
│   └── index.css           # グローバルスタイル
├── public/                 # 静的ファイル
├── package.json           # 依存関係
└── vite.config.ts         # ビルド設定
```

### 現在のコードを確認

```bash
# メインアプリケーションを確認
cat front/src/App.tsx
```

## 3. JSX の基本

### JSX とは？

- JavaScript の拡張構文
- HTML に似た構文でUIを記述
- 最終的にJavaScriptのコードに変換される

### JSX の基本ルール

```jsx
// 1. 必ず1つの親要素で囲む
function Component() {
  return (
    <div>  {/* 親要素 */}
      <h1>タイトル</h1>
      <p>内容</p>
    </div>
  );
}

// 2. JavaScript式は{}で囲む
function Greeting({ name }) {
  const message = "こんにちは";
  return <h1>{message}、{name}さん！</h1>;
}

// 3. classはclassNameを使用
function Card() {
  return <div className="card">カード</div>;
}

// 4. 要素のリスト表示にはkey属性が必要
function List({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

## 4. コンポーネントの基本

> **🧩 補足**: コンポーネントについて詳しく知りたい場合は **[コンポーネントとは何か？](./what-is-component.md)** を参照してください。

### 関数コンポーネント（推奨）

```jsx
// 基本的な関数コンポーネント
function Welcome(props) {
  return <h1>Hello, {props.name}!</h1>;
}

// アロー関数での書き方
const Welcome = (props) => {
  return <h1>Hello, {props.name}!</h1>;
}

// 分割代入を使った書き方（推奨）
function Welcome({ name }) {
  return <h1>Hello, {name}!</h1>;
}
```

### プロパティ（props）

Rails の部分テンプレートにローカル変数を渡すのと似ています：

```ruby
# Rails: 部分テンプレート呼び出し
<%= render 'user_card', user: @user %>
```

```jsx
// React: コンポーネント呼び出し
<UserCard user={user} />
```

## 5. State（状態）管理

> **🔄 補足**: useStateの仕組みについて詳しく知りたい場合は **[useStateの内部動作](./how-usestate-works.md)** を参照してください。

### useState フック

Railsでは状態はサーバー側で管理されますが、Reactでは各コンポーネントが状態を持てます：

```jsx
import { useState } from 'react';

function Counter() {
  // state の宣言: [現在の値, 更新関数] = useState(初期値)
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>カウント: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        +1
      </button>
      <button onClick={() => setCount(count - 1)}>
        -1
      </button>
    </div>
  );
}
```

### 複雑なstate

```jsx
function UserForm() {
  const [user, setUser] = useState({
    name: '',
    email: ''
  });

  const handleNameChange = (e) => {
    setUser({ ...user, name: e.target.value });
  };

  const handleEmailChange = (e) => {
    setUser({ ...user, email: e.target.value });
  };

  return (
    <form>
      <input 
        type="text" 
        value={user.name}
        onChange={handleNameChange}
        placeholder="名前"
      />
      <input 
        type="email" 
        value={user.email}
        onChange={handleEmailChange}
        placeholder="メールアドレス"
      />
      <p>名前: {user.name}</p>
      <p>メール: {user.email}</p>
    </form>
  );
}
```

## 6. イベントハンドリング

### 基本的なイベント処理

```jsx
function ButtonExample() {
  const handleClick = () => {
    alert('ボタンがクリックされました！');
  };

  const handleSubmit = (e) => {
    e.preventDefault(); // フォームのデフォルト送信を防ぐ
    console.log('フォームが送信されました');
  };

  return (
    <div>
      <button onClick={handleClick}>
        クリック
      </button>
      
      <form onSubmit={handleSubmit}>
        <input type="text" />
        <button type="submit">送信</button>
      </form>
    </div>
  );
}
```

## 7. 実際にコンポーネントを作ってみよう

### Step 1: 開発サーバーの起動

```bash
# Docker環境の場合（推奨）
docker compose up -d

# ブラウザで http://localhost:8080 を開く
# ホットリロードが効くので、コード変更は自動で反映されます
```

### Step 2: 新しいコンポーネントを作成

**重要**: ファイルの作成はVSCodeで行うことを推奨します（Dockerでファイルを作成するとパーミッションエラーが発生することがあります）

VSCodeで `front/src/components/UserCard.tsx` を作成:

```jsx
interface User {
  id: number;
  name: string;
  email: string;
}

interface UserCardProps {
  user: User;
}

function UserCard({ user }: UserCardProps) {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={() => alert(`${user.name}さんの詳細`)}>
        詳細
      </button>
    </div>
  );
}

export default UserCard;
```

### Step 3: CSS を追加

VSCodeで `front/src/components/UserCard.css` を作成:

```css
.user-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;
  margin: 8px 0;
  background-color: #f9f9f9;
}

.user-card h3 {
  margin: 0 0 8px 0;
  color: #333;
}

.user-card p {
  margin: 0 0 12px 0;
  color: #666;
}

.user-card button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}

.user-card button:hover {
  background-color: #0056b3;
}
```

### Step 4: App.tsx で使用

VSCodeで `front/src/App.tsx` を更新:

```jsx
import { useState } from 'react';
import UserCard from './components/UserCard';
import './App.css';

interface User {
  id: number;
  name: string;
  email: string;
}

function App() {
  const [users] = useState<User[]>([
    { id: 1, name: "田中太郎", email: "tanaka@example.com" },
    { id: 2, name: "佐藤花子", email: "sato@example.com" },
    { id: 3, name: "鈴木次郎", email: "suzuki@example.com" }
  ]);

  return (
    <div className="App">
      <header className="App-header">
        <h1>React ユーザー管理</h1>
        <div className="user-list">
          {users.map(user => (
            <UserCard key={user.id} user={user} />
          ))}
        </div>
      </header>
    </div>
  );
}

export default App;
```

## 8. TypeScript について

このプロジェクトではTypeScriptを使用しています：

### TypeScript の利点

- **型安全**: コンパイル時にエラーを検出
- **自動補完**: IDEでの開発効率向上
- **保守性**: 大規模プロジェクトでの品質向上

### 基本的な型定義

```typescript
// インターフェース定義
interface User {
  id: number;
  name: string;
  email: string;
  age?: number; // オプショナル
}

// コンポーネントのProps型
interface UserCardProps {
  user: User;
  onEdit?: (user: User) => void; // 関数型
}

// 関数コンポーネントの型
const UserCard: React.FC<UserCardProps> = ({ user, onEdit }) => {
  // ...
};
```

## 9. React Hooksについて

> **🪝 補足**: Hooksについて詳しく知りたい場合は **[React Hooks 一覧](./react-hooks-overview.md)** を参照してください。

Reactでは、useState以外にも様々なHooks（フック）という機能を使用できます。Hooksを使うことで、コンポーネントに様々な機能を追加できます。

### 主なHooks

- **useState**: 状態管理（今回学習済み）
- **useCallback**: 関数の安定化、API通信に推奨
- **useContext**: コンポーネント間でのデータ共有
- **useEffect**: DOM操作、タイマーなど（API通信には推奨されません）

詳細は補足資料を参照してください。

## 10. まとめ

### 学習した内容

- ✅ ReactとRails Viewの違い
- ✅ JSXの基本文法
- ✅ コンポーネントの作成と再利用
- ✅ props と state の管理
- ✅ イベントハンドリング
- ✅ TypeScriptの基本的な使い方

### Railsとの対応関係

| Rails概念 | React概念 | 説明 |
|----------|-----------|------|
| 部分テンプレート | コンポーネント | 再利用可能なUI部品 |
| ローカル変数 | props | 親から子への データ渡し |
| インスタンス変数 | state | コンポーネント内の状態 |
| Helper | カスタムフック | ロジックの再利用 |

### 次のステップ

次は **[Rails API + React連携編](../03-api-react-integration/README.md)** で、バックエンドAPIとフロントエンドを連携させる方法を学びましょう。

### 参考資料

- [React公式ドキュメント](https://ja.react.dev/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

## 練習問題

以下の練習問題に挑戦してみましょう。分からない場合は、メンターやLLMに相談してください。

1. ToDoアイテムを表示する `TodoItem` コンポーネントを作成してください
2. カウンターアプリを拡張して、リセット機能を追加してください  
3. フォームコンポーネントを作成して、入力値をstateで管理してください

> **💡 ヒント**: ファイルの作成・編集はVSCodeで行い、`docker compose up -d`でサーバーを起動しましょう。