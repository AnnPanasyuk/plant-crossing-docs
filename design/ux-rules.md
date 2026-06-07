# UX Rules

Глобальні правила для всієї платформи. Не прив'язані до конкретних компонентів.

---

## Inputs

- Без слайдерів і range inputs — ніде на платформі.
- Без calendar picker — дати через текстовий інпут з пресетом формату `dd.mm.yyyy`.
- Без dropdown — використовувати Chip select.

## Layout

- Без горизонтального скролу — ніде на платформі.
- Два брейкпоінти: mobile `< 1024px` (touch, включає планшети) / desktop `≥ 1024px` (hover).
  Немає `sm / md / lg` — тільки `mobile / desktop`.
- Без breadcrumbs — структура платформи плоска. Замінено на BackButton.

## Полив

`wateringIndicator` описує стан ґрунту, ніколи частоту:
- ✅ "коли верхній шар 2–3 см сухий"
- ❌ "кожні 3 дні"

## Авторизація

- Каталог — відкритий без логіну.
- Кошик — додавати можна без логіну.
- Логін тригериться при: checkout · клік "Запропонувати обмін" · клік wishlist.
- Email верифікація — відсутня (знижує friction).

## Реєстрація

Обов'язкові поля: ім'я, фото, місто, email.

## PlantCard

- `overflow: hidden` — тільки на `.photo`, не на `.card`. На `.card` — `overflow: visible`.
  Причина: `overflow: hidden` на батьку тригерить layout pass при `transform: translateY` на hover.
- `will-change: transform` — на `.card`.
- Hover ефекти — тільки `@media (hover: hover)`. На touch скидати.
- Без `box-shadow` на hover.
- Wishlist кнопка — завжди видима (desktop і mobile), `position: absolute top-right`.

## Feedback

- Toast — автозакривається через 4 секунди.
- Empty states — watercolor ілюстрація допустима тільки тут і в hero зоні.
