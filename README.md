# @patimweb/pi-nodejs-bestpractices

A [Pi coding agent](https://github.com/earendil-works/pi-mono) skill that keeps agents current on
**Node.js 26** and the **2026 best-practice catalog**. Built to fight the "the agent's Node knowledge
is outdated" problem.

## What's Included

| Resource | Path | Purpose |
|----------|------|---------|
| Skill | `skills/node-26-best-practices/SKILL.md` | Concise trigger + checklist + pitfalls (loaded on demand) |
| Reference | `skills/node-26-best-practices/references/node-26-whats-new.md` | Full Node 26 features + breaking changes with code examples |
| Reference | `skills/node-26-best-practices/references/evergreen-best-practices.md` | Complete 2026 best-practice catalog (architecture → Docker) |
| Reference | `skills/node-26-best-practices/references/security-deep-dive.md` | Concrete security reference — headers, OWASP, crypto, secrets, injection, XSS, auth |
| Reference | `skills/node-26-best-practices/references/framework-and-testing.md` | Framework selection guide + testing patterns (five outcomes, AAA) |

**Covers:**
- Node 26 lifecycle (Current → LTS 2026-10-28, EOL 2029-04-30)
- **Temporal API** (enabled by default) vs legacy `Date`
- **Native TypeScript** type-stripping (`node app.ts`) and what was removed (`--experimental-transform-types`)
- **`node:sqlite`** (`DatabaseSync`) replacing `better-sqlite3`
- **Undici 8** HTTP client changes
- V8 14.6 additions (`Map.getOrInsert`, `Iterator.concat`)
- Post-quantum crypto (ML-KEM/ML-DSA) and other crypto changes
- All **deprecations/removals** that break apps (verified: DEP0182/0201/0203/0204, `writeHeader`, `_stream_*`, `module.register`)
- The **evergreen best practices** (structure, error handling, testing, production, security, performance, Docker)

## Install

### From npm

```bash
pi install npm:@patimweb/pi-nodejs-bestpractices
```

### From a git repo

```bash
pi install git:github.com/Smotherer007/pi-nodejs-bestpractices@v0.1.0
```

### Try without installing

```bash
pi -e npm:@patimweb/pi-nodejs-bestpractices
# or locally:
pi -e ./
```

## Usage

No configuration needed. Once installed, the skill loads automatically when you ask Pi anything
about Node.js 26, modern Node patterns, or migrating code to Node 26 — or force it explicitly:

```
/skill:node-26-best-practices
```

## Release / Publish

### Via npm

```bash
npm login
npm publish --access public
```

### Via git + GitHub

```bash
git remote add origin git@github.com:Smotherer007/pi-nodejs-bestpractices.git
git push -u origin main --tags
```

Others then install with `pi install git:github.com/Smotherer007/pi-nodejs-bestpractices@v0.1.0`.

## License

[MIT](LICENSE)
