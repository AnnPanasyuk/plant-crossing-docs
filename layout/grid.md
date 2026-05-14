# Grid Layout

Базова сіткова система платформи.
Tailwind v4 використовується для grid/flex wrappers.
CSS Modules — для компонентів всередині сітки.

---

## Breakpoints

Breakpoints визначені за **interaction model**, а не тільки за розміром екрану.

| Назва | Діапазон | Пристрій | Interaction |
|-------|----------|----------|-------------|
| `sm` | < 480px | Мобайл | Touch, без hover |
| `md` | 480px – 1023px | Планшет | Touch, без hover |
| `lg` | ≥ 1024px | Десктоп | Mouse, hover, keyboard |

**Джерело даних:** StatCounter, серпень 2024 – серпень 2025
https://gs.statcounter.com/screen-resolution-stats

**Ключові факти:**
- Мобайл: 59.16% трафіку, CSS viewport 360–430px
- Планшет: 1.55% трафіку, домінує 768×1024 (iPad)
- Десктоп: 39.29% трафіку

---

## Поведінкові відмінності по breakpoints

| Елемент | sm (мобайл) | md (планшет) | lg (десктоп) |
|---------|-------------|--------------|--------------|
| Hover ефекти | ❌ | ❌ | ✅ |
| PlantCardActions | wishlist іконка завжди видима | wishlist іконка завжди видима | reveal на hover |
| CartDrawer | bottom sheet | bottom sheet | side drawer |
| FilterButton | sticky внизу | sticky внизу | sticky внизу |
| FAB (нове оголошення) | ✅ внизу праворуч | ✅ внизу праворуч | ❌ (в хедері) |
| Анімації | мінімальні | мінімальні | повні |
| Header пошук | іконка → розгортає інпут | іконка → розгортає інпут | завжди видимий інпут |
| AccountPopup | bottom sheet | popover | popover |

---

## Page Container

Максимальна ширина контенту: `1280px`
Горизонтальний padding:

| Breakpoint | Padding |
|------------|---------|
| sm | `var(--spacing-page-x)` = 24px |
| md | 40px |
| lg | 64px |

---

## Каталог — Грід карток

| Breakpoint | Колонки | Gap |
|------------|---------|-----|
| sm | 2 | 12px |
| md | 3 | 16px |
| lg | 4 | 20px |

**Правила:**
- Картки рівновисокі в межах рядка (align-items: stretch)
- Фото в картці: фіксований aspect ratio, не розтягується
- Без горизонтального scroll на будь-якому breakpoint

---

## Сторінка деталей — Двоколонковий layout

| Breakpoint | Структура |
|------------|-----------|
| sm | 1 колонка: фото зверху, деталі знизу |
| md | 1 колонка |
| lg | 2 колонки: 45% фото (sticky) / 55% деталі |

**Правила:**
- Ліва колонка (фото): sticky тільки на `lg`
- Gap між колонками: 40px на `lg`

---

## Spacing система

Базова одиниця: `4px`. Tailwind spacing scale використовується як є.

| Токен | Значення | Використання |
|-------|----------|--------------|
| `--spacing-card-pad` | `16px` | Внутрішній відступ картки |
| `--spacing-section` | `48px` | Відступ між секціями сторінки |
| `--spacing-page-x` | `24px` | Горизонтальний padding (sm) |

---

## Z-index шари

| Шар | Z-index | Елемент |
|-----|---------|---------|
| Base | 0 | Основний контент |
| Sticky | 10 | Sticky фото колонка |
| FilterButton | 20 | Sticky фільтр іконка |
| FAB | 25 | Floating action button |
| StickyCTA | 30 | Sticky CTA бар знизу |
| Drawer | 40 | CartDrawer, FilterPopup |
| Header | 50 | Глобальний хедер |
| Popup | 60 | AccountPopup, Toast |
| Overlay | 70 | Backdrop для модалок |

---

## Що не робимо

- Hover ефекти на `sm` і `md` — touch пристрої їх не підтримують
- Горизонтальний scroll в гріді карток
- Фіксована висота карток
- Side drawer на `sm` і `md` — використовуємо bottom sheet
