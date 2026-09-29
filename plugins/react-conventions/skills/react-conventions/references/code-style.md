# Code style

## Contents
- Formatting and imports
- Naming and files
- File size and splitting
- TypeScript
- Components and hooks
- Comments
- Pure logic and tests
- Commits and delivery

## Formatting and imports

Prettier owns formatting: `printWidth 80`, 2 spaces, double quotes,
semicolons, `trailingComma: "all"`, and the import-sort plugin with blank
lines between groups. Don't order imports by hand; run Prettier on the files
you changed only — a folder-wide run reformats files other people are working
on. The result looks like this:

```tsx
import { Button } from "@core/ui/atoms/button";
import { Table } from "@core/ui/organisms/table";

import { SearchIcon } from "lucide-react";
import { useMemo } from "react";
import { useTranslation } from "react-i18next";

import { useGetPassagesInfinite } from "../api";
import type { Passage } from "../types/search.types";
```

Use `import type` for type-only imports and the inline `type` modifier when
mixing (`import { type Camera, cameraKeys } from "@core/api"`).

## Naming and files

- Files are kebab-case with a role suffix where one exists:
  `camera-group-list.tsx`, `use-camera-lookups.ts`, `plate-filter-rules.ts`,
  `camera-group.schema.ts`, `.types.ts`, `.constants.ts`, `.queries.ts`,
  `.feature.tsx`, `variants.ts`.
- Components are PascalCase and named for what they are on screen
  (`PlateTypeFilter`, `PassageTable`); the component that owns a screen's
  state is `<Name>Screen`.
- Hooks `useX`, column models `useXColumns`, callback props `onX`, local
  handlers `handleX`, booleans as questions (`isEdit`, `hasValidity`).
- Constants `UPPER_SNAKE` (`DEFAULT_SEARCH_FILTERS`); enum value lists
  `as const satisfies readonly T[]`, so the list and the union can't drift.

## File size and splitting

- At most **300 code lines per file** and **150 per function or component**
  (blank lines and comments excluded). Split instead of raising a limit or
  disabling the rule.
- **One component per file** in features and pages; a helper component moves
  to its own file beside its parent. Core UI modules may keep small internal
  parts together.
- Split by responsibility: sub-components → `components/`, state and effects
  → `hooks/`, pure logic → `lib/`, column models, constants and field configs
  → their own files.

## TypeScript

- Strict mode and no `any`; an unavoidable one gets an eslint-disable with the
  reason on the same line.
- Domain enums are string-literal unions
  (`type PlateMatch = "sequential" | "fuzzy" | "even" | "odd"`).
- Props are an `interface XProps` right above the component; mutually
  exclusive prop sets are a discriminated union on `variant`.
- `null` means "no value" in state (`siteId: string | null`).
- A necessary cast carries one comment saying why it's safe.

## Components and hooks

- Components are function declarations; destructure props, with defaults, in
  the signature.
- Derive flags into named consts before the JSX
  (`const input = plateQueryInput(value.match);`); render with `&&` blocks and
  at most one level of ternary.
- `useMemo` for column models, field configs and option lists; `useCallback`
  for handlers passed deep; nothing for trivia.
- Stable empty values are module constants (`const NO_HOLIDAYS: Holiday[] = [];`).
- Effects only sync with the outside (DOM measurement, subscriptions,
  resetting a form when a dialog opens), always with cleanup.
- Grouped state changes through patch handlers:
  `onChange: (patch: Partial<PlateFilter>) => void`.

## Comments

The default is no comment. Clear names, small functions and types carry the
meaning; every extra comment makes the code longer and goes stale. Write one
only when the code can't say the reason: a non-obvious constraint, a
workaround, a decision that looks wrong without context. Keep it to a line
or two.

```ts
// Without this, opening the calendar with no value commits today: an empty
// date filter silently became "today" the moment the box was clicked.
onOpenPickNewDate={false}
```

Not worth a comment:

```ts
/** The search screen. */          // ❌ restates the name
export function SearchScreen() {

// Filter the rows by status       // ❌ restates the next line
const active = rows.filter(isActive);
```

- No JSDoc just because something is exported; add one only for a public
  API contract a caller can't infer from the types.
- No section banners, change narratives or commented-out code. When editing,
  delete comments that are stale or redundant.
- Tool directives (`eslint-disable` with its reason) and license headers stay.

## Pure logic and tests

- Business rules — which controls show, how dependent fields reset,
  formatting — are pure functions in `lib/` with a sibling `*.test.ts`.
- Unit tests: Vitest with explicit inputs; `describe` per function,
  `it("does X when Y")`. Pure logic runs in Node; a component's own DOM
  behaviour (keyboard, paste) may run in jsdom via
  `// @vitest-environment jsdom` on that file, rendering the real component.
  Run the project's no-mocks check with them.
- E2E tests: Playwright against the real app and a real backend; never
  intercept requests.

```ts
describe("updatePlateFilter", () => {
  it("forces a fuzzy match for damaged plates", () => {
    expect(updatePlateFilter(initial, { domain: "damaged" }).match).toBe("fuzzy");
  });
});
```

## Commits and delivery

- Conventional Commits: `type(scope): imperative summary` with types `feat`,
  `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf` and scopes such
  as `ui`, `feature`, `pages`, `api`, `scripts`. The summary says what changes
  for the product; the body (wrapped at ~72) says why.
- Separate design-system changes from the feature that uses them.
- Work on a branch when asked; never `--no-verify`, never force-push.
