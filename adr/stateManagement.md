# State Management

Стратегія управління станом на фронтенді.
Стек: Next.js App Router, Zustand, TanStack Query.

---

## Розподіл відповідальності

| Тип стану | Приклад | Інструмент |
|-----------|---------|------------|
| Серверний (статичний) | каталог, деталі оголошення, профіль | Server Components + fetch |
| Серверний (динамічний) | статус обміну, wishlist items | TanStack Query |
| Глобальний UI | кошик, wishlist count, drawer open/close | Zustand |
| Локальний UI | активний таб, відкритий попап, форма | useState |

Головне правило: **Server Components — за замовчуванням**. Client state додається тільки там де без нього не обійтись.

---

## Zustand — глобальний UI стан

Використовується для стану що:
- потрібен у кількох непов'язаних компонентах
- має переживати навігацію між сторінками
- не залежить від серверних даних

### Кошик

```ts
// stores/cart.ts
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

interface CartItem {
  id: string
  name: string
  price: number
  photo_url: string
}

interface CartStore {
  items: CartItem[]
  add: (item: CartItem) => void
  remove: (id: string) => void
  clear: () => void
  count: () => number
}

export const useCartStore = create<CartStore>()(
  persist(
    (set, get) => ({
      items: [],
      add: (item) =>
        set((s) => ({
          items: s.items.some((i) => i.id === item.id)
            ? s.items
            : [...s.items, item],
        })),
      remove: (id) =>
        set((s) => ({ items: s.items.filter((i) => i.id !== id) })),
      clear: () => set({ items: [] }),
      count: () => get().items.length,
    }),
    { name: 'cart-storage' } // persist в localStorage автоматично
  )
)
```

`persist` middleware зберігає кошик в localStorage — виживає після перезавантаження сторінки. Юзер може додати рослину без логіну, закрити браузер і повернутись — кошик збережено.

### Wishlist count (UI badge)

Кількість збережених рослин в іконці хедера — глобальний UI стан.
Самі items wishlist — через TanStack Query (серверні дані).

```ts
// stores/wishlist.ts
import { create } from 'zustand'

interface WishlistStore {
  count: number
  setCount: (count: number) => void
  increment: () => void
  decrement: () => void
}

export const useWishlistStore = create<WishlistStore>((set) => ({
  count: 0,
  setCount: (count) => set({ count }),
  increment: () => set((s) => ({ count: s.count + 1 })),
  decrement: () => set((s) => ({ count: Math.max(0, s.count - 1) })),
}))
```

### UI стан (drawer, popups)

```ts
// stores/ui.ts
import { create } from 'zustand'

interface UIStore {
  cartDrawerOpen: boolean
  filterPopupOpen: boolean
  openCartDrawer: () => void
  closeCartDrawer: () => void
  openFilterPopup: () => void
  closeFilterPopup: () => void
}

export const useUIStore = create<UIStore>((set) => ({
  cartDrawerOpen: false,
  filterPopupOpen: false,
  openCartDrawer: () => set({ cartDrawerOpen: true }),
  closeCartDrawer: () => set({ cartDrawerOpen: false }),
  openFilterPopup: () => set({ filterPopupOpen: true }),
  closeFilterPopup: () => set({ filterPopupOpen: false }),
}))
```

---

## TanStack Query — серверний динамічний стан

Використовується для даних що:
- потребують фонового оновлення (polling)
- мають оптимістичні апдейти
- викликаються з client components

### Налаштування

```ts
// app/providers.tsx
'use client'

import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { useState } from 'react'

export function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000, // 60 секунд — не рефетчити якщо дані свіжі
          },
        },
      })
  )

  return (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  )
}
```

```ts
// app/layout.tsx
import { Providers } from './providers'

export default function RootLayout({ children }) {
  return (
    <html lang="uk">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  )
}
```

### Polling статусу обміну

Використовується на сторінці активного обміну — юзер чекає поки партнер підтвердить.

```ts
// hooks/useListingStatus.ts
import { useQuery } from '@tanstack/react-query'

export function useListingStatus(id: string) {
  return useQuery({
    queryKey: ['listing-status', id],
    queryFn: async () => {
      const res = await fetch(`/api/listings/${id}`)
      if (!res.ok) throw new Error('Failed to fetch')
      return res.json()
    },
    refetchInterval: 10_000,       // polling кожні 10 секунд
    refetchIntervalInBackground: false, // зупиняємо якщо вкладка неактивна
  })
}
```

```tsx
// використання в компоненті
'use client'

export function SwapStatusBanner({ listingId }: { listingId: string }) {
  const { data, isLoading } = useListingStatus(listingId)

  if (isLoading) return <Skeleton />
  if (data?.status === 'COMPLETED') return <Banner text="Обмін завершено 🌿" />
  if (data?.status === 'BOTH_SHIPPED') return <Banner text="Обидві рослини в дорозі" />

  return null
}
```

### Wishlist — оптимістичні апдейти

Серце іконки заповнюється миттєво — не чекаємо відповіді сервера.

```ts
// hooks/useWishlist.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'

export function useWishlist() {
  const queryClient = useQueryClient()

  const { data: items = [] } = useQuery({
    queryKey: ['wishlist'],
    queryFn: () => fetch('/api/wishlist').then((r) => r.json()),
  })

  const add = useMutation({
    mutationFn: (listingId: string) =>
      fetch('/api/wishlist', {
        method: 'POST',
        body: JSON.stringify({ listingId }),
      }),
    onMutate: async (listingId) => {
      // скасовуємо in-flight запити щоб уникнути race condition
      await queryClient.cancelQueries({ queryKey: ['wishlist'] })

      // зберігаємо попередній стан для rollback
      const previous = queryClient.getQueryData(['wishlist'])

      // оптимістично додаємо — серце заповнюється миттєво
      queryClient.setQueryData(['wishlist'], (old: string[]) => [
        ...old,
        listingId,
      ])

      return { previous }
    },
    onError: (_err, _listingId, context) => {
      // rollback якщо сервер повернув помилку
      queryClient.setQueryData(['wishlist'], context?.previous)
    },
    onSettled: () => {
      // синхронізуємо з сервером після завершення
      queryClient.invalidateQueries({ queryKey: ['wishlist'] })
    },
  })

  const remove = useMutation({
    mutationFn: (listingId: string) =>
      fetch(`/api/wishlist/${listingId}`, { method: 'DELETE' }),
    onMutate: async (listingId) => {
      await queryClient.cancelQueries({ queryKey: ['wishlist'] })
      const previous = queryClient.getQueryData(['wishlist'])

      queryClient.setQueryData(['wishlist'], (old: string[]) =>
        old.filter((id) => id !== listingId)
      )

      return { previous }
    },
    onError: (_err, _listingId, context) => {
      queryClient.setQueryData(['wishlist'], context?.previous)
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['wishlist'] })
    },
  })

  return { items, add: add.mutate, remove: remove.mutate }
}
```

---

## Server Components — серверний статичний стан

Каталог, деталі оголошення, профіль — рендеряться на сервері без будь-яких бібліотек.
Дані фетчаться напряму в компоненті через Prisma або fetch.

```ts
// app/listings/[id]/page.tsx — без useQuery, без useState
export default async function ListingPage({ params }) {
  const listing = await prisma.plant.findUnique({
    where: { id: params.id },
    include: { user: true, photos: true },
  })

  if (!listing) notFound()

  return <ListingDetails listing={listing} />
}
```

TanStack Query тут не потрібен — Server Component сам є "запитом".

---

## Структура файлів

```
stores/
  cart.ts          — кошик з persist
  wishlist.ts      — wishlist count badge
  ui.ts            — drawer, popup стани

hooks/
  useWishlist.ts   — TanStack Query: items + optimistic updates
  useListingStatus.ts — TanStack Query: polling статусу
  useExchangeRequest.ts — TanStack Query: статус обміну (post-MVP)
```

---

## QueryKey конвенція

Консистентні ключі запобігають конфліктам і спрощують інвалідацію.

```ts
// конвенція: [entity, id?, filters?]
['listings']                          // весь каталог
['listings', { city: 'Kyiv' }]       // каталог з фільтром
['listing-status', id]               // статус конкретного оголошення
['wishlist']                          // wishlist поточного юзера
['exchange-request', id]             // конкретний обмін
['profile', userId]                  // профіль юзера
```

---

## MVP scope

Стартуємо з Zustand — він потрібен одразу для кошика і UI стану.
TanStack Query додаємо в Phase 3 коли з'явиться сторінка активного обміну з polling.

| Фаза | Що додаємо |
|------|------------|
| Phase 2 (Core MVP) | Zustand: cart, ui |
| Phase 3 (Swap) | TanStack Query: polling, wishlist optimistic updates |
| Post-MVP | Zustand wishlist count синхронізація з TanStack Query |
