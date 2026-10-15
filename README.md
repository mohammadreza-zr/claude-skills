# claude-skills

Claude Code skills by [mohammadreza-zr](https://github.com/mohammadreza-zr), published as a plugin marketplace.

## Install

```
/plugin marketplace add mohammadreza-zr/claude-skills
/plugin install react-conventions@mohammadreza-skills
/plugin install one-source-of-truth@mohammadreza-skills
```

Update later with `/plugin marketplace update mohammadreza-skills`.

## Skills

### react-conventions

Opinionated React + TypeScript house style and architecture rules:

- Feature-sliced layering (`pages → features → core`)
- Atomic design system with `cva` variants and semantic design tokens
- Generated OpenAPI client + React Query data layer
- Schema-driven forms, i18n, strict file-size and one-component-per-file limits
- No-mocks testing, English-only dev prose, Conventional Commits

It triggers automatically when Claude writes, reviews or refactors React/TSX code.
A project's own `AGENTS.md` / `CLAUDE.md` always takes precedence over the skill.

Some rules assume tooling you may not use (for example graphify
for code lookup); the skill falls back to other indexes when it is absent.

### one-source-of-truth

Stack-agnostic rules for backend, frontend and admin code, plus an audit:

- One entity, one type: every type, enum, constant and helper has one home; everything else derives from it
- One name end to end: the database column, entity, API key, frontend type and form field share one name, with no renaming or case-conversion layer
- No code comments (tool directives only)
- Clean-code rules for names, functions, values, errors and structure
- A step-by-step audit with search commands, a report format and a fix order

It triggers when Claude writes, reviews, refactors or audits code, or when you ask it to find duplication or align names between backend and frontend.

## License

MIT
