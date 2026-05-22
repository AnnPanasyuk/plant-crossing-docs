# Header Layout

Глобальний хедер присутній на всіх сторінках платформи.
`position: sticky; top: 0; z-index: var(--z-header)`

---

## Стилі

```css
height: 56px;
padding: 0 40px;
background: linear-gradient(to bottom, #E8EBE8 70%, rgba(232,235,232,0) 100%);
border-bottom: var(--border-size) solid var(--color-inverse-muted-4);
box-shadow: var(--shadow-sticky);
backdrop-filter: none;
```

---

## Структура

`grid-template-columns: auto 1fr auto`
[ Лого ]        [ Пошук ]        [ Нове оголошення · Акаунт · Кошик ]

### Колонка 1 — Лого
- `flex-shrink: 0`, ліворуч
- Logo mark (26×26px, border-radius 7px, accent gradient) + текст "PlantCrossing"
- Клік → головна сторінка (каталог)

### Колонка 2 — Пошук
- `justify-self: center; width: 100%; max-width: 380px`
- Візуально ідентичний `Button white`: `background: #fff`, `border`, `border-radius: var(--btn-radius)`, `padding: var(--btn-padding)`
- Іконка пошуку (14px) + placeholder
- На mobile: іконка → розгортає інпут на повну ширину

### Колонка 3 — Дії (праворуч)
Всі три елементи — компонент `Button white` з токенами `--btn-padding`, `--btn-radius`, `--btn-font-size`:

1. **Нове оголошення** — іконка `+` + текст підпис
   - На mobile: прибирається з хедера → FAB внизу праворуч
2. **Акаунт** — іконка юзера (icon-only, `padding: 8px 10px`)
   - Клік → AccountPopup (3 сценарії, див. нижче)
3. **Кошик** — іконка кошика (icon-only, `padding: 8px 10px`) + badge з кількістю
   - Клік → CartDrawer
   - Додавати товари можна без авторизації

---

## Сценарії AccountPopup

### Сценарій 1 — Гість, signup
Форма реєстрації. Поля: ім'я, фото (обов'язкове), місто (обов'язкове), email.
SSO: Google, Apple — основні. Email/password — опціональний.
Під кнопкою: `Реєструючись, ти погоджуєшся з [Умовами використання]`

### Сценарій 2 — Гість, login
Форма входу. SSO: Google, Apple.
Посилання "Забули пароль?" під полем пароля.

### Сценарій 3 — Авторизований
Ім'я + email + 4 посилання: Профіль · Замовлення · Обміни · Вішліст.

---

## Адаптивність

| Breakpoint | Поведінка |
|------------|-----------|
| desktop (≥1024px) | Всі три колонки, пошук розгорнутий, кнопка "Нове оголошення" видима |
| mobile (<1024px) | Лого + іконка пошуку + акаунт + кошик. FAB для нового оголошення |

---

## Що не робимо

- `backdrop-filter` на хедері — тільки gradient background
- Hover-reveal для пошуку і нового оголошення
- Темний фон хедера
- Яскравий announcement bar
