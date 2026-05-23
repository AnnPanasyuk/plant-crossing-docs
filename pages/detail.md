# Page: Detail

Сторінка деталей оголошення. Доступна без авторизації.

---

## Layout

Desktop:
[ Header ]
[ Two-column: PlantGallery 50% sticky | PlantDetails 40% scroll ]
[ Footer з padding-bottom для StickyCTA ]
[ StickyCTA fixed ]

Mobile:
[ Header ]
[ PlantGallery ]
[ PlantDetails ]
[ SimilarListings ]
[ Footer ]
[ StickyCTA fixed ]

---

## Page padding

`padding: 0 100px` на desktop. Без `max-width`.

---

## Ліва колонка — PlantGallery

```css
width: 50%;
flex-shrink: 0;
display: flex;
flex-direction: column;
position: sticky;
top: 68px;
z-index: var(--z-sticky); /* 10 */
```

### Головне фото
```css
aspect-ratio: 1/1;
border-radius: var(--radius-lg);
background: var(--color-inverse-muted-3);
border: var(--border-size) solid var(--color-inverse-muted-4);
overflow: hidden;
position: relative;
```
Висота визначається виключно `aspect-ratio: 1/1` — `height` не задається.

### Лічильник
```css
position: absolute;
top: 12px; left: 14px;
background: rgba(0,0,0,0.22);
backdrop-filter: blur(6px);
border-radius: var(--radius-full);
padding: 3px 9px;
font-size: 11px; color: rgba(255,255,255,0.85);
```

### Thumbnails
Розташовані поверх фото, абсолютно позиціоновані горизонтально внизу:
```css
position: absolute;
bottom: 12px;
left: 50%;
transform: translateX(-50%);
display: flex;
gap: 6px;
z-index: 2;
backdrop-filter: blur(4px);
```

Розмір кожного thumb:
ширина = (ширина головного фото - горизонтальні padding'и) / кількість
aspect-ratio: 1/1  /* height не задається */
border-radius: var(--radius-sm)

| Breakpoint | Кількість thumbs |
|------------|-----------------|
| desktop | максимум 5 |
| mobile | 3 |

Active thumb:
```css
border: 2px solid #fff;
box-shadow: 0 0 0 1px var(--color-accent-main);
```

---

## Права колонка — PlantDetails

```css
width: 40%;
min-width: 40%;
margin: 0 auto auto;
display: flex;
flex-direction: column;
gap: 16px;
```

### 1. BackButton
```css
width: calc(33.333%); /* 1/3 від ширини колонки */
```
Variant: `ghost`. Текст: `← Назад до каталогу`.
Замінює breadcrumbs — структура платформи плоска.

### 2. StatusBanner (умовний)

Показується тільки якщо `ListingStatus !== 'ACTIVE'`. Розташований між BackButton і InfoCard.

| Статус | Текст | Вигляд |
|--------|-------|--------|
| `RESERVED` | "Зараз в процесі передачі. Якщо угода не відбудеться — рослина знову з'явиться." | warning |
| `COMPLETED` | "Рослина вже знайшла новий дім 🌿" + CTA "Переглянути схожі" | success |
| `ARCHIVED` | "Оголошення знято з публікації." | neutral |
| `DRAFT` | "Це чернетка. Тільки ти її бачиш." + CTA "Опублікувати" | neutral |

### 3. InfoCard `card-white`
- Назва: `font-size: 20px; font-weight: 500`
- Latin: `font-size: 12px; color: var(--color-text-secondary); font-style: italic`
- Теги: компонент `Tag` (Місто, Indoor тощо)
- Характеристики під тегами, **порядок зверху вниз:**
   1. Стан (`condition-badge` зелений)
   2. Горщик (діаметр + висота рослини)
   3. Складність догляду
   4. Полив — завжди soil-state: `"Коли верхній шар ґрунту (2–3 см) підсох"`, ніколи не частота

Кожен рядок: іконка 22×22px (accent bg) · ключ (flex: 0 0 80px) · значення

### 4. ActionCard `card-pink`

**Стан `ACTIVE`:**
```
[ 850 грн ]         ← font-size: 26px; font-weight: 500; color: #fff
[ або обмін ]       ← font-size: 12px; color: inverse-muted-4
[ Button white "Придбати" (3fr) ] [ Button secondary "♡ Вішліст" (2fr) ]
[ Button secondary full "⇄ Запропонувати обмін" ]
[ swap-label: підказка як працює обмін ]
```

**Стан `RESERVED`:**
```
[ 850 грн · Зарезервовано ]   ← ціна залишається, badge поруч (приглушений)
[ Button white "Придбати" disabled + tooltip "Зараз в процесі передачі" ]
[ Button secondary "♡ Вішліст" (активна) ]
[ Button secondary full "⇄ Запропонувати обмін" disabled + той самий tooltip ]
```

**Стан `COMPLETED` / `ARCHIVED`:**
```
[ 850 грн ]   ← ціна залишається для прозорості
[ кнопки Придбати і Обмін — приховані повністю ]
[ Button secondary "♡ Вішліст" — залишається ]
```

### 5. DescriptionCard `card-white`
- `section-label` "Від продавця" + текст продавця
- Divider
- `section-label` "Догляд" + AI-згенерований текст
- `Tag inverse` "згенеровано AI" після тексту

### 6. SellerCard `card-white`
- `section-label` "Продавець"
- Avatar (36px, круглий) + `@handle` + мета (місто · кількість оголошень)
- Контакти продавця (тільки для авторизованих):
  - Telegram `@username` — якщо заповнено в профілі
  - Instagram `@handle` — якщо заповнено в профілі
  - Гарантовано присутній хоча б один — валідація на реєстрації
- `Button ghost` "Інші оголошення продавця" — `width: fit-content`
- Карта локації: схематичний блакитний градієнт, `height: 200px`, `border-radius: var(--radius-md)`, пін з районом і містом

### 7. SimilarListings
- `section-label` "Схожі оголошення"
- Grid: `grid-template-columns: repeat(4, 1fr); gap: 10px`
- 2 ряди (8 карток)
- Без горизонтального scroll

---

## StickyCTA

Самостійний компонент, не картка.

```css
position: fixed;
bottom: 0; left: 0; right: 0;
z-index: var(--z-sticky-cta); /* 30 */
padding: 12px 40px 16px;
background: linear-gradient(to top, #E8EBE8 70%, rgba(232,235,232,0) 100%);
border-top: var(--border-size) solid var(--color-inverse-muted-4);
box-shadow: var(--shadow-sticky);
/* без backdrop-filter */
```

Анатомія:
[ .sticky-name-block flex:1 ]          [ Button white ] [ Button primary ]
Monstera Deliciosa                    ⇄ Запропонувати   Придбати · 850 грн
Thai Constellation (italic, secondary, 12px)

**Стан `RESERVED`:** кнопки Придбати і Обмін — disabled + tooltip.
**Стан `COMPLETED` / `ARCHIVED`:** кнопки Придбати і Обмін — приховані.

- Назва: `font-size: 14px; font-weight: 500; color: var(--color-text-primary)`
- Latin: `font-size: 12px; font-weight: 400; color: var(--color-text-secondary); font-style: italic`
- Footer отримує `padding-bottom: calc(60px + 16px)` для компенсації

---

## Button tokens (єдиний розмір)

```css
--btn-padding: 8px 14px;
--btn-font-size: var(--font-size-m); /* 14px */
--btn-radius: var(--radius-full);
font-weight: 400; /* для всіх variant, крім primary */
```

| Кнопка | Variant |
|--------|---------|
| Назад до каталогу | ghost |
| Придбати | white |
| Вішліст | secondary |
| Запропонувати обмін (ActionCard) | secondary |
| Інші оголошення продавця | ghost |
| StickyCTA обмін | white |
| StickyCTA придбати | primary |

---

## Авторизація

| Дія | Без логіну | З логіном |
|-----|-----------|-----------|
| Перегляд деталей | ✅ | ✅ |
| Перегляд контактів продавця | ❌ (приховано) | ✅ |
| Wishlist | клік → AccountPopup | ✅ |
| Купити | AccountPopup при checkout | ✅ |
| Запропонувати обмін | клік → AccountPopup | ✅ |

---

## Стани сторінки

| Стан | Поведінка |
|------|-----------|
| Завантаження | Skeleton для галереї і деталей |
| `ACTIVE` | Стандартний вигляд, всі CTA активні |
| `RESERVED` | StatusBanner + CTA disabled |
| `COMPLETED` | StatusBanner + CTA приховані |
| `ARCHIVED` | StatusBanner + CTA приховані |
| `DRAFT` | StatusBanner + CTA "Опублікувати" |
| Конфлікт stock | Toast при checkout |
| Не знайдено | EmptyState + кнопка "До каталогу" |

---

## SEO

- SSR (Next.js App Router)
- `export const revalidate = 60` + `revalidatePath` при мутації статусу
- Title: `{Назва рослини} — PlantCrossing`
- Meta description: перші 160 символів опису
- OG image: головне фото рослини

---

## Що не робимо

- Breadcrumbs
- Частота поливу ("кожні X днів")
- Телефон продавця — прибрано повністю
- Контакти продавця для неавторизованих
- Hover ефекти на mobile
- Кількість в наявності — прихована до конфлікту при checkout
- `max-width` на page container
- Горизонтальний scroll
- Дублювання статус-індикатора (банер зверху + badge в ActionCard одночасно)
