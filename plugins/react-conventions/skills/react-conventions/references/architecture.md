# Architecture

## Contents
- Layers and ownership
- Feature anatomy
- Exports
- Pages and routing
- State
- Adding a feature

## Layers and ownership

```
src/
├── core/        domain-free and shared
│   ├── ui/      the design system (see ui-and-styling.md)
│   ├── hooks/   shared hooks
│   ├── lib/     pure helpers
│   ├── services/ HTTP wrappers
│   ├── api/     the API contract and its generated client (see data-and-forms.md)
│   ├── types/   shared types
│   └── store/   the one global client store
├── features/    one folder per business module
├── pages/       one file per route
├── router/      route tables
└── stories/     storybook onboarding only
```

Path aliases: `@core/*`, `@features/*`, `@pages/*`, `@router/*`, `@assets/*`.

- **Imports flow `pages → features → core`, never backwards.** A core module
  that imports a feature can't be reused; a feature importing another feature
  creates coupling that breaks silently when either changes.
- **Core holds only what is domain-free.** Being used twice is not enough —
  ask whether it belongs to no particular business domain. Premature sharing
  is as harmful as duplication.
- **Sub-features are private** to their parent feature.
- **Pages only compose**: no business logic, state or API calls.

## Feature anatomy

Create only the folders a feature needs.

```
features/camera-group/
├── api/                         the feature's API surface
├── components/                  the screen and its parts
├── constants/                   limits, base paths, enum lists
├── hooks/                       state and effects pulled out of components
├── lib/                         pure logic and its tests
├── schemas/                     validation schemas
├── types/                       the feature's types
├── camera-group.feature.tsx     the feature entry
└── index.ts                     export { default } from "./camera-group.feature";
```

The feature entry loads the feature's i18n namespace, then renders the screen
component that owns the state:

```tsx
import { useI18nScope } from "@core/hooks";

import { CameraGroupScreen } from "./components/camera-group-screen";

export default function CameraGroupFeature() {
  const { ready } = useI18nScope("cameraGroup");
  if (!ready) return null;

  return <CameraGroupScreen />;
}
```

## Exports

- Core: **named exports through each module's `index.ts`**; no deep imports.
- UI: import one module, never a layer barrel — a barrel pulls the whole layer
  (date engines, the table, the combobox) into the chunk that touches it.
- Features and pages: **a single default export**; everything else inside a
  feature uses named exports.

```ts
import { useOffsetInfiniteList } from "@core/hooks";           // ✅
import { apiService } from "@core/services/api-service";      // ❌ deep import
import { Button } from "@core/ui/atoms/button";                // ✅
import { Button } from "@core/ui/atoms";                       // ❌ layer barrel
import CameraGroupFeature from "@features/camera-group";       // ✅
```

## Pages and routing

```tsx
import CameraGroupFeature from "@features/camera-group";

export default function CameraGroupPage() {
  return <CameraGroupFeature />;
}
```

Every page is lazy-loaded. In a React Router data router use route-level
`lazy` rather than `React.lazy`: the router awaits the chunk as part of the
navigation, so the layout can show its pending overlay instead of freezing on
the old screen.

```tsx
const page = (load: () => Promise<{ default: ComponentType }>) => ({
  lazy: async () => ({ Component: (await load()).default }),
  HydrateFallback: RoutePending,
});

{ path: "search", ...page(() => import("@pages/search")) },
```

A module with its own list/create/edit screens mounts with a splat
(`"configuration/camera/*"`) and owns its nested routes.

## State

- Server data: React Query only (`data-and-forms.md`).
- UI state: `useState`/`useReducer` in the component, lifted only to the
  nearest common owner (the `*-screen.tsx`), or a feature hook.
- Global client state: the single approved store, only when distant parts
  genuinely share it. Never add another state library.

## Adding a feature

1. Folders as in *Feature anatomy*.
2. Types and API surface — `data-and-forms.md`.
3. Locale files — `data-and-forms.md` → i18n.
4. Components — `ui-and-styling.md`; the `*-screen.tsx` owns state.
5. Feature entry + `index.ts`, then `pages/<name>.tsx` and its route.
6. Tests for `lib/` — `code-style.md` → Pure logic and tests.
