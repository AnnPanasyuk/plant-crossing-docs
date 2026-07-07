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

Обов'язкові поля: ім'я, фото, місто, email, **Telegram username або Instagram handle
(хоча б одне з двох)**.

<!--
  ВИПРАВЛЕНО: попередня версія не згадувала Telegram/Instagram.
  auth.md, layout/header.md і components-legacy.md незалежно фіксують це поле
  як обов'язкове ("хоча б одне") — форма не сабмітиться без нього.
-->

## PlantCard

- `overflow: hidden` — тільки на `.photo`, не на `.card`. На `.card` — `overflow: visible`.
  Причина: `overflow: hidden` на батьку тригерить layout pass при `transform: translateY` на hover.
- `will-change: transform` — на `.card`.
- Hover ефекти — тільки `@media (hover: hover)`. На touch скидати.
- Без `box-shadow` на hover.
- Wishlist кнопка:
  - **desktop** — reveal on hover (з'являється тільки при наведенні на картку)
  - **mobile** — завжди видима, `position: absolute top-right`

<!--
  ВИПРАВЛЕНО: попередня версія казала "завжди видима (desktop і mobile)" —
  суперечило ux-decisions.md ("hover reveal на десктопі, на мобайлі завжди
  видима") і layout/grid.md (таблиця PlantCardActions: mobile — завжди видима,
  desktop — reveal on hover). Обидва незалежні джерела узгоджені між собою,
  тому виправлено на їхню версію. Якщо в коді PlantCard зараз реалізовано
  інакше — звірити з `PlantCard.module.css` перед тим як вважати це фінальним.
-->

## Feedback

- Toast — автозакривається через 4 секунди.
- Empty states — watercolor ілюстрація допустима тільки тут і в hero зоні.
