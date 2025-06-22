# Claude Code 初期セットアップファイルの使い方ガイド

このガイドでは、`claude-code-init.md`を使用してClaude Codeプロジェクトを効率的にセットアップする方法を説明します。

## 概要

`claude-code-init.md`は、Claude Codeの初期セットアップを自動化し、ベストプラクティスに基づいた開発環境を構築するためのテンプレートファイルです。

## セットアップ手順

### ステップ1: プロジェクトディレクトリの作成

```bash
mkdir my-project
cd my-project
```

### ステップ2: claude-code-init.mdの配置

[claude-code-init.md](./claude-code-init.md)をダウンロードして、プロジェクトのルートディレクトリに配置します。

```bash
# wgetを使用する場合
wget https://raw.githubusercontent.com/furuya-kento/claude-code-tips/main/claude-code-init.md

# またはcurlを使用する場合
curl -O https://raw.githubusercontent.com/furuya-kento/claude-code-tips/main/claude-code-init.md
```

### ステップ3: Claude Codeの起動

```bash
claude
```

### ステップ4: 初期化コマンドの実行

Claude Codeのプロンプトで以下のコマンドを実行：

```
> /init
```

このコマンドにより、Claude Codeは`claude-code-init.md`の内容を読み込み、以下の処理を自動的に実行します：

1. プロジェクト構造の分析
2. `.claude/`ディレクトリの作成
3. 必要な設定ファイルの生成
4. プロジェクト固有の`CLAUDE.md`の作成

## 期待される結果

`/init`コマンド実行後、以下のような構造が作成されます：

```
my-project/
├── claude-code-init.md
├── CLAUDE.md              # 自動生成されたプロジェクト設定
└── .claude/               # 知識管理ディレクトリ
    ├── context.md         # プロジェクトコンテキスト
    ├── project-knowledge.md
    ├── project-improvements.md
    ├── common-patterns.md
    └── debug-log.md
```

## カスタマイズ方法

### プロジェクト固有の設定

生成された`CLAUDE.md`を編集して、プロジェクト固有の情報を追加：

```markdown
## 開発環境
- 言語: JavaScript/TypeScript
- フレームワーク: React
- パッケージマネージャー: npm

## 基本コマンド
- ビルド: `npm run build`
- テスト: `npm test`
- リント: `npm run lint`
- 開発サーバー: `npm run dev`
```

### 知識ファイルの更新

プロジェクトの進行に応じて、`.claude/`内のファイルを更新：

1. **context.md** - ビジネス要件や制約を記載
2. **project-knowledge.md** - 技術的な決定事項を記録
3. **common-patterns.md** - よく使うコードパターンを保存
4. **debug-log.md** - 解決した問題と対策を記録

## 実践的な使用例

### 例1: Reactプロジェクトの初期化

```bash
# 1. Reactプロジェクトを作成
npx create-react-app my-app
cd my-app

# 2. claude-code-init.mdを配置
curl -O https://raw.githubusercontent.com/furuya-kento/claude-code-tips/main/claude-code-init.md

# 3. Claude Codeを起動
claude

# 4. 初期化
> /init

# 5. プロジェクト固有の設定を追加
> このプロジェクトはReactアプリケーションです。TypeScriptを使用し、Material-UIをUIライブラリとして使用します。
```

### 例2: 既存プロジェクトへの導入

```bash
# 1. 既存プロジェクトのルートへ移動
cd existing-project

# 2. claude-code-init.mdを配置
wget https://raw.githubusercontent.com/furuya-kento/claude-code-tips/main/claude-code-init.md

# 3. Claude Codeを起動して初期化
claude
> /init

# 4. 既存の設定を分析させる
> package.jsonとREADME.mdを読んで、プロジェクトの設定をCLAUDE.mdに反映してください
```

## トラブルシューティング

### よくある問題と解決方法

#### Q: `/init`コマンドが`claude-code-init.md`を認識しない

A: ファイルがプロジェクトのルートディレクトリに配置されているか確認してください。

```bash
ls -la | grep claude-code-init.md
```

#### Q: `.claude/`ディレクトリが作成されない

A: 手動で作成してから再度`/init`を実行：

```bash
mkdir -p .claude
claude
> /init
```

#### Q: 生成されたCLAUDE.mdが不完全

A: プロジェクトに関する追加情報を提供：

```
> プロジェクトの詳細を教えてください。使用している言語、フレームワーク、主な依存関係などを含めてCLAUDE.mdを更新してください。
```

## ベストプラクティス

### 1. 定期的な更新

- 週次でCLAUDE.mdと知識ファイルを見直す
- 新しい発見や学びを記録

### 2. チーム共有

- `.claude/`ディレクトリをGitで管理
- チームメンバー間で知識を共有

### 3. プロジェクトテンプレート

- 組織内で標準的な`claude-code-init.md`を作成
- プロジェクトタイプ別にテンプレートを用意

## 次のステップ

1. [Claude Code Tips](./claude-code-tips.md)で詳細なベストプラクティスを確認
2. プロジェクトに合わせて設定をカスタマイズ
3. チームメンバーと知識を共有

## サポート

問題や質問がある場合は、[GitHubのIssue](https://github.com/furuya-kento/claude-code-tips/issues)で報告してください。