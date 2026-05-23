# API Routes & Caching Strategy

Документація серверних маршрутів, caching поведінки і on-demand revalidation.
Стек: Next.js App Router, Prisma, PostgreSQL (Neon).

---

## Ключові концепції

### ISR (Incremental Static Regeneration)

Сторінка генерується один раз як статичний HTML і кешується на CDN.
Після закінчення TTL — при наступному запиті Next.js перегенерує її у фоні й оновить кеш.
Юзер завжди отримує відповідь миттєво.

```ts
// app/listings/[id]/page.tsx
export const revalidate = 60 // секунди
```

Означає: "перегенеруй не частіше ніж раз на 60 секунд".

### On-demand revalidation

Активне інвалідування кешу при мутації. При зміні статусу в Route Handler —
Next.js негайно помічає кеш сторінки як stale. Наступний запит отримує свіжу версію.

`revalidatePath` доступний тільки в **Server Actions** і **Route Handlers**.
З client components не викликається.

```ts
import { revalidatePath } from 'next/cache'

// після мутації в БД:
revalidatePath(`/listings/${id}`)
```

### Повний цикл при зміні статусу

```
Юзер A: ініціює swap
  → POST /api/listings/[id]/status
    → prisma.plant.update({ status: 'RESERVED' })
    → revalidatePath('/listings/abc123')   ← кеш помічено stale

Юзер B: відкриває /listings/abc123
  → Next.js: кеш stale → запускає page function
    → prisma.plant.findUnique()            ← повертає RESERVED
    → рендериться з банером "Зарезервовано"
    → кеш оновлено
  → Юзер B бачить актуальну сторінку
```

`revalidate = 60` — fallback: навіть якщо `revalidatePath` не спрацював,
через 60 секунд кеш застаріє сам.

---

## Структура Route Handlers

```
app/
  api/
    listings/
      route.ts                  GET (каталог) · POST (створити)
      [id]/
        route.ts                GET (деталі) · PATCH · DELETE
        status/
          route.ts              PATCH (змінити статус)
        reserve/
          route.ts              POST (зарезервувати)
    exchange-requests/
      route.ts                  GET · POST
      [id]/
        route.ts                GET
        accept/
          route.ts              POST
        decline/
          route.ts              POST
        confirm-shipped/
          route.ts              POST
        confirm-received/
          route.ts              POST
    wishlist/
      route.ts                  GET · POST · DELETE
    users/
      [id]/
        route.ts                GET · PATCH
    cron/
      release-reservations/
        route.ts                GET (Vercel Cron)
```

---

## Listings

### `GET /api/listings` — каталог

Повертає тільки `ACTIVE` і `RESERVED` оголошення.
`DRAFT`, `COMPLETED`, `ARCHIVED` — не видаються публічно.

```ts
// app/api/listings/route.ts
import { prisma } from '@/lib/prisma'
import { NextRequest, NextResponse } from 'next/server'

export async function GET(req: NextRequest) {
  const { searchParams } = req.nextUrl
  const city = searchParams.get('city')
  const category = searchParams.get('category')

  const listings = await prisma.plant.findMany({
    where: {
      status: { in: ['ACTIVE', 'RESERVED'] },
      ...(city && { user: { city } }),
      ...(category && { category }),
    },
    include: {
      user: { select: { id: true, full_name: true, city: true, photo_url: true } },
      photos: { take: 1 },
    },
    orderBy: { created_at: 'desc' },
  })

  return NextResponse.json(listings)
}
```

### `GET /api/listings/[id]` — деталі оголошення

Повертає для **будь-якого** статусу — сторінка деталей рендериться завжди.

```ts
// app/api/listings/[id]/route.ts
export async function GET(
  _req: NextRequest,
  { params }: { params: { id: string } }
) {
  const listing = await prisma.plant.findUnique({
    where: { id: params.id },
    include: {
      user: { select: { id: true, full_name: true, city: true, photo_url: true } },
      photos: true,
    },
  })

  if (!listing) {
    return NextResponse.json({ error: 'Not found' }, { status: 404 })
  }

  return NextResponse.json(listing)
}
```

### `PATCH /api/listings/[id]/status` — зміна статусу

Мутація + on-demand revalidation. Захищений middleware (тільки власник або system).

```ts
// app/api/listings/[id]/status/route.ts
import { revalidatePath } from 'next/cache'
import { prisma } from '@/lib/prisma'
import { NextRequest, NextResponse } from 'next/server'

export async function PATCH(
  req: NextRequest,
  { params }: { params: { id: string } }
) {
  const { status, reason } = await req.json()

  const listing = await prisma.plant.update({
    where: { id: params.id },
    data: {
      status,
      // скидаємо reserved_until якщо статус більше не RESERVED
      ...(status !== 'RESERVED' && { reserved_until: null }),
    },
  })

  // логуємо перехід в ActionLog
  await prisma.actionLog.create({
    data: {
      entity_id: params.id,
      action_type: `LISTING_${status}`,
      metadata: { reason, previous_status: listing.status },
    },
  })

  // інвалідуємо кеш сторінки деталей
  revalidatePath(`/listings/${params.id}`)

  return NextResponse.json(listing)
}
```

### `POST /api/listings/[id]/reserve` — резервація

Атомарна операція: встановлює `RESERVED` + `reserved_until` + інвалідує кеш.

```ts
// app/api/listings/[id]/reserve/route.ts
import { revalidatePath } from 'next/cache'
import { prisma } from '@/lib/prisma'

const RESERVATION_TTL_HOURS = 24

export async function POST(
  _req: NextRequest,
  { params }: { params: { id: string } }
) {
  const reserved_until = new Date(
    Date.now() + RESERVATION_TTL_HOURS * 60 * 60 * 1000
  )

  const listing = await prisma.plant.update({
    where: {
      id: params.id,
      status: 'ACTIVE', // guard: резервуємо тільки активне
    },
    data: { status: 'RESERVED', reserved_until },
  })

  revalidatePath(`/listings/${params.id}`)

  return NextResponse.json(listing)
}
```

---

## Exchange Requests (Swap Flow)

### `POST /api/exchange-requests` — надіслати запит на обмін

```ts
// app/api/exchange-requests/route.ts
export async function POST(req: NextRequest) {
  const { listing_id, offered_plant_id } = await req.json()
  const session = await auth() // Auth.js

  // резервуємо listing
  await fetch(`/api/listings/${listing_id}/reserve`, { method: 'POST' })

  const exchangeRequest = await prisma.exchangeRequest.create({
    data: {
      listing_id,
      offered_plant_id,
      requester_id: session.user.id,
      status: 'PENDING',
    },
  })

  return NextResponse.json(exchangeRequest, { status: 201 })
}
```

### `POST /api/exchange-requests/[id]/accept` — seller підтверджує

```ts
// app/api/exchange-requests/[id]/accept/route.ts
export async function POST(
  _req: NextRequest,
  { params }: { params: { id: string } }
) {
  const exchangeRequest = await prisma.exchangeRequest.update({
    where: { id: params.id },
    data: { status: 'ACCEPTED' },
    include: { listing: true },
  })

  await prisma.actionLog.create({
    data: {
      entity_id: params.id,
      action_type: 'SWAP_ACCEPTED',
    },
  })

  return NextResponse.json(exchangeRequest)
}
```

### `POST /api/exchange-requests/[id]/confirm-shipped` — підтвердження відправки

Кожна сторона підтверджує окремо. Коли обидва = true → статус автоматично `BOTH_SHIPPED`.

```ts
// app/api/exchange-requests/[id]/confirm-shipped/route.ts
export async function POST(
  _req: NextRequest,
  { params }: { params: { id: string } }
) {
  const session = await auth()

  const current = await prisma.exchangeRequest.findUniqueOrThrow({
    where: { id: params.id },
    include: { listing: true },
  })

  const isRequester = current.requester_id === session.user.id

  const updated = await prisma.exchangeRequest.update({
    where: { id: params.id },
    data: {
      ...(isRequester
        ? { requester_confirmed_shipped: true }
        : { seller_confirmed_shipped: true }),
    },
  })

  // якщо обидва підтвердили — переводимо в BOTH_SHIPPED
  if (updated.requester_confirmed_shipped && updated.seller_confirmed_shipped) {
    await prisma.exchangeRequest.update({
      where: { id: params.id },
      data: { status: 'BOTH_SHIPPED' },
    })
  }

  return NextResponse.json(updated)
}
```

### `POST /api/exchange-requests/[id]/confirm-received` — підтвердження отримання

Аналогічно confirm-shipped. Коли обидва = true → `COMPLETED` + listing `COMPLETED`.

```ts
// app/api/exchange-requests/[id]/confirm-received/route.ts
export async function POST(
  _req: NextRequest,
  { params }: { params: { id: string } }
) {
  const session = await auth()

  const current = await prisma.exchangeRequest.findUniqueOrThrow({
    where: { id: params.id },
    include: { listing: true },
  })

  const isRequester = current.requester_id === session.user.id

  const updated = await prisma.exchangeRequest.update({
    where: { id: params.id },
    data: {
      ...(isRequester
        ? { requester_confirmed_received: true }
        : { seller_confirmed_received: true }),
    },
  })

  if (updated.requester_confirmed_received && updated.seller_confirmed_received) {
    // закриваємо обмін
    await prisma.exchangeRequest.update({
      where: { id: params.id },
      data: { status: 'COMPLETED' },
    })

    // закриваємо оголошення
    await prisma.plant.update({
      where: { id: current.listing_id },
      data: { status: 'COMPLETED', reserved_until: null },
    })

    // інвалідуємо кеш сторінки деталей
    revalidatePath(`/listings/${current.listing_id}`)
  }

  return NextResponse.json(updated)
}
```

---

## Cron: автоскидання прострочених резервацій

Vercel Cron Job. Запускається кожні 5 хвилин.
Скидає `RESERVED → ACTIVE` для оголошень де `reserved_until` вже минув.

Конфігурація в `vercel.json`:

```json
{
  "crons": [
    {
      "path": "/api/cron/release-reservations",
      "schedule": "*/5 * * * *"
    }
  ]
}
```

Route Handler:

```ts
// app/api/cron/release-reservations/route.ts
import { revalidatePath } from 'next/cache'
import { prisma } from '@/lib/prisma'
import { NextRequest, NextResponse } from 'next/server'

export async function GET(req: NextRequest) {
  // захист: Vercel передає секретний заголовок
  const authHeader = req.headers.get('authorization')
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const expired = await prisma.plant.findMany({
    where: {
      status: 'RESERVED',
      reserved_until: { lt: new Date() },
    },
    select: { id: true },
  })

  if (expired.length === 0) {
    return NextResponse.json({ released: 0 })
  }

  await prisma.plant.updateMany({
    where: { id: { in: expired.map((p) => p.id) } },
    data: { status: 'ACTIVE', reserved_until: null },
  })

  // інвалідуємо кеш для кожного звільненого оголошення
  expired.forEach((p) => revalidatePath(`/listings/${p.id}`))

  return NextResponse.json({ released: expired.length })
}
```

---

## Сторінка деталей — ISR + revalidation

```ts
// app/listings/[id]/page.tsx
import { notFound } from 'next/navigation'
import { prisma } from '@/lib/prisma'

export const revalidate = 60 // fallback TTL

export default async function ListingPage({
  params,
}: {
  params: { id: string }
}) {
  const listing = await prisma.plant.findUnique({
    where: { id: params.id },
    include: {
      user: { select: { id: true, full_name: true, city: true, photo_url: true } },
      photos: true,
    },
  })

  if (!listing) notFound()

  return (
    <>
      {listing.status !== 'ACTIVE' && (
        <StatusBanner status={listing.status} />
      )}
      <ListingDetails listing={listing} />
    </>
  )
}
```

```ts
// components/StatusBanner.tsx
const BANNER_CONFIG = {
  RESERVED: {
    text: 'Зараз в процесі передачі. Якщо угода не відбудеться — рослина знову з\'явиться.',
    variant: 'warning',
  },
  COMPLETED: {
    text: 'Рослина вже знайшла новий дім 🌿',
    variant: 'success',
    cta: { label: 'Переглянути схожі', href: '/listings' },
  },
  ARCHIVED: {
    text: 'Оголошення знято з публікації.',
    variant: 'neutral',
  },
  DRAFT: {
    text: 'Це чернетка. Тільки ти її бачиш.',
    variant: 'neutral',
    cta: { label: 'Опублікувати', action: 'publish' },
  },
} as const
```

---

## Правила

- `revalidatePath` викликається **після кожної мутації** що змінює `ListingStatus`
- Route Handlers що мутують стан — завжди логують в `ActionLog`
- Резервація оголошення — атомарна: `status + reserved_until` в одному `prisma.update`
- Cron захищений `CRON_SECRET` env змінною — Vercel передає її автоматично
- `prisma.updateMany` у Cron не підтримує `include` — окремий `findMany` перед апдейтом
