# Frontend
フロントエンドは[pnpm](https://pnpm.io/ja/)を使用してパッケージ管理をしています.
- npmやyarnと比較して早い
- 依存関係の管理が独自であり、ディスク容量が節約できて軽い
詳しくは[こちら](https://pnpm.io/ja/motivation)を見てみてください

## パッケージを追加したい場合
```bash
pnpm add <追加したいパッケージ名>
```

## パッケージを削除したい場合
```bash
pnpm remove <パッケージ名>
```

## パッケージをアップデートしたい場合
```bash
pnpm up <パッケージ名>@<バージョン> # 特定のパッケージをアップデートする場合
pnpm up # 全てのパッケージを依存関係を維持してアップデートする場合
pnpm up --latest # 依存関係を無視して全てのライブラリを最新版にする場合
```

## ローカルで立ち上げ・Lintをする場合
- 立ち上げ(立ち上がると`localhost:8080`でアクセス可能です)
```bash
pnpm dev
```

- Lint(Typescriptの構文エラーを見つけてくれます)
```bash
pnpm lint
```