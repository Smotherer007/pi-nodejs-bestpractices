# @patimweb/pi-nodejs-bestpractices

A [Pi coding agent](https://github.com/earendil-works/pi-mono) skill that keeps agents current on
**Node.js best practices** (current version: **Node 26**) and the **2026 best-practice catalog**.
Built to fight the "the agent's Node knowledge is outdated" problem.

> **Attribution:** the best-practice catalog is curated from
> [**goldbergyoni/nodebestpractices**](https://github.com/goldbergyoni/nodebestpractices)
> (2026 edition, by [Yoni Goldberg](https://github.com/goldbergyoni), MIT License), and the
> version-specific facts (Temporal, type-stripping, `node:sqlite`, Undici 8, deprecations) are
> sourced from the [Node.js release blog](https://nodejs.org/en/blog/release/v26.0.0) and
> [Node.js docs](https://nodejs.org/api/). 🙏 Thanks to the `nodebestpractices` maintainers.

## What's Included

| Resource | Path | Purpose |
|----------|------|---------|
| Skill | `skills/nodejs-best-practices/SKILL.md` | Trigger + checklist + pitfalls (loaded on demand) |
| Reference | `skills/nodejs-best-practices/references/evergreen-best-practices.md` | **Complete** 2026 catalog — all 8 sections, ~108 practices |
| Reference | `skills/nodejs-best-practices/references/node-26-whats-new.md` | Node 26 features + breaking changes with code examples |
| Reference | `skills/nodejs-best-practices/references/security-deep-dive.md` | Concrete security reference — headers, OWASP, crypto, injection, XSS, auth |
| Reference | `skills/nodejs-best-practices/references/framework-and-testing.md` | Framework selection + testing patterns (five outcomes, AAA) |

**Covers:** architecture, error handling, code style, testing, production, security (OWASP), performance,
Docker — plus Node 26 specifics (Temporal API, native TypeScript, `node:sqlite`, Undici 8, V8 14.6,
post-quantum crypto, and all deprecations/removals).

## Install

### From npm

```bash
pi install npm:@patimweb/pi-nodejs-bestpractices
```

### From a git repo

```bash
pi install git:github.com/Smotherer007/pi-nodejs-bestpractices@v1.0.0
```

### Try without installing

```bash
pi -e npm:@patimweb/pi-nodejs-bestpractices
# or locally:
pi -e ./
```

## Usage

No configuration needed. Once installed, the skill loads automatically when you ask Pi anything
about Node.js best practices, Node 26, or migrating code — or force it explicitly:

```
/skill:nodejs-best-practices
```

## Development

```bash
npm install
npm test        # validates skill frontmatter (node:test)
```

The package is skills-only (markdown), no build step.

## Release

Releases are fully automated via [**semantic-release**](https://semantic-release.gitbook.io/)
(see `.github/workflows/release.yml`). Pushing to `main` with a conventional commit
(`feat:`/`fix:`) triggers: version bump → `npm publish` → GitHub release + tag.

**One-time setup:** add an npm publish token as the `NPM_TOKEN` repository secret
(GitHub → repo → Settings → Secrets and variables → Actions → `NPM_TOKEN`), generated at
[npmjs.com → Access Tokens](https://www.npmjs.com/settings/tokens) (type: Automation/Publish).

## License

[MIT](LICENSE). Best-practice content is derived from `nodebestpractices` (MIT) — see attribution above.
