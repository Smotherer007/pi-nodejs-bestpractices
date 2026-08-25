---
name: node-26-best-practices
description: Node.js 26 (Current, LTS Oct 2026) features, breaking changes, and current (2026) best practices. Use when writing, reviewing, or migrating Node.js code to Node 26+, or whenever an answer about Node risks being outdated (Temporal API, native TypeScript, node:sqlite, Undici 8, deprecations/removals).
license: MIT
compatibility: Node.js 26.x (baseline for type-stripping also applies to 22.18+ and 24.12+)
---

# Node 26 Best Practices

## When to Use

- Writing new Node.js code that targets Node 26 (Current → LTS from 2026-10-28).
- Migrating an app from Node 20/22/24 to Node 26 and hitting removed/deprecated APIs.
- Answering "what is the modern way to do X in Node" — dates/time, HTTP, SQLite, TypeScript, tests.
- Reviewing PRs against outdated Node patterns (e.g. `moment`, `better-sqlite3`, `writeHeader`, legacy streams).

## Key Facts (Node 26)

- **26.0.0** released 2026-05-05 as "Current". **LTS from 2026-10-28**, EOL 2029-04-30. Patch line as of Aug 2026: 26.7.x. Starting with Node 27 the release cycle becomes annual and every major goes LTS.
- **Temporal API enabled by default** (ECMAScript 2026). Prefer it over `Date` for new code.
- **V8 14.6** → `Map.prototype.getOrInsert()` / `getOrInsertComputed()`, `WeakMap`/`WeakSet` variants, `Iterator.concat()`.
- **Undici 8.0** (built-in `fetch`/HTTP): new handler/dispatcher API for custom dispatchers; HTTP/2 is opt-in, set `allowH2: false` for H1-only.
- **Native TypeScript type-stripping** on by default (`node app.ts`). `--experimental-transform-types` was **removed** → enums/namespaces/parameter-properties require a build step.
- **`node:sqlite`** (`DatabaseSync`) built in — no native build for basic SQLite.
- **`node:ffi`** (experimental, 26.1.0) — `--experimental-ffi` + `--allow-ffi` (permission model); inherently unsafe.

## Removals & Deprecations That Break Apps

| In Node 26 | Action |
|---|---|
| `http.Server#writeHeader()` — removed | use `writeHead()` |
| `_stream_wrap`, `_stream_readable`, `_stream_writable`, `_stream_duplex`, `_stream_transform`, `_stream_passthrough` — removed | use `node:stream` / `node:stream/*` |
| GCM short auth tags without `authTagLength` (DEP0182) — End-of-Life | always pass `authTagLength` / correct `setAuthTag()` |
| webcrypto `CryptoKey` passed into `node:crypto` (DEP0203) — runtime-deprecated | use `KeyObject`, or stay entirely in WebCrypto API |
| `KeyObject.from()` with non-extractable `CryptoKey` (DEP0204) — runtime-deprecated | keep key extractable or use `node:crypto` key format |
| `Duplex.toWeb({ type })` (DEP0201) — runtime-deprecated | use `node:stream` converters |
| `module.register()` — runtime-deprecated | `registerHooks()` from `node:module` |

## Procedure

1. **Pin the runtime.** Add `.nvmrc` + `"engines": { "node": ">=26" }`; use `volta`/`fnm`. Run an LTS line in prod (Node 26 LTS from Oct 2026; Node 24 LTS before that).
2. **Dates/time → Temporal.** `Instant` for timestamps, `ZonedDateTime` for wall-clock/DST, `PlainDate` for calendar-only. Drop `moment`/`date-fns` in new code.
3. **TypeScript.** Run `.ts` directly via type-stripping, but keep `tsc --noEmit` in CI (type-stripping ≠ type-checking). Avoid enums/namespaces/parameter-properties if you want a zero-build setup.
4. **HTTP.** Prefer built-in `fetch`/Undici 8. If you pin HTTP/1.1, set `allowH2: false`.
5. **Embedded DB.** `node:sqlite` `DatabaseSync` instead of `better-sqlite3` for basic needs.
6. **Config.** `node --env-file=.env` (no `dotenv`); validate with `zod` (fail fast).
7. **Tests.** Built-in `node --test`; `--experimental-test-coverage` for coverage.
8. **Deps & security.** `npm ci` in CI, lockfiles committed, `node:`-prefixed imports, run as non-root, secrets out of code.

## Pitfalls

- **Temporal is not a drop-in `Date` replacement** and `Date` is *not* removed. Handle serialization explicitly (e.g. store `Instant` as ISO-8601 string; `JSON.stringify` doesn't special-case Temporal).
- **Type-stripping silently ignores types** — enums/namespaces/parameter-properties throw at runtime unless transpiled. Keep `tsc --noEmit` in CI to catch type errors.
- **`node:sqlite` is synchronous and Stability 1.2** — great for tools/CLIs/medium loads; verify under high concurrency before production-critical use.
- **`node:ffi` is experimental and unsafe** (invalid pointers etc.) — not for production.
- **Mixing WebCrypto `CryptoKey` with `node:crypto` is now deprecated** — choose one API surface per key path.

## Verification

- `node -v` reports 26.x.
- `node --test` is green; `tsc --noEmit` is clean.
- `node --trace-warnings app.js` shows no `DEP0XXX` warnings.
- `.nvmrc`/`engines` enforce the target; CI runs Node 26 (plus 24 LTS if you support both).

## References

- [references/node-26-whats-new.md](references/node-26-whats-new.md) — full feature + breaking-change details with code examples.
- [references/evergreen-best-practices.md](references/evergreen-best-practices.md) — the complete current (2026) best-practice catalog.
- [references/security-deep-dive.md](references/security-deep-dive.md) — concrete security reference (headers, OWASP, crypto, secrets, injection, XSS, auth).
- [references/framework-and-testing.md](references/framework-and-testing.md) — framework selection guide + testing patterns (five outcomes, AAA).
