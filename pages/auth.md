# Page: Auth

Авторизація і реєстрація реалізовані як inline форми всередині `AccountPopup`.
Окремих сторінок для login/signup немає — все відбувається в попапі без зміни URL.
Виняток: `/reset-password` — окрема сторінка (технічна необхідність email флоу).

---

## Сценарії

### Сценарій 1 — Signup
Тригери:
- Клік на іконку акаунту (гість)
- Клік Wishlist без логіну
- Клік Swap без логіну
- Клік "Нове оголошення" без логіну
- Перехід до checkout з кошика

**Компонент:** `SignupForm` всередині `AccountPopup`

**Анатомія форми (зверху вниз):**
1. Заголовок: "Приєднатись"
2. `SSOButton` Google
3. `SSOButton` Apple
4. Роздільник: `— або —`
5. `Input` Ім'я (обов'язкове)
6. `Input` Email (обов'язкове, `autocomplete="email"`)
7. `Input` Пароль (опціональне, `autocomplete="new-password"`)
8. `ImageUpload` Фото профілю (обов'язкове)
9. `Input` Місто (обов'язкове, з autocomplete)
10. `Input` Telegram username (обов'язкове якщо Instagram не заповнено)
11. `Input` Instagram handle (обов'язкове якщо Telegram не заповнено)
12. `Button primary lg` "Зареєструватись"
13. ToS: `Реєструючись, ти погоджуєшся з [Умовами використання]`
14. Перемикач: "Вже є акаунт? [Увійти]"

**Правила:**
- Фото і місто — обов'язкові поля
- Telegram або Instagram — обов'язково хоча б одне; обидва опціонально
- Валідація: форма не сабмітиться якщо обидва поля контакту порожні
- Email верифікація відсутня — знижує friction
- Пароль опціональний якщо вибрано SSO
- ToS посилання під кнопкою — завжди видиме

---

### Сценарій 2 — Login
Тригер: клік "Увійти" в попапі або перемикач зі signup.

**Компонент:** `LoginForm` всередині `AccountPopup`

**Анатомія форми (зверху вниз):**
1. Заголовок: "Увійти"
2. `SSOButton` Google
3. `SSOButton` Apple
4. Роздільник: `— або —`
5. `Input` Email (`autocomplete="email"`)
6. `Input` Пароль + посилання "Забули пароль?" праворуч (`autocomplete="current-password"`)
7. `Button primary lg` "Увійти"
8. Перемикач: "Немає акаунту? [Зареєструватись]"

---

### Сценарій 3 — Авторизований користувач
Тригер: клік на іконку акаунту (авторизований).

**Компонент:** `AccountPopup` авторизований стан

**Анатомія (зверху вниз):**
1. `Avatar` md + Ім'я + Email
2. Divider
3. Посилання: Профіль
4. Посилання: Замовлення
5. Посилання: Обміни
6. Посилання: Вішліст
7. Divider
8. `Button ghost` "Вийти"

---

## Forgot Password

Реалізується як окремий крок всередині `LoginForm` → редірект на окрему сторінку.

**Флоу:**
1. Клік "Забули пароль?" в попапі
2. Попап закривається
3. Редірект на `/reset-password`
4. `Input` Email + `Button primary` "Надіслати посилання"
5. Success: "Перевір пошту — ми надіслали посилання"
6. Посилання в email → `/reset-password?token=...` → форма нового пароля

**Правила:**
- `/reset-password` — єдина окрема auth сторінка
- Токен одноразовий, expires після використання або через 1 годину

---

## SSO флоу

**Провайдери MVP:** Google, Apple
**Реалізація:** Auth.js (NextAuth) — сесії в PostgreSQL

**Флоу:**
1. Клік SSO кнопки в попапі
2. Повний редірект на провайдера (неминуче — OAuth 2.0)
3. Auth.js зберігає `callbackUrl` — після авторизації повертає на попередню сторінку
4. Якщо новий юзер і відсутнє фото, місто або контакт → inline prompt в попапі для заповнення

**Правила:**
- `callbackUrl` — обробляється Auth.js з коробки
- Не створювати акаунти без явної дії користувача

---

## AccountPopup — технічні правила

**На `lg` (десктоп):**
- Popover прив'язаний до іконки акаунту
- Border radius: `var(--radius-xl)`
- Shadow: `var(--shadow-drawer)`
- Закривається: клік поза попапом або Escape

**На `sm`/`md` (touch):**
- Bottom sheet
- Border radius: `var(--radius-xl)` верхні кути
- Закривається: свайп вниз або клік на backdrop

---

## Безпека

Auth попап рендериться в контексті батьківської сторінки.
Нижче — обов'язкові заходи для нівелювання ризиків.

### Обов'язково для MVP

**XSS захист**
- Ніколи не використовувати `dangerouslySetInnerHTML` для user-generated content
- Весь UGC (назви рослин, описи, імена) — тільки через JSX (Next.js екранує автоматично)

**httpOnly cookies**
- Токени сесії зберігаються тільки в `httpOnly` cookies
- Ніколи в `localStorage` або `sessionStorage`
- Auth.js реалізує це з коробки — не перевизначати

**autocomplete атрибути**
- Правильні атрибути на всіх інпутах форми:
```html
<!-- Login -->
<input autocomplete="email" />
<input autocomplete="current-password" />

<!-- Signup -->
<input autocomplete="email" />
<input autocomplete="new-password" />
```

**CSRF захист**
- Auth.js включає CSRF токени для всіх form submissions з коробки
- Не вимикати, не обходити

**Third-party скрипти**
- Аналітика і будь-який third-party JS — завантажувати з `defer`
- Мінімізувати кількість сторонніх скриптів на сторінках з auth попапом

### Бажано (можна після MVP)

**CSP заголовки**
```js
// next.config.js
headers: [{
  key: 'Content-Security-Policy',
  value: "script-src 'self' accounts.google.com appleid.apple.com"
}]
```

---

## Модель профілю

Єдина модель для всіх користувачів (Shafa/Vinted патерн).
Немає окремих buyer/seller акаунтів.

**Обов'язкові поля при реєстрації:**
- Ім'я
- Фото
- Місто
- Email
- Telegram username або Instagram handle (хоча б одне)

**Зарезервовано в БД:**
- `account_type` — для майбутніх комерційних акаунтів (post-MVP)

---

## Що не робимо

- Окремі сторінки `/login` `/signup` — все в попапі
- Зберігати токени в `localStorage`
- Використовувати `dangerouslySetInnerHTML` для UGC
- Збирати або показувати телефон — прибрано повністю
- Email верифікація — post-MVP
- 2FA — post-MVP
- Terms of Service enforcement — post-MVP
- Dropdown або calendar picker в будь-яких полях форми
