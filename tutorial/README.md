# Rails API + React チュートリアル

Rails経験者だけど、APIモード未経験＆React初心者向けの学習教材です。

## 学習の進め方

1. **[Rails APIモード基礎編](./01-rails-api-basics/README.md)** 
   - Rails APIモードの基本概念
   - 従来のRailsとの違い
   - API設計の考え方

2. **[React基礎編](./02-react-basics/README.md)**
   - Reactの基本概念
   - コンポーネント、state、props
   - イベントハンドリング

3. **[Rails API + React連携編](./03-api-react-integration/README.md)**
   - フロントエンドとバックエンドの連携
   - axios を使ったHTTP通信
   - CORS設定

4. **[実践編：ToDoアプリ作成](./04-practical-todo-app/README.md)**
   - CRUD操作の実装
   - エラーハンドリング
   - UI/UXの改善

## 前提知識

- Ruby on Rails の基本的な使い方（MVC、ルーティング、Active Record等）
- HTML/CSS の基本知識
- JavaScript の基本文法

## 使用技術

### バックエンド
- Ruby 3.3.4
- Rails 8.0 (APIモード)
- PostgreSQL 16

### フロントエンド  
- React 19
- TypeScript
- Vite (ビルドツール)
- axios (HTTP クライアント)

### 開発環境
- Docker & Docker Compose
- pnpm (パッケージマネージャー)

## 開発環境の準備

```bash
# リポジトリをクローン（既に完了している場合はスキップ）
cd TOKIUM-summer-intern

# Docker コンテナをビルド・起動
docker compose build
docker compose up -d

# フロントエンド（React）: http://localhost:8080
# バックエンドAPI（Rails）: http://localhost:3000
```

各章で詳しい環境構築手順も説明します。

## 学習のポイント

- **段階的学習**: 一つずつ確実に理解してから次へ
- **実際に手を動かす**: コピペではなく自分で書いてみる
- **エラーを恐れない**: エラーメッセージから学ぶことは多い
- **公式ドキュメントも読む**: チュートリアルと合わせて理解を深める

頑張って学習を進めましょう！