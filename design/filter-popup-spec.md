# FilterButton + FilterPopup

Кнопка-тригер та wizard-popup для фільтрації каталогу. 6 кроків, по одному фільтру на крок.

---

## FilterButton

Кнопка у sticky sub-header каталогу. Відкриває FilterPopup через Radix UI Dialog.

### Стани

| Стан | Умова | Вигляд |
|------|-------|--------|
| Default | 0 активних фільтрів | білий фон, muted border, secondary icon + text |
| Hover | hover на Default | посилений shadow |
| Active | ≥1 активний фільтр | accent border, accent icon + text, badge з кількістю |
| Active + Hover | hover на Active | accent border + рожевий shadow, badge |

### Токени

```
padding:       10px 18px
border-radius: var(--radius-full)
background:    #fff
font-size:     13px
```

**Default border:**
```css
box-shadow: inset 0 0 0 0.5px rgba(122,136,120,0.22);
```

**Hover:**
```css
box-shadow: inset 0 0 0 0.5px rgba(122,136,120,0.4), 0 4px 16px rgba(74,90,72,0.14);
```

**Active:**
```css
box-shadow: inset 0 0 0 0.5px rgba(196,122,136,0.45);
color: var(--color-text-accent);
/* icon: stroke var(--color-text-accent) */
```

**Active + Hover:**
```css
box-shadow: inset 0 0 0 0.5px rgba(196,122,136,0.6), 0 4px 16px rgba(196,122,136,0.18);
```

### Badge

Показується тільки коли є активні фільтри. Загальна кількість активних фільтрів.

```
min-width:    18px
height:       18px
padding:      0 4px
border-radius: var(--radius-full)
background:   var(--color-accent-main)
font-size:    9px
font-weight:  600
color:        #fff
```

### Заборонено

- Preview chips (назви вибраних фільтрів всередині кнопки) — не реалізується

---

## FilterPopup

Wizard-popup для фільтрації каталогу. 6 кроків, по одному фільтру на крок. Відкривається з `FilterButton` через Radix UI Dialog.

---

## Файлова структура

```
components/
  features/
    catalog/
      FilterPopup/
        FilterPopup.tsx          — Radix Dialog shell, оркестрація кроків, стан
        FilterPopup.module.css   — стилі попапу, progress bar, header, footer
        FilterStep.tsx           — один крок: title + info + body slot + footer
        FilterStep.module.css    — стилі кроку
      CatalogFilters/
        CatalogFilters.tsx       — FilterButton + FilterPopup разом
```

---

## Кроки

| # | Назва | Тип body | Варіанти |
|---|-------|----------|---------|
| 1 | Стан рослини | chips | Хороший, Задовільний |
| 2 | Складність догляду | chips | Easy, Medium, Hard |
| 3 | Діаметр горщика | chips | XS · до 9 см, S · 9–15 см, M · 15–25 см, L · 25+ см |
| 4 | Тип угоди | chips | Обмін, Тільки продаж, Будь-який |
| 5 | Ціна | два інпути | Від, ₴ / До, ₴ |
| 6 | Місто | один інпут | Місто |

Мульти-селект для всіх chip-кроків (немає обмеження на кількість вибраних).

---

## Структура попапу

```
┌──────────────────────────────────────────────────────────┐
│  [■ ■ □ □ □ □]                                      [×]   │  ← progress-row
│                                                            │
│  Назва кроку  ⓘ                                           │  ← header
│                                                            │
│  [body: chips або inputs]                                 │  ← body
│                                                            │
│  [←]  Пропустити      [Показати результати (N)]    [→]    │  ← footer, кроки 1–5
│  [←]  ↺ Скинути все   [Показати результати (N)]    [→]⃠   │  ← footer, крок 6 (→ disabled)
└──────────────────────────────────────────────────────────┘
```

Ширина: `660px`. `border-radius: var(--radius-xl)` = 24px. `box-shadow: var(--shadow-drawer)`.

---

## Progress bar

Рядок з 6 сегментів (`height: 2px`, `border-radius: full`, `gap: 4px`).

- **Пройдені + поточний** → `background: var(--color-accent-main)`
- **Майбутні** → `background: rgba(122,136,120,0.18)`

Прогрес заповнюється вперед при переході на наступний крок, скорочується при поверненні назад.

Кнопка закриття `×` (28×28px, circle, `border: 0.5px solid rgba(122,136,120,0.25)`) — в одному flex-рядку з progress dots, `margin-left: 8px`.

```
padding: 18px 20px 0
```

---

## Header

Title + info icon в окремому рядку під progress bar.

```
padding: 10px 20px 12px
```

- **Title** — `font-size: 14px`, `font-weight: 500`, `color: var(--color-text-primary)`, left-aligned
- **Info icon** — `<Button variant="plain" size="sm" iconLeft={<Info />} aria-label="Детальніше" />`. Без бордера, без фону. Відкриває `<Tooltip>` з поясненням фільтру.

---

## Body

```
padding: 0 20px 14px
```

### Chips

Використовує компонент `<Chip>` з `ui/`. Мульти-селект через `isSelected` prop.

Токени компонента Chip:
```
padding:    2px 7px
font-size:  12px
radius:     radius-full
bg:         var(--color-base-linear-gradient)
border:     0.5px solid var(--color-secondary)
color:      var(--color-secondary)
```

Selected state:
```
bg:         linear-gradient(160deg, var(--color-accent-light), var(--color-accent-main))
border:     0.5px solid rgba(255,255,255,0.28)
color:      #fff
```

### Ціна (крок 5)

Два `<Input>` в `grid-template-columns: 1fr 1fr`, `gap: 8px`. Placeholder: `Від, ₴` / `До, ₴`. Тип `text`, format validation на blur.

### Місто (крок 6)

Один `<Input>` на всю ширину. Placeholder: `Місто`. Free-text, без автокомпліту в MVP.

---

## Footer

Уніфікована структура на **всіх 6 кроках** — один layout, без окремих mid-step / last-step варіантів:

```
padding:         10px 20px 16px
border-top:      0.5px solid rgba(122,136,120,0.10)
display:         flex
align-items:     center
gap:             8px
```

```
[nav-left]  [middle-button]  [CTA "Показати результати (N)", flex: 1]  [nav-right]
```

| Слот | Елемент | Кроки 1–5 | Крок 6 (`isLast`) |
|------|---------|-----------|---------------------|
| nav-left | `<Button variant="ghost" size="sm" iconLeft={<ChevronLeft />} />` | enabled (`disabled` на кроці 1 / `isFirst`) | enabled |
| middle | див. нижче | "Пропустити" | "↺ Скинути все" |
| CTA | `<Button variant="primary" size="md" />`, `flex: 1` | "Показати результати (N)" | "Показати результати (N)" |
| nav-right | `<Button variant="ghost" size="sm" iconLeft={<ChevronRight />} />` | enabled | **disabled** (`isLast`) |

### Disabled states

- `nav-left`: `disabled` тільки коли `isFirst === true` (`opacity: 0.3; pointer-events: none`)
- `nav-right`: `disabled` тільки коли `isLast === true`. Кнопка **лишається в DOM** (не прибирається, як було в v2) — для візуальної симетрії footer.

### Middle-button

Перемикається залежно від `isLast`:

```tsx
{isLast ? (
  <Button variant="ghost" size="sm" iconLeft={<RotateCcw />} onClick={onResetAll}>
    Скинути все
  </Button>
) : (
  <Button variant="plain" size="sm" onClick={onSkip}>
    Пропустити
  </Button>
)}
```

### "Пропустити" vs nav-right "→"

Дублювання дії — **навмисне**:

- **"Пропустити"** (middle, текстова кнопка, кроки 1–5) — UX-сигнал "за цим параметром можна не фільтрувати", знімає тривогу обов'язкового вибору на кожному кроці.
- **"→"** (nav-right, іконка, завжди) — компактна навігація для тих, хто вже зрозумів флоу.

Обидві кнопки виконують **однаковий** перехід: крок → крок+1, без зміни selection поточного кроку. Якщо колись постане питання прибрати дублювання — прибирається `→`, а не "Пропустити".

---

## Props

### FilterPopup

```tsx
type FilterPopup = {
  open: boolean
  onOpenChange: (open: boolean) => void
  filters: FilterState
  onFiltersChange: (filters: FilterState) => void
  resultCount: number
}
```

### FilterState

```tsx
type FilterState = {
  condition:  ('good' | 'fair')[]
  difficulty: ('easy' | 'medium' | 'hard')[]
  potSize:    ('xs' | 's' | 'm' | 'l')[]
  dealType:   ('swap' | 'sale' | 'any')[]
  priceMin:   string
  priceMax:   string
  city:       string
}
```

### FilterStep

```tsx
type FilterStep = {
  stepIndex:   number           // 0-based
  totalSteps:  number
  isFirst:     boolean          // stepIndex === 0
  isLast:      boolean          // stepIndex === totalSteps - 1
  title:       string
  tooltip?:    ReactNode
  children:    ReactNode        // body slot
  resultCount: number
  onBack:      () => void       // nav-left: крок-1, без зміни selection. disabled якщо isFirst
  onSkip:      () => void       // "Пропустити" / nav-right: крок+1, без зміни selection. nav-right disabled якщо isLast
  onResetAll:  () => void       // "Скинути все" (тільки isLast) — глобальний reset усього FilterState
  onSubmit:    () => void       // "Показати результати" — submit поточного FilterState + close, на будь-якому кроці
  onClose:     () => void       // × / backdrop
}
```

---

## Анімація

### Відкриття (FilterButton → popup)

```
duration:          350ms  — var(--duration-slow)
easing:            cubic-bezier(0, 0, 0.2, 1)  — var(--easing-enter)
transform-origin:  bottom center
```

Що анімується:
```
border-radius:      9999px → 24px
width:              auto → 660px
opacity (content):  0 → 1  (delay: 150ms)
```

### Закриття

```
duration:  250ms
easing:    cubic-bezier(0.4, 0, 1, 1)  — var(--easing-exit)
```

### Перемикання кроків

```
content fade:     opacity 0 → 1, 150ms
slide (вперед):   translateX(12px) → 0
slide (назад):    translateX(-12px) → 0
```

Реалізація через Radix Dialog + CSS `data-state="open|closed"` на content-елементі кроку.

---

## Overlay

Radix Dialog Portal. `data-state` керує анімацією overlay:
```css
[data-state="open"]  { opacity: 1 }
[data-state="closed"] { opacity: 0 }
```

`z-index: var(--z-overlay)` = 70.

---

## Поведінка

- Натиск на `×` або backdrop → закриття без збереження змін.
- `←` (nav-left) → `disabled` тільки на кроці 1 (`isFirst`). Інакше — крок-1, без зміни selection.
- `→` (nav-right) → `disabled` тільки на кроці 6 (`isLast`), залишаючись у DOM. Інакше — крок+1, без зміни selection (дія ідентична "Пропустити").
- "Пропустити" (кроки 1–5, middle-слот) → крок+1, без зміни selection поточного кроку. UX-підказка: цей фільтр можна не заповнювати.
- "Скинути все" (тільки крок 6, middle-слот) → **глобальний** reset `FilterState` до початкового стану (усі 6 кроків, не лише поточний), залишається на кроці 6, `N` перераховується синхронно без "мигання".
- "Показати результати (N)" → submit поточного `FilterState` + close popup. **Ідентична поведінка з будь-якого з 6 кроків** — не "зберегти стан і перейти далі".
- Модель даних: відсутність вибраного chip / порожній input на кроці = "без обмеження за цим параметром", а НЕ "застосувати всі варіанти цього параметра".
- `N` у кнопці оновлюється реактивно при зміні будь-якого фільтру (debounce 300ms).

---

## Заборонено

- Слайдери / range inputs для ціни
- Dropdown або calendar picker
- Горизонтальний скрол всередині popup
- Кастомні chip-елементи замість `<Chip>` з `ui/`
- Inline styles в TSX
- `border: var(--border-size)` на info іконці — тільки `variant="plain"`
- Окремі `--mid` / `--last` варіанти footer (v2) — замінено на єдину структуру
