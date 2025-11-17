---
description: '建構 TanStack Start 應用程式的指南'
applyTo: '**/*.ts, **/*.tsx, **/*.js, **/*.jsx, **/*.css, **/*.scss, **/*.json'
---

# TanStack Start 與 Shadcn/ui 開發指南

你是專精於使用現代 React 模式建構 TanStack Start 應用程式的 TypeScript 專家開發者。

## 技術堆疊
- TypeScript（嚴格模式）
- TanStack Start（路由和 SSR）
- Shadcn/ui（UI 元件）
- Tailwind CSS（樣式）
- Zod（驗證）
- TanStack Query（客戶端狀態）

## 程式碼風格規則

- 永遠不要使用 `any` 類型 - 始終使用適當的 TypeScript 類型
- 優先使用函數元件而非類別元件
- 始終使用 Zod 模式驗證外部資料
- 為所有路由包含錯誤和待處理邊界
- 遵循無障礙最佳實踐，使用 ARIA 屬性

## 元件模式

使用具有適當 TypeScript 介面的函數元件：

```typescript
interface ButtonProps {
  children: React.ReactNode;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
}

export default function Button({ children, onClick, variant = 'primary' }: ButtonProps) {
  return (
    <button onClick={onClick} className={cn(buttonVariants({ variant }))}>
      {children}
    </button>
  );
}
```

## 資料擷取

使用路由載入器用於：
- 渲染所需的初始頁面資料
- SSR 需求
- SEO 關鍵資料

使用 React Query 用於：
- 頻繁更新的資料
- 可選/次要資料
- 帶有樂觀更新的客戶端變更

```typescript
// 路由載入器
export const Route = createFileRoute('/users')({
  loader: async () => {
    const users = await fetchUsers()
    return { users: userListSchema.parse(users) }
  },
  component: UserList,
})

// React Query
const { data: stats } = useQuery({
  queryKey: ['user-stats', userId],
  queryFn: () => fetchUserStats(userId),
  refetchInterval: 30000,
});
```

## Zod 驗證

始終驗證外部資料。在 `src/lib/schemas.ts` 中定義模式：

```typescript
export const userSchema = z.object({
  id: z.string(),
  name: z.string().min(1).max(100),
  email: z.string().email().optional(),
  role: z.enum(['admin', 'user']).default('user'),
})

export type User = z.infer<typeof userSchema>

// 安全解析
const result = userSchema.safeParse(data)
if (!result.success) {
  console.error('Validation failed:', result.error.format())
  return null
}
```

## 路由

在 `src/routes/` 中使用基於檔案的路由結構化路由。始終包含錯誤和待處理邊界：

```typescript
export const Route = createFileRoute('/users/$id')({
  loader: async ({ params }) => {
    const user = await fetchUser(params.id);
    return { user: userSchema.parse(user) };
  },
  component: UserDetail,
  errorBoundary: ({ error }) => (
    <div className="text-red-600 p-4">Error: {error.message}</div>
  ),
  pendingBoundary: () => (
    <div className="flex items-center justify-center p-4">
      <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-primary" />
    </div>
  ),
});
```

## UI 元件

始終優先使用 Shadcn/ui 元件而非自訂元件：

```typescript
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';

<Card>
  <CardHeader>
    <CardTitle>User Details</CardTitle>
  </CardHeader>
  <CardContent>
    <Button onClick={handleSave}>Save</Button>
  </CardContent>
</Card>
```

使用 Tailwind 進行樣式設計，具有響應式設計：

```typescript
<div className="flex flex-col gap-4 p-6 md:flex-row md:gap-6">
  <Button className="w-full md:w-auto">Action</Button>
</div>
```

## 無障礙

首先使用語義 HTML。只有在不存在語義等效時才新增 ARIA：

```typescript
// ✅ 良好：語義 HTML，最少 ARIA
<button onClick={toggleMenu}>
  <MenuIcon aria-hidden="true" />
  <span className="sr-only">Toggle Menu</span>
</button>

// ✅ 良好：僅在需要時使用 ARIA（用於動態狀態）
<button
  aria-expanded={isOpen}
  aria-controls="menu"
  onClick={toggleMenu}
>
  Menu
</button>

// ✅ 良好：語義表單元素
<label htmlFor="email">Email Address</label>
<input id="email" type="email" />
{errors.email && (
  <p role="alert">{errors.email}</p>
)}
```

## 檔案組織

```
src/
├── components/ui/    # Shadcn/ui 元件
├── lib/schemas.ts    # Zod 模式
├── routes/          # 基於檔案的路由
└── routes/api/      # 伺服器路由 (.ts)
```

## 匯入標準

對所有內部匯入使用 `@/` 別名：

```typescript
// ✅ 良好
import { Button } from '@/components/ui/button'
import { userSchema } from '@/lib/schemas'

// ❌ 不好
import { Button } from '../components/ui/button'
```

## 新增元件

在需要時安裝 Shadcn 元件：

```bash
npx shadcn@latest add button card input dialog
```

## 常見模式

- 始終使用 Zod 驗證外部資料
- 使用路由載入器取得初始資料，React Query 用於更新
- 在所有路由上包含錯誤/待處理邊界
- 優先使用 Shadcn 元件而非自訂 UI
- 一致使用 `@/` 匯入
- 遵循無障礙最佳實踐
