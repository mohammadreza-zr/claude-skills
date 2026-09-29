# Data and forms

## Contents
- The API source of truth
- The feature's api/ barrel
- Infinite lists
- Mutations
- Forms
- i18n

## The API source of truth

- `openapi.yaml` is the contract. Never edit it or the generated client by
  hand; regenerate (`npm run build:api`) and adapt the features.
- Feature types are **aliases** of generated models, so a backend rename
  fails the build instead of drifting:

```ts
import type { Invoice as ApiInvoice } from "@core/api";

/** An alias, deliberately: an object alias gets the implicit index signature
 *  the pickers need, and an `interface` does not. */
export type Invoice = ApiInvoice;
```

- An endpoint that doesn't exist yet is a missing-data question (SKILL.md →
  Principle 3). When the user confirms building the UI ahead of it, type the
  UI with one minimal local type named after the future model, to be replaced
  by the generated alias.
- Every response is an envelope (`{ result, code, message, data }`). The
  `apiGet/apiPost/...` wrappers unwrap it and throw a typed error; branch on
  that, never on HTTP status.

## The feature's api/ barrel

Screens import data hooks from their feature's `api/`, never from `@core/api`
or the HTTP service.

```ts
// features/camera-group/api/index.ts
export {
  cameraGroupsEndpoints,
  cameraGroupsKeys,
  useCreateCameraGroupMutation,
  useDeleteCameraGroupMutation,
} from "@core/api";
export { useGetCameraGroupsInfinite } from "./camera-group.queries";
```

Only what the spec can't express — a `select`, a `staleTime`, an infinite
window — is written by hand, in `api/<name>.queries.ts`, as a query around the
generated **request function**:

```ts
export const useGetInvoiceSummary = (id: number) =>
  useQuery({
    queryKey: [...billingKeys.details(), "summary", id] as const,
    queryFn: ({ signal }) => getInvoiceSummary(id, { signal }),
    staleTime: 5 * 60 * 1000,
  });
```

Query keys always extend the generated factory (`all`, `lists()`,
`details()`, per-operation keys); never a hardcoded key.

## Infinite lists

Product lists are offset-based infinite scroll; numbered pagination is kept
only for special cases.

```ts
export const useGetCameraGroupsInfinite = (filters: CameraGroupListFilters) => {
  const fetchPage = useCallback(
    (window: OffsetParams, signal: AbortSignal) =>
      listCameraGroups({ ...window, ...filters }, { signal }),
    [filters],
  );

  return useOffsetInfiniteList({
    queryKey: [...cameraGroupsKeys.lists(), "infinite", filters] as const,
    fetchPage,
    staleTime: 30_000,
  });
};
```

Keep a query idle with `enabled` until its input exists (a selected id, an
open picker). Map column filters to query params in one small pure function.

## Mutations

- Use the generated mutation hooks. On success show a toast from the
  feature's locale and close the dialog; the global handler already reports
  failures, so a `catch` that only swallows says so in a comment.
- Guard double submits (`if (isPending) return;`) and cancel in-flight
  mutations when the dialog closes.

## Forms

1. **Schema factory** in `schemas/<name>.schema.ts`, taking `t` so messages
   translate, with limits imported from `constants/`. The values type is
   derived from the schema, never written by hand:

```ts
export const createCameraGroupSchema = (t: TFunction<any, any>) =>
  z.object({
    title: z
      .string()
      .trim()
      .min(1, { message: t("cameraGroup:validation.nameRequired") })
      .max(CAMERA_GROUP_NAME_MAX_LENGTH, {
        message: t("cameraGroup:validation.nameTooLong"),
      }),
    description: z
      .string()
      .trim()
      .optional()
      .or(z.literal(""))
      .transform((value) => value || undefined),
  });

export type CameraGroupFormValues = z.input<ReturnType<typeof createCameraGroupSchema>>;
```

2. **Fields as data:** a memoised `FormFieldConfig<Values>[]` rendered by
   `FormBuilder`, never a hand-wired `Controller` plus input. A field the
   builder can't express is added to the builder (or built with its `custom`
   field type) first.

```tsx
const form = useI18nZodForm(createCameraGroupSchema, { defaultValues: EMPTY_VALUES, mode: "onSubmit" });

const fields = useMemo<FormFieldConfig<CameraGroupFormValues>[]>(
  () => [
    { name: "title", type: "text", label: t("form.name"), required: true, width: "100%" },
    { name: "description", type: "textarea", label: t("form.description"), width: "100%" },
  ],
  [t],
);
```

3. **One dialog for create and edit** (frame: `ui-and-styling.md` → Dialogs);
   `reset` fills it from the record when it opens.

Surfaces outside the form that react to its values use the builder's watch
hook instead of their own subscription.

## i18n

- One namespace per feature: `public/locales/fa/<ns>.json` and
  `public/locales/en/<ns>.json` with the same keys; register the namespace in
  the typed resources map when the project has one.
- Keys are English and nested by area: `list.title`, `columns.site`,
  `actions.delete`, `empty.title`, `toast.created`, `validation.nameRequired`.
- Option labels come from enum constants (`t(\`plateType.${type}\`)`), so the
  enum is the one list of values.
