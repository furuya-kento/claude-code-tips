# よく使うパターン集

## Claude Code コマンドパターン

### 新機能開発のフロー
```
> 新しい商品検索機能を実装してください。以下の要件を満たしてください：
- キーワード検索
- カテゴリフィルター
- 価格範囲フィルター
- 並び替え機能（価格、人気度、新着順）

まず既存の検索機能を確認してから、段階的に実装してください。
```

### バグ修正のフロー
```
> カート機能で以下のバグが発生しています：
[エラーメッセージをここに貼り付け]

以下の手順で調査してください：
1. 関連するファイルを特定
2. エラーの原因を調査
3. 修正案を提案
4. テストケースも含めて修正
```

### リファクタリング
```
> components/ProductCard.tsx の複雑度が高くなっています。
以下の観点でリファクタリングしてください：
- Single Responsibility Principleに従った分割
- カスタムフックの抽出
- Propsの型安全性向上
- パフォーマンス最適化

既存の機能は保持したまま改善してください。
```

## コードパターン

### React コンポーネント定義

#### 基本パターン
```typescript
interface ProductCardProps {
  product: Product
  onAddToCart: (product: Product) => void
  className?: string
}

export const ProductCard: React.FC<ProductCardProps> = ({
  product,
  onAddToCart,
  className
}) => {
  const [isLoading, setIsLoading] = useState(false)
  
  const handleAddToCart = async () => {
    setIsLoading(true)
    try {
      await onAddToCart(product)
    } finally {
      setIsLoading(false)
    }
  }
  
  return (
    <div className={cn('product-card', className)}>
      {/* コンポーネント内容 */}
    </div>
  )
}
```

#### フォームコンポーネント
```typescript
interface ContactFormData {
  name: string
  email: string
  message: string
}

export const ContactForm = () => {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting }
  } = useForm<ContactFormData>()
  
  const onSubmit = async (data: ContactFormData) => {
    // フォーム送信処理
  }
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* フォーム要素 */}
    </form>
  )
}
```

### カスタムフック

#### データフェッチフック
```typescript
export const useProduct = (productId: string) => {
  const [product, setProduct] = useState<Product | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)
  
  useEffect(() => {
    const fetchProduct = async () => {
      try {
        setLoading(true)
        const data = await api.products.getById(productId)
        setProduct(data)
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Unknown error')
      } finally {
        setLoading(false)
      }
    }
    
    fetchProduct()
  }, [productId])
  
  return { product, loading, error }
}
```

#### ローカルストレージフック
```typescript
export const useLocalStorage = <T>(key: string, initialValue: T) => {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue
    
    try {
      const item = window.localStorage.getItem(key)
      return item ? JSON.parse(item) : initialValue
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error)
      return initialValue
    }
  })
  
  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value
      setStoredValue(valueToStore)
      if (typeof window !== 'undefined') {
        window.localStorage.setItem(key, JSON.stringify(valueToStore))
      }
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error)
    }
  }
  
  return [storedValue, setValue] as const
}
```

### API呼び出しパターン

#### 基本的なAPI関数
```typescript
export const api = {
  products: {
    getAll: async (params?: ProductSearchParams): Promise<Product[]> => {
      const url = new URL('/api/products', process.env.NEXT_PUBLIC_API_URL)
      if (params) {
        Object.entries(params).forEach(([key, value]) => {
          if (value !== undefined) url.searchParams.set(key, String(value))
        })
      }
      
      const response = await fetch(url.toString())
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`)
      }
      return response.json()
    },
    
    getById: async (id: string): Promise<Product> => {
      const response = await fetch(`/api/products/${id}`)
      if (!response.ok) {
        throw new Error(`Product not found: ${id}`)
      }
      return response.json()
    },
    
    create: async (product: CreateProductRequest): Promise<Product> => {
      const response = await fetch('/api/products', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(product)
      })
      if (!response.ok) {
        throw new Error('Failed to create product')
      }
      return response.json()
    }
  }
}
```

### エラーハンドリング

#### Error Boundary
```typescript
interface ErrorBoundaryState {
  hasError: boolean
  error?: Error
}

export class ErrorBoundary extends React.Component<
  React.PropsWithChildren<{}>,
  ErrorBoundaryState
> {
  constructor(props: React.PropsWithChildren<{}>) {
    super(props)
    this.state = { hasError: false }
  }
  
  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error }
  }
  
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo)
    // エラーレポーティングサービスに送信
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="error-fallback">
          <h2>Something went wrong.</h2>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      )
    }
    
    return this.props.children
  }
}
```

### テストパターン

#### コンポーネントテスト
```typescript
import { render, screen, fireEvent, waitFor } from '@testing-library/react'
import { ProductCard } from '../ProductCard'

const mockProduct: Product = {
  id: '1',
  name: 'Test Product',
  price: 1000,
  imageUrl: '/test-image.jpg'
}

describe('ProductCard', () => {
  it('商品情報を正しく表示する', () => {
    const onAddToCart = jest.fn()
    
    render(
      <ProductCard product={mockProduct} onAddToCart={onAddToCart} />
    )
    
    expect(screen.getByText('Test Product')).toBeInTheDocument()
    expect(screen.getByText('¥1,000')).toBeInTheDocument()
  })
  
  it('カートに追加ボタンをクリックすると関数が呼ばれる', async () => {
    const onAddToCart = jest.fn()
    
    render(
      <ProductCard product={mockProduct} onAddToCart={onAddToCart} />
    )
    
    fireEvent.click(screen.getByText('カートに追加'))
    
    await waitFor(() => {
      expect(onAddToCart).toHaveBeenCalledWith(mockProduct)
    })
  })
})
```

#### APIテスト
```typescript
import { api } from '../api'

// MSW (Mock Service Worker) を使用
import { rest } from 'msw'
import { setupServer } from 'msw/node'

const server = setupServer(
  rest.get('/api/products', (req, res, ctx) => {
    return res(ctx.json([mockProduct]))
  })
)

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())

describe('Products API', () => {
  it('商品一覧を取得できる', async () => {
    const products = await api.products.getAll()
    expect(products).toHaveLength(1)
    expect(products[0].name).toBe('Test Product')
  })
})
```

## デバッグパターン

### Claude Codeでのデバッグ依頼
```
> 以下のエラーが発生しています。原因を調査して修正してください：

```
TypeError: Cannot read properties of undefined (reading 'map')
  at ProductList (/components/ProductList.tsx:15:23)
```

関連する可能性があるファイル：
- components/ProductList.tsx
- hooks/useProducts.ts
- pages/api/products.js

エラーの再現手順：
1. トップページにアクセス
2. カテゴリ「Electronics」を選択
3. エラーが発生

デバッグ情報も含めて解決策を提案してください。
```

### パフォーマンス調査
```
> ProductList コンポーネントのパフォーマンスを分析してください。
現在1000件の商品表示に3秒かかっています。

以下の観点で最適化案を提案してください：
- レンダリング最適化
- データフェッチ最適化
- メモ化の適用
- 仮想スクロールの導入

React DevTools Profilerの結果も含めて分析してください。
```

## 設定ファイルパターン

### Prettier設定
```json
{
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 80,
  "bracketSpacing": true,
  "arrowParens": "avoid"
}
```

### ESLint設定
```json
{
  "extends": [
    "next/core-web-vitals",
    "@typescript-eslint/recommended",
    "prettier"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "warn",
    "prefer-const": "error",
    "no-console": ["warn", { "allow": ["warn", "error"] }]
  }
}
```

### TypeScript設定
```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "es6"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"],
      "@/hooks/*": ["./src/hooks/*"],
      "@/lib/*": ["./src/lib/*"],
      "@/types/*": ["./src/types/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```