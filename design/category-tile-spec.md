# CategoryTile · CategoryTileGroup — спека

**Статус:** узгоджено (крок 2 воркфлоу)
**Дизайн:** `plant-crossing/design/pages/plant-create/category_tile.html`
**Іконки:** `components/features/plant/CategoryTileGroup/icons/*.tsx` (17 шт., нормалізовані — див. §5; у репозиторії docs їх немає)
**Використання:** виключно крок 2 форми створення оголошення («Назва та категорія»).
Каталог отримає власний категорійний компонент — цей не переюзається.

---

## 1. Що змінюється в наявних документах

| Файл | Що правимо |
|---|---|
| `listing-form-spec-v2.md` §3 Крок 1 | ❌ «opacity іконки покривається станом `disabled` компонента `Field`» — **скасовано**. При ліміті 3/3 недоступні лише необрані тайли, а не поле цілком; `Field.disabled` цього не виражає. Opacity іконки — власний токен тайла. |
| `listing-form-spec-v2.md` §9 п.5 | ❌ вимога «`Field` отримує формальний `disabled`-стан заради тайлів» — **знята**. |
| `listing-form-spec-v2.md` §9 п.3 | ❌ «Іконка — 56×56, виняток fly 37×37» — **виняток скасовано**. Після нормалізації всі 17 іконок рендеряться в 56×56 без винятків (§5). |
| `listing-form-spec-v2.md` §3.0 | Tag у банері рекомендованих категорій: `label` = **«згенеровано AI»**, не «AI». |
| `listing-form-spec-v2.md` §2 | `identifyPlant` — таймаут, `AbortController`, стан кнопки «Далі» (§8). |
| `Створення оголошення` (standalone) | **Оновлено** — категорійний блок переверстано: `absolute` для тексту й іконки прибирається, іконки замінюються нормалізованими, `iconOpacity`/`tileBg`/`nameColor` як інлайн-обчислення зникають на користь CSS-станів. |

---

## 2. CategoryTile

Презентаційний, stateless, без знання про ліміт і сусідів.

### Пропи

```ts
type CategoryTileProps = Omit<
  ComponentPropsWithoutRef<'button'>,
  'children' | 'type' | 'className' | 'style'
> & {
  label: string;
  example?: string;
  icon: ComponentType<SVGProps<SVGSVGElement>>;
  selected: boolean;
  order?: number;   // рендериться тільки коли selected
  disabled?: boolean;
};
```

`iconSize` **відсутній** — після нормалізації всі іконки одного оптичного розміру.
`order` без `selected` ігнорується. `disabled` і `selected` одночасно неможливі — гарантує група.

### DOM-структура

```
<button class={tile}>                                ← flex column, min-height
  {selected && <span class={badge}>{order}</span>}   ← єдиний absolute, top/right
  <span class={icon}><Icon /></span>                 ← flex-shrink: 0, justify-content: flex-end
                                                        + padding-right: --icon-inset-right
  <span class={spacer} />                            ← flex: 1
  <span class={text}>
    <span class={label}>{label}</span>
    {example && <span class={example}>{example}</span>}
  </span>
</button>
```

Ані текст, ані іконка не позиціонуються через `position: absolute` — тільки потік.
Це переписує попередній дизайн, де текстовий блок був `absolute; bottom: 9px`.

### Обрізання тексту

- `label` — `-webkit-line-clamp: 2`
- `example` — `-webkit-line-clamp: 3`

Однорядкове обрізання назви заборонено: назва — єдиний носій сенсу тайла. Найдовша з 17 («Декоративно-листяні») влазить у 2 рядки навіть у 2-колонковому гріді на 320px, тож clamp — страховка, а не робочий механізм.

### Анімація

`transition: background-color .12s ease, box-shadow .12s ease, color .12s ease`.
`transition: all` заборонено. Під `prefers-reduced-motion: reduce` → `transition: none`.

### Стани

| Стан | Умова | Візуально |
|---|---|---|
| `default` | необраний, доступний | `--bg`, іконка `.4` |
| `hover` | тільки `@media (hover: hover)` | `--hover-bg` |
| `focus-visible` | клавіатура | `--focus-ring` |
| `selected` | обраний | `--selected-bg` + `--selected-shadow` + бейдж, іконка в accent |
| `selected + hover` | | `--selected-hover-bg` |
| `selected + focus-visible` | | `--selected-shadow, --focus-ring` в одному `box-shadow` |
| `disabled` | необраний при ліміті | `--disabled-bg`, іконка `.18`, поза tab-order, `cursor: default` |

Skeleton/loading — **немає**: список локальний і статичний.

---

## 3. Токени — `CategoryTile/styles.module.css`

Неймспейс `--category-tile-*`. У `globals.css` не виносяться.

### Похідні від глобальних токенів

| Токен | Значення |
|---|---|
| `--category-tile-radius` | `var(--radius-lg)` |
| `--category-tile-label-size` | `var(--font-size-s)` |
| `--category-tile-label-weight` | `var(--font-weight-bold)` |
| `--category-tile-icon-color` | `var(--color-secondary)` |
| `--category-tile-label-color` | `var(--color-text-primary)` |
| `--category-tile-focus-ring` | `0 0 0 2px var(--color-accent-deep)` |
| `--category-tile-selected-shadow` | `inset 0 0 0 1px var(--color-accent-main)` |
| `--category-tile-selected-label-color` | `var(--color-accent-main)` |
| `--category-tile-selected-icon-color` | `var(--color-accent-main)` |
| `--category-tile-badge-bg` | `var(--color-accent-main)` |
| `--category-tile-badge-color` | `var(--color-inverse)` |
| `--category-tile-badge-font-size` | `var(--font-size-xs)` |

### Локальні кольори категорій

Не мають відповідників у `globals.css` і **свідомо лишаються локальними** — існують тільки в контексті категорійних тайлів.

| Токен | Значення | Що це |
|---|---|---|
| `--category-tile-bg` | `#dde5da` | зелена поверхня тайла |
| `--category-tile-hover-bg` | `#d5dfd1` | на ~4% темніша |
| `--category-tile-example-color` | `#5f6b5d` | приглушений green-grey, темніший за `--color-secondary` |
| `--category-tile-selected-bg` | `rgba(196, 122, 136, 0.06)` | accent-заливка |
| `--category-tile-selected-example-color` | `#a06874` | accent, темніший |
| `--category-tile-selected-hover-bg` | `rgba(196, 122, 136, 0.11)` | |
| `--category-tile-disabled-bg` | `#eef1ed` | |
| `--category-tile-disabled-label-color` | `#a4ada2` | |
| `--category-tile-disabled-example-color` | `#b6bdb4` | |
| `--category-tile-example-size` | `11px` | у `globals.css` немає токена 11px — див. нижче |

### Розміри

| Токен | Значення |
|---|---|
| `--category-tile-min-height` | `132px` |
| `--category-tile-pad-top` | `16px` *(збільшено з 8px)* |
| `--category-tile-pad-x` | `12px` |
| `--category-tile-pad-bottom` | `12px` |
| `--category-tile-icon-size` | `56px` |
| `--category-tile-icon-slot-height` | `56px` |
| `--category-tile-icon-inset-right` | `12px` |
| `--category-tile-icon-opacity` | `0.4` |
| `--category-tile-icon-opacity-disabled` | `0.18` |
| `--category-tile-label-line-height` | `1.2` |
| `--category-tile-example-size` | `11px` (локальний — див. нижче) |
| `--category-tile-example-line-height` | `1.25` |
| `--category-tile-text-gap` | `4px` |
| `--category-tile-badge-size` | `18px` |
| `--category-tile-badge-offset` | `8px` (від правого верхнього кута) |

**⚠️ Розбіжність у `globals.css`, яку треба закрити окремо.** Коментар у блоці TYPOGRAPHY заявляє `xs=11`, а фактичне значення — `--font-size-xs: 10px`. `example` тайла = 11px, тобто або `--font-size-xs` треба виправити на 11px (і перевірити всі місця, де він уже вжитий), або лишити 10px і виправити коментар. Поки що 11px живе як локальний токен тайла — це тимчасове рішення, не остаточне.

**Line-height — нижня межа 1.2, не 1.** У `globals.css` є `--line-height-base: 1`, і для однорядкових елементів (кнопки, теги, лічильники) це коректно. Для тексту, що переноситься, `1` фізично неможливий: у Inter ascender+descender ≈ 1.21em, тож при `line-height: 1` рядки накладаються — «р», «у», «ц» з першого рядка врізаються у другий. Тому `label` = `1.2` (мінімум без накладання), `example` = `1.25` (три рядки, потрібен мінімальний повітряний зазор). Це не питання смаку, а метрики шрифта.

**Позиція іконки й бейджа.** Іконка притиснута до правого краю (`justify-content: flex-end` + `padding-right: 12px`), однаково у всіх тайлів незалежно від пропорцій малюнка. Бейдж порядку — `position: absolute`, правий верхній кут, `8px`. Бейдж і верхній правий край іконки геометрично перетинаються; це прийнято свідомо — бейдж непрозорий і лежить над іконкою, яка на `opacity: 0.4`, тож візуально конфлікту немає. Рішення візуальне, не розрахункове: не зводити `--icon-inset-right` до `≥ 34px` заради формального розведення.

**⚠️ Задокументоване відхилення від конвенції:** бордер обраного тайла — `inset 0 0 0 1px`, не `0.5px`. На фоні `rgba(196,122,136,.06)` півпіксельний бордер візуально зникає, і стан `selected` тримається майже виключно на кольорі тексту. Механізм (`box-shadow: inset`) конвенції відповідає, відхилення тільки в товщині.

---

## 4. CategoryTileGroup

Тільки сітка. Не малює ні лейбл, ні лічильник, ні хінт, ні помилку — усе це дає `Field`.

### Пропи

```ts
type CategoryTileGroupProps = {
  categories: typeof CATEGORIES;   // довжина фіксується структурно, див. §5
  value: readonly CategoryId[];
  onChange: (next: CategoryId[]) => void;
  max?: number;                    // дефолт 3
};
```

`label`, `required`, `error` **відсутні** — дублювати пропи `Field` заборонено.

### Відповідальність

- `order` обраного = `value.indexOf(id) + 1` — **похідна, не збережене значення**. Знімаєш №1 → колишній №2 автоматично стає №1.
- `disabled` тайла = `!selected && value.length >= max`
- тогл: обраний → прибрати з `value`; необраний і `value.length < max` → додати в кінець
- `aria-live="polite"` з текстом «Обрано {N} з {max}»

### Стани

| Стан | Умова | Що змінюється в сітці |
|---|---|---|
| `empty` | 0 обрано | нічого |
| `partial` | 1…max−1 | обрані тайли з бейджами |
| `full` | max обрано | решта тайлів `disabled` |

`error` станом групи не є — його малює `Field`.
`prefilled` станом не є — AI-prefill візуально ідентичний `partial`, відрізняє його лише `InfoBanner` над `Field`.

### Композиція у кроці

Крок не знає ні про лейбл, ні про лічильник, ні про хінт — усе це всередині `CategoryField`:

```tsx
<CategoryField
  categories={CATEGORIES}
  value={value}
  onChange={setValue}
  max={3}
  error={errors.categories}
/>
```

- Стилі `labelRow` / `required` / `counter` / `counterFull` живуть у `CategoryField/styles.module.css`.
- `Field.counter` тут **не використовується**: це лічильник символів у нижньому рядку поруч із hint/error, інша сутність.
- Лічильник рахується там, де вже живе `value` — дублювання стану немає.

**⚠️ Відомий ризик, прийнятий свідомо.** `Field` рендерить `<label>`, який обгортає контент. `<label>` може бути прив'язаний лише до labelable-елемента; сітка з 17 кнопок таким не є, тож браузер прив'яже лейбл до першої кнопки. Рішення не чіпати `Field` прийняте свідомо; якщо колись доведеться закривати — це робиться режимом `role="group"` у `Field`, а не окремим `CategoryTileGroupField`.

### Грід

```css
.group { container-type: inline-size; }
.grid  { display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px; align-items: stretch; }
@container (max-width: 520px) { .grid { grid-template-columns: repeat(3, 1fr); } }
@container (max-width: 380px) { .grid { grid-template-columns: repeat(2, 1fr); } }
```

`align-items: stretch` обов'язково — найвищий тайл рядка диктує висоту решті, тому 2-рядковий label не ламає вирівнювання.

Media query неприпустимий: група живе всередині `StepPopup` фіксованої ширини `660px`, viewport про це нічого не знає. Прецедент container query в проєкті — `plant_card.html`.

17 категорій / 4 колонки → останній рядок неповний. Прийнято як є.

## 5. Конфіг категорій

```ts
export type CategoryOption = {
  id: string;
  label: string;
  example: string;
  icon: ComponentType<SVGProps<SVGSVGElement>>;
};

export const CATEGORIES = [ /* 17 елементів, порядок нижче */ ]
  as const satisfies readonly CategoryOption[];

export type CategoryId = (typeof CATEGORIES)[number]['id'];
```

Довжина фіксується структурно через `typeof CATEGORIES` і оновлюється сама при додаванні категорії. Хардкод `17` у tuple-типі відхилено як brittle. Кількість **обраних** (`max`) у типах не виражається — це рантайм-стан.

`CategoryId` виводиться з конфігу, тому `value`/`onChange` типобезпечні без окремого union.

### Список — 17 категорій (порядок значущий)

| # | id | label | example | icon |
|---|---|---|---|---|
| 1 | `balcony` | Балконні | напр. гортензія, петунія, пеларгонія | `hydrangea` |
| 2 | `orchid` | Орхідні | напр. ванда, фаленопсис, цимбідіум | `orchid` |
| 3 | `palm` | Пальми | напр. вашингтонія, драцена, цикус | `palm` |
| 4 | `tree` | Дерева | напр. фікус, кротон, шефлера | `ficus` |
| 5 | `bulb` | Цибулинні | напр. амарилис, гіппеаструм, клівія | `hyacinth` |
| 6 | `succulent` | Сукуленти | напр. кактус, алое, ехеверія, агава | `cactus` |
| 7 | `carnivorous` | Хижаки | напр. венерина мухоловка, росичка, непентес | `fly` |
| 8 | `conifer` | Хвойні | напр. араукарія | `conifer` |
| 9 | `foliage` | Декоративно-листяні | напр. монстера, алоказія, бегонія | `foliage` |
| 10 | `variegated` | Варіегатні | строкаті, химери, секторальні форми | `alocasia` |
| 11 | `citrus` | Цитрусові | напр. мандарин, лимон, кумкват | `lemon` |
| 12 | `flowering` | Квітучі | напр. медініла, спатифілум, гібіскус | `magnolia` |
| 13 | `vine` | В'юнкі | напр. плющ, епіпремнум, хоя, хедера | `ivy` |
| 14 | `fern` | Папороті | напр. адіантум, флебодіум, платіцеріум | `fern` |
| 15 | `bromeliad` | Бромелієві | напр. тілландсія, ананас, неорегелія | `tillandsia` |
| 16 | `cutting` | Живці | стебла, листки, гілки, бульби | `cutting` |
| 17 | `moss` | Мохи | напр. сфагнум, купинний мох, ягель | `moss` |

`example` рендериться як є — префікс «напр.» лежить у тексті конфігу, компонент його не додає (пор. «Живці» → «стебла, листки, гілки, бульби»).

### Іконки — нормалізація

Сирі файли **не були оптично нормалізовані**: контент займав від 28×77 юнітів (`fern`) до 92×70 (`palm`); `moss` сидів смугою 80×16 біля нижнього краю (центр y = 80 замість 50), `palm` — центр y = 60, `foliage` — 40, `ivy` — 41. При рендері в однаковий слот 56×56 це давало різний видимий розмір і різну висоту посадки.

Нормалізовані файли отримали обгортку:

```svg
<svg viewBox="0 0 100 100" fill="none" stroke="currentColor"
     stroke-width="{4 / scale}" stroke-linecap="round" stroke-linejoin="round">
  <g transform="translate(tx, ty) scale(scale)"> … </g>
</svg>
```

- контент вписано в спільний оптичний бокс **84×84** зі збереженням пропорцій;
- центр контенту зведено до `(50, 50)`;
- `stroke-width` перерахований як `4 / scale`, щоб товщина лінії лишалась однаковою у всіх 17 після масштабування;
- `stroke="#7a8878"` замінено на `currentColor` — інакше `selected`-стан не пофарбує іконку в accent;
- атрибути `width`/`height` прибрано — розмір задає CSS.

**Наслідок:** виняток `fly` 37×37 більше не потрібен, `iconSize` як проп прибрано.

`moss` після нормалізації лишається горизонтальною смугою (пропорція 5:1) — це властивість самого малюнка, а не позиціонування. Якщо в макеті вона виглядає надто легкою на тлі решти, іконку треба перемалювати, а не масштабувати.

Іконки підключаються як інлайн React-SVG-компоненти, **не `<img src>`** — стану «іконка не завантажилась» не існує.

---

## 6. Доступність

- тайл — `<button type="button">` з `aria-pressed={selected}`
- `aria-live="polite"`: «Обрано {N} з 3»
- `disabled` тайли поза tab-order — свідоме рішення (WCAG виключає вимкнені контроли з 1.4.3); пояснення дає хінт під грідом
- фокус-рінг видимий на всіх станах, включно з `selected`

---

## 7. Файлова структура

```
components/features/plant/CategoryTile/
  CategoryTile.tsx          // презентаційний тайл, default export
  types.ts                  // CategoryTileProps
  styles.module.css         // токени --category-tile-* + стилі тайла
  index.ts

components/features/plant/CategoryTileGroup/
  CategoryTileGroup.tsx     // сітка + логіка вибору/ліміту, default export
  types.ts                  // CategoryTileGroupProps, CategoryOption, CategoryId
  constants.ts              // CATEGORIES (17)
  styles.module.css         // тільки грід
  index.ts
  icons/
    index.ts                // барель 17 іконок
    HydrangeaIcon.tsx  OrchidIcon.tsx     PalmIcon.tsx       FicusIcon.tsx
    HyacinthIcon.tsx   CactusIcon.tsx     FlyIcon.tsx        ConiferIcon.tsx
    FoliageIcon.tsx    AlocasiaIcon.tsx   LemonIcon.tsx      MagnoliaIcon.tsx
    IvyIcon.tsx        FernIcon.tsx       TillandsiaIcon.tsx
    CuttingIcon.tsx    MossIcon.tsx

components/features/plant/CategoryField/
  CategoryField.tsx         // BaseField + лейбл-рядок + hint/error, default export
  types.ts                  // CategoryFieldProps
  styles.module.css         // .labelRow .required .counter .counterFull
  index.ts

app/dev/category-tile/
  page.tsx                  // CategoryTileDemoPage
  styles.module.css
```

**Три окремі папки, не колокація.** `CLAUDE.md` вимагає власну папку для кожного компонента; виняток для `*Field` поруч із примітивом стосується `InputField`/`TextareaField`/`SwitchField`, і поширювати його сюди підстав немає.

**Entity-рівень — `plant`, без проміжних рівнів.** Усі три компоненти доменні (категорії рослин), тому лежать під `components/features/plant/`, а не в `ui/`. Шлях відповідає розділу «Domain nesting feature-компонентів» у `CLAUDE.md`: `components/features/<entity>/<ComponentName>/`.

**`CATEGORIES` — у `CategoryTileGroup/constants.ts`**, разом з іконками, які вони посилаються. Реекспортується через `index.ts` групи.

### `CategoryField`

```ts
type CategoryFieldProps = {
  categories: typeof CATEGORIES;
  value: readonly CategoryId[];
  onChange: (next: CategoryId[]) => void;
  max?: number;      // дефолт 3
  error?: string;
};
```

`children` **немає** — `CategoryField` сам рендерить `CategoryTileGroup`. Це та сама композиція, що описана в `CLAUDE.md` для `*Field`: `BaseField` + примітив (`InputField` = `BaseField` + `Input`).

Відповідальність:

- збирає лейбл-рядок: «Категорія *» ліворуч, `{value.length} / {max}` праворуч
- лічильник при `value.length === max` → `--color-accent-main`
- `hint` = «Максимум 3 категорії — зніми одну, щоб обрати іншу.» при `value.length === max`, інакше `undefined`
- прокидає `error` у `BaseField`
- прокидає `categories` / `value` / `onChange` / `max` у `CategoryTileGroup`

`categories` лишається пропом, а не імпортом усередині — за правилом «`variant` ніколи не хардкодиться в `*Field`, завжди прокидається з батька». `CATEGORIES` передає крок.

Лічильник має `aria-hidden="true"` — озвучує його `aria-live` у `CategoryTileGroup`.

Обгортка лейбла — `<span>`, не `<div>`: `<label>` приймає лише phrasing content.

### Що змінюється у `Field`

`components/ui/Field/types.ts` — тип `label` зі `string` на `ReactNode`. Більше нічого: ні `BaseField.tsx`, ні стилі, ні решта API.

## 8. `identifyPlant` — таймаут, race condition, стан кроку 1

Рішення, узгоджене разом із компонентом (перенести в `listing-form-spec-v2.md` §2):

- `identifyPlant` стартує на «Далі» з кроку 1 (фото), **паралельно з `checkPhotos`**, не після — послідовний запуск дає суму латентностей замість максимуму.
- Поки AI працює, кнопка «Далі» на кроці 1 — `disabled` + `loading`-стан (`Button` уже має shimmer-токени для loading).
- Таймаут **2500 мс**. По таймауту — `AbortController.abort()`, перехід на крок 2 відбувається, банер рекомендованих категорій не показується, користувач обирає вручну.
- **Пізній результат відкидається назавжди.** Без abort відповідь на 4-й секунді перезапише вибір, який користувач уже зробив руками — таймаут без abort створює новий баг замість того, щоб закрити старий.
- Prefill ніколи не перезаписує непорожній `value`.

---

## 9. Відкриті питання

Немає.
