# claude-code-tips

Claude Codeを効果的に使用するためのtipsとベストプラクティスをまとめたリポジトリです。

## 概要

このリポジトリでは、Claude Codeの様々なtipsをソースとしてまとめ、開発時に活用できる実践的な知識を整理しています。他のプロジェクトでも利用できるように、汎用的な内容を心がけています。

## ドキュメント

### [Claude Code Tips](./claude-code-tips.md)
Claude Codeの包括的なtipsとベストプラクティス集です。初期セットアップから高度な使用方法まで、実践的な知識を体系的にまとめています。
- 知識管理システム（`.claude/`ディレクトリ）の構築方法
- 効果的なプロンプト戦略と思考モードの活用
- デバッグ、トラブルシューティング、生産性向上のヒント
- セキュリティ、パフォーマンス、チーム開発のベストプラクティス

### [Claude Code Init](./claude-code-init.md)
プロジェクトの初期セットアップを自動化するためのテンプレートファイルです。`/init`コマンドで読み込まれ、プロジェクトに最適な設定を構築します。
- 知識管理システムの自動構築
- プロジェクト分析と設定ファイルの生成
- 推奨されるワークフローとプロンプト戦略
- カスタマイズ可能な設定テンプレート

### [使い方ガイド](./how-to-use.md)
`claude-code-init.md`を使用してClaude Codeプロジェクトを効率的にセットアップする方法の詳細ガイドです。
- ステップバイステップのセットアップ手順
- 新規・既存プロジェクトへの導入方法
- 実践的な使用例とカスタマイズ方法
- トラブルシューティングとFAQ

### [サンプルファイル集](./examples/)
知識管理システムで使用する各ファイルの実用的なサンプル集です。
- [CLAUDE.md](./examples/CLAUDE.md) - プロジェクト設定ファイルのサンプル（React/TypeScript Eコマースアプリ例）
- [context.md](./examples/context.md) - ビジネス背景とプロジェクトコンテキストのサンプル
- [project-knowledge.md](./examples/project-knowledge.md) - 技術的な深い知識とアーキテクチャ決定のサンプル
- [project-improvements.md](./examples/project-improvements.md) - 改善履歴とログのサンプル
- [common-patterns.md](./examples/common-patterns.md) - よく使うコードパターンとClaude Codeコマンド例
- [debug-log.md](./examples/debug-log.md) - デバッグログと問題解決の記録サンプル


## 使い方

### 新規プロジェクトの場合
1. プロジェクトディレクトリに[claude-code-init.md](./claude-code-init.md)を配置
2. Claude Codeで`/init`コマンドを実行
3. 自動生成された設定をカスタマイズ

### 既存プロジェクトの場合
1. [claude-code-tips.md](./claude-code-tips.md)を参照して、必要な情報を確認
2. 自分のプロジェクトに`.claude/`ディレクトリを作成し、知識管理システムを構築
3. 記載されているベストプラクティスを実践

詳細は[使い方ガイド](./how-to-use.md)を参照してください。

## 参考記事・リンク

このリポジトリの作成にあたって参考にした記事やリンクの一覧です：

- [X (Twitter) - robj3d3のポスト](https://x.com/robj3d3/status/1935466820959617479)
- [Zenn - Claude Codeで素早くアプリケーションをつくる](https://zenn.dev/hokuto_tech/articles/86d1edb33da61a)
- [Zenn - Claude Codeのtipsと使い方](https://zenn.dev/medicalforce/articles/8bc0b6afbbb8a7)
- [Zenn - Claude Codeの使い方とベストプラクティス](https://zenn.dev/driller/articles/2a23ef94f1d603)

## 貢献

新しいtipsや改善案がある場合は、PRを歓迎します。

## ライセンス

このリポジトリの内容は自由に使用・改変できます。