> [!CAUTION]
> このRepoをForkしてプロダクトを作る際には、PR作成時に自分のRepositoryにマージ先が向いているか確認してください

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
