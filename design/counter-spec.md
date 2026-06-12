# Counter

Чистий UI-примітив: кружечок з числом. Без логіки, без позиціонування, без варіантів.

Використовується як badge на іконках хедера (cart count, wishlist count). Позиціонування (absolute, top/right offset) та рішення про відображення при `value === 0` — відповідальність батьківського компонента (Header). Counter сам по собі — `display: inline-flex`, нічого більше.

---

## Дизайн (затверджено)

- Один розмір: **14×14px**, коло (`border-radius: 50%`)
- `font-size: 8px`, `font-weight: 600`
- Background: `var(--color-accent-main)`, текст: `var(--color-inverse)`
- Border: `box-shadow: 0 0 0 1.5px var(--color-base-bg-2)` (без `inset` — кільце малюється за межами 14×14, видимий діаметр ~17px, layout-розмір залишається 14×14)
- Значення: число `0–99`. Двозначні числа — без спецобробки, без `max`/`"99+"`
- Без станів/варіантів — компонент завжди виглядає однаково
- `aria-hidden="true"` — текстове значення семантично озвучує батьківський компонент через власний `aria-label`

---

## Токени → `app/globals.css`

Консолідувати існуючі `--counter-s-*` / `--counter-m-*` (рядки 527–541) в єдиний набір:

```css
/* === COUNTER === */
--counter-size:        14px;
--counter-font-size:   8px;
--counter-font-weight: 600;
--counter-radius:      50%;
--counter-bg:          var(--color-accent-main);
--counter-color:       var(--color-inverse);
--counter-box-shadow:  0 0 0 1.5px var(--color-base-bg-2);
```

Старі токени (`--counter-s-size`, `--counter-s-font-size`, `--counter-s-font-weight`, `--counter-s-radius`, `--counter-s-bg`, `--counter-s-color`, `--counter-m-size`, `--counter-m-font-size`, `--counter-m-font-weight`, `--counter-m-radius`, `--counter-m-bg`, `--counter-m-color`) — видалити, ніде більше не використовуються.

---

## Counter.tsx — відповідальність

```tsx
export interface CounterProps {
  value: number; // 0–99, не валідується runtime
}
```

- Рендерить `<span className={styles.counter} aria-hidden="true">{value}</span>`
- Жодних інших props, жодного internal state
- `value` не клемпиться і не форматується — відповідальність викликаючого коду (батьківський компонент гарантує діапазон і вирішує, чи рендерити Counter при `0`)

## Counter.module.css — відповідальність

```css
.counter {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  width: var(--counter-size);
  height: var(--counter-size);
  border-radius: var(--counter-radius);
  background: var(--counter-bg);
  color: var(--counter-color);
  font-size: var(--counter-font-size);
  font-weight: var(--counter-font-weight);
  box-shadow: var(--counter-box-shadow);
  line-height: 1;
  font-family: 'Inter', sans-serif;
}
```

---

## Файлова структура

```
components/ui/Counter/
  Counter.tsx          — СТВОРИТИ: компонент + CounterProps
  Counter.module.css   — СТВОРИТИ: .counter
  index.ts             — СТВОРИТИ: re-export Counter, CounterProps

app/globals.css        — ОНОВИТИ: консолідувати токени --counter-* (див. вище)

app/dev/counter/
  page.tsx             — СТВОРИТИ: dev-сторінка (деталі — в промті Claude Code)
  CounterDemo.module.css — СТВОРИТИ
app/dev/page.tsx       — ОНОВИТИ: додати посилання на Counter
```

---

## Заборонено

- `size`/`variant` props — компонент завжди однаковий
- Логіка приховування при `value === 0` всередині Counter
- Логіка `"99+"` / overflow всередині Counter
- Власне позиціонування (`position: absolute` тощо) всередині Counter
- `aria-label` на Counter (тільки `aria-hidden`)
- Hardcode значень — тільки токени з globals.css
