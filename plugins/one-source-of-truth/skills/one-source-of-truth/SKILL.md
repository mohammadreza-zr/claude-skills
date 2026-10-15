---
name: one-source-of-truth
description: One entity, one type, one name end to end — every domain entity, enum, constant, rule and helper is defined exactly once and everything else derives from it; a field keeps the exact same name in the database column, the backend entity, the API JSON, the frontend type, the form field and the query param, with no renaming, case-conversion or mapper layer in between (only omitting private fields and adding computed ones); zero code comments; clean-code rules; and a repeatable audit that finds duplicated types, copy-pasted logic, naming drift and mapping layers across backend, frontend and admin repos. Use this skill whenever writing, reviewing, refactoring or auditing code in any stack (NestJS/TypeORM, Express, React, Vue, admin panels) — adding an entity, column, DTO, endpoint, API hook, type, form, enum or constant — and whenever the user asks to find duplication, unify types, remove mappers, align field names between backend and frontend, clean up code, or check that "input and output are the same", even if they don't name this skill.
---

# One source of truth

Every fact in a codebase has exactly one home. Every other place either
imports it or derives from it. Data keeps one name from the database to the
screen and back.

**The repository beats this skill.** Read the project's `AGENTS.md` /
`CLAUDE.md` / `docs/` first. Where they contradict this skill, follow them and
tell the user about the conflict instead of silently picking one.

## The four rules

1. **One entity, one type.** Each domain entity (User, Product, Order…) has
   one canonical definition. Request bodies, responses, form values, table
   rows and test fixtures derive from it (`Pick`, `Omit`, `Partial`,
   `PickType`/`OmitType`, `z.infer`, generated clients, indexed access).
   Two interfaces with the same fields in two files is a bug, even across
   repos. Details: `references/types-and-entities.md`.
2. **One name end to end.** A field is spelled identically in the database
   column, the entity property, the API JSON key, the frontend type, the form
   field `name`, the query param and the validation schema. No renaming, no
   camelCase↔snake_case conversion layer, no `toDto`/`fromApi` that copies
   field by field under new names. Allowed: omitting private fields
   (password hashes, tokens, internal flags), adding computed fields under
   their own honest name, and nesting a related entity. The label a user
   sees describes the same thing the name says («شماره موبایل» ⇒
   `phone_number`, never `mobile`, `tel`, `p` or `value1`). Details:
   `references/naming-and-data-flow.md`.
3. **No code comments.** Names, types and small functions carry the meaning.
   Only tool directives survive: `eslint-disable…` (with its reason inside
   the directive), `@ts-expect-error`, `prettier-ignore`,
   `/// <reference …>`, `/* @vite-ignore */`, `/*#__PURE__*/`, license
   headers the license requires. Knowledge that does not fit in a name goes
   to `docs/`, a commit message or a test name.
4. **Clean code.** Small single-purpose functions, intention-revealing names,
   early returns, no dead code, no magic values, one way to do each thing.
   Details: `references/clean-code.md`.

## Workflow when writing code

1. **Find before you write.** Search for an existing type, enum, constant,
   helper, component or endpoint that already covers the need — by name and
   by shape (grep the field names, not only the type name). Reuse it.
2. **Name from the source.** Take field names from the database schema or
   the API contract, never from the screen copy or your own preference. If
   the source name is wrong, rename it at the source and everywhere at once
   (with a migration that renames the column, never drop-and-add).
3. **Derive, don't redeclare.** Build the new shape from the canonical one.
4. **Second occurrence ⇒ extract.** The second time markup, logic, a
   constant or a literal appears, move it to its one home and import it.
5. **Re-read the diff** for duplicated shapes, renamed fields, mapping code
   and comments before calling it done.

## Workflow when auditing

Follow `references/audit.md`: inventory entities and their names per layer,
scan for duplicated types and copy-paste, trace each entity through
DB → backend → API → frontend → form and back, list every rename or mapper,
then write the report in the format given there and fix it in the order it
sets (data-loss risks first).

## Reference index

| Need | File |
|---|---|
| Canonical types, deriving variants, enums/constants, cross-repo contracts | `references/types-and-entities.md` |
| Identical names DB → API → UI, what may and may not be transformed, safe renames | `references/naming-and-data-flow.md` |
| Naming, functions, errors, files, dead code, comments | `references/clean-code.md` |
| Step-by-step audit, search commands, report format, fix order | `references/audit.md` |
