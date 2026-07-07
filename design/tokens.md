# Design Tokens

Єдине джерело правди для всіх візуальних констант платформи.
Всі токени визначені в `globals.css` (app repo) через Tailwind v4 `@theme`.
CSS Modules компонентів використовують ці змінні через `var(--...)`.

> Цей документ — довідник. Якщо він розійшовся з `globals.css`, правий `globals.css`.

---

## Кольори

### Base (фон сторінки)
Фон сторінки — завжди градієнт (135deg), ніколи чистий колір.

| Токен | Значення | Використання |
|-------|----------|--------------|
| `--color-base-bg-1` | `#EEF0EC` | Градієнт, старт |
| `--color-base-bg-2` | `#E8EBE8` | Градієнт, середина; також header/sticky-cta gradient base |
| `--color-base-bg-3` | `#E4E8E6` | Градієнт, кінець |
| `--color-base-linear-gradient` | `linear-gradient(135deg, bg-1 0%, bg-2 50%, bg-3 100%)` | `body` background, tag/chip default bg |

⚠️ Не змішувати cream і light green в одному елементі — виглядає непослідовно.

### Accent (пильний рожевий, матовий)
Не яскравий, не гламурний. `light → main` — напрямок градієнта кнопок і карток.

| Токен | Значення | Використання |
|-------|----------|--------------|
| `--color-accent-light` | `rgba(212, 145, 158, 1)` / `#D4919E` | Градієнт start (button primary, card pink) |
| `--color-accent-main` | `rgba(196, 122, 136, 1)` / `#C47A88` | Градієнт end, primary акценти |
| `--color-accent-main-muted` | `rgba(196, 122, 136, 0.1)` | Фон іконок (StatusIcon) |
| `--color-accent-deep` | `rgba(212, 145, 158, 0.38)` | Focus ring |

### Secondary (приглушений зелено-сірий)
Вторинний текст, бордери ghost-елементів, іконки в нейтральному стані.

| Токен | Значення |
|-------|----------|
| `--color-secondary` | `#7A8878` |
| `--color-secondary-border` | `var(--color-secondary)` |

### Inverse (білий і прозорі варіанти)
Використовуються на кольорових поверхнях (рожева картка, matte-фон).
`muted-1 → muted-4`: від майже прозорого до 72% білого.

| Токен | Значення |
|-------|----------|
| `--color-inverse` | `#FFFFFF` |
| `--color-inverse-muted-1` | `rgba(255,255,255,0.18)` |
| `--color-inverse-muted-2` | `rgba(255,255,255,0.28)` |
| `--color-inverse-muted-3` | `rgba(255,255,255,0.48)` |
| `--color-inverse-muted-4` | `rgba(255,255,255,0.72)` |

### Text

| Токен | Значення | Використання |
|-------|----------|--------------|
| `--color-text-primary` | `#1C1C1C` | Основний текст |
| `--color-text-secondary` | `var(--color-secondary)` | Вторинний текст, мета-інфо |
| `--color-text-inverse` | `var(--color-inverse)` | Текст на темних/кольорових поверхнях |
| `--color-text-accent` | `var(--color-accent-main)` | Акцентний текст, посилання |
| `--color-text-swap` | `#9E7B5A` | Текст, специфічний для swap-функціоналу |
| `--color-placeholder` | `rgba(28,28,28,0.32)` | Плейсхолдери інпутів (похідне від text-primary) |

🚫 Темно-зелений текст заборонений — конкурує з фото рослин.

### Semantic

| Токен | Значення | Використання |
|-------|----------|--------------|
| `--color-success-primary` | `rgba(74,110,40,1)` / `#4A6E28` | Успішні стани |
| `--color-success-secondary` | `rgba(181,210,150,0.2)` | Фон успішних станів |
| `--color-warning-primary` | `rgba(160,110,40,1)` | Попередження |
| `--color-warning-secondary` | `rgba(212,184,150,0.2)` | Фон попереджень |
| `--color-error-primary` | `#BF3D52` | Помилки |
| `--color-error-secondary` | `rgba(191,61,82,0.1)` | Фон помилок |
| `--color-error-muted` | `rgba(191,61,82,0.35)` | Приглушений error border |

---

## Типографія

Єдиний шрифт платформи: **Inter**. Серифні шрифти протестовані й відхилені.

| Токен | Значення |
|-------|----------|
| `--font-sans` | `'Inter', sans-serif` |
| `--font-size-xxs` | `9px` |
| `--font-size-xs` | `10px` |
| `--font-size-s` | `12px` |
| `--font-size-m` | `14px` (базовий розмір `body`) |
| `--font-size-l` | `20px` (заголовки компонентів) |
| `--font-weight-normal` | `400` |
| `--font-weight-bold` | `500` |
| `--line-height-base` | `1` |

Правило ваги: `400` всюди, крім назв і primary-кнопок (`500`). Немає `600` — важчих ваг у системі нема.

---

## Border

Єдиний розмір бордера в системі — тонкий, ненав'язливий.

| Токен | Значення |
|-------|----------|
| `--border-size` | `0.5px` |

---

## Border Radius

| Токен | Значення | Використання |
|-------|----------|--------------|
| `--radius-sm` | `8px` | Badges, теги, chips |
| `--radius-md` | `12px` | Інпути, невеликі картки, StatusIcon |
| `--radius-lg` | `16px` | Картки товарів, модалі |
| `--radius-xl` | `24px` | Великі панелі, StepPopup |
| `--radius-full` | `9999px` | Кнопки-пілюлі, теги, chips, аватари |

Дизайн загалом тяжіє до великих радіусів — картки, кнопки, поля завжди rounded.

---

## Spacing

Базова одиниця — `4px`, Tailwind v4 spacing scale використовується як є.

| Токен | Значення | Використання |
|-------|----------|--------------|
| `--spacing-card-pad` | `16px` | Внутрішній padding картки |
| `--spacing-section` | `48px` | Відступ між секціями |
| `--spacing-page-x` | `24px` | Горизонтальний padding сторінки (mobile) |

Desktop page padding (`100px`) і header/footer/sticky-cta padding (`40px`) задаються inline на рівні сторінки/компонента, не токенізовані як CSS-змінна spacing-групи (див. `--header-padding-x`, `--footer-padding-x`, `--sticky-cta-padding-x` нижче).

Без `max-width` на page container.

---

## Тіні

| Токен | Значення | Використання |
|-------|----------|--------------|
| `--shadow-card` | `0 2px 12px rgba(74,90,72,0.08)` | Картки в спокої |
| `--shadow-card-hover` | `0 4px 24px rgba(74,90,72,0.14)` | Картки на hover |
| `--shadow-drawer` | `0 0 40px rgba(74,90,72,0.12)` | Side drawer, bottom sheet |
| `--shadow-sticky` | `0 -2px 16px rgba(74,90,72,0.08)` | Sticky CTA, header (той самий токен, напрямок через box-shadow) |

Матові тіні, без різких країв. Кольорова база тіней — `rgba(74,90,72,*)` (sage), не коричневий.

---

## Motion

Mobile — мінімальні анімації. Desktop — повні. Ніколи не блокувати UI під час async операцій.

| Токен | Значення | Використання |
|-------|----------|--------------|
| `--duration-fast` | `120ms` | Hover, fade кнопок |
| `--duration-base` | `200ms` | Більшість переходів |
| `--duration-slow` | `350ms` | Модалі, drawer, попапи |
| `--duration-enter` | `400ms` | Поява великих елементів |
| `--easing-base` | `cubic-bezier(0.4, 0, 0.2, 1)` | Базовий easing |
| `--easing-enter` | `cubic-bezier(0, 0, 0.2, 1)` | Вхідні анімації |
| `--easing-exit` | `cubic-bezier(0.4, 0, 1, 1)` | Вихідні анімації |
| `--easing-spring` | `cubic-bezier(0.34, 1.56, 0.64, 1)` | Пружинні (spring) переходи |

---

## Z-Index

Порядок: `sticky < filter < fab < sticky-cta < drawer < tabs < step-popup-backdrop < step-popup < header < popup < overlay < tooltip`

| Токен | Значення | Використання |
|-------|----------|--------------|
| `--z-sticky` | `10` | Sticky фото-колонка (деталі) |
| `--z-filter-btn` | `20` | Sticky фільтр іконка |
| `--z-fab` | `25` | Floating action button |
| `--z-sticky-cta` | `30` | Sticky CTA бар знизу |
| `--z-drawer` | `40` | CartDrawer, FilterPopup |
| `--z-tabs` | `45` | Sticky таби профілю |
| `--z-step-popup-backdrop` | `48` | StepPopup backdrop |
| `--z-step-popup` | `49` | StepPopup |
| `--z-header` | `50` | Глобальний хедер |
| `--z-popup` | `60` | AccountPopup, Toast |
| `--z-overlay` | `70` | Backdrop модалок |
| `--z-tooltip` | `100` | Tooltip (портал, поверх усього) |

---

## Компонентні токени

Компонентні токени — похідні від примітивів вище (кольори, radius, spacing, border, shadow). Кожен компонент має власний блок у `globals.css` з повним набором станів. Тут — огляд по компонентах; за точними значеннями конкретного стану звертатись до `globals.css`.

### Button (`--btn-*`)
Один розмір per variant по факту (sm/md/lg відрізняються тільки padding/font-size), `--btn-radius: var(--radius-full)` завжди.
Варіанти: `primary` (accent gradient, білий текст, font-weight 500), `white` (білий фон, accent текст), `secondary` (inverse-muted-1, для рожевих карток), `ghost` (прозорий, secondary border/текст), `danger` (crimson gradient), `plain` (текстова кнопка без фону).
Кожен варіант має набір станів: default / hover / focus / disabled / failure. Плюс shimmer-токени для loading-стану.

### Card (`--card-*`)
Три варіанти: `white` (inverse-muted-3 фон, без тіні на page-bg), `pink` (accent gradient, рожева тінь, inverse текст), `matte` (inverse-muted-1 + `blur(16px) saturate(1.35)`, inverse текст).
`radius-lg` і `spacing-card-pad` — спільні для всіх варіантів.

### Tag (`--tag-*`)
Два варіанти: `default` (base-gradient фон, secondary текст) і `white` (inverse-muted-3, тільки на translucent картках). Border — через `inset box-shadow 0.5px`, не `border`. Два розміри: `s` (картки) і `l` (детальна сторінка). Іконка — лише stroke, без fill.

### Chip (`--chip-*`)
Фільтри, тулбар сортування. Стани: default / hover / focus-visible / selected / selected+hover / selected+focus-visible / disabled. `radius-full`.

### Input / Textarea (`--input-*`, `--textarea-*`)
Два варіанти поверхні: `white` (світлий фон) і `ghost` (напівпрозорий, на кольорових поверхнях). Стани: default / focus / filled / disabled / readonly / error / success. Textarea переважно наслідує токени Input через `var()`, крім `radius` (`radius-lg` замість `radius-full`) і `min-height: 200px`.

### Status Badge / Status Icon (`--status-*`, `--status-icon-*`)
StatusBadge: 3 варіанти (`healthy` / `care` / `sick`), кожен = color + bg + dot з відповідного semantic-токена. StatusIcon: фіксований розмір `36px`, `radius-md`, accent-muted фон.

### Tooltip (`--tooltip-*`)
Темний фон `rgba(95,107,93,0.95)`, білий текст, `max-width: 220px`. Trigger — окрема іконка `20px` з власним hover/focus.

### Step Popup (`--step-popup-*`)
Модальний попап `480px` шириною, `radius-xl`, білий фон, окремий backdrop-токен.

### Step Progress (`--progress-*`, `--step-progress-*`)
Тонка лінія прогресу (`2px`) + stage-індикатори з окремими токенами upcoming/done/active.

### Counter (`--counter-*`)
Маленький бейдж-лічильник (`14px`, `50%` radius), накладається на інші елементи — має `box-shadow` "obводку" кольором фону сторінки.

### CharsList (`--chars-*`)
Рядки характеристик товару. `--chars-row-label-width: 105px` — фіксована ширина колонки лейблів (легасі-токен; поточна реалізація компонента використовує CSS Grid + Subgrid замість хардкоду, токен лишається для сумісності).

### Listing Status Banner / Inline (`--lsb-*`, `--listing-status-*`)
Дві форми відображення статусу оголошення (банер на сторінці деталей, інлайн-бейдж у списках). Три статуси: `reserved` / `completed` / `archived`, кожен з власним набором color/bg/border/dot.

### Header / Footer / Sticky CTA (`--header-*`, `--footer-*`, `--sticky-cta-*`)
Спільний горизонтальний padding — `40px` — у всіх трьох. Header і Sticky CTA — sticky/fixed з gradient fade (без `backdrop-filter`), протилежні напрямки (вниз / вгору відповідно). Footer — прозорий, успадковує фон сторінки.

---

## Ефекти

**Frosted glass** — застосовується sparingly (matte-картки, попапи).
```css
backdrop-filter: blur(16px) saturate(1.35);
```
Header і Sticky CTA свідомо **без** `backdrop-filter` — тільки gradient fade.

**Watercolor texture** — тільки в hero і empty state зонах. Ніколи на картках товарів.

---

## Що не використовуємо

- Чорний (`#000000`) і білий (`#FFFFFF`) як основні кольори великих поверхонь — замінені на sage-градієнт
- Темно-зелений текст — конкурує з фото рослин
- Яскравий помаранчевий, лаймовий зелений — порушують тональність
- Змішування cream і light green в одному елементі
- `font-weight: 600` і вище — у системі лише `400` / `500`
- Dark mode — не в MVP
- Хардкод border-radius/spacing/color значень у компонентах — тільки токени з `globals.css`
