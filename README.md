# claude-skills

Claude Code skills by [mohammadreza-zr](https://github.com/mohammadreza-zr), published as a plugin marketplace.

## Install

```
/plugin marketplace add mohammadreza-zr/claude-skills
/plugin install react-conventions@mohammadreza-skills
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

Some rules assume tooling you may not use (for example [graphify](https://github.com/safishamsi/graphify)
for code lookup); the skill falls back to other indexes when it is absent.

## License

MIT
