# InfoBanner — Спека компонента (v1)

**Статус:** новий компонент, замінює `ListingStatusBanner`. Витіснено з дизайну
`listing_status_banner.html` + узгоджено зі `listing-form-spec-v2.md` §3.0 (`InfoBanner`
у ListingForm лишається тим самим компонентом, тепер описаним тут повністю, а не частково).

**Конфлікт із `listing-form-spec-v2.md` §3.0, який ця спека вирішує явно:**
там `variant: 'neutral' | 'success' | 'warning'` (без `error`, `neutral` замість `info`).
Ця спека є новим джерелом правди: `variant: 'info' | 'warning' | 'success' | 'error'`.
Усі місця використання в ListingForm, де стояло `variant: 'neutral'`, тепер `variant: 'info'`
(значення не змінилось, змінилась лише назва).

---

## 1. Призначення

Домено-незалежний примітив для інформаційних, попереджувальних, успішних і помилкових
повідомлень будь-де в проєкті — не тільки в контексті оголошень.

`components/ui/InfoBanner/` — pure primitive, без знання про сутності (`Listing`, `Plant` тощо).

---

## 2. Пропси

| Проп | Тип | Дефолт | Обов'язковий | Примітка |
|---|---|---|---|---|
| `text` | `string` | — | **так** | Основний текст банера |
| `variant` | `InfoBannerVariant` (`'info' \| 'warning' \| 'success' \| 'error'`) | `'info'` |ні | Керує кольором bg/border/text/cta **і дефолтною іконкою** (див. §5) |
| `icon` | `InfoBannerIcon` (`'info' \| 'success' \| 'error' \| 'warning' \| 'sparkle'`) | похідний від `variant` |ні | **Змінено відносно v0 цієї спеки:** раніше був обов'язковим без дефолту. Тепер опційний — кожен `variant` має свою іконку за замовчуванням (§5), проп лишається лазівкою для override (головний кейс — `'sparkle'` для AI-контексту незалежно від variant, але можна передати будь-яке значення) |
| `title` | `string` | `undefined` |ні | Коли відсутній — `text` вирівнюється по центру відносно іконки (немає окремого title-рядка) |
| `action` | `InfoBannerAction \| undefined` | `undefined` |ні | Див. §3 |
| `tag` | `{ label: string; variant: 'white' } \| undefined` | `undefined` |ні | Рендериться в одному рядку з `text` (flex-wrap) |
| `dismissible` | `boolean` | `false` | ні | Коли `false` — хрестик не рендериться взагалі (не просто прихований) |
| `onDismiss` | `() => void` | `undefined` | ні | Використовується, лише якщо `dismissible === true`. Якщо `dismissible === true`, а `onDismiss` не передано — компонент все одно рендерить хрестик, але клік нічого не робить; **зупинись і повідом**, якщо Claude Code зустріне цю комбінацію в реальному використанні, не імпровізуй з fallback-поведінкою |

```ts
type InfoBannerVariant = 'info' | 'warning' | 'success' | 'error';
type InfoBannerIcon = 'info' | 'success' | 'error' | 'warning' | 'sparkle';

type InfoBannerAction = {
  label: string;
  href?: string;
  onClick?: () => void;
};
```

---

## 3. Проп `action` (екшн-кнопка)

Рендериться як `<a>`, якщо передано `href`, інакше як `<button>`, якщо передано `onClick`.
Якщо передано і `href`, і `onClick` — рендер як `<a>`, `onClick` ігнорується;
**зупинись і повідом**, не вигадуй пріоритет самостійно, якщо це станеться в реальному коді.
Стиль однаковий для обох тегів (клас `.info-banner__cta`), різниця лише в семантиці тега.
Зі стрілкою `→` в кінці лейбла — рішення на рівні контенту, не компонента (компонент не додає
стрілку автоматично).

---

## 4. Структура DOM (порядок рендеру)

```
<div class="info-banner info-banner--{variant}" [info-banner--dismissible коли dismissible]>
  <svg class="info-banner__icon" />                      — завжди
  <div class="info-banner__body">
    <span class="info-banner__title">{title}</span>       — умовно, якщо title передано
    <div class="info-banner__text-row">
      <span class="info-banner__text">{text}</span>       — завжди
      <span class="info-banner__tag">{tag.label}</span>   — умовно, якщо tag передано
    </div>
    <a|button class="info-banner__cta">{action.label}</a> — умовно, якщо action передано
  </div>
  <button class="info-banner__dismiss" aria-label="Закрити"> — умовно, якщо dismissible === true
</div>
```

Коли `title` відсутній — `.info-banner__body` не має title-рядка, `.info-banner__text-row` стає першим і єдиним
верхнім елементом, вертикальне вирівнювання іконки лишається `align-items: flex-start` на
`.info-banner`, іконка отримує `margin-top: 1px` для оптичного вирівнювання з першим рядком тексту
(без змін відносно старої `ListingStatusBanner`).

---

## 5. Іконки — `lucide-react`, дефолт від `variant`, override через `icon`

Іконка **завжди** рендериться. Джерело — `lucide-react` (вже в стеку проєкту), не inline SVG
і не окремий файл кастомних іконок, як планувалось раніше в цій спеці (§9, п. якщо буде
override-history — ця версія витісняє попередню, де іконки малювались вручну).

**Дефолтна іконка — похідна від `variant`, якщо `icon` не переданий:**

| `variant` | Дефолтна іконка (`icon` не передано) | Імпорт з `lucide-react` |
|---|---|---|
| `info` | інформація | `Info` |
| `success` | галочка в колі | `CircleCheck` |
| `error` | хрестик в колі | `CircleX` |
| `warning` | оклику в колі | `CircleAlert` |

**Override:** якщо `icon` передано явно — рендериться відповідна іконка незалежно від
`variant`. П'яте значення, якого немає серед дефолтів variant-ів, — `icon="sparkle"` →
`Sparkles` з `lucide-react`, зарезервовано для AI-контексту (наприклад `variant="info"` +
`icon="sparkle"` — так і лишається основний кейс use, як і в v0 цієї спеки).

Значення `icon` 1:1 мапляться на компонент з `lucide-react`:

```ts
const ICON_MAP: Record<InfoBannerIcon, LucideIcon> = {
  info: Info,
  success: CircleCheck,
  error: CircleX,
  warning: CircleAlert,
  sparkle: Sparkles,
};
```

Колір іконки — `currentColor` (lucide-компоненти успадковують `color` з батьківського
елемента за замовчуванням, окремий `color`-проп на іконку не передається) — той самий
принцип, що й раніше: колір іконки = `--info-banner-{variant}-color`, окремого token'а кольору
іконки немає.

Розмір — `size={16}` проп лучid-компонента, що відповідає `--info-banner-icon-size`. **Розмір
задається пропом `size`, не CSS-класом** (на відміну від попередньої версії з inline SVG) —
токен `--info-banner-icon-size` лишається джерелом правди для значення, але прокидається в
JS/TS-константу, не через `var()` напряму в SVG. Якщо це створює розсинхрон між CSS-токеном
і хардкод-числом в TSX — зафіксуй це як компромісне рішення, не намагайся зробити
`size="var(--info-banner-icon-size)"` (lucide приймає `number`, не CSS-рядок).

---

## 6. Файлова структура

```
components/ui/InfoBanner/
  InfoBanner.tsx           — компонент, default export
  InfoBanner.module.css    — стилі, локальні класи .info-banner__*
  icons.ts                 — ICON_MAP (InfoBannerIcon → lucide-react компонент), named export
  types.ts                 — InfoBannerVariant, InfoBannerIcon, InfoBannerAction (named exports)
  index.ts                 — export { default as InfoBanner } from './InfoBanner'
                              export type { InfoBannerVariant, InfoBannerIcon, InfoBannerAction } from './types'
```

Кастомних inline-SVG іконок більше немає — усі п'ять значень `icon` мапляться на існуючі
компоненти `lucide-react` (`Info`, `CircleCheck`, `CircleX`, `CircleAlert`, `Sparkles`),
імпортовані напряму в `icons.ts`.

`InfoBanner.module.css` — перший рядок файлу: коментар зі списком CSS custom properties, що
інжектяться ззовні через `style` (тут таких немає — усі варіанти покриваються класами
`.info-banner--info/.info-banner--warning/.info-banner--success/.info-banner--error`, не інлайновими стилями; окремий коментар
про "unresolved variable" тут не застосовний).

---

## 7. Токени (`globals.css`)

Секція `--lsb-*` перейменовується на `--info-banner-*`. Значення структурних і типографічних токенів —
без змін, лише префікс. Повна таблиця значень — у файлі мокапу
`info_banner.html` (розділ "Токени · globals.css").

Нове відносно старого `--lsb-*`:
- `--info-banner-cta-gap` (3px) — раніше хардкод `gap: 3px` у стилі `.lsb__cta`, тепер токенізовано.
- `--info-banner-dismiss-size`, `--info-banner-dismiss-opacity`, `--info-banner-dismiss-hover-opacity` — новий блок,
  дизайну dismiss-хреста в старому `ListingStatusBanner` не було, погоджено в цій сесії.
- `--info-banner-tag-gap` — відступ між `text` і `tag` в одному рядку.
- `--info-banner-{variant}-cta-color`, `--info-banner-{variant}-cta-border` — раніше існували тільки для
  `completed` (тепер `success`). Тепер `action` доступний для будь-якого variant, тому
  додано аналогічну пару токенів для `info`, `warning`, `error`.
- Блок `error` — цілком новий, значень-попередників немає. `--info-banner-error-color: #8f2e3d`
  — ручне затемнення `--color-error-primary` (#bf3d52) за тим самим принципом, що й
  `success`/`warning` (текст темніший за primary-колір для читабельності на світлому фоні).
  **Перевір контраст на реальному фоні `--info-banner-error-bg` перед мержем** — це підібране, не
  виміряне значення.

`--listing-status-*` (компонент `ListingStatusInline`, пігулка зі статусом на картці
оголошення) — **не перейменовується і не чіпається**. Це окремий компонент, зав'язаний
конкретно на `ListingStatus` (`RESERVED/COMPLETED/ARCHIVED`), не на InfoBanner. Якщо мала на
увазі й його — скажи явно, я окремо пройдусь.

---

## 8. Заміна використань

- `listing-form-spec-v2.md` §3.0: усі рядки таблиці "Використання по кроках форми" —
  `variant: 'neutral'` → `variant: 'info'`. Значення пропу не змінюється по суті, тільки назва.
- Будь-де, де в коді ще залишився імпорт `ListingStatusBanner` — замінити на `InfoBanner` з
  відповідним мапінгом variant: `reserved→warning`, `completed→success`, `archived→info`.
  Якщо на момент виконання прompt'а компонент `ListingStatusBanner` ще не був реалізований у
  коді (тільки в дизайні) — цей пункт неактуальний, Claude Code про це повідомляє, не шукає
  файли, яких немає.

---

## 9. Відкриті питання (не блокують реалізацію, але не вирішені мною одноосібно)

1. Точне значення `--info-banner-error-color` (#8f2e3d) — підібране за аналогією, не звірене з
   дизайнером/контраст-чекером.
2. `size={16}` для lucide-іконки — хардкод-число в TSX, синхронізоване з `--info-banner-icon-size`
   вручну, не через CSS-змінну (lucide цього не підтримує). Якщо колись `--info-banner-icon-size`
   зміниться в `globals.css`, число в `icons.ts`/`InfoBanner.tsx` треба буде оновити окремо
   — точка можливого розсинхрону, варто тримати в голові при рефакторингу токенів.
