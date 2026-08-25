# @patimweb/pi-nodejs-bestpractices

Node.js best practices for the [Pi coding agent](https://github.com/earendil-works/pi-mono), shipping
**Matteo Collina's `node` skill** — vendored **1:1** from [mcollina/skills](https://github.com/mcollina/skills).

> **Attribution:** skill content is © 2026 [Matteo Collina](https://github.com/mcollina), MIT License
> (see `skills/node/LICENSE`). Packaging by Patrick Weppelmann.

## What's Included

| Resource | Path | Purpose |
|----------|------|---------|
| Skill | `skills/node/SKILL.md` | Collina's `node` skill — Node.js best practices (TypeScript-first) |
| Rules | `skills/node/rules/*.md` | 14 rules: async patterns, error handling, streams, caching, performance, profiling, logging, modules, testing, TypeScript, graceful shutdown, environment, flaky tests, stuck processes |
| Assets | `skills/node/rules/assets/*.ts` | Graceful-server reference implementation + test |

## Install

### From npm

```bash
pi install npm:@patimweb/pi-nodejs-bestpractices
```

### From a git repo

```bash
pi install git:github.com/Smotherer007/pi-nodejs-bestpractices@v0.2.0
```

### Try without installing

```bash
pi -e ./
```

## Usage

The `node` skill loads automatically for Node.js questions, or force it explicitly:

```
/skill:node
```

## Development

```bash
npm install
npm test        # validates skill frontmatter (node:test)
```

## Release

Releases are automated via [**semantic-release**](https://semantic-release.gitbook.io/)
(`.github/workflows/release.yml`). Pushing a conventional commit (`feat:`/`fix:`) to `main` triggers:
version bump → `npm publish` → GitHub release + tag.

## License

[MIT](LICENSE). Skill content © Matteo Collina (MIT) — see `skills/node/LICENSE`.
