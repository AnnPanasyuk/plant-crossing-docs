# Спека: CharsList

**Дизайн-референс:** `chars_list.html` (Variant B — 1px border)
**Статус залежностей:** `IconBadge` (s, 22px) — вважається реалізованим; `PlantHealthLabel` — вважається реалізованим.

---

## Що це

UI-примітив `components/ui/CharsList`. Рендерить список рядків «іконка + мітка + значення»
всередині картки з від'ємним `margin-inline` (bleed), щоб градієнт рядків виходив впритул
до внутрішнього краю картки.

Компонент не знає нічого про домен. Feature-шар формує масив `CharsListItem[]` і передає вже
готові реакт-компоненти (напр. `<PlantHealthLabel>`) через `type: 'node'`.

---

## Типи

```ts
import { LucideIcon }  from 'lucide-react'
import { ReactNode }   from 'react'

type CharValueText    = { type: 'text';    value: string }
type CharValueStacked = { type: 'stacked'; primary: string; secondary: string }
type CharValueNode    = { type: 'node';    content: ReactNode }

export type CharValue = CharValueText | CharValueStacked | CharValueNode

export type CharsListItem = {
  id:    string
  icon:  LucideIcon
  label: string
  value: CharValue
}

type CharsListProps = {
  items: CharsListItem[]
}
```

---

## Пропси

| Prop    | Тип               | Обов'язковий | Опис                                        |
|---------|-------------------|:---:|----------------------------------------------|
| `items` | `CharsListItem[]` | ✓   | Масив рядків. Порожній масив — відповідальність батька. |

---

## Варіанти значення (CharValue)

### `type: 'text'`
Простий текстовий рядок. Primary-колір, `font-size: --chars-font-size`.

```tsx
{ type: 'text', value: 'Легка' }
{ type: 'text', value: 'Яскраве непряме' }
```

### `type: 'stacked'`
Два рядки: основний (primary) + другорядний (muted, менший шрифт, secondary-колір).

```tsx
{ type: 'stacked', primary: 'ø 20 см', secondary: 'Висота ~120 см' }
```

### `type: 'node'`
Довільний `ReactNode`. Feature-шар передає готовий компонент.

```tsx
{ type: 'node', content: <StatusBadge variant="healthy">Здорова</StatusBadge> }
```

---

## Семантика

```html
<dl>                          <!-- список пар ключ–значення -->
  <div>                       <!-- обгортка пари (валідний HTML per spec) -->
    <dt>                      <!-- мітка: IconBadge + текст -->
    <dd>                      <!-- значення: text / stacked / node -->
  </div>
</dl>
```

`IconBadge` має `aria-hidden="true"` — іконки декоративні, мітка читається як текст `<dt>`.

---

## Розмітка колонок

Ширина колонки мітки (`<dt>`) визначається автоматично по найдовшому лейблу —
через `grid-template-columns: max-content 1fr` на `<dl>`.
Жодного фіксованого значення в пікселях, жодного токена ширини.

### Механізм

```
.charsList → display: grid; grid-template-columns: max-content 1fr;
.charRow   → grid-column: 1 / -1; display: grid; grid-template-columns: subgrid;
```

Кожен `.charRow` є subgrid-item — успадковує колонки батьківського `<dl>`.
`max-content` для колонки 1 = ширина найширшого `<dt>` (IconBadge + gap + текст).
Всі рядки автоматично вирівнюються по цій ширині.

Сумісність: CSS Subgrid підтримується в Chrome 117+, Firefox 71+, Safari 16+.

### Bleed + padding

```
.charsList → margin-inline: calc(-1 * var(--spacing-card-pad))   ← виходить за межі картки
.charRow   → padding-inline: var(--spacing-card-pad)             ← компенсує відступ
```

Фон-градієнт рядка заповнює всю ширину `.charRow` (включно з bleed).
Контент (`<dt>`, `<dd>`) розміщується у відступленій зоні.

### Візуальна структура рядка

```
◄── bleed ──►[   IconBadge sm   Мітка (max-content)   │gap│   Значення (1fr)   ]◄── bleed ──►
              ↑ padding-inline: --spacing-card-pad                               ↑
                         ↑ gap: --chars-label-gap між badge і текстом
```

---

## Градієнт рядка

Горизонтальний `linear-gradient` (fade in/out по краях):

```
var(--chars-row-bg-from) 0% ──► var(--chars-row-bg-mid) 10% ... 90% ──► var(--chars-row-bg-from) 100%
```

Токени: `--chars-row-bg-from` = `rgba(255,255,255,0)`, `--chars-row-bg-mid` = `rgba(255,255,255,0.22)`.

---

## Роздільник

```css
box-shadow: inset 0 calc(-1 * var(--chars-row-border-size)) 0 0 var(--chars-row-border);
```

Не CSS `border` — зберігає box-model консистентним (CLAUDE.md).
Останній рядок (`:last-child`) — `box-shadow: none`.

---

## Файлова структура

```
components/
  ui/
    CharsList/
      CharsList.tsx          — компонент (default export)
      CharsList.module.css   — стилі
      index.ts               — re-export: export { default as CharsList }

app/
  dev/
    chars-list/
      page.tsx               — dev-сторінка (default export: CharsListDemoPage)
      styles.module.css
```

---

## CSS-токени

### Оновити в `globals.css` — секція `/* === CHARS LIST === */`

| Токен | Дія | Поточне → нове |
|---|---|---|
| `--chars-row-border-size` | змінити | `0.5px` → `1px` (Variant B) |
| `--chars-row-label-width` | **видалити** | більше не використовується |

### Додати до тієї ж секції (після `--chars-row-bg-mid`)

```css
--chars-row-padding-v:    7px;
--chars-row-gap:          8px;   /* column-gap між dt і dd */
--chars-label-gap:        6px;   /* gap між IconBadge і текстом мітки */
--chars-font-size:        var(--font-size-s);   /* 12px */
--chars-stack-gap:        1px;
```

---

## CharsList.tsx

```tsx
import { ReactElement } from 'react'
import { IconBadge }    from '@/components/ui/IconBadge'
import styles           from './CharsList.module.css'
import { CharValue, CharsListItem } from './index'

type CharsListProps = {
  items: CharsListItem[]
}

function renderValue(value: CharValue): ReactElement {
  if (value.type === 'stacked') {
    return (
      <span className={styles.charValueStack}>
        <span>{value.primary}</span>
        <span className={styles.charValueMuted}>{value.secondary}</span>
      </span>
    )
  }
  if (value.type === 'node') {
    return <>{value.content}</>
  }
  return <>{value.value}</>
}

export default function CharsList({ items }: CharsListProps): ReactElement {
  return (
    <dl className={styles.charsList}>
      {items.map((item) => (
        <div key={item.id} className={styles.charRow}>
          <dt className={styles.charLabel}>
            <IconBadge icon={item.icon} size="sm" aria-hidden />
            {item.label}
          </dt>
          <dd className={styles.charValue}>{renderValue(item.value)}</dd>
        </div>
      ))}
    </dl>
  )
}
```

---

## CharsList.module.css

```css
/* Externally injected custom properties: none */

.charsList {
  display: grid;
  grid-template-columns: max-content 1fr;
  column-gap: var(--chars-row-gap);
  margin-inline: calc(-1 * var(--spacing-card-pad));
}

.charRow {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: subgrid;
  align-items: center;
  font-size: var(--chars-font-size);
  line-height: var(--line-height-base);
  padding-block: var(--chars-row-padding-v);
  padding-inline: var(--spacing-card-pad);
  background: linear-gradient(
    to right,
    var(--chars-row-bg-from)  0%,
    var(--chars-row-bg-mid)  10%,
    var(--chars-row-bg-mid)  90%,
    var(--chars-row-bg-from) 100%
  );
  box-shadow: inset 0 calc(-1 * var(--chars-row-border-size)) 0 0 var(--chars-row-border);
}

.charRow:last-child {
  box-shadow: none;
}

.charLabel {
  display: flex;
  align-items: center;
  gap: var(--chars-label-gap);
  color: var(--color-text-secondary);
}

.charValue {
  color: var(--color-text-primary);
}

.charValueStack {
  display: flex;
  flex-direction: column;
  gap: var(--chars-stack-gap);
}

.charValueMuted {
  color: var(--color-text-secondary);
}
```

---

## index.ts

```ts
export { default as CharsList } from './CharsList'
export type { CharsListItem, CharValue } from './CharsList'
```

> Типи — named export з файлу компонента. Якщо TypeScript скаржиться на circular,
> виноси типи в окремий `types.ts` усередині папки.

---

## Dev-сторінка

**Файл:** `app/dev/chars-list/page.tsx`

Колонки на рожевому та зеленому фоні, загорнуті у `card-white`. Кожна колонка — окремий сценарій.

| # | Заголовок колонки | Що демонструє |
|---|---|---|
| 1 | `type: text — всі рядки` | 5 рядків з text-значеннями |
| 2 | `type: stacked` | Рядок з двома підрядками (Горщик / Висота) |
| 3 | `type: node — StatusBadge` | Стан: healthy / care / sick — три окремі CharsList по 1 рядку |
| 4 | `single item` | 1 рядок → без border-bottom |
| 5 | `змішаний (production-like)` | Стан=StatusBadge, Горщик=stacked, решта=text |

Іконки для dev: `Heart`, `Sprout`, `Star`, `Droplet`, `Sun` з `lucide-react`.

---

## Файли для оновлення

### `globals.css` — секція `/* === CHARS LIST === */`

1. `--chars-row-border-size: 0.5px` → `1px`
2. Видалити рядок `--chars-row-label-width: 105px`
3. Додати після `--chars-row-bg-mid`:

```css
--chars-row-padding-v:    7px;
--chars-row-gap:          8px;
--chars-label-gap:        6px;
--chars-font-size:        var(--font-size-s);
--chars-stack-gap:        1px;
```

### `app/dev/page.tsx`

Додати посилання: `<a href="/dev/chars-list">CharsList</a>`

---

## Заборонено

- Розуміти домен: не знати що таке "Стан", "Горщик" — тільки `id`, `icon`, `label`, `value`
- Рендерити `StatusBadge` самостійно — він приходить готовим через `type: 'node'`
- Хардкодити значення кольорів або розмірів — тільки токени з `globals.css`
- Додавати `className` prop
- Використовувати `useCallback`/`useMemo` без реальної потреби
- Додавати `'use client'` без необхідності
- Inline-стилі в TSX
- Створювати файли не зі списку вище
