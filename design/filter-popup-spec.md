# FilterButton + FilterPopup

Компонент фільтрації каталогу. Два стани: collapsed (кнопка) і expanded (wizard попап).

---

## Архітектура

```
components/
  ui/
    FilterButton/
      FilterButton.tsx       — collapsed кнопка
      FilterButton.module.css
    FilterPopup/
      FilterPopup.tsx        — wizard контейнер
      FilterStep.tsx         — один крок
      FilterPopup.module.css
  features/
    catalog/
      CatalogFilters.tsx     — orchestrator: стан фільтрів, відкриття/закриття
```

---

## FilterButton (collapsed)

### Props

```ts
interface FilterButtonProps {
  activeCount: number          // кількість активних фільтрів (0 = без badge)
  onClick: () => void
  activeLabels?: string[]      // preview chips (опціонально, max 2 + "+N")
}
```

### Стани

| Стан | Вигляд |
|------|--------|
| Default | Іконка фільтра + "Фільтри" · без badge |
| Hover | Тінь посилюється · border темніший |
| Active (є фільтри) | Іконка accent · badge з кількістю |
| Active + preview | Іконка · divider · chips preview · "+N" |
| Active + Hover | Комбінація |

### Tokens

```css
/* position */
position: fixed;
bottom: 28px;
left: 50%;
transform: translateX(-50%);
z-index: var(--z-filter-btn); /* 20 */

/* appearance */
background: #fff;
border: var(--border-size) solid rgba(122,136,120,0.22);
border-radius: var(--radius-full);
padding: 10px 18px;
box-shadow: 0 4px 20px rgba(74,90,72,0.16);
font-size: var(--font-size-s); /* 12px */

/* hover */
box-shadow: 0 8px 28px rgba(74,90,72,0.20);
border-color: rgba(122,136,120,0.40);

/* active filters */
border-color: rgba(196,122,136,0.35);
icon stroke: var(--color-text-accent);

/* badge */
background: var(--color-accent-main);
size: 18px × 18px;
font-size: 9px; font-weight: 600; color: #fff;
```

---

## FilterPopup (expanded wizard)

### Props

```ts
interface FilterPopupProps {
  isOpen: boolean
  onClose: () => void
  onApply: (filters: CatalogFilters) => void
  onReset: () => void
  initialFilters?: CatalogFilters
  resultCount: number          // оновлюється в реальному часі при виборі
}

interface CatalogFilters {
  condition?: 'good' | 'fair'
  difficulty?: ('easy' | 'medium' | 'hard')[]
  potSize?: ('xs' | 's' | 'm' | 'l' | 'xl')[]
  dealType?: 'swap' | 'sale' | 'any'
  priceFrom?: number
  priceTo?: number
  city?: string
}
```

### 5 кроків wizard

| # | Крок | Компонент | Мультиселект |
|---|------|-----------|--------------|
| 1 | Стан рослини | Chips | Ні (single) |
| 2 | Складність догляду | Chips | Так |
| 3 | Діаметр горщика | Chips | Так |
| 4 | Тип угоди | Chips | Ні (single) |
| 5 | Ціна і місто | 2× Input + Input | — |

### Tooltip тексти

```ts
const tooltips = {
  condition: 'Хороший — рослина здорова, без пошкоджень. Задовільний — є незначні дефекти, але жива і росте.',
  difficulty: 'Easy — поливати рідко, вибачає помилки. Medium — потребує уваги. Hard — для досвідчених, специфічні умови.',
  potSize: 'Розмір горщика відображає вік і розмір рослини. XS = малюки до 9 см, XL = дорослі від 25 см.',
  dealType: 'Обмін — продавець готовий обмінятись на іншу рослину. Тільки продаж — без обміну, лише за гроші.',
  priceCity: 'Вкажи бюджет у гривнях і місто продавця. Обидва поля опціональні.',
}
```

### Поведінка

- **Перший захід на каталог** → попап відкривається автоматично на кроці 1
- **Навігація** → `<` назад / `>` вперед (або кнопки "Назад" / "Далі")
- **Пропустити** → перехід до наступного кроку без вибору
- **Apply** → тільки по кнопці "Показати результати" (останній крок)
- **resultCount** → оновлюється при кожній зміні вибору (debounce 300ms)
- **Закриття** → клік на backdrop або Escape → морф назад у кнопку
- **Reset** → "Скинути все" (тільки на останньому кроці) → скидає всі кроки, залишає попап відкритим на кроці 1

### Tokens

```css
/* popup */
width: 480px;
border-radius: var(--radius-xl); /* 24px */
background: #fff;
border: var(--border-size) solid rgba(122,136,120,0.15);
box-shadow: var(--shadow-drawer);
position: fixed; bottom: 24px; left: 50%; transform: translateX(-50%);
z-index: var(--z-drawer); /* 40 */

/* progress bar dots */
height: 2px;
active: var(--color-accent-main);
done: rgba(196,122,136,0.35);
empty: rgba(122,136,120,0.2);

/* nav buttons */
size: 28px × 28px; border-radius: 50%;
border: var(--border-size) solid rgba(122,136,120,0.25);
disabled opacity: 0.3;

/* tooltip bubble */
background: #1C1C1C; color: #fff;
font-size: 11px; border-radius: var(--radius-md);
width: 220px;
arrow: border trick, bottom center;
```

---

## Morph animation

### Відкриття (кнопка → попап)

```css
.filter-popup {
  transform-origin: bottom center;
  animation: popup-open 350ms cubic-bezier(0, 0, 0.2, 1) forwards;
}

@keyframes popup-open {
  from {
    border-radius: 9999px;
    width: 140px;        /* ширина кнопки */
    opacity: 0.6;
  }
  to {
    border-radius: 24px;
    width: 480px;
    opacity: 1;
  }
}

/* контент попапу з'являється з затримкою */
.filter-popup-content {
  animation: content-fade 150ms ease 150ms forwards;
  opacity: 0;
}
@keyframes content-fade {
  to { opacity: 1; }
}
```

### Закриття (попап → кнопка)

```css
animation: popup-close 250ms cubic-bezier(0.4, 0, 1, 1) forwards;

@keyframes popup-close {
  from { border-radius: 24px; width: 480px; opacity: 1; }
  to   { border-radius: 9999px; width: 140px; opacity: 0.6; }
}
```

### Перемикання кроків

```css
/* вперед */
@keyframes step-forward {
  from { opacity: 0; transform: translateX(12px); }
  to   { opacity: 1; transform: translateX(0); }
}

/* назад */
@keyframes step-back {
  from { opacity: 0; transform: translateX(-12px); }
  to   { opacity: 1; transform: translateX(0); }
}

/* duration: 150ms, easing: ease */
```

---

## CategoryBar (sticky підхедер)

Окремий компонент. Завжди видимий, не залежить від FilterPopup.

```ts
interface CategoryBarProps {
  categories: Category[]
  selected: string[]           // мультиселект
  onChange: (ids: string[]) => void
}

// "Всі" = selected порожній масив
// position: sticky; top: 56px (висота хедера); z-index: var(--z-tabs) — 45
```

```css
.category-bar {
  position: sticky;
  top: 56px;
  z-index: var(--z-tabs); /* 45 */
  padding: 10px 100px;   /* desktop */
  background: linear-gradient(to bottom, #E8EBE8 80%, rgba(232,235,232,0) 100%);
  border-bottom: var(--border-size) solid var(--color-inverse-muted-4);
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;       /* ніякого горизонтального скролу */
}
```

---

## URL параметри (SEO + shareable)

```ts
// /catalog?condition=good&difficulty=easy,medium&city=Київ
const filterToParams = (filters: CatalogFilters): URLSearchParams => { ... }
const paramsToFilter = (params: URLSearchParams): CatalogFilters => { ... }
```

---

## Edge cases

- Якщо `resultCount === 0` → кнопка "Показати результати" disabled, підпис "0 рослин"
- Якщо `resultCount` завантажується → spinner замість числа
- Mobile (< 1024px) → FilterPopup стає bottom sheet на повну ширину, border-radius тільки верхні кути
- Escape закриває попап без apply (зберігає попередні фільтри)
