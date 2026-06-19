# Skeleton — специфікація компонента

**Дата:** 2026-06-18
**Статус:** готово до імплементації

---

## Контекст і роль

Skeleton — Suspense fallback останньої надії, не основна UX-стратегія.

**Коли показується:**
- Cold first load + повільна мережа → `<Suspense fallback={<CatalogGridSkeleton />}>` навколо `CatalogGrid`

**Коли НЕ показується:**
- Зміна фільтрів → `keepPreviousData` + pending-стан на grid
- Повторні візити на каталог → TanStack Query cache, дані вже є
- Більшість навігацій → prefetch або staleTime покриває потребу

---

## Scope цієї сесії

| Компонент | Тип | Призначення |
|---|---|---|
| `ui/Skeleton` | primitive | domain-agnostic shimmer-блок |
| `features/catalog/CatalogGridSkeleton` | composite | Suspense fallback для каталогу |

`PlantDetailsSkeleton`, `ProfileHeaderSkeleton`, `OrderItemSkeleton` — поза scope, окремі сесії.

---

## Нові токени в `globals.css`

Додати в секцію `/* === COMPONENTS ===` перед `/* === HEADER ===`:

```css
/* === SKELETON ===
   Shimmer-анімація для loading placeholder.
   Кольори похідні від page background (base-bg-2 / base-bg-3).
   Не використовувати поза Skeleton компонентом.
============================================ */
--skeleton-bg-base:      rgba(195, 206, 193, 0.70);
--skeleton-bg-highlight: rgba(232, 238, 230, 0.90);
--skeleton-duration:     1.5s;
```

**Обґрунтування кольорів:** `--skeleton-bg-base` — приглушений sage, видимий на `#e8ebe8` фоні сторінки. `--skeleton-bg-highlight` — світліший peak shimmer. Обидва в тому ж hue що й `--color-base-bg-*` — виглядає як "вицвілий" варіант фону, не чужорідний елемент.

---

## `ui/Skeleton`

### Props

```ts
interface SkeletonProps {
  shape?: 'rect' | 'circle' | 'text'; // default: 'rect'
  width?: string;   // CSS value: '100%', '72px'. Default: '100%'
  height?: string;  // CSS value: '120px', '1em'. Default: '1em'
}
```

`width` і `height` передаються через `style` prop як CSS custom properties:

```tsx
style={{
  '--skeleton-width': width ?? '100%',
  '--skeleton-height': height ?? '1em',
} as React.CSSProperties}
```

### Anatomy

Один `<span>` (inline-block) — не `<div>`, щоб можна було вставляти в текстові контексти.

```tsx
<span
  aria-hidden="true"
  className={cn(styles.skeleton, styles[`skeleton--${shape}`])}
  style={{ '--skeleton-width': width ?? '100%', '--skeleton-height': height ?? '1em' } as React.CSSProperties}
/>
```

### States

| Стан | Опис |
|---|---|
| default | shimmer-анімація, завжди активна |

Немає hover, focus, disabled — компонент ніколи не інтерактивний.

### CSS (Skeleton.module.css)

```css
@keyframes skeleton-shimmer {
  0%   { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

.skeleton {
  display: inline-block;
  width: var(--skeleton-width, 100%);
  height: var(--skeleton-height, 1em);
  background: linear-gradient(
    90deg,
    var(--skeleton-bg-base)      25%,
    var(--skeleton-bg-highlight) 50%,
    var(--skeleton-bg-base)      75%
  );
  background-size: 200% 100%;
  animation: skeleton-shimmer var(--skeleton-duration) infinite linear;
}

/* shape variants */
.skeleton--rect   { border-radius: var(--radius-md); }
.skeleton--circle { border-radius: var(--radius-full); }
.skeleton--text   { border-radius: var(--radius-sm); }
```

### Accessibility

- `aria-hidden="true"` завжди — placeholder-геометрія нічого не говорить screen reader.
- Контейнер, що рендерить skeleton-и (`CatalogGridSkeleton`), відповідає за `role="status"` і `aria-label`.

---

## `features/catalog/CatalogGridSkeleton`

### Призначення

Suspense fallback для `<CatalogGrid />`. Рендерить 4 спрощені картки у тому самому grid layout що і реальний каталог.

**Фіксована кількість: 4.** Не динамічна — skeleton не знає скільки результатів прийде.

### Anatomy

```tsx
// CatalogGridSkeleton.tsx
export default function CatalogGridSkeleton() {
  return (
    <div
      className={styles.grid}
      role="status"
      aria-label="Завантаження оголошень"
      aria-live="polite"
    >
      {Array.from({ length: 4 }).map((_, i) => (
        <div key={i} className={styles.card} aria-hidden="true">
          <Skeleton shape="rect" width="100%" height="100%" />
          <div className={styles.content}>
            <Skeleton shape="text" width="70%" height="14px" />
            <Skeleton shape="text" width="42%" height="14px" />
            <Skeleton shape="text" width="28%" height="12px" />
          </div>
        </div>
      ))}
    </div>
  );
}
```

### Структура картки-placeholder

Не копіює DOM `PlantCard`. Проста ієрархія:

```
.card
  Skeleton shape="rect"        ← весь фото-блок, aspect-ratio: 1/1
  .content
    Skeleton shape="text"      ← назва рослини (70% ширини)
    Skeleton shape="text"      ← ціна або swap (42%)
    Skeleton shape="text"      ← місто (28%)
```

### CSS (CatalogGridSkeleton.module.css)

```css
.grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
}

.card {
  display: flex;
  flex-direction: column;
  border-radius: var(--radius-lg);
  overflow: hidden;
  background: rgba(255, 255, 255, 0.48);
  box-shadow: var(--shadow-card);
}

/* Фото-блок: aspect-ratio як у реального PlantCard */
.card > span:first-child {  /* Skeleton shape=rect */
  aspect-ratio: 1 / 1;
  flex-shrink: 0;
  border-radius: 0;  /* override — кути round тільки у .card */
}

.content {
  display: flex;
  flex-direction: column;
  gap: 6px;
  padding: 12px 14px 14px;
}
```

**Grid breakpoints:** успадковуються від батьківської обгортки сторінки каталогу, якщо `CatalogGrid` і `CatalogGridSkeleton` рендеруться в одному контейнері — без дублювання breakpoints.

### Accessibility

- `role="status"` + `aria-live="polite"` на `.grid` — screen reader оголосить "Завантаження оголошень" коли компонент з'явиться.
- `aria-hidden="true"` на кожній `.card` — самі блоки не несуть змісту.
- Коли `CatalogGrid` замінить fallback, React автоматично видалить `CatalogGridSkeleton` з DOM.

---

## Файлова структура

```
src/
  components/
    ui/
      Skeleton/
        Skeleton.tsx          ← компонент, SkeletonProps
        Skeleton.module.css   ← @keyframes + .skeleton + shape variants
        index.ts              ← re-export
    features/
      catalog/
        CatalogGridSkeleton/
          CatalogGridSkeleton.tsx        ← composite, 4 cartки
          CatalogGridSkeleton.module.css ← .grid + .card + .content
          index.ts                       ← re-export
  app/
    globals.css  ← нові === SKELETON === токени
    (catalog)/
      catalog/
        page.tsx  ← використовує <Suspense fallback={<CatalogGridSkeleton />}>
```

### Skeleton.tsx — відповідальність

- Приймає `shape`, `width`, `height`
- Рендерить `<span>` з `aria-hidden="true"`
- Передає `width`/`height` як CSS custom properties через `style` prop
- Без `'use client'` — немає жодної клієнтської логіки
- Export — тільки `default`, не named

### Skeleton.module.css — відповідальність

- `@keyframes skeleton-shimmer` — тільки тут, не в globals
- `.skeleton` — base shimmer з токенами `--skeleton-bg-*` і `--skeleton-duration`
- `.skeleton--rect` / `.skeleton--circle` / `.skeleton--text` — border-radius per shape

### CatalogGridSkeleton.tsx — відповідальність

- Рендерить grid з 4 картками через `Array.from({ length: 4 })`
- `role="status"` / `aria-live="polite"` / `aria-label` — на grid-контейнері
- `aria-hidden="true"` — на кожній картці
- Без `'use client'` — статичний Server Component
- Export — тільки `default`, не named

### CatalogGridSkeleton.module.css — відповідальність

- `.grid` — 4-column grid layout (повторює layout реального CatalogGrid)
- `.card` — картка-placeholder: background, border-radius, shadow, flex column
- `.content` — відступи і gap між text-лініями
- `aspect-ratio: 1/1` на фото-блоці

### index.ts (обидва)

```ts
export { default as Skeleton } from './Skeleton'
export type { SkeletonProps } from './Skeleton'
```

```ts
export { default as CatalogGridSkeleton } from './CatalogGridSkeleton'
```

---

## Заборонено

- `aria-hidden` на контейнері `CatalogGridSkeleton` — він несе `role="status"` для a11y
- Динамічна кількість карток у `CatalogGridSkeleton` — завжди 4
- `border` замість `box-shadow: inset` де потрібні видимі бордери (для `.card` — немає видимого бордеру, тільки shadow)
- Хардкод кольорів поза токенами — shimmer тільки через `--skeleton-bg-base` / `--skeleton-bg-highlight`
- `useEffect`, `useState` або будь-яка клієнтська логіка — обидва компоненти Server Components
- `className` prop на `Skeleton` — розміри тільки через `width`/`height` props
- Pixel-perfect копіювання DOM структури `PlantCard` в `CatalogGridSkeleton` — вона не повинна дублювати PlantCard
- Окремі skeleton-компоненти для `PlantDetailsSkeleton`, `ProfileHeaderSkeleton` тощо — поза scope цієї сесії
