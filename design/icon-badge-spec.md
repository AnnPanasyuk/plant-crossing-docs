# IconBadge — Spec

Дизайн: `icon_badge.html`

---

## Опис

Декоративний контейнер для lucide-react іконки. Використовується як візуальний акцент у рядках деталей та покрокових секціях. Не є інтерактивним — `aria-hidden="true"` завжди.

---

## Props

| Prop | Тип | Default | Обов'язково |
|------|-----|---------|-------------|
| `icon` | `LucideIcon` | — | ✅ |
| `size` | `'s' \| 'l'` | `'s'` | — |

Немає `className` prop. Позиціонування — через wrapper у батьківському компоненті.

---

## Розміри

| Size | Бейдж | Іконка | Контекст |
|------|-------|--------|----------|
| `s` (default) | 22×22px | 14px   | Plant Detail, рядки деталей |
| `l` | 32×32px | 18px   | Swap Popup, покрокові секції |

---

## Стани

Статичний компонент. Інтерактивних станів немає: без hover, focus, active, disabled.

---

## Візуальні властивості

- `border-radius: var(--icon-badge-radius)` → `--radius-sm` (8px)
- `background: var(--icon-badge-bg)` → `--color-accent-main-muted`
- `color: var(--icon-badge-color)` → `--color-accent-main` (lucide використовує `currentColor` для `stroke`)
- `flex-shrink: 0` — обов'язково, компонент завжди в рядковому flex-контейнері
- `strokeWidth={1.5}` — передається як prop до Icon компонента (не через CSS)
- lucide defaults `strokeLinecap="round"` і `strokeLinejoin="round"` — CSS override не потрібен
- lucide default `fill="none"` — CSS override не потрібен

---

## Токени globals.css

Додати після блоку `/* === STATUS BADGE === */`:

```css
/* === ICON BADGE === */
--icon-badge-size-s: 22px;
--icon-badge-size-l: 32px;
--icon-badge-icon-s: 14px;
--icon-badge-icon-l: 18px;
--icon-badge-radius: var(--radius-sm);
--icon-badge-bg: var(--color-accent-main-muted);
--icon-badge-color: var(--color-accent-main);
--icon-badge-stroke-width: 1.5;
```

`--icon-badge-stroke-width` — токен-документація дизайн-рішення.
В CSS не використовується — значення передається як TSX-константа `strokeWidth={1.5}` у Icon prop.

---

## CSS Module — логіка класів

| Клас | Що робить |
|------|-----------|
| `.iconBadge` | Base: розмір `s` (22px), border-radius, background, `color` для lucide currentColor, flex centering, flex-shrink |
| `.iconBadge svg` | Розмір іконки `s`: width/height 10px |
| `.large` | Override розміру контейнера: 32×32px |
| `.large svg` | Override розміру іконки: 14×14px |

Stroke властивості (`stroke-width`, `stroke-linecap`, `stroke-linejoin`) — не в CSS, контролюються через lucide props і lucide defaults.

---

## Файлова структура

### `components/ui/IconBadge/IconBadge.tsx`
Компонент. Рендерить `<div aria-hidden="true">` + `<Icon strokeWidth={1.5} />`.
`strokeWidth={1.5}` — константа в тілі файлу, не inline значення.
Клас: `styles.iconBadge` (s) або `styles.iconBadge + styles.large` (l).
Типи `IconBadgeSize`, `IconBadgeProps` — named exports з цього ж файлу.
`export default function IconBadge`.
Немає `'use client'`.

### `components/ui/IconBadge/IconBadge.module.css`
Стилі. Класи: `.iconBadge`, `.iconBadge svg`, `.large`, `.large svg`.
Тільки токени з globals.css, без hardcode значень.

### `components/ui/IconBadge/index.ts`
```ts
export { default as IconBadge } from './IconBadge';
export type { IconBadgeProps, IconBadgeSize } from './IconBadge';
```

### `app/dev/icon-badge/page.tsx`
`export default function IconBadgeDemoPage`.
Немає `'use client'`.
Секції (детальніше нижче).

### `app/dev/icon-badge/styles.module.css`
Стилі dev сторінки.

---

## Dev сторінка

### 1. Розміри
Два варіанти поряд: `s` і `l`.
Під кожним — підпис з розміром і позначкою `default` для `s`.
Іконка для демо: `MapPin`.

### 2. Варіанти іконок · s
8 IconBadge size `s` в рядок:
`MapPin`, `Droplets`, `Package`, `Clock`, `Lock`, `Grid2X2`, `AlignLeft`, `CheckCircle`

### 3. Варіанти іконок · l
Ті ж самі 8 іконок, size `l`.

### 4. Контекст · Plant Detail (s)
Рядки: flex row, gap 12px, align-items center.
Мітка ширина 120px, колір secondary.

| Іконка | Мітка | Значення |
|--------|-------|---------|
| `CheckCircle` | Стан | Здорова |
| `Droplets` | Полив | Коли верхній шар ґрунту (2–3 см) підсох |
| `Package` | Горщик | ø 18 см |
| `MapPin` | Місто | Київ |
| `Lock` | Обмін | Лише обмін |

Розділювач між рядками: `box-shadow: inset 0 -0.5px 0 rgba(122,136,120,0.18)`.
Останній рядок без розділювача.

### 5. Контекст · Swap Popup (l)
3 кроки: flex row, gap 12px.
Текстовий блок: title (14px, 500) + desc (12px, secondary, line-height 1.55).

| Іконка | Title | Desc |
|--------|-------|------|
| `AlignLeft` | Ви пропонуєте свою рослину | Обери рослину зі своїх оголошень, яку хочеш обміняти. |
| `Clock` | Продавець підтверджує обмін | Відповідь побачиш у розділі Обміни. |
| `Package` | Обоє відправляють рослини | Після підтвердження кожна сторона відправляє своє і підтверджує отримання. |

Розділювач між кроками: `box-shadow: inset 0 -0.5px 0 rgba(0,0,0,0.05)`.
Останній крок без розділювача.

---

## Використання

```tsx
import { IconBadge } from '@/components/ui/IconBadge';
import { MapPin } from 'lucide-react';

<IconBadge icon={MapPin} />
<IconBadge icon={MapPin} size="l" />
```
