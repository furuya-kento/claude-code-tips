# CLAUDE.md

このファイルは、このリポジトリでコードを扱う際に Claude Code に対してガイダンスを提供します。

## プロジェクト概要

このプロジェクトは、React と TypeScript を使用したEコマースWebアプリケーションです。
ユーザーが商品を検索、閲覧、購入できるオンラインショッピングプラットフォームを提供します。

### 主な機能
- 商品カタログの表示
- ショッピングカート機能
- ユーザー認証とアカウント管理
- 決済処理
- 注文履歴の管理

## 開発環境

- **言語**: TypeScript 5.3
- **フレームワーク**: React 18.2, Next.js 14
- **スタイリング**: Tailwind CSS 3.4
- **状態管理**: Zustand 4.5
- **パッケージマネージャー**: npm 10.2
- **Node.js**: 20.10 LTS

## 基本コマンド

```bash
# 依存関係のインストール
npm install

# 開発サーバーの起動
npm run dev

# ビルド
npm run build

# 本番環境での起動
npm start

# テストの実行
npm test

# E2Eテストの実行
npm run e2e

# リントの実行
npm run lint

# 型チェック
npm run type-check

# フォーマット
npm run format
```

## プロジェクト構造

```
src/
├── app/           # Next.js App Router
├── components/    # 再利用可能なコンポーネント
├── hooks/         # カスタムフック
├── lib/           # ユーティリティ関数とライブラリ
├── services/      # API通信ロジック
├── stores/        # Zustand ストア
├── types/         # TypeScript型定義
└── styles/        # グローバルスタイル
```

## コーディング規約

### 命名規則
- **コンポーネント**: PascalCase (例: `ProductCard.tsx`)
- **フック**: camelCase with "use" prefix (例: `useCart.ts`)
- **ユーティリティ関数**: camelCase (例: `formatPrice.ts`)
- **型**: PascalCase with "T" prefix for generic types (例: `TProduct`)
- **定数**: SCREAMING_SNAKE_CASE (例: `MAX_ITEMS_PER_PAGE`)

### ファイル構成
- 1ファイル1コンポーネント/機能
- index.tsを使用してディレクトリからのエクスポートを整理
- テストファイルは同じディレクトリに `.test.ts` または `.spec.ts` として配置

### コードスタイル
- Prettierの設定に従う
- ESLintルールを厳守
- 明示的な型定義を優先（any型の使用を避ける）
- 関数型プログラミングのパラダイムを採用

## API仕様

### エンドポイント
- ベースURL: `https://api.example.com/v1`
- 認証: Bearer Token (JWTを使用)
- レスポンス形式: JSON

### 主要なエンドポイント
- `GET /products` - 商品一覧の取得
- `GET /products/:id` - 商品詳細の取得
- `POST /cart/items` - カートに商品を追加
- `DELETE /cart/items/:id` - カートから商品を削除
- `POST /orders` - 注文の作成

## 環境変数

必要な環境変数は `.env.example` を参照してください：

```
NEXT_PUBLIC_API_URL=
DATABASE_URL=
JWT_SECRET=
STRIPE_SECRET_KEY=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
```

## デプロイ

### 本番環境
- Vercelにデプロイ
- GitHub Actionsでの自動デプロイ設定済み
- mainブランチへのマージで自動デプロイ

### ステージング環境
- developブランチからの自動デプロイ
- URLはREADMEに記載

## 注意事項

1. **パフォーマンス**: 
   - 画像は次世代フォーマット（WebP）を使用
   - 遅延読み込みを実装
   - コンポーネントの適切なメモ化

2. **セキュリティ**:
   - APIキーは環境変数で管理
   - XSS対策のため、ユーザー入力は必ずサニタイズ
   - CSRFトークンの実装

3. **アクセシビリティ**:
   - WCAG 2.1 AA準拠を目指す
   - セマンティックHTMLの使用
   - キーボードナビゲーション対応

4. **データベース**:
   - マイグレーションは必ずレビュー後に実行
   - データベーススキーマの変更は慎重に

## トラブルシューティング

よくある問題と解決方法：

1. **npm install が失敗する場合**
   - Node.jsのバージョンを確認（20.10以上が必要）
   - `npm cache clean --force` を実行

2. **型エラーが発生する場合**
   - `npm run type-check` で詳細を確認
   - VSCodeの場合、TypeScriptのバージョンを確認

3. **テストが失敗する場合**
   - テスト用の環境変数が設定されているか確認
   - データベースの接続を確認

## 関連ドキュメント

- [API仕様書](./docs/api.md)
- [アーキテクチャ設計書](./docs/architecture.md)
- [コントリビューションガイド](./CONTRIBUTING.md)