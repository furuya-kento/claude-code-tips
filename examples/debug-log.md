# デバッグログ

## 2024-03-20: 決済処理でタイムアウトエラー

### 症状
- 注文確定ボタンを押すと30秒後にタイムアウト
- エラーメッセージ: "Request timeout after 30000ms"
- 影響範囲: 全ユーザー、注文完了率が85%から60%に低下

### エラーの詳細
```
Error: AxiosError: timeout of 30000ms exceeded
  at createError (/node_modules/axios/lib/core/createError.js:16:15)
  at RedirectableRequest.handleRequestTimeout (/node_modules/axios/lib/adapters/http.js:281:16)
  at Object.onceWrapper (events.js:421:28)
  at RedirectableRequest.emit (events.js:315:20)
  at Timeout.<anonymous> (/node_modules/follow-redirects/index.js:178:9)
```

### 調査プロセス
1. **ログ確認**
   - API呼び出し時間: 45秒平均
   - Stripe APIの応答時間: 2秒以内（正常）
   - データベースクエリ: 40秒以上

2. **データベース調査**
   ```sql
   EXPLAIN ANALYZE SELECT * FROM orders 
   JOIN order_items ON orders.id = order_items.order_id 
   JOIN products ON order_items.product_id = products.id 
   WHERE orders.user_id = 'user_123';
   
   -- 実行時間: 42.3秒
   -- 原因: order_itemsテーブルにインデックスなし
   ```

3. **パフォーマンス分析**
   - 1つの注文で平均15個の商品
   - N+1クエリ問題が発生
   - order_itemsテーブルのレコード数: 500万件

### 根本原因
1. **インデックス不足**
   - `order_items.order_id`にインデックスなし
   - `order_items.product_id`にインデックスなし

2. **N+1クエリ問題**
   ```typescript
   // 問題のあるコード
   const order = await prisma.order.findUnique({ where: { id } })
   for (const itemId of order.itemIds) {
     const item = await prisma.orderItem.findUnique({ where: { id: itemId } })
     const product = await prisma.product.findUnique({ where: { id: item.productId } })
   }
   ```

### 解決方法
1. **インデックス追加**
   ```sql
   CREATE INDEX idx_order_items_order_id ON order_items(order_id);
   CREATE INDEX idx_order_items_product_id ON order_items(product_id);
   CREATE INDEX idx_orders_user_id ON orders(user_id);
   ```

2. **クエリ最適化**
   ```typescript
   // 修正後のコード
   const order = await prisma.order.findUnique({
     where: { id },
     include: {
       items: {
         include: {
           product: true
         }
       }
     }
   })
   ```

3. **タイムアウト時間の調整**
   ```typescript
   // axios設定
   const api = axios.create({
     timeout: 60000, // 30秒 → 60秒
     retry: 3
   })
   ```

### 結果
- API応答時間: 45秒 → 1.2秒
- 注文完了率: 60% → 95%
- データベース負荷: 90% → 15%

### 予防策
1. **監視の強化**
   ```typescript
   // パフォーマンス監視
   const startTime = Date.now()
   const result = await expensiveOperation()
   const duration = Date.now() - startTime
   
   if (duration > 5000) {
     logger.warn(`Slow operation detected: ${duration}ms`)
   }
   ```

2. **定期的なパフォーマンステスト**
   - 週次でEXPLAIN ANALYZEを実行
   - スロークエリログの確認

3. **アラート設定**
   - API応答時間 > 5秒でアラート
   - エラー率 > 5%でアラート

---

## 2024-03-15: React Hydration エラーの大量発生

### 症状
- 本番環境でHydration errorが大量発生
- ユーザーに白画面が表示される
- エラー発生率: 25%のセッション

### エラーの詳細
```
Warning: Text content did not match. Server: "2024年3月15日" Client: "2024年3月16日"
  at span
  at DateDisplay
  at ProductCard
```

### 調査プロセス
1. **エラーの特定**
   - サーバーサイドとクライアントサイドで時刻の差異
   - タイムゾーンの処理に問題

2. **再現手順**
   ```typescript
   // 問題のあるコード
   const DateDisplay = () => {
     return <span>{new Date().toLocaleDateString('ja-JP')}</span>
   }
   ```

### 根本原因
- サーバー（UTC）とクライアント（ローカルタイムゾーン）で日付が異なる
- 日付をまたぐタイミングで発生頻度が高い

### 解決方法
1. **suppressHydrationWarningの使用**
   ```typescript
   const DateDisplay = () => {
     return (
       <span suppressHydrationWarning>
         {new Date().toLocaleDateString('ja-JP')}
       </span>
     )
   }
   ```

2. **クライアントサイドレンダリングに変更**
   ```typescript
   const DateDisplay = () => {
     const [mounted, setMounted] = useState(false)
     
     useEffect(() => {
       setMounted(true)
     }, [])
     
     if (!mounted) {
       return <span>読み込み中...</span>
     }
     
     return <span>{new Date().toLocaleDateString('ja-JP')}</span>
   }
   ```

3. **サーバーで固定時刻を使用**
   ```typescript
   // getServerSideProps で取得
   export const getServerSideProps = () => {
     return {
       props: {
         serverTime: new Date().toISOString()
       }
     }
   }
   ```

### 予防策
- 時刻表示はクライアントサイドで実行
- Hydration errorの監視を強化
- E2Eテストで複数タイムゾーンをテスト

---

## 2024-03-10: メモリリークによるサーバークラッシュ

### 症状
- 本番サーバーが6時間おきにクラッシュ
- メモリ使用量が徐々に増加
- PM2の自動再起動が頻繁に発生

### 調査プロセス
1. **メモリ使用量の監視**
   ```bash
   # ヒープダンプの取得
   node --inspect server.js
   # Chrome DevTools でメモリ分析
   ```

2. **原因の特定**
   ```typescript
   // 問題のあるコード - イベントリスナーの解除忘れ
   useEffect(() => {
     const handleResize = () => setWindowSize(window.innerWidth)
     window.addEventListener('resize', handleResize)
     // return文が不足
   }, [])
   ```

### 根本原因
- React コンポーネントのイベントリスナー解除忘れ
- setInterval/setTimeoutのクリア忘れ
- WebSocket接続のクローズ忘れ

### 解決方法
```typescript
// 修正後のコード
useEffect(() => {
  const handleResize = () => setWindowSize(window.innerWidth)
  window.addEventListener('resize', handleResize)
  
  return () => {
    window.removeEventListener('resize', handleResize)
  }
}, [])

// タイマーの適切な処理
useEffect(() => {
  const timer = setInterval(() => {
    fetchData()
  }, 5000)
  
  return () => clearInterval(timer)
}, [])
```

### 予防策
- ESLintルールの追加
- メモリ使用量の監視アラート
- 定期的なヒープダンプ分析

---

## 2024-03-05: CORS エラーで API 呼び出し失敗

### 症状
- フロントエンドからのAPI呼び出しがCORSエラー
- 開発環境では正常、本番環境でのみ発生
- エラー: "Access to fetch at 'https://api.example.com' from origin 'https://app.example.com' has been blocked by CORS policy"

### 調査プロセス
1. **CORS設定確認**
   ```typescript
   // Next.js API Routes
   export default function handler(req, res) {
     res.setHeader('Access-Control-Allow-Origin', 'http://localhost:3000') // 問題：本番ドメイン未設定
     res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE')
     res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization')
   }
   ```

### 解決方法
```typescript
// 修正版
export default function handler(req, res) {
  const allowedOrigins = [
    'http://localhost:3000',
    'https://app.example.com',
    'https://staging.example.com'
  ]
  
  const origin = req.headers.origin
  if (allowedOrigins.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin)
  }
  
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE')
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization')
  
  if (req.method === 'OPTIONS') {
    res.status(200).end()
    return
  }
}
```

### 学んだこと
- 本番環境とステージング環境のドメイン設定を事前に確認
- 環境変数でオリジンを管理する
- プリフライトリクエスト（OPTIONS）の適切な処理

---

## よくある問題と解決パターン

### TypeScript 型エラー
```typescript
// 問題: Property 'xxx' does not exist on type 'yyy'
// 解決: 型ガードの使用
function isUser(obj: any): obj is User {
  return obj && typeof obj.id === 'string' && typeof obj.name === 'string'
}

if (isUser(data)) {
  // この中では data は User 型として扱われる
  console.log(data.name)
}
```

### Async/Await エラーハンドリング
```typescript
// 問題: エラーハンドリングの漏れ
const fetchData = async () => {
  try {
    const response = await api.getData()
    return response.data
  } catch (error) {
    // ログ記録
    logger.error('Data fetch failed:', error)
    // ユーザーに通知
    toast.error('データの取得に失敗しました')
    // デフォルト値を返す
    return []
  }
}
```

### State 更新のバッチング
```typescript
// 問題: 複数のstate更新で再レンダリング
const handleUpdate = () => {
  setName('New Name')      // 再レンダリング
  setEmail('new@email')    // 再レンダリング
  setAge(25)              // 再レンダリング
}

// 解決: useTransition または setState のコールバック
const handleUpdate = () => {
  startTransition(() => {
    setName('New Name')
    setEmail('new@email')
    setAge(25)            // まとめて1回の再レンダリング
  })
}
```