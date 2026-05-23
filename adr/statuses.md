# Listing & Transaction Status

Документація статусів оголошень і транзакцій платформи.
Два окремих entity з незалежними life cycles.

---

## Ключовий принцип

URL оголошення (`/listings/[id]`) — permanent. Ніколи не видаляється і не редіректиться.
Контент сторінки адаптується залежно від поточного `ListingStatus`.
Це забезпечує SEO, shareable лінки і коректну поведінку збережених закладок.

---

## Частина 1 — `ListingStatus` (оголошення)

### Enum

```prisma
enum ListingStatus {
  DRAFT       // чернетка, не опублікована
  ACTIVE      // активне, доступне в каталозі
  RESERVED    // заблоковане на час активної транзакції
  COMPLETED   // рослина передана, угода завершена
  ARCHIVED    // знято з публікації власником (soft delete)
}
```

### Transition map

```
DRAFT → ACTIVE
ACTIVE → RESERVED        (при старті purchase або swap flow)
ACTIVE → ARCHIVED        (власник знімає оголошення)
RESERVED → ACTIVE        (транзакція скасована або TTL вичерпано)
RESERVED → COMPLETED     (транзакція успішно завершена)
COMPLETED → (terminal)
ARCHIVED → ACTIVE        (власник повторно публікує)
```

### Поведінка в каталозі (грід)

| Статус | Показується в гріді | Вигляд картки |
|--------|---------------------|---------------|
| `DRAFT` | ні | — |
| `ACTIVE` | так | стандартний |
| `RESERVED` | так | opacity знижена, ціна замінена на бейдж "Зарезервовано" |
| `COMPLETED` | ні | — |
| `ARCHIVED` | ні | — |

`RESERVED` залишається видимим у каталозі — патерн Vinted/OLX.
Причина: юзер бачить що рослина існує; якщо резервація впаде — картка стане доступною без будь-яких дій.

Картка `RESERVED` — клікабельна, веде на сторінку деталей у readonly режимі.

### Поведінка на сторінці деталей (`/listings/[id]`)

Сторінка рендериться для **всіх** статусів. Статус-банер відображається над контентом.

| Статус | Банер | CTA кнопки |
|--------|-------|------------|
| `ACTIVE` | відсутній | Купити · Запропонувати обмін |
| `RESERVED` | "Зараз в процесі передачі. Якщо угода не відбудеться — рослина знову з'явиться." | disabled + tooltip |
| `COMPLETED` | "Рослина вже знайшла новий дім 🌿" + CTA "Переглянути схожі" | приховані |
| `ARCHIVED` | "Оголошення знято з публікації" | приховані |
| `DRAFT` | "Це чернетка. Тільки ти її бачиш." | Опублікувати |

### Revalidation стратегія

Сторінка деталей використовує ISR з коротким TTL + on-demand revalidation при мутації статусу.

```ts
// app/listings/[id]/page.tsx
export const revalidate = 60
```

При зміні `ListingStatus` в API route — обов'язково викликати:

```ts
revalidatePath(`/listings/${id}`)
```

Це гарантує що збережений лінк покаже актуальний стан максимум через 60 секунд без SSR на кожен запит.

### `RESERVED` + TTL автоскидання

Поле `reserved_until: DateTime?` на моделі `Plant`.

Vercel Cron Job перевіряє прострочені резервації і скидає `RESERVED → ACTIVE`.
Це простіше ніж saga pattern і достатньо для MVP.

```ts
// app/api/cron/release-reservations/route.ts
// Vercel Cron: */5 * * * *
```

---

## Частина 2 — `ExchangeRequestStatus` (swap flow)

### Enum

```prisma
enum ExchangeRequestStatus {
  PENDING       // requester надіслав запит, очікує відповіді seller
  ACCEPTED      // seller підтвердив, обидва готуються до відправки
  DECLINED      // seller відхилив
  BOTH_SHIPPED  // обидві сторони підтвердили відправку
  COMPLETED     // обидві сторони підтвердили отримання
  CANCELLED     // скасовано будь-якою стороною після ACCEPTED
}
```

### Transition map

```
PENDING → ACCEPTED      (seller підтверджує)
PENDING → DECLINED      (seller відхиляє)
ACCEPTED → BOTH_SHIPPED
ACCEPTED → CANCELLED
BOTH_SHIPPED → COMPLETED
BOTH_SHIPPED → CANCELLED  (крайній випадок, диспут — post-MVP)
```

### Двостороннє підтвердження

Замість одного поля `status = SHIPPED` — два boolean на моделі:

```prisma
requester_confirmed_shipped  Boolean @default(false)
seller_confirmed_shipped     Boolean @default(false)
requester_confirmed_received Boolean @default(false)
seller_confirmed_received    Boolean @default(false)
```

`BOTH_SHIPPED` виставляється автоматично коли обидва `_shipped` = `true`.
`COMPLETED` — коли обидва `_received` = `true`.

---

## Частина 3 — `OrderStatus` (purchase flow, post-MVP)

Описано для повноти. Purchase flow реалізується після MVP.

```prisma
enum OrderStatus {
  PENDING_PAYMENT  // створено, очікує оплати
  PAID             // оплата підтверджена
  SHIPPED          // seller підтвердив відправку
  DELIVERED        // buyer підтвердив отримання
  COMPLETED        // угода закрита
  CANCELLED        // скасовано (auto після TTL або вручну)
  DISPUTED         // відкрито диспут
}
```

`CANCELLED` замінює окремий стан `ERROR`. Причина скасування зберігається в полі
`cancellation_reason: String?` — не потрібно плодити окремі enum-значення для кожного edge case.

`PENDING_PAYMENT` + TTL — auto-cancel якщо оплата не надійшла протягом N годин
(той самий Cron патерн що і `RESERVED`).

---

## Частина 4 — `ActionLog`

Всі переходи між статусами пишуться в `ActionLog`. Записи immutable — ніколи не перезаписуються.

Це audit trail для:
- відлагодження транзакційних багів
- підтримки диспутів (post-MVP)
- аналітики воронки обміну

```prisma
model ActionLog {
  id          String   @id @default(uuid())
  user_id     String
  action_type String   // "LISTING_RESERVED", "SWAP_ACCEPTED", etc.
  entity_id   String   // id оголошення або транзакції
  metadata    Json?    // додаткові дані (reason, old_status, new_status)
  created_at  DateTime @default(now())
}
```

---

## MVP scope

Мінімум для Phase 3 (Walking Skeleton):

**ListingStatus:** `DRAFT | ACTIVE | RESERVED | COMPLETED | ARCHIVED`
**ExchangeRequestStatus:** `PENDING | ACCEPTED | DECLINED | COMPLETED | CANCELLED`

`OrderStatus` і `DISPUTED` — post-MVP.
