# PlantCard — специфікація компонента

## Огляд

Основна картка оголошення платформи. Один компонент, що використовується в чотирьох
контекстах (каталог, схожі оголошення, SwapPopup, рекомендації в кошику) і адаптується
єдиним механізмом:

- **`compact` проп (boolean)** — керує всім: типографікою (`name`/`price`/`city`/`tag`),
  padding (`.meta`/`.footer`), розміром `WishlistToggleButton`. Жодного впливу на форму
  картки — фото завжди `aspect-ratio: 1/1`, у будь-якому контексті.

`@container` query свідомо не використовується зараз — мікс `compact`-пропу з CSS-кверями
для тієї самої задачі (типографіка) визнаний небажаним. Адаптація під мобільний каталог
і дуже широкі екрани — окреме рішення пізніше, поза цією версією спеки.

Композиція з трьох власних sub-primitives, кожен описаний в межах цього файлу — єдиний
споживач усіх трьох це `PlantCard`, окремих spec-файлів вони не потребують:

- `WishlistToggleButton` — toggle-іконка вішліста, top-right.
- `PhotoBadge` — статус-бейдж на фото, top-left.
- `CheckedIndicator` — індикатор вибору в SwapPopup, top-right (замінює wishlist-кнопку
  в режимі вибору, не одночасно з нею).

---

## Контексти використання

| Контекст | `compact` | Колонок |
|---|---|---|
| Каталог desktop | `false` | 4 |
| Схожі оголошення (деталі) | `false` | 4 |
| SwapPopup "Оберіть рослину" | `true` | 3 |
| Рекомендації в CartDrawer | `true` | 1 (вертикальний список) |

Мобільний каталог і дуже широкі екрани — не визначені цією версією спеки, рішення
відкладено.

---

## Props

```ts
type ImageStatus      = 'loading' | 'loaded' | 'error'
type ListingStatus    = 'reserved' | 'completed' | 'archived'
type TradeType         = 'swapOnly' | 'saleOnly'
type CareLevel          = 'easy' | 'medium' | 'hard'
type PhotoBadgeTone     = 'gray' | 'white'

interface PlantCardProps {
  id: string
  name: string
  price: number | null              // null якщо tradeType === 'swapOnly'
  city: string
  photoUrl: string
  careLevel: CareLevel
  status?: ListingStatus | null      // публічно в каталозі реально приходить лише 'reserved'
  tradeType?: TradeType | null

  compact?: boolean                  // тільки розмір WishlistToggleButton (24 замість 28); default: false
  priority?: boolean                 // прокидається в next/image; default: false

  isWishlisted: boolean
  onWishlistToggle: () => void

  onAddToCart?: () => void           // "В кошик" (default) / "Купити" (saleOnly, full-width)
  onSwap?: () => void                 // "Обмін" (default) / "Запропонувати обмін" (swapOnly, full-width)

  checked?: boolean                  // якщо визначений — картка в режимі вибору (SwapPopup)
  onCheckedChange?: () => void
}
```

`status`/`tradeType` — дані з API, не вигадуються компонентом. `completed`/`archived`
теоретично не повинні долітати в каталожний контекст (фільтрація на рівні запиту
бекенду), але умови `showReservedBadge`/`showTradeTypeBadge` обробляють їх безпечно
(просто не показують бейдж), а не кидають помилку — захист на випадок, якщо
фільтрація колись зламається.

### Режим вибору (`checked`)

`checked` — не альтернативна іконка поруч із wishlist, а перемикач режиму всієї
верхньої частини фото:

| `checked` | Рендер top-right | Рендер `PhotoBadge` |
|---|---|---|
| `undefined` (каталог/кошик) | `WishlistToggleButton` | за `showReservedBadge`/`showTradeTypeBadge` |
| `false` (SwapPopup, не вибрано) | нічого | нічого |
| `true` (SwapPopup, вибрано) | `CheckedIndicator` | нічого |

Коли `checked` визначений — `status`/`tradeType` ігноруються для відображення (вибір
свого оголошення для обміну не повинен показувати "зарезервовано" чи "тільки продаж" —
це інформація іншого контексту). Дані `status`/`tradeType` можуть передаватись, просто
не впливають на вигляд у цьому режимі.

---

## Анатомія

```
.card
 ├─ .photo                     aspect-ratio: 1/1 (завжди)
 │   ├─ <Image>                next/image, loading={priority ? 'eager' : 'lazy'}
 │   ├─ .badgeSlot              top-left, обгортка з позиціюванням; умовна
 │   │   └─ ReservedPhotoBadge | TradePhotoBadge   взаємовиключні, кожен сам рендерить PhotoBadge
 │   ├─ .topRightSlot           top-right, обгортка з позиціюванням
 │   │   └─ WishlistToggleButton | CheckedIndicator   взаємовиключні, всередині однієї обгортки
 │   └─ .actions                bottom overlay, тільки на hover (@media hover:hover),
 │                               умовний — див. "Hover actions" нижче
 ├─ .meta
 │   ├─ .name
 │   ├─ .price  | .swapLabel    залежно від tradeType
 │   └─ .city
 └─ .footer
     └─ .tag                    рівень догляду — завжди, незалежно від status
```

Позиціювання (`position: absolute; top; right`/`left`) живе в `.badgeSlot`/`.topRightSlot` —
класах `PlantCard.module.css`, застосованих до `<div>`-обгортки. Жоден з `PhotoBadge`/
`ReservedPhotoBadge`/`TradePhotoBadge`/`WishlistToggleButton`/`CheckedIndicator` не приймає
`className` ззовні — кожен має лише власні стилі у власному модулі, без можливості батька
вплинути на них напряму (глобальне правило, не специфічне для цього компонента).

Footer ніколи не замінюється на статус-інформацію — `RESERVED` показується лише через
`PhotoBadge` на фото й через `opacity` картки, тег рівня догляду лишається.

Root-елемент `.card` (`<div>` з прихованим `Link` всередині, чи `<button>` без `Link`) —
залежить від режиму, див. "Навігація" нижче.

---

## Стани картки

| Стан | Тригер | Фото (opacity) | `PhotoBadge` | Footer |
|---|---|---|---|---|
| `ACTIVE`, ціна, обмін доступний | `status` відсутній/`null`, `tradeType` відсутній | 1 | — | тег |
| `ACTIVE`, лише продаж | `tradeType === 'saleOnly'` | 1 | `tone="white"`, "Тільки продаж" | тег |
| `ACTIVE`, лише обмін | `tradeType === 'swapOnly'` | 1 | `tone="white"`, "Тільки обмін" | тег |
| `RESERVED` | `status === 'reserved'` | 0.82 (картка), 0.68 (фото) | `tone="gray"`, "Зарезервовано" | тег |
| Вибір (SwapPopup) | `checked` визначений | 1 | — (нівелюється) | тег |
| Помилка фото | `imageStatus === 'error'` | 1 | за `status`/`tradeType` як завжди | тег |
| Skeleton | окремий компонент `PlantCardSkeleton`, не сам `PlantCard` | — | — | однотонна заглушка (тимчасово) |

`RESERVED` + `checked === true` одночасно не передбачені бізнес-логікою (SwapPopup
показує лише активні оголошення власника); якщо все ж трапиться — `checked` має
пріоритет, `PhotoBadge`/opacity-затемнення не застосовуються.

### Стан фото (`imageStatus`) — окремо від `Skeleton`

Це новий, відсутній досі в документації стан: дані картки (назва, ціна, тег) вже є,
просто байти фото ще не прийшли. Не плутати зі `Skeleton` (немає взагалі ніяких даних).

| `imageStatus` | Коли | Вигляд `.photo` |
|---|---|---|
| `loading` | до `onLoad`/`onError` на `<Image>` | shimmer лише в межах `.photo`; `.meta`/`.footer` вже видні нормально |
| `loaded` | після `onLoad` | фото, fade-in `150ms` |
| `error` | після `onError` | заглушка з іконкою (вже задокументований дизайн) |

`priority` прокидається з батьківського грід-компонента (`CatalogGrid`/`SimilarListings`/
`SwapPopup`) в `PlantCard`, а звідти — в `<Image priority={priority}>`. Скільки саме перших
карток вважати "above the fold" — рішення грід-компонента, не `PlantCard`: залежить від
кількості колонок (4 на каталозі/"Схожих", 3 у SwapPopup) і
висоти viewport, тож фіксоване число тут недоречне. Орієнтир — `колонок × 2` (приблизно
два повні ряди), але остаточне правило належить окремій спеці `CatalogGrid`, не цьому
документу. Без `priority` навіть видимі одразу картки чекають lazy-черги, що шкодить LCP.
Решта карток — дефолтний `loading="lazy"`, без додаткової логіки: браузер сам не робить
мережевий запит за байтами, поки картка не наблизиться до viewport.

---

## Hover actions (`.actions`)

Overlay внизу `.photo`, з'являється лише на hover (`@media (hover: hover)`), `opacity: 0 → 1`,
`background: linear-gradient(to top, rgba(0,0,0,0.18) 0%, transparent 100%)`. Окремий від
hover-ефекту самої картки (`translateY`/`box-shadow`) — той же тригер, різні властивості.

| Контекст | Рендериться? | Кнопки |
|---|---|---|
| `compact === true` | ні | — |
| `checked` визначений | ні | — |
| `status === 'reserved'` | ні | — |
| `tradeType` відсутній (обидва способи) | так, 2 кнопки `flex: 1` кожна | "В кошик" → `onAddToCart` · "Обмін" → `onSwap` |
| `tradeType === 'swapOnly'` | так, 1 кнопка `width: 100%` | "Запропонувати обмін" → `onSwap` |
| `tradeType === 'saleOnly'` | так, 1 кнопка `width: 100%` | "Купити" → `onAddToCart` |

`onAddToCart` — єдиний handler незалежно від `tradeType`; "Купити" на sale-only картці
це той самий "додати в кошик", не окремий buy-now flow. Лейбл "Купити" зберігається в UI
свідомо (підтверджено), хоча фактична дія — додавання в кошик, не миттєва покупка;
користувач отримає `CartDrawer`, не checkout.

### Типографіка та padding

| Елемент | default | `compact` |
|---|---|---|
| `.name` / `.price` / `.swapLabel` | 14px | 12px |
| `.city` | 12px | 10px |
| `.tag` (footer) | 12px (`<Tag size="m">` чи відповідний) | 10px (`<Tag size="s">`) |

`.tag` — значення йдуть із самого `Tag`-компонента через `size`-проп, не дублюються
тут як окремий токен; `PlantCard` лише обирає правильний `size` залежно від `compact`.

| Контейнер | default padding | `compact` padding | default gap | `compact` gap |
|---|---|---|---|---|
| `.meta` (name + price/swapLabel + city) | `10px 12px` | `10px` (рівномірно) | `4px` | `2px` |
| `.footer` (tag) | `0 12px 12px` | `10px` (рівномірно) | — | — |

### Локальні токени (`PlantCard.module.css`)

```css
--plant-card-actions-padding:    10px;
--plant-card-actions-gap:        6px;
--plant-card-actions-bg:         linear-gradient(to top, rgba(0, 0, 0, 0.18) 0%, transparent 100%);

--plant-card-action-radius:      var(--radius-full);
--plant-card-action-font-size:   11px;
--plant-card-action-cart-bg:     rgba(255, 255, 255, 0.88);
--plant-card-action-cart-color:  var(--color-text-primary); /* глобальний токен */
--plant-card-action-swap-bg:     rgba(255, 255, 255, 0.28);
--plant-card-action-swap-border: rgba(255, 255, 255, 0.45);
--plant-card-action-swap-color:  #fff;

--plant-card-name-font-size:     14px;
--plant-card-city-font-size:     12px;
--plant-card-city-color:         #a89e88;

--plant-card-meta-padding:       10px 12px;
--plant-card-meta-gap:           4px;
--plant-card-footer-padding:     0 12px 12px;

/* .compact { } — override типографіки й padding, решта токенів спільні */
.compact {
  --plant-card-name-font-size:   12px;
  --plant-card-city-font-size:   10px;
  --plant-card-meta-padding:     10px;
  --plant-card-meta-gap:         2px;
  --plant-card-footer-padding:   10px;
}
```

`.badgeSlot` / `.topRightSlot` — позиціюючі обгортки (не токени, самі правила):

```css
.badgeSlot,
.topRightSlot {
  position: absolute;
  pointer-events: auto;
  z-index: 1;
}
.badgeSlot     { top: 8px; left: 8px; }
.topRightSlot  { top: 8px; right: 8px; }
```

---

## Skeleton — тимчасова заглушка

Справжній `ui/Skeleton` розробляється окремо (інша сесія) і ще не готовий. Зараз —
мінімальний, явно тимчасовий стаб: окремий компонент `PlantCardSkeleton`, не режим
`PlantCard` (без `loading`-пропу на `PlantCardProps` — не плодимо умовну логіку для
тимчасового рішення). Без пропів, фіксовані розміри що повторюють `.card` (`aspect-ratio:
1/1` для фото, ті самі padding токени для `.meta`/`.footer`), один колір замість shimmer-
анімації — `background: var(--color-inverse-muted-2)` на всіх трьох блоках (фото/назва-
рядок/тег-рядок). Коли `ui/Skeleton` буде готовий — `PlantCardSkeleton.tsx` видаляється,
`CatalogGrid` перемикається на справжній компонент; ніяких слідів стаба не лишається в
самому `PlantCard`.

---

## Навігація

Уся картка — посилання на сторінку деталей (`/listings/{id}`), окрім `checked`-режиму,
де клік означає вибір, а не перехід. Це різні root-елементи `.card`, не один елемент
з умовною поведінкою:

### Default / compact (каталог, схожі, кошик)

`.card` лишається `<div>`. Невидимий `<Link href={`/listings/${id}`}>` — **сіблінг**
візуальному контенту, не обгортка навколо нього (`position: absolute; inset: 0`),
**першим** у DOM-порядку, перед `.photo`/`.meta`/`.footer` — без явного `z-index` він
би малювався під ними за дефолтним DOM-порядком, що й потрібно:

```
.card (div, position: relative)
 ├─ <Link>                       position: absolute; inset: 0; z-index: 0; pointer-events: auto
 └─ .photo / .meta / .footer    pointer-events: none
```

`.topRightSlot`, `.badgeSlot` і кнопки `.actions` — `pointer-events: auto` **і**
`z-index: 1` явно в `PlantCard.module.css` (вони вже вище за `Link` за DOM-порядком,
бо вкладені в `.photo`, що йде після `Link`, але явний `z-index` — захисна практика,
щоб не залежати виключно від порядку рендеру, якщо хтось його змінить пізніше).

`aria-label` на `Link`: `` `${name}, ${price ? price + ' грн' : 'обмін'}` `` — повний
контекст для скрін-рідера без потреби заходити всередину картки окремо.

`.card:focus-within` отримує той самий lifted-вигляд, що й `.card:hover` (`translateY`,
`box-shadow`, світліший фон) — клавіатурний фокус на `Link` чи на `WishlistToggleButton`
візуально підтверджує активність картки.

### Checked-режим (SwapPopup)

`.card` — не `<div>` з `Link`, а `<button type="button" aria-pressed={checked} onClick={onCheckedChange}>`.
`Link` не рендериться взагалі. У цьому режимі вкладених interactive-елементів немає
(`WishlistToggleButton`/`.actions` вже придушені раніше), тож конфлікту клік-в-клік
немає — нативна кнопка покриває весь UX без додаткових pointer-events правил.

---

## Edge cases

- Відсутнє місто — не хендлиться в картці; вважається помилкою даних на рівні каталогу.
- Довга назва рослини — `text-overflow: ellipsis`, `white-space: nowrap` (вже в CSS).
- `RESERVED` + `checked` — заборонено типобезпечно неможливо (різні режими використання
  компонента), пріоритет `checked`, якщо все ж трапиться.
- Hover-ефекти — лише `@media (hover: hover)`: `transform: translateY(-2px)`,
  світліший `background`, `border-color`, і `box-shadow: var(--shadow-card-hover)`
  (глобальний токен, вже існує в `globals.css`). Свідома зміна щодо `ux-rules.md` і
  `components-legacy.md` — обидва документи досі забороняють `box-shadow` на hover
  у списку "Що не робимо"; ці два файли потребують окремого оновлення під цю спеку,
  бо зараз вони суперечать одне одному.

---

## Sub-component: `WishlistToggleButton`

Презентаційний primitive, без знання про `useWishlist`/auth-gate — повністю controlled.

```ts
interface WishlistToggleButtonProps {
  pressed: boolean
  onClick: () => void
  compact?: boolean                 // 24px замість 28px; default: false
  'aria-label': string
}
```

### Розміри

| `compact` | Розмір кнопки | Розмір іконки | Контекст |
|---|---|---|---|
| `false` | 28px | 14px | каталог, схожі оголошення |
| `true` | 24px | 12px | CartDrawer, SwapPopup |

### Стани

| Стан | Іконка `Heart` | Trigger |
|---|---|---|
| Default (`pressed=false`) | `fill="none"`, контур | `aria-pressed="false"` |
| Toggled (`pressed=true`) | `fill="currentColor"` | `aria-pressed="true"` |
| Hover (`@media (hover: hover)`) | `transform: scale(1.18)` | однаково для default/toggled |
| Focus-visible | focus ring токен | клавіатурна навігація |

`aria-label` — статичний текст ("Додати до вішліста"), не змінюється залежно від
`pressed` — стан передає `aria-pressed`, не лейбл (WAI-ARIA APG toggle button pattern).

Іконка — хардкодний `Heart` з `lucide-react`, не проп. Колір (`stroke`/`fill`) —
`currentColor`, встановлюється через CSS `color` на батьківському `<button>`, не
напряму на SVG. Позиціювання (`top-right`) — відповідальність `PlantCard`: компонент
рендериться всередині `.topRightSlot` (`PlantCard.module.css`), сам про позицію не знає.

### Локальні токени (`WishlistToggleButton.module.css`)

```css
--wishlist-toggle-size:      28px;
--wishlist-toggle-icon-size: 14px;

--wishlist-toggle-bg:               rgba(255, 255, 255, 0.55);
--wishlist-toggle-backdrop-filter:  blur(8px) saturate(1.4);
--wishlist-toggle-border:           rgba(255, 255, 255, 0.75);
--wishlist-toggle-color:            var(--color-text-accent); /* глобальний токен */

--wishlist-toggle-hover-scale:      1.18;
--wishlist-toggle-focus-ring:       rgba(212, 145, 158, 0.18); /* дефолт з CLAUDE.md */
--wishlist-toggle-focus-ring-size:  2px;

/* .compact { } — override двох токенів розміру, решта спільні */
.compact {
  --wishlist-toggle-size:      24px;
  --wishlist-toggle-icon-size: 12px;
}
```

`--wishlist-toggle-color` посилається на глобальний `--color-text-accent` (вже існує
в `globals.css`) — це свідомий виняток із "токени локально": сам колір — глобальна
сутність (`accent`), локальний тут лише факт, що саме цей компонент його використовує.

---

## Sub-component: `PhotoBadge`

Чистий примітив без знання про домен — тільки `tone` і `label`, скалярні проп. Не
використовується напряму в `PlantCard` — тільки через `ReservedPhotoBadge`/
`TradePhotoBadge` (нижче).

```ts
interface PhotoBadgeProps {
  tone: PhotoBadgeTone
  label: string
}
```

| `tone` | background | color | Використання |
|---|---|---|---|
| `gray` | `rgba(122, 136, 120, 0.72)` + `backdrop-filter: blur(6px)` | `#fff` | `ListingStatus` |
| `white` | `rgba(255, 255, 255, 0.82)` | `var(--color-text-secondary)` | `TradeType` |

Анатомія: крапка (`background: currentColor`) + текст, `display: inline-flex`,
`border-radius: var(--radius-full)`. Позиціювання (`top-left`) — відповідальність
`PlantCard`: компонент рендериться всередині `.badgeSlot`, сам про позицію не знає.

### `ReservedPhotoBadge` і `TradePhotoBadge`

Замість одного резолвера + одного `PhotoBadge` — два конкретні компоненти, кожен
явно прив'язаний до своєї доменної сутності. Обидва внутрішньо рендерять `PhotoBadge`,
самі лишаючись у `features/PlantCard/`, не в `ui/`:

```ts
// ReservedPhotoBadge.tsx — без пропів, один фіксований вигляд
function ReservedPhotoBadge(): ReactElement {
  return <PhotoBadge tone="gray" label="Зарезервовано" />
}

// TradePhotoBadge.tsx
interface TradePhotoBadgeProps {
  tradeType: TradeType
}
function TradePhotoBadge({ tradeType }: TradePhotoBadgeProps): ReactElement {
  return <PhotoBadge tone="white" label={TRADE_TYPE_LABELS[tradeType]} />
}
```

`TRADE_TYPE_LABELS` — мапа в `PlantCard.constants.ts`, не inline у `TradePhotoBadge.tsx`
(правило `CLAUDE.md`: доменні LABEL-мапи живуть в `Entity.constants.ts`, не в файлі
компонента).

`PlantCard` вирішує, який з двох рендерити (чи жодного), зберігаючи той самий пріоритет,
що й раніше:

```ts
const showReservedBadge = status === 'reserved'
const showTradeTypeBadge = !showReservedBadge && tradeType != null
```

`reserved` виграє пріоритет над `tradeType`, бо резервація нівелює можливість отримати
рослину будь-яким способом — показувати "тільки продаж" на зарезервованій рослині
вводить в оману. `completed`/`archived` свідомо не мають кейсу — у каталозі не з'являються.

Обидва бейджі — **завжди видимі**, коли умова виконана, не hover-only. `opacity`/hover-
reveal — це поведінка `.actions`, не `PhotoBadge` чи його обгорток.

### `ListingStatus` — два контексти, дві мапи лейблів

`ListingStatus` — єдиний enum, що описує lifecycle оголошення. Використовується і в
`PhotoBadge` (каталог, лаконічно), і в уже готовому `ListingStatusInline` (профіль "Мої
оголошення", повний lifecycle). Лейбли різні для кожного контексту — мапа лейблів
лежить у відповідному компоненті, не в самому enum:

| `ListingStatus` | `PhotoBadge` label | `ListingStatusInline` label |
|---|---|---|
| `reserved` | "Зарезервовано" | "В процесі передачі" |
| `completed` | — (не рендериться) | "Знайшла новий дім" |
| `archived` | — (не рендериться) | "Знято з публікації" |

### Локальні токени (`PhotoBadge.module.css`)

```css
--photo-badge-font-size: 10px;
--photo-badge-padding:   2px 8px;
--photo-badge-dot-size:  5px;
--photo-badge-gap:       5px;

--photo-badge-gray-bg:    rgba(122, 136, 120, 0.72);
--photo-badge-gray-color: #fff;
--photo-badge-gray-backdrop-filter: blur(6px);

--photo-badge-white-bg:    rgba(255, 255, 255, 0.82);
--photo-badge-white-color: var(--color-text-secondary); /* глобальний токен */
```

---

## Sub-component: `CheckedIndicator`

Презентаційний, без пропів узагалі — рендериться лише коли `checked === true` (умова
на рівні `PlantCard`).

```ts
// CheckedIndicatorProps не існує — компонент без пропів
function CheckedIndicator(): ReactElement { ... }
```

Коло 20px — єдиний розмір, без `compact`-варіанта взагалі (на відміну від
`WishlistToggleButton`, тут нема контексту з іншим розміром). Іконка checkmark 14px,
`stroke: #fff`, `stroke-width: 2.25`. Позиціювання — той самий `.topRightSlot`, що й
у `WishlistToggleButton` (взаємовиключні, обидва без власної позиції).

### Локальні токени (`CheckedIndicator.module.css`)

```css
--checked-indicator-size:      20px;
--checked-indicator-icon-size: 14px;
--checked-indicator-bg:        var(--color-accent-main); /* глобальний токен */
```

Картка в `checked`-стані отримує єдиний `box-shadow` (без `border` — inset box-shadow,
як і всюди в проєкті):

```css
box-shadow:
  inset 0 0 0 1.5px var(--color-accent-main),
  0 0 0 3px rgba(196, 122, 136, 0.15);
```

Hover-ефект картки вимикається (`transform: none`) — вибрана картка нерухома.

---

## Файлова структура

```
components/
  features/
    PlantCard/
      PlantCard.tsx                    — основний компонент, композиція
      PlantCard.module.css             — токени картки, типографіка/padding (compact)
      WishlistToggleButton.tsx
      WishlistToggleButton.module.css
      PhotoBadge.tsx
      PhotoBadge.module.css
      ReservedPhotoBadge.tsx
      TradePhotoBadge.tsx
      CheckedIndicator.tsx
      CheckedIndicator.module.css
      PlantCardSkeleton.tsx             — тимчасова заглушка, видалити коли готовий ui/Skeleton
      PlantCardSkeleton.module.css      — тимчасова заглушка
      PlantCard.constants.ts            — TRADE_TYPE_LABELS
      types.ts                          — ListingStatus, TradeType, PlantCardProps
      index.ts                          — re-export PlantCard (sub-primitives не експортуються назовні)
app/
  dev/
    plant-card/
      page.tsx                          — усі стани/варіанти, окремо від цього кроку
      PlantCardDemo.module.css
```

`WishlistToggleButton`/`PhotoBadge`/`CheckedIndicator` не отримують власного `index.ts`
експорту з пакету — вони private до `PlantCard`, імпортуються лише всередині нього.

---

## Заборонено

- Видалення `box-shadow: var(--shadow-card-hover)` на hover — тепер частина спеки.
- Експорт sub-primitives назовні `features/PlantCard/` — single-consumer, не передбачені
  для повторного використання.
- `PhotoBadge`/`WishlistToggleButton`/`CheckedIndicator`, що знають про `status`/`tradeType`/
  `useWishlist` — домен резолвиться в `ReservedPhotoBadge`/`TradePhotoBadge`/`PlantCard`,
  не в самих примітивах.
- LABEL-мапи (`TRADE_TYPE_LABELS` тощо) inline у файлі компонента — тільки
  `PlantCard.constants.ts`.
- Кастомна `IntersectionObserver`-логіка для lazy-load фото — `next/image` з
  `loading="lazy"` (дефолт) це вже вирішує; `PlantCard` лише прокидає `priority`.
- Один і той самий розмірний токен для `WishlistToggleButton` і `CheckedIndicator` —
  вони незалежні: 20px фіксовано для `CheckedIndicator`, 28/24px для
  `WishlistToggleButton` залежно від `compact`.
- Виносити кнопки `.actions` (cart/swap/buy) в окремий sub-component — на відміну від
  `WishlistToggleButton`/`PhotoBadge`/`CheckedIndicator`, тут немає toggle-логіки чи
  повторного використання поза `.photo`, що варто б капсулювати; звичайні `<button>` з
  класами `PlantCard.module.css`.
- `stopPropagation()` для розв'язання конфлікту клік-навігація — лише `pointer-events`
  і DOM-порядок (`Link` першим), описано в розділі "Навігація".
- Обгортати весь `.card` у `<Link>` (замість sibling-паттерну) — тоді
  `WishlistToggleButton`/`.actions` стають interactive-елементами всередині interactive
  `<a>`, що невалідний HTML і обмежена доступність.
