# Rails API + React 学習プロジェクト

このリポジトリは、Rails経験者がRails APIモードとReactを学習するための教材とサンプルプロジェクトです。
> [!CAUTION]
> このRepoをForkしてプロダクトを作る際には、PR作成時に自分のRepositoryにマージ先が向いているか確認してください

## 📚 学習教材・チュートリアル

### 初めての方へ

**Rails経験者でAPIモード未経験・React初心者の方** は、まず `tutorial/` ディレクトリの教材から始めることを強く推奨します。

### 📖 チュートリアル構成

1. **[Rails API基礎編](./tutorial/01-rails-api-basics/README.md)**
   - Rails APIモードの基本概念
   - JSON、CORS、StatelessとStatefulの違い
   - 既存のRailsとの違いを理解

2. **[React基礎編](./tutorial/02-react-basics/README.md)**
   - React コンポーネントの基本
   - JSX、Props、State管理
   - TypeScript との連携

3. **[Rails API + React連携編](./tutorial/03-api-react-integration/README.md)**
   - HTTP通信とAPI連携
   - 非同期処理（Promise、async/await）
   - CRUD操作の実装

4. **[実践編：ToDoアプリ作成](./tutorial/04-practical-todo-app/README.md)**
   - 実際のアプリケーション開発
   - 完全なCRUDアプリケーション
   - ベストプラクティスの実装

### 🎯 学習の進め方

```bash
# 1. 環境構築（下記の手順に従ってください）
docker compose up -d

# 2. チュートリアルを順番に進める
# tutorial/README.md から開始してください

# 3. 疑問があれば questions/ ディレクトリを確認
# 過去のQ&Aが参考になるかもしれません
```

### 💡 学習サポート

- **質問集**: `questions/` ディレクトリに過去のQ&Aを収録
- **補足資料**: 各章に理解が難しい概念の詳細解説
- **段階的学習**: Railsの知識を活かしながら段階的にReactを習得

---



## 環境構築

### 必要な物
このリポジトリの実行には
- Docker
- npm
- pnpm
のインストールが必要です。
pnpmのインストールは[公式ドキュメント](https://pnpm.io/ja/installation)を参照してください

### クローン
forkした場合はリポジトリのリンクやディレクトリ名を変えてください
```bash
cd 作業ディレクトリ
git clone git@github.com:Gild-shogi/TOKIUM-summer-intern.git
cd TOKIUM-summer-intern
```

### Start server
```bash
docker compose build
docker compose up -d
```


### ローカルサーバーへのアクセス
すべてが正常に動作している場合、ブラウザで http://localhost:3000 , http://localhost:8080 にアクセスして、アプリケーションが期待通りに動作しているか確認してください
