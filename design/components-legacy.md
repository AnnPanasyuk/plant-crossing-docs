# Components

Документація всіх UI компонентів платформи.
Архітектура: CSS Modules для стилів компонентів, Tailwind тільки для one-off layout wrappers.
Іконки: Lucide React. Шрифт: Inter.

---

## Глобальні / Shared

### Button
Основний інтерактивний елемент платформи.

**Варіанти:**
| Variant | Використання |
|---------|--------------|
| `primary` | Головна дія на сторінці (Купити, Зареєструватись) |
| `secondary` | Вторинна дія (Запропонувати обмін) |
| `ghost` | Третинна дія, навігація (Назад, Скасувати) |
| `danger` | Деструктивні дії (Видалити оголошення) |

**Розміри:**
| Size | Використання |
|------|--------------|
| `sm` | Inline дії, фільтри |
| `md` | Основний розмір |
| `lg` | CTA в hero, sticky бар |

**Стани:** default · hover · disabled · loading (spinner)

**Правила:**
- Border radius: `var(--radius-full)` для pill-кнопок, `var(--radius-md)` для прямокутних
- Primary background: `var(--color-accent-dusty)`
- Текст primary: `var(--color-text-inverse)`
- Disabled opacity: `0.5`

---

### Input
Текстове поле вводу.

**Варіанти:**
- Default
- З іконкою зліва (пошук)
- З іконкою справа (clear, show/hide password)
- Error стан (border `var(--color-error)` + error message знизу)

**Правила:**
- Border radius: `var(--radius-md)`
- Border color default: `var(--color-neutral-stone)`
- Border color focus: `var(--color-accent-dusty)`
- Placeholder color: `var(--color-text-secondary)`
- Дата: текстовий інпут з пресетом формату `dd.mm.yyyy` — без calendar picker

---

### Badge
Маленький лейбл для статусів і лічильників.

**Варіанти:**
- `status` — стан рослини (Хороший / Задовільний)
- `count` — лічильник кошика на іконці хедера
- `tag` — категорія, тег рослини

**Правила:**
- Border radius: `var(--radius-sm)`
- Розмір тексту: `var(--text-xs)`

---

### Avatar
Фото профілю користувача.

**Варіанти:**
- З фото (обов'язкове при реєстрації)
- Fallback: ініціали на фоні `var(--color-accent-blush)`

**Розміри:** sm (24px) · md (40px) · lg (64px)

**Правила:**
- Border radius: `var(--radius-full)`
- Object-fit: cover

---

### Icon
Wrapper над Lucide React для консистентних розмірів.

**Розміри:** sm (16px) · md (20px) · lg (24px)

**Правила:**
- Колір успадковується від батьківського елемента через `currentColor`
- Не використовувати кольори іконок напряму — тільки через контекст

---

### Chip
Selectable елемент для фільтрів і категорій.

**Стани:** default · selected · disabled

**Правила:**
- Border radius: `var(--radius-sm)`
- Selected background: `var(--color-accent-dusty)`
- Selected текст: `var(--color-text-inverse)`
- Default border: `var(--color-neutral-stone)`

---

### Toggle
Бінарний перемикач.

**Використання:** фільтр "Є обмін", бінарні налаштування

**Стани:** on · off · disabled

**Правила:**
- On color: `var(--color-accent-dusty)`
- Off color: `var(--color-neutral-sand)`
- Без label зліва/справа — підпис завжди зовні компонента

---

### Tooltip
Підказка для іконок без текстового підпису.

**Правила:**
- З'являється на hover (десктоп)
- Border radius: `var(--radius-sm)`
- Background: `var(--color-text-primary)`
- Текст: `var(--color-text-inverse)`
- Розмір тексту: `var(--text-xs)`

---

### Divider
Горизонтальний роздільник.

**Правила:**
- Color: `var(--color-neutral-sand)`
- Height: 1px

---

## Навігація

### Header
Глобальний хедер, детальна документація в `layout/header.md`.

**Компоненти всередині:** Logo · SearchInput · Icon(+) · AccountPopup trigger · CartIcon з Badge

---

### AccountPopup
Попап акаунту з трьома сценаріями.

**Сценарій 1 — Гість, signup:** LoginForm
**Сценарій 2 — Гість, login:** SignupForm
**Сценарій 3 — Авторизований:** Avatar + ім'я + email + 4 посилання + logout

**Правила:**
- Відкривається на клік по іконці акаунту
- Border radius: `var(--radius-xl)`
- Shadow: `var(--shadow-drawer)`
- Закривається кліком поза попапом або Escape

---

### Footer
Легкий інформаційний футер.

**Містить:**
- Лого + короткий опис платформи
- Посилання: About · Contact · Terms of Use · Privacy · Cookies
- Copyright

**Правила:**
- Background: `var(--color-base-warm)`
- Без темного фону — не конкурує з контентом
- Детальна документація в `layout/footer.md`

---

### BackButton
Кнопка повернення до каталогу на сторінці деталей.

**Вигляд:** ← Назад до каталогу

**Правила:**
- Variant: `ghost`
- Позиція: початок правої колонки сторінки деталей
- Замінює breadcrumbs — структура платформи плоска

---

## Auth

### LoginForm
Форма входу, використовується всередині AccountPopup.

**Поля:** Email · Password
**Елементи:** SSOButton (Google) · SSOButton (Apple) · посилання "Забули пароль?"

**Правила:**
- "Забули пароль?" — під полем пароля, вирівнювання праворуч
- Розділювач між SSO і email/password: `--- або ---`

---

### SignupForm
Форма реєстрації, використовується всередині AccountPopup.

**Поля:** Ім'я · Фото (обов'язкове) · Місто (обов'язкове) · Email · Password (опціональний)
**Елементи:** SSOButton (Google) · SSOButton (Apple) · ToS посилання

**Правила:**
- Під кнопкою реєстрації: `Реєструючись, ти погоджуєшся з [Умовами використання]`
- Email верифікація — відсутня (знижує friction)
- Фото і місто — обов'язкові поля

---

### SSOButton
Кнопка входу через зовнішній провайдер.

**Варіанти:** Google · Apple

**Правила:**
- Variant: `secondary` або окремий `sso` variant з логотипом провайдера
- Повна ширина всередині форми

---

## Картка товару

### PlantCard

Основна картка оголошення. Один компонент — використовується в каталозі
та блоці схожих оголошень, адаптується через container queries.

**Анатомія (зверху вниз):**
1. Фото (`aspect-ratio: 1/1`, `object-fit: cover`)
   — wishlist кнопка `absolute top-right`, frosted glass, завжди видима
2. Назва рослини
3. Ціна (`--color-text-accent`) або "Обмін" (`--color-text-swap`)
4. Місто (`#A89E88` — теплий беж, світліший за `--color-text-secondary`)
5. Footer — один елемент, пріоритет залежить від контексту:
    - категорія задана фільтром → тег рівня догляду (Easy / Medium / Hard)
    - всі рослини → тег категорії (Indoor / Сукулент / …)
    - статус `RESERVED` → `ListingStatusInline` замість тегу

**Стани картки:**

| Стан | Фото | Footer | Opacity |
|------|------|--------|---------|
| `ACTIVE` + ціна | звичайне | тег | 1 |
| `ACTIVE` + обмін | звичайне | тег | 1 |
| `RESERVED` | opacity 0.68 | `ListingStatusInline` | 0.82 |
| Skeleton | shimmer анімація | shimmer рядок | — |
| Фото error | заглушка з іконкою | тег | 1 |

**Wishlist кнопка:**
- Завжди видима (і mobile, і desktop) — `absolute top-right`
- Frosted glass: `rgba(255,255,255,0.55)` + `backdrop-filter: blur(8px) saturate(1.4)`
- Border: `0.5px solid rgba(255,255,255,0.75)`
- Active стан: `fill: var(--color-text-accent)` на іконці серця
- Hover: `scale(1.18)`, `transition: 120ms var(--easing-spring)`

**Hover стан картки** (тільки `@media (hover: hover)`):
- `transform: translateY(-2px)`
- `background: rgba(255,255,255,0.68)` (світлішає)
- `border-color: rgba(255,255,255,0.92)`
- Без `box-shadow`
- `transition: 200ms var(--easing-base)`

**Адаптивність через container queries:**
- Батько (`CatalogGrid`, `SimilarListings`) отримує `container-type: inline-size`
- `@container (max-width: 180px)` — compact: менший padding і font-size
- `@container (max-width: 140px)` — мінімум: тег і місто приховані
- Компонент не знає де рендериться — адаптується автоматично

**Контексти використання:**

| Контекст | Колонок | Gap | Особливості |
|----------|---------|-----|-------------|
| Каталог desktop | 6 | 16px | базовий розмір |
| Каталог mobile | 2 | 12px | compact через container |
| Схожі оголошення | 4 | 14px | `max-width: 50%` від viewport |

**Фікс стрибка при hover:**
- `.card { overflow: visible; will-change: transform }` — не `hidden`
- `.photo { overflow: hidden; border-radius: var(--radius-lg) var(--radius-lg) 0 0 }`
- `overflow: hidden` на батьку тригерить layout pass при `transform` — тому переноситься на `.photo`

**Файли:**
- `components/PlantCard/PlantCard.module.css` — токени на `.card {}`, стилі, container queries
- `components/CatalogGrid/CatalogGrid.module.css` — `container-type: inline-size`
- `components/SimilarListings/SimilarListings.module.css` — `container-type: inline-size`

**Що не робимо:**
- `box-shadow` на hover
- `overflow: hidden` на `.card`
- Більше одного тега в footer
- Hover ефекти на touch (`@media (hover: none)` скидає всі hover стани)

---

### PlantCardActions
Іконки дій на картці, з'являються на hover.

**Іконки:** Wishlist (серце) · Swap · Add to cart

**Правила:**
- Десктоп: reveal на hover (overlay знизу картки або кути)
- Мобайл: іконка wishlist завжди видима у верхньому куті
- Клік Wishlist без логіну → тригерить AccountPopup (login сценарій)
- Клік Swap без логіну → тригерить AccountPopup (login сценарій)

---

## Сторінка деталей

### PlantGallery
Ліва колонка сторінки деталей з фото рослини.

**Правила:**
- Sticky: залишається у фокусі поки скролиться права колонка
- На мобайлі: sticky вимкнено, фото вгорі
- Підтримка кількох фото (галерея з thumbnails)

---

### PlantDetails
Права колонка сторінки деталей.

**Анатомія (зверху вниз):**
1. BackButton (← Назад до каталогу)
2. Назва рослини
3. Ціна / мітка "Обмін"
4. Кнопки: Купити · Запропонувати обмін · Wishlist
5. Опис і характеристики
6. AI-згенерований текст догляду
7. Інфо про продавця (Avatar + ім'я + місто)

**Правила:**
- Полив: завжди soil-state-based ("коли верхній шар ґрунту висох"), ніколи частота ("кожні 3 дні")
- Breadcrumbs відсутні — замінені на BackButton

---

### StickyCTA
Sticky бар знизу сторінки деталей, поверх контенту.

**Анатомія:** Назва рослини · Button(primary, "Купити") · Button(secondary, "Запропонувати обмін")

**Правила:**
- Position: fixed bottom
- Shadow: `var(--shadow-sticky)`
- Background: `var(--color-base-canvas)` з легкою прозорістю
- Z-index вище основного контенту
- На мобайлі: кнопки на повну ширину

---

### SimilarListings
Блок схожих оголошень під правою колонкою сторінки деталей.

**Вигляд:** грід 4 колонки, `max-width: 50%` (відповідає ширині правої колонки деталей)

**Правила:**
- Заголовок секції: "Схожі оголошення"
- `container-type: inline-size` — картки адаптуються до compact розміру автоматично
- Той самий `PlantCard` компонент що і в каталозі


---

## Кошик

### CartDrawer
Бічна панель кошика.

**Правила:**
- Десктоп: side drawer справа
- Мобайл: bottom sheet
- Відкривається при кліку на іконку кошика в хедері або після додавання товару
- Закривається: клік поза drawer · Escape · кнопка ×
- Містить: список CartItem · підсумок · Button "Оформити" · Button ghost "Продовжити шопінг"
- Border radius: `var(--radius-xl)` (ліві кути для drawer, верхні для bottom sheet)
- Shadow: `var(--shadow-drawer)`

---

### CartItem
Рядок товару всередині CartDrawer.

**Анатомія:** фото · назва · ціна · кнопка видалення

**Правила:**
- Фото: маленьке, border radius `var(--radius-md)`
- Кількість не показується якщо stock > 1 — конфлікт показується тільки при checkout

---

## Фільтри

### FilterButton
Sticky іконка фільтра внизу каталогу.

**Правила:**
- Position: fixed bottom center або bottom right
- На мобайлі поруч з FAB нового оголошення (або замість, залежно від сторінки)
- Клік → відкриває FilterPopup
- Badge з кількістю активних фільтрів

---

### FilterPopup
Модальне вікно з фільтрами каталогу.

**6 опціональних кроків:**
1. Категорія → Chip (мультиселект)
2. Стан рослини → Toggle (2 варіанти: Хороший / Задовільний)
3. Діаметр горщика → Chip (XS / S / M / L / XL)
4. Є обмін → Toggle
5. Ціна → два Input (від / до) — без слайдера
6. Місто → Input з autocomplete

**Правила:**
- Всі кроки опціональні
- Border radius: `var(--radius-xl)`
- Shadow: `var(--shadow-drawer)`
- Кнопки внизу: "Скинути" (ghost) · "Показати результати" (primary)
- Без dropdown і range slider — відповідно до глобальних UX правил

---

## Форми / Create listing

### ListingForm
Форма створення нового оголошення.

**Поля:**
- ImageUpload (обов'язкове)
- Назва рослини (AI autocomplete після скану)
- Категорія (Chip select)
- Стан рослини (Toggle)
- Діаметр горщика (Chip select)
- Ціна або "Тільки обмін" (Toggle)
- Опис
- Місто (Input з autocomplete)

**Правила:**
- AI скан фото → pre-fill полів (вид, складність, умови) як autocomplete
- Доступна тільки авторизованим користувачам

---

### ImageUpload
Завантаження фото рослини.

**Правила:**
- Drag & drop + клік для вибору файлу
- Preview після завантаження
- Border radius: `var(--radius-lg)`
- Border: dashed `var(--color-neutral-stone)` в стані очікування

---

## Feedback / Empty states

### EmptyState
Порожній стан для каталогу, вішліста, результатів пошуку.

**Анатомія:** watercolor ілюстрація · заголовок · підзаголовок · опціональний CTA

**Правила:**
- Watercolor texture дозволена тільки тут і в hero зоні
- Тон: дружній, community-відчуття

---

### Toast
Системне повідомлення про результат дії.

**Варіанти:**
| Variant | Колір | Використання |
|---------|-------|--------------|
| `success` | `var(--color-success)` | Товар додано, обмін запропоновано |
| `error` | `var(--color-error)` | Помилка, конфлікт stock |
| `warning` | `var(--color-warning)` | Попередження |

**Правила:**
- Позиція: top right (десктоп) / top center (мобайл)
- Автозакривається через 4 секунди
- Border radius: `var(--radius-md)`

---

### Skeleton
Loading стан компонентів під час завантаження даних.

**Варіанти:** PlantCard skeleton · PlantDetails skeleton

**Правила:**
- Анімація: pulse (opacity 1 → 0.5 → 1)
- Background: `var(--color-neutral-sand)`
- Border radius відповідає реальному компоненту
