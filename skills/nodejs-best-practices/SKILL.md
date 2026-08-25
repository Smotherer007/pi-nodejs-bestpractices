---
name: nodejs-best-practices
description: Node.js best practices (current version: Node 26) — architecture, error handling, code style, testing, production, security, performance, and Docker. Use when writing, reviewing, or migrating Node.js code, choosing a framework, hardening security, or whenever an answer about Node risks being outdated (Temporal API, native TypeScript, node:sqlite, Undici 8, deprecations/removals).
license: MIT
compatibility: Node.js 26.x (type-stripping baseline also applies to 22.18+ and 24.12+)
---

# Node.js Best Practices (current: Node 26)

## When to Use

- Writing or reviewing Node.js code (architecture, error handling, testing, production, security, Docker).
- Migrating an app to Node 26 and hitting removed/deprecated APIs.
- Answering "what is the modern way to do X in Node" — dates/time, HTTP, SQLite, TypeScript, tests, config.
- Choosing a framework, hardening security, or setting up CI/release for a Node package.

## Current-Version Facts (Node 26)

- **26.0.0** released 2026-05-05 (Current). **LTS from 2026-10-28**, EOL 2029-04-30. Latest patch line as of Aug 2026: 26.7.x. From Node 27 the release cycle is annual and every major goes LTS.
- **Temporal API** enabled by default (ECMAScript 2026). Prefer it over `Date` for new code.
- **V8 14.6** → `Map.prototype.getOrInsert()`/`getOrInsertComputed()`, `WeakMap`/`WeakSet` variants, `Iterator.concat()`.
- **Undici 8** (built-in `fetch`/HTTP): new handler/dispatcher API; HTTP/2 opt-in (`allowH2: false` for H1-only).
- **Native TypeScript** type-stripping on by default (`node app.ts`). `--experimental-transform-types` was **removed** → enums/namespaces/parameter-properties need a build step.
- **`node:sqlite`** (`DatabaseSync`) built in — no native build for basic SQLite.
- **`node:ffi`** (experimental, 26.1.0) — `--experimental-ffi` + `--allow-ffi`; unsafe by design.

## Removals & Deprecations That Break Apps

| In Node 26 | Action |
|---|---|
| `http.Server#writeHeader()` — removed | use `writeHead()` |
| `_stream_wrap`, `_stream_readable`, `_stream_writable`, `_stream_duplex`, `_stream_transform`, `_stream_passthrough` — removed | use `node:stream` / `node:stream/*` |
| GCM short auth tags without `authTagLength` (DEP0182) — End-of-Life | always pass `authTagLength` / correct `setAuthTag()` |
| webcrypto `CryptoKey` into `node:crypto` (DEP0203) — runtime-deprecated | use `KeyObject`, or stay in WebCrypto API |
| `KeyObject.from()` with non-extractable `CryptoKey` (DEP0204) — runtime-deprecated | keep key extractable or use `node:crypto` key format |
| `Duplex.toWeb({ type })` (DEP0201) — runtime-deprecated | use `node:stream` converters |
| `module.register()` — runtime-deprecated | `registerHooks()` from `node:module` |

## Procedure

1. **Architecture** — structure by business component, layer in 3 tiers (entry-points / domain / data-access).
2. **Errors** — async/await, extend `Error`, separate operational vs programmer errors, handle centrally, log JSON to stdout (`pino`).
3. **Dates/time** — Temporal: `Instant` for storage, `ZonedDateTime` for display, `PlainDate` for calendar-only.
4. **TypeScript** — run `.ts` directly (type-stripping) but keep `tsc --noEmit` in CI; avoid enums/namespaces for zero-build.
5. **HTTP** — built-in `fetch`/Undici 8; `allowH2: false` if H1-only.
6. **DB** — `node:sqlite` `DatabaseSync` for embedded SQLite.
7. **Config** — `node --env-file=.env` + validate with `zod` (fail fast).
8. **Tests** — built-in `node --test`; coverage; AAA; per-test data; mock HTTP with Undici `MockAgent`.
9. **Security** — helmet, parameterized queries, `crypto.scrypt`/argon2, validate input, rate-limit, non-root, `node:` imports.
10. **Release** — `engines`/`.nvmrc` pinned, `npm ci`, semantic-release (see `.github/workflows/release.yml`).

## Pitfalls

- **Temporal is not a drop-in `Date` replacement** and `Date` is *not* removed. Handle serialization explicitly (store `Instant` as ISO-8601 string; `JSON.stringify` doesn't special-case Temporal).
- **Type-stripping silently ignores types** — enums/namespaces/parameter-properties throw at runtime unless transpiled. Keep `tsc --noEmit`.
- **`node:sqlite` is synchronous and Stability 1.2** — great for tools/CLIs/medium loads; verify under high concurrency.
- **`node:ffi` is experimental and unsafe** — not for production.
- **Mixing WebCrypto `CryptoKey` with `node:crypto` is deprecated** — pick one API surface per key path.

## Verification

- `node -v` is 26.x; `npm test` green; `tsc --noEmit` clean (where applicable).
- `node --trace-warnings app.js` shows no `DEP0XXX` warnings.
- `.nvmrc`/`engines` enforce the target; CI runs Node 24 + 26.

## References

- [references/node-26-whats-new.md](references/node-26-whats-new.md) — Node 26 features + breaking changes with code examples.
- [references/evergreen-best-practices.md](references/evergreen-best-practices.md) — the complete current (2026) best-practice catalog.
- [references/security-deep-dive.md](references/security-deep-dive.md) — concrete security reference (headers, OWASP, crypto, secrets, injection, XSS, auth).
- [references/framework-and-testing.md](references/framework-and-testing.md) — framework selection guide + testing patterns (five outcomes, AAA).
