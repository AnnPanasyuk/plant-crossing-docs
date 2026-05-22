# Grid Layout

Базова сіткова система платформи.
Tailwind v4 — для grid/flex wrappers. CSS Modules — для компонентів всередині сітки.

---

## Breakpoints

Два breakpoints за **interaction model**:

| Назва | Діапазон | Пристрій | Interaction |
|-------|----------|----------|-------------|
| `mobile` | < 1024px | Телефон, планшет | Touch, без hover |
| `desktop` | ≥ 1024px | Десктоп, широкі екрани | Mouse, hover, keyboard |

Планшет входить у mobile-зону — interaction model touch, не mouse.

**Джерело даних:** StatCounter, серп. 2024 – серп. 2025
- Mobile+tablet: ~60.7% трафіку
- Desktop: ~39.3% трафіку

---

## Поведінкові відмінності

| Елемент | mobile | desktop |
|---------|--------|---------|
| Hover ефекти | ❌ | ✅ |
| PlantCardActions | wishlist іконка завжди видима | reveal на hover |
| CartDrawer | bottom sheet | side drawer |
| FilterButton | sticky внизу | sticky внизу |
| FAB (нове оголошення) | ✅ внизу праворуч | ❌ (в хедері) |
| Анімації | мінімальні | повні |
| Header пошук | іконка → розгортає інпут | завжди видимий інпут |
| AccountPopup | bottom sheet | popover |

---

## Page Container

Без `max-width`. Сторінки розтягуються на повну ширину viewport.
Горизонтальний padding задається на рівні кожної сторінки:

| Breakpoint | Padding |
|------------|---------|
| mobile | `var(--spacing-page-x)` = 24px |
| desktop | 100px |

Вюпорти керують позицією та функціоналом компонентів, не шириною контейнера.

---

## Каталог — Грід карток

| Breakpoint | Колонки | Gap |
|------------|---------|-----|
| mobile | 2 | 12px |
| desktop | 4 | 20px |

**Правила:**
- Картки рівновисокі в межах рядка (`align-items: stretch`)
- Фото в картці: фіксований `aspect-ratio`, не розтягується
- Горизонтального scroll немає ніде на платформі

---

## Сторінка деталей — Двоколонковий layout

| Breakpoint | Структура |
|------------|-----------|
| mobile | 1 колонка: фото зверху, деталі знизу |
| desktop | 2 колонки: `50%` фото (sticky) / `40%` деталі |

**Правила:**
- Gap між колонками: `16px`
- Ліва колонка (фото): `position: sticky; top: 68px; z-index: var(--z-sticky)`
- Права колонка: `margin: 0 auto auto`
- Висота лівої колонки визначається виключно `aspect-ratio: 1/1` — `height` не задається

---

## Spacing система

| Токен | Значення | Використання |
|-------|----------|--------------|
| `--spacing-card-pad` | `16px` | Внутрішній відступ картки |
| `--spacing-section` | `48px` | Відступ між секціями сторінки |
| `--spacing-page-x` | `24px` | Горизонтальний padding (mobile) |

---

## Z-index шари

| Шар | Z-index | Токен | Елемент |
|-----|---------|-------|---------|
| Base | 0 | — | Основний контент |
| Sticky | 10 | `--z-sticky` | Sticky фото колонка (деталі) |
| FilterButton | 20 | `--z-filter-btn` | Sticky фільтр іконка |
| FAB | 25 | `--z-fab` | Floating action button |
| StickyCTA | 30 | `--z-sticky-cta` | Sticky CTA бар знизу |
| Drawer | 40 | `--z-drawer` | CartDrawer, FilterPopup |
| Tabs | 45 | `--z-tabs` | Sticky таби профілю |
| Header | 50 | `--z-header` | Глобальний хедер |
| Popup | 60 | `--z-popup` | AccountPopup, Toast |
| Overlay | 70 | `--z-overlay` | Backdrop для модалок |

---

## Що не робимо

- Hover ефекти на mobile — touch пристрої їх не підтримують
- Горизонтальний scroll — ніде на платформі
- Фіксована висота карток
- Side drawer на mobile — використовуємо bottom sheet
- `max-width` на page container
