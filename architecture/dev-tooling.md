# Dev Tooling

Налаштування інструментів розробки для PlantCrossing.

---

## Stack

| Інструмент | Версія | Призначення |
|---|---|---|
| TypeScript | ~5.x | Статична типізація |
| ESLint | ~9.x (flat config) | Лінтинг коду |
| Prettier | ~3.x | Форматування |
| `@trivago/prettier-plugin-sort-imports` | latest | Автосортування імпортів |

---

## TypeScript

Конфіг: `tsconfig.json` в корені проекту.

**Ключові налаштування:**
- `strict: true` + додаткові перевірки (`noUnusedLocals`, `noUnusedParameters`, `noImplicitReturns`, `noFallthroughCasesInSwitch`, `noImplicitOverride`)
- `moduleResolution: bundler` — Next.js 13.4+ вимагає
- `jsx: preserve` — Next.js сам трансформує JSX
- `plugins: [{ "name": "next" }]` — IntelliSense для App Router
- `paths: { "@/*": ["./*"] }` — аліас для імпортів

**Аліас `@/` в імпортах:**
```ts
// замість
import { Button } from '../../components/ui/Button'
// пиши
import { Button } from '@/components/ui/Button'
```

---

## ESLint

Конфіг: `eslint.config.mjs` (flat config формат, ESLint 9+).

**Плагіни:**
- `typescript-eslint` — TypeScript правила з type checking
- `eslint-plugin-react` + `eslint-plugin-react-hooks`
- `eslint-plugin-import` — контроль імпортів (no-cycle, no-duplicates)
- `@next/eslint-plugin-next` — Next.js специфічні правила
- `eslint-config-prettier` — вимикає конфліктуючі з Prettier правила

**Що перевіряється:**
- Naming convention: `strictCamelCase` для змінних і функцій, `StrictPascalCase` для типів і компонентів, `UPPER_CASE` для enum members
- React компоненти — тільки arrow functions (крім Next.js page/layout файлів)
- Циклічні імпорти — заборонені
- `@ts-ignore` та подібні — заборонені (`ban-ts-comment: error`)

**Виключення з правил:**
- `**/route.ts` — `naming-convention` вимкнено (дозволяє `GET`, `POST`, `PUT` і т.д.)
- `**/page.tsx`, `**/layout.tsx`, `**/loading.tsx`, `**/error.tsx`, `**/not-found.tsx` — `function-component-definition` вимкнено (Next.js вимагає `default export function`)

**Запуск:**
```bash
npx eslint .                    # перевірка
npx eslint . --fix              # автовиправлення
npx eslint . --max-warnings 0   # строгий режим (CI)
```

---

## Prettier

Конфіг: `prettier.config.mjs` в корені проекту.

**Налаштування:**
```js
{
  singleQuote: true,        // одинарні лапки
  jsxSingleQuote: true,     // одинарні в JSX
  trailingComma: 'all',     // кома після останнього елемента
  printWidth: 120,          // ширина рядка
  tabWidth: 2,              // відступ
  semi: false,              // без крапки з комою (якщо так налаштовано)
  arrowParens: 'always',    // завжди дужки у arrow functions
}
```

**Порядок імпортів** (автоматично через `@trivago/prettier-plugin-sort-imports`):
1. `react`, `react-dom`
2. `next`
3. Всі зовнішні пакети
4. `@/components/**`
5. `@/lib/**`
6. `@/app/**`
7. Відносні імпорти (`./`, `../`)
8. CSS Modules (завжди останні)

**Ігноруються** (`.prettierignore`):
- `node_modules`, `.next`, `out`, `public`
- `**/*.html` — дизайн-файли
- `design/` — вся папка з макетами
- `*.md`

**Запуск:**
```bash
npx prettier --check .    # перевірка
npx prettier --write .    # форматування всього проекту
```

---

## PhpStorm

Папку `design/` позначено як **Excluded** (Mark Directory as → Excluded) — PhpStorm не показує warnings для HTML макетів.

**Рекомендовані налаштування:**
- ESLint: Settings → Languages & Frameworks → JavaScript → Code Quality Tools → ESLint → **Automatic ESLint configuration**
- Prettier: Settings → Languages & Frameworks → JavaScript → Prettier → **On save** + **On reformat**

---

## Scripts в `package.json`

```json
{
  "scripts": {
    "lint": "eslint . --max-warnings 0",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "type-check": "tsc --noEmit"
  }
}
```

Додай ці скрипти якщо їх ще немає.

---

## Pre-commit (опціонально, post-MVP)

Можна додати `husky` + `lint-staged` щоб лінтинг запускався автоматично перед кожним комітом. Відкладено до стабілізації проекту.
