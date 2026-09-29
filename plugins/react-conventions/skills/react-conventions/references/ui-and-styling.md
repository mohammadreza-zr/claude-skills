# UI and styling

## Contents
- Finding and extending components
- Atomic layers
- The variants pattern (cva)
- Styling from outside a component
- Tokens and Tailwind
- Feature-local looks
- Tables, empty states, dialogs, checkboxes
- Accessibility and RTL
- Stories

## Finding and extending components

1. Locate candidates through the index (SKILL.md → *Finding code*); the
   component catalog lists each module's import path and exports.
2. Open the component's props type, `variants.ts` and stories. Don't infer an
   API from a name.
3. When the component exists but differs (SKILL.md → Principle 2), extend
   it: a new variant, an optional prop, a new member of a discriminated
   `variant` union.
   Keep current defaults so other screens don't move, add a story for the new
   state, update the catalog, and tell the user what you extended and why.

Examples: widening a segmented control from three options to four; a
`width: "fill" | "hug"` variant; a `variant: "none"` member for a compound
input that sometimes has no input; a `padding: "bar"` step on a surface.

## Atomic layers

| Layer | Holds | Examples |
|---|---|---|
| atoms | single primitives with variants | Button, Checkbox, Text, Surface, SegmentedButton |
| molecules | small compositions | TextInput, SelectInput, Dialog, EmptyState, RowActionsMenu |
| organisms | complex widgets | Table, FormBuilder, DatePicker, Map |
| templates | screen skeletons | Panel, ListPanel, FormPage, StatusPage |

The shadcn CLI target (root `ui/`) is never imported; UI comes from
`@core/ui/...` only.

## The variants pattern (cva)

A styled component keeps its look in a sibling `variants.ts`; the component
file only arranges. Variant names mirror the design board.

```ts
// core/ui/atoms/surface/variants.ts
import { cva } from "class-variance-authority";

export const surfaceVariants = cva("", {
  variants: {
    tone: {
      surface: "bg-neutral-surface text-neutral-on-surface",
      tertiary: "bg-neutral-tertiary-surface text-neutral-on-tertiary-surface",
      primary: "bg-primary-10",
    },
    padding: {
      none: "",
      sm: "p-2.5",
      md: "p-4",
      bar: "px-3 py-1",
    },
    radius: { none: "", sm: "rounded-sm", md: "rounded-md", full: "rounded-full" },
  },
});
```

```tsx
// core/ui/atoms/surface/surface.tsx
function Surface({ tone, padding, radius, asChild = false, className, ...props }: SurfaceProps) {
  const Comp = asChild ? Slot.Root : "div";
  return (
    <Comp
      data-slot="surface"
      className={cn(surfaceVariants({ tone, padding, radius }), className)}
      {...props}
    />
  );
}
```

Shape: `data-slot="<name>"` on the root, `cn(xVariants(...), className)`,
`asChild` through Radix `Slot` where useful, explicit `defaultVariants`,
props typed as `ComponentProps<"div"> & VariantProps<typeof xVariants>`.

## Styling from outside a component

Outside `core`, `className` only **places and sizes**: flex/grid, gap,
width/height, min/max, margin, position, overflow, z-index, grow/shrink.
Colour, typography, border, radius, shadow, padding and motion come from the
component's variants. Never reach into children with `[&_…]` or `*:`.

```tsx
// ❌ look from outside
<Button className="bg-primary text-sm rounded-full" />
<p className="text-sm text-destructive">{error}</p>
<SegmentedButton className="[&_[data-slot=segmented-button-item]]:text-sm" />

// ✅ layout from outside, look from variants and components
<Button variant="primary" size="md" className="ms-auto w-36" />
<FieldError>{error}</FieldError>
<Text size="md" tone="muted" className="shrink-0">{label}</Text>
<SegmentedButton size="xs" tone="tertiary" width="hug" />
```

When no variant gives the look, add the variant in core: a look defined in one
place changes in one place.

## Tokens and Tailwind

- Only semantic tokens generated from the design tokens (`bg-card`,
  `bg-neutral-surface`, `text-destructive`, opacity stops like
  `bg-primary-10`, `text-neutral-40`); never Tailwind palette classes
  (`bg-white`, `text-gray-400`).
- Check what a token resolves to before trusting its name.
- The radius scale may differ from Tailwind's defaults — choose by the
  control's size.
- Write class names out in full; Tailwind can't see `` `bg-${tone}` ``.
- Token CSS is generated; change the generator, not its output.

## Feature-local looks

A look only one feature uses becomes a component inside that feature, with
its classes in `<name>.variants.ts` built with `cva` like the core ones. A
small wrapper that gives repeated markup one home is exactly right:

```tsx
export function FilterGroup({ children }: FilterGroupProps) {
  return (
    <Surface tone="surface" radius="md" padding="bar" className="flex min-w-0 items-center gap-3">
      {children}
    </Surface>
  );
}
```

## Tables, empty states, dialogs, checkboxes

- **Tables:** the shared `Table` organism with `infinite={list.tableProps}`
  (the data side is in `data-and-forms.md` → Infinite lists), `fillHeight`
  inside a flex column whose ancestors carry `min-h-0`, `showIndex` for the
  row number. Column models live in a `useXColumns()` hook built with
  `createTableColumnHelper` inside `useMemo`; row menus use `RowActionsMenu`
  with `DropdownMenuItem`s.
- **Empty states:** the shared `EmptyState` with a domain scope; callers pass
  only translated copy. Tell "nothing exists yet" apart from "no results for
  this filter".
- **Dialogs:** the shared `FormDialog` and `DeleteDialog` frames with the
  shared delete-confirmation hook; never a hand-assembled Dialog, header and
  footer.
- **Checkboxes:** only the shared `Checkbox` atom with a linked
  `Label htmlFor`; size and colour come from its variants.

## Accessibility and RTL

- Icon-only buttons get `aria-label` and `title`; every radiogroup or control
  gets a name (pass the visible label as `aria-label` when it isn't a
  `<label>`).
- Use logical utilities (`ms-`, `me-`, `ps-`, `start-`, `inset-inline-start`)
  so layouts mirror in RTL; content that is always LTR (plates, codes) sets
  `dir="ltr"`.
- Keep the loading, error and empty states the components already provide.

## Stories

Every core component has a `*.stories.tsx` beside it. A new variant or prop
gets a story. Stories document; they aren't a test suite.
