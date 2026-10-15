# Audit

Run the audit read-only first, write the report, then fix in the order below.

## 1. Inventory

For each repository list the entities and where each one is declared:

- Backend: entities/models, DTOs, mappers/serializers, response builders,
  enums, constants, socket event payload types.
- Frontend/admin: API types, feature `types.ts`, Zod schemas, form value
  types, table column models, mock/fixture types, constants.

## 2. Duplicated definitions

- Same type name declared more than once:
  `rg -n "^(export )?(interface|type|class|enum) (\w+)" -o --no-filename | sort | uniq -d`
- Overlapping field sets: for each entity, grep two or three of its distinctive
  field names together (`rg -l "compare_at_price" | xargs rg -l "stock"`) and
  open the hits that declare a shape.
- Repeated string unions and enums: grep each status value
  (`rg -n "'pending_payment'"`) and count declarations, not uses.
- Copy-paste logic: `npx jscpd --min-lines 6 --reporters console src` per repo.
- Dead exports and files: `npx ts-prune` / `npx knip`.

## 3. Naming drift and mapping layers

For each entity build a table:

| Field meaning | DB column | Entity prop | API key | Frontend type | Form name | Request key | Label |
|---|---|---|---|---|---|---|---|

Sources:

- DB columns: entity decorators (`@Column({ name })`) or migrations.
- API keys: controller responses / mappers / contract doc / MSW handlers.
- Frontend: feature types, schemas, `register("…")` / `name="…"`, request
  bodies in API hooks.
- Labels: locale files.

Every row where the cells differ is a finding. Also search for the mapping
machinery itself:

`rg -n "toSnake|toCamel|snakeCase|camelCase|mapKeys|fromApi|toApi|toPayload|toDto|\.mapper\.ts"`

## 4. Comments

`rg -n "^\s*(//|/\*|\*)" --glob "*.{ts,tsx,js,jsx,css}"` minus tool
directives. Remove with a parser-based tool (TypeScript compiler API comment
ranges), never with a regex: `//` inside strings, URLs, regexes and JSX text
is not a comment.

## 5. Report format

One file, grouped by repository, each finding as:

```
[severity] area — file:line
What: one sentence.
Why it matters: one sentence (bug, drift risk, data loss, security).
Fix: the concrete change and the single home it moves to.
```

Severity: **critical** (data loss, security, wrong money/data on the wire),
**high** (two sources already disagree, a rename breaks a flow),
**medium** (duplicate that agrees today), **low** (style, naming polish).

## 6. Fix order

1. Anything that can lose data or leak data.
2. Disagreeing duplicates and broken name chains (pick the source, align
   the rest in one change per entity, backend and frontend together).
3. Remove mapping layers once both sides use the same names.
4. Agreeing duplicates, dead code, comments, clean-code polish.

After each step: type-check, build, unit and end-to-end tests on every
affected repo; for UI, open the screens. For ORM renames, the schema diff
against a copy of the real database must be empty.
