---
name: react-conventions
description: Opinionated React + TypeScript house style and architecture rules — existing components and types first, asking the user up front whenever data, design or meaning is unclear; finding code through graphify (or another categorised index / RAG) instead of reading whole files; zero duplication of types, code, docs or rules; feature-sliced layering (pages → features → core); atomic design system with cva variants; generated OpenAPI + React Query data layer; schema-driven forms; i18n; strict file-size and one-component-per-file limits; no-mocks testing; English-only dev prose; Conventional Commits. Use this skill whenever writing, reviewing, refactoring or scaffolding React/TSX code — a new page, feature, component, hook, form, table, API hook, story, test or project doc — in the user's projects, even when the request doesn't mention "conventions" or "style" (e.g. "add a users page", "build this screen from the design", "review this component", "split this file").
---

# React conventions

These are the rules and the handwriting of the user's React codebases. Code
written with this skill should look as if the rest of the repository wrote it.

**The repository beats this skill.** Read the project's own `AGENTS.md` /
`CLAUDE.md` / `docs/` first and follow them where they speak; use this skill
where they are silent. If the project lacks a piece a reference assumes (a
generated API client, `FormBuilder`, a component catalog), keep the principle
and use what the project has — don't build the missing infrastructure unasked.

Each rule below and in `references/` is stated **once**. Where two sources
seem to disagree, the project's own docs win; point the conflict out to the
user instead of silently picking one.

## Principles

1. **One home for every fact — never write anything twice.** This covers
   types, code, docs and rules alike:
   - *Types:* derive, don't redeclare. Alias generated models; build variants
     with `Pick`, `Omit`, `Partial`, indexed access (`Props["tone"]`),
     `z.infer`/`z.input`, `ReturnType`, `typeof CONST[number]`. A shape that
     appears in two signatures gets a name and is imported.
   - *Code:* the second time markup or logic appears, extract a component,
     hook or function; constants live in one `constants` file.
   - *Docs and rules:* write a rule where it belongs and link to it from
     elsewhere; summaries that restate it are duplication. Delete a stale or
     contradicting rule rather than adding a competing one.
2. **Existing components and types come first.** Before writing anything,
   find the components, hooks, schemas and types that already cover the need
   (*Finding code*) and use them as they are. Only when nothing fits, or what
   exists behaves differently, build: extend the existing piece when the
   difference is a variant or prop, otherwise create a new one. Either way,
   follow the rules in `references/` and leave it fully clean: typed,
   story-covered and duplicate-free. How to extend a design-system
   component: `references/ui-and-styling.md`.
3. **Ask instead of guessing.** Invent no requirements, API fields, business
   rules, designs, props or data. When data is insufficient, something isn't
   clear, a design or image is missing or unreadable, or two sources disagree,
   ask the user. Ask at the start of the task, all questions together, before
   writing code. When a question only surfaces mid-task, stop and ask it then;
   don't pick a reading and carry on. Never cover uncertainty with cosmetic
   edits (rewording comments, renaming, reformatting) and report the task as
   done; say what is unclear instead.
4. **English for everything developers read** (identifiers, comments, docs,
   tests, commits), whatever language the user writes in. Product copy stays
   in the locale files (`references/data-and-forms.md` → i18n).
5. **Real data only.** No mocks, stubs, spies, fake timers, MSW or fabricated
   responses — in app code, tests or stories. A missing endpoint means a real
   empty state, not placeholder rows (testing rules:
   `references/code-style.md` → Pure logic and tests).

## Finding code

Don't list directories or read whole files to find things. Ask a
categorised, described index first, and open only the lines it points to.
Use the first source that exists, in this order:

1. **graphify** — when `graphify-out/graph.json` exists:
   - `graphify query "<question>"` returns the relevant nodes with file, line
     and community (the group a file belongs to). Narrow the question or
     raise `--budget` when the output says it was truncated.
   - `graphify explain "<Symbol>"` shows one node and its neighbours (who
     uses a component, what a hook depends on).
   - `graphify path "<A>" "<B>"` shows how two modules connect, for example
     to check a layering violation.
   - `graphify-out/wiki/index.md` is for browsing by area.
     `GRAPH_REPORT.md` is only for an architecture-wide review.
   - After changing code, run `graphify update .` so the next lookup is
     current.
2. **The project's own indexes**: the component catalog, `docs/` pages, and
   module `index.ts` barrels (which list what a module exports).
3. **A RAG or semantic-search tool** (an MCP server or a CLI), when the project
   has one configured and the question is conceptual rather than a symbol name.
4. **Targeted search** (`grep`/`rg` for an exact symbol or string), only
   after the index has narrowed the area. Then read the specific range with
   offsets, not the whole file.

If none of 1–3 exists and the task spans much of the codebase, suggest
setting up graphify (`/graphify`) instead of scanning everything.

## Workflow

1. **Orient.** Read the project rules, then use *Finding code* to reach the
   closest existing example of the same kind (a sibling feature, a similar
   component and its story). Copy its shape.
2. **Clarify.** List what is missing or unclear and ask it now (Principle 3).
   Continue once it is answered.
3. **Place the code** by ownership — `references/architecture.md`.
4. **Build the UI** from the design system — `references/ui-and-styling.md`.
5. **Wire data and forms** — `references/data-and-forms.md`.
6. **Write it in house style** — `references/code-style.md`.
7. **De-duplicate.** Re-read the diff for repeated type shapes, markup,
   logic, strings and prose (Principle 1) and for rules that now contradict.
8. **Verify.** Run the project's type-check, lint, unit tests and no-mocks
   check; for UI, run the app, look at it and compare with the design. Report
   every failure honestly, including pre-existing ones.
9. **Deliver.** Update the component catalog/docs where public exports
   changed, refresh the index (*Finding code*), then commit
   (`references/code-style.md` → Commits and delivery).

## Reference index

| Need | File |
|---|---|
| Layers, feature anatomy, exports, pages/routing, state, new-feature steps | `references/architecture.md` |
| Design-system reuse, cva variants, external-styling ban, tokens, tables/dialogs/checkboxes, a11y/RTL, stories | `references/ui-and-styling.md` |
| OpenAPI source of truth, feature `api/` barrel, infinite lists, mutations, forms, i18n | `references/data-and-forms.md` |
| Formatting/imports, naming, TypeScript, component shape, comments, tests, commits | `references/code-style.md` |
