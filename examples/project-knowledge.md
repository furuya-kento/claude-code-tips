# プロジェクト技術知識

## アーキテクチャ決定事項

### フロントエンドアーキテクチャ

#### Next.js App Routerの採用理由
- **Server Components**: 初期ロード時間の短縮
- **Streaming**: ユーザー体験の向上
- **Built-in SEO**: ECサイトに必要なSEO対策
- **File-based Routing**: 開発効率の向上

#### 状態管理にZustandを選択
```typescript
// stores/useCartStore.ts
interface CartStore {
  items: CartItem[]
  addItem: (product: Product) => void
  removeItem: (productId: string) => void
  clearCart: () => void
}
```
- Reduxよりも軽量でシンプル
- TypeScriptとの親和性が高い
- DevToolsサポート

### バックエンドアーキテクチャ

#### APIアーキテクチャ
- **GraphQL不採用の理由**: 
  - チームの学習コストを考慮
  - REST APIで十分な要件
  - 将来的な移行は可能

#### データベース設計
```sql
-- 主要なテーブル構造
CREATE TABLE products (
  id UUID PRIMARY KEY,
  shop_id UUID NOT NULL,
  name VARCHAR(255) NOT NULL,
  price DECIMAL(10, 2) NOT NULL,
  inventory_count INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW()
);

-- インデックス戦略
CREATE INDEX idx_products_shop_id ON products(shop_id);
CREATE INDEX idx_products_created_at ON products(created_at DESC);
```

## 技術的な深い知識

### パフォーマンス最適化

#### 画像最適化戦略
```typescript
// next.config.js
module.exports = {
  images: {
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  }
}
```

#### キャッシング戦略
1. **ブラウザキャッシュ**
   - 静的アセット: 1年間
   - API応答: 状況に応じて5分〜1時間

2. **CDNキャッシュ**
   - 商品画像: 30日間
   - 静的ページ: 1時間

3. **アプリケーションキャッシュ**
   ```typescript
   // Redis実装例
   const getCachedProduct = async (id: string) => {
     const cached = await redis.get(`product:${id}`)
     if (cached) return JSON.parse(cached)
     
     const product = await db.product.findUnique({ where: { id } })
     await redis.set(`product:${id}`, JSON.stringify(product), 'EX', 300)
     return product
   }
   ```

### セキュリティ実装

#### 認証・認可
```typescript
// middleware.ts
export async function middleware(request: NextRequest) {
  const token = request.cookies.get('auth-token')
  
  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url))
  }
  
  try {
    const payload = await verifyJWT(token.value)
    // リクエストに認証情報を追加
    request.headers.set('X-User-Id', payload.userId)
  } catch {
    return NextResponse.redirect(new URL('/login', request.url))
  }
}
```

#### XSS対策
- React のデフォルトエスケープに依存
- dangerouslySetInnerHTMLの使用を制限
- Content Security Policy の実装

### データベース最適化

#### クエリ最適化
```typescript
// 悪い例
const orders = await db.order.findMany()
for (const order of orders) {
  order.items = await db.orderItem.findMany({ where: { orderId: order.id } })
}

// 良い例
const orders = await db.order.findMany({
  include: {
    items: true
  }
})
```

#### インデックス戦略
- 検索頻度の高いカラムにインデックス
- 複合インデックスの順序に注意
- EXPLAIN ANALYZEで定期的に確認

## 外部サービス統合

### Stripe統合
```typescript
// services/stripe.ts
export const createPaymentIntent = async (amount: number, currency: string) => {
  const paymentIntent = await stripe.paymentIntents.create({
    amount,
    currency,
    automatic_payment_methods: { enabled: true },
    metadata: {
      integration_check: 'accept_a_payment'
    }
  })
  return paymentIntent
}
```

### SendGrid設定
```typescript
// lib/email.ts
sgMail.setApiKey(process.env.SENDGRID_API_KEY)

export const sendOrderConfirmation = async (order: Order) => {
  const msg = {
    to: order.customerEmail,
    from: 'noreply@example.com',
    templateId: 'd-xxxxxxxxxxxxx',
    dynamicTemplateData: {
      orderNumber: order.number,
      totalAmount: order.total,
      items: order.items
    }
  }
  await sgMail.send(msg)
}
```

## 開発ワークフロー

### Git ブランチ戦略
```
main
├── develop
│   ├── feature/product-search
│   ├── feature/checkout-flow
│   └── feature/admin-dashboard
├── release/v1.2.0
└── hotfix/payment-bug
```

### CI/CD パイプライン
```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: npm test
      - name: Build
        run: npm run build
      - name: Deploy to Vercel
        run: vercel --prod
```

## トラブルシューティング知識

### よくある問題と解決策

#### Hydration エラー
```typescript
// 問題のあるコード
<div>{new Date().toLocaleString()}</div>

// 解決策
<div suppressHydrationWarning>{new Date().toLocaleString()}</div>
// または
const [mounted, setMounted] = useState(false)
useEffect(() => setMounted(true), [])
if (!mounted) return null
```

#### メモリリーク
```typescript
// 問題: イベントリスナーの解除忘れ
useEffect(() => {
  window.addEventListener('resize', handleResize)
  // return忘れ
}, [])

// 解決策
useEffect(() => {
  window.addEventListener('resize', handleResize)
  return () => window.removeEventListener('resize', handleResize)
}, [])
```

## パフォーマンスベンチマーク

### 現在の計測値
- First Contentful Paint: 1.2s
- Largest Contentful Paint: 2.1s
- Time to Interactive: 2.5s
- Cumulative Layout Shift: 0.05

### 改善履歴
1. 画像の遅延読み込み実装 → LCP 3.5s → 2.1s
2. JavaScriptバンドル最適化 → TTI 4.0s → 2.5s
3. フォントの事前読み込み → CLS 0.15 → 0.05