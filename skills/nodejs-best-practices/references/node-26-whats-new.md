# Node.js 26 — What's New & What Changed

> Source: Node.js release blog (v26.0.0, 2026-05-05), `nodejs.org/api/deprecations.html`,
> `nodejs.org/api/*`, `github.com/nodejs/node` changelogs. Verified Aug 2026.

## Release & Lifecycle

| Fact | Value |
|---|---|
| 26.0.0 released | 2026-05-05 ("Current") |
| Enters LTS | 2026-10-28 |
| End-of-Life | 2029-04-30 |
| Latest (Aug 2026) | 26.7.0 (2026-08-05) |
| NODE_MODULE_VERSION | 147 |
| V8 | 14.6 (Chromium 146) |
| Undici | 8.0.2 |
| ICU | 78.3 |

Starting with **Node 27**, the release cadence becomes **annual** and **every major version goes LTS**
(no more alternating even/odd "Current vs LTS" split). Node 26 is the last even-numbered major under the old model.

---

## 1. Temporal API (enabled by default)

Temporal is the ECMAScript 2026 date/time API and a full replacement for `Date` (which is **not** removed).
No polyfill needed server-side on Node 26.

**Core types and when to use them:**

| Type | Meaning | Use for |
|---|---|---|
| `Temporal.Instant` | exact UTC point in time | `created_at`, `updated_at` timestamps |
| `Temporal.ZonedDateTime` | instant + IANA tz + calendar | wall-clock time, DST-sensitive display, scheduling |
| `Temporal.PlainDate` | calendar date, no time/tz | birthdays, deadlines |
| `Temporal.PlainTime` | time of day, no date/tz | "daily at 09:00" |
| `Temporal.PlainDateTime` | date + time, no tz | local/floating datetime |
| `Temporal.PlainYearMonth` / `PlainMonthDay` | partial dates | recurring events, anniversaries |
| `Temporal.Duration` | length of time | elapsed time, timeouts |
| `Temporal.Now` | current values | `Temporal.Now.instant()` etc. |

**Examples:**

```js
// Current instant (recommended for server-side timestamps)
const now = Temporal.Now.instant();

// From a legacy Date
const instant = Temporal.Instant.fromEpochMilliseconds(date.getTime());

// Zoned (wall-clock aware)
const zdt = instant.toZonedDateTimeISO('Europe/Berlin');
zdt.toPlainDate(); // -> PlainDate
zdt.toPlainTime(); // -> PlainTime

// Calendar-only
const deadline = Temporal.PlainDate.from({ year: 2026, month: 12, day: 31 });

// Duration arithmetic (handles DST/calendar correctly)
const in30d = now.add(Temporal.Duration.from({ days: 30 }));

// Round-trip via string (ISO 8601 / RFC 9557)
const s = now.toString();
const back = Temporal.Instant.from(s);
```

**Best practice:** store timestamps as `Instant` (UTC) in the DB; convert to `ZonedDateTime` at the edge
for display; use `PlainDate` for genuinely calendar-only data. `Intl.DateTimeFormat` can format Temporal
objects directly.

**Caveats:**
- `JSON.stringify` does not special-case Temporal → serialize to string and parse with `Temporal.*.from()`.
- Browser support is uneven (Firefox 139+, Chrome 144+; Safari still needs a polyfill as of Aug 2026) —
  fine on the server, but polyfill on the frontend if you share Temporal logic.

---

## 2. V8 14.6 language additions

- **Upsert** (TC39 proposal-upsert):
  - `Map.prototype.getOrInsert(key, defaultValue)`
  - `Map.prototype.getOrInsertComputed(key, fn)`
  - `WeakMap.prototype.getOrInsertComputed(key, fn)`
  - `WeakSet.prototype.getOrInsertComputed(value, fn)`
- **Iterator sequencing** (TC39 proposal-iterator-sequencing): `Iterator.concat(...iterables)`.

```js
const counts = new Map();
counts.getOrInsert('a', 0);          // 0, then increments below
counts.getOrInsertComputed('b', () => 0);

for (const x of Iterator.concat([1, 2], new Set([3, 4]))) {
  // 1, 2, 3, 4
}
```

---

## 3. Native TypeScript (type-stripping)

- Running `.ts` files directly (`node app.ts`) is **on by default** (type stripping). No `tsc`/`ts-node`/`tsx` for simple cases.
- Node strips **erasable** syntax only: type annotations, interfaces, type aliases, generics, `as` casts, `import type`.
- **Removed in Node 26:** `--experimental-transform-types` (present in 22.7+ / 24). This flag handled
  **non-erasable** features. Without it, the following **throw at runtime** in Node 26:
  - `enum`
  - `namespace`
  - constructor parameter properties (`constructor(private x: number)`)
  - legacy experimental decorators
- **Type-stripping does NOT type-check.** Always keep `tsc --noEmit` (or `--checkJs`) in CI.

**Guidance:** prefer erasable-only TS (use `const` objects / string-literal unions instead of `enum`).
If you need enums/decorators, keep a build step (`tsc`, `esbuild`, `swc`).

---

## 4. Undici 8 (built-in fetch / HTTP client)

- Built-in HTTP client bumped to Undici 8.0.2.
- **Breaking for custom dispatchers:** the handler/dispatcher API was reworked. Custom `Dispatcher` /
  `Interceptor` implementations need updating (see undici "Migrating from v7 to v8").
- **HTTP/2 is opt-in.** HTTP/1.1-only behavior now requires `allowH2: false` explicitly where H2 negotiation
  might otherwise occur.

---

## 5. `node:sqlite` (DatabaseSync)

Built-in SQLite, zero native build. Replaces `better-sqlite3`/`sqlite3` for basic use.

```js
import { DatabaseSync } from 'node:sqlite';

const db = new DatabaseSync(':memory:'); // or a file path
db.exec('CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT)');

const insert = db.prepare('INSERT INTO users (name) VALUES (?)');
insert.run('Alice');

const rows = db.prepare('SELECT * FROM users').all();
```

**Notes:** synchronous API only; Stability 1.2 (release candidate) as of Node 26 — safe for tools/CLIs
and moderate loads, verify before high-concurrency production use.

---

## 6. `node:ffi` (experimental)

Added in 26.1.0: load dynamic libraries and call native symbols from JS.

- Gated behind `--experimental-ffi`.
- Requires `--allow-ffi` when the Permission Model is enabled.
- **Inherently unsafe** (invalid pointers, no memory-safety guarantees). Not for production.

---

## 7. Crypto changes

- **Post-quantum:** ML-KEM (Kyber) and ML-DSA (Dilithium) PKCS#8 exports now default to **seed-only** format.
- Ed25519: added `context` parameter support.
- `KeyObject`: added **raw key format** support.
- Async crypto errors now include OpenSSL error details.
- Deprecations/removals (see table in SKILL.md):
  - DEP0182 (short GCM auth tags without `authTagLength`) → **End-of-Life**: always pass `authTagLength`
    when creating the cipher, or pass the correct tag length to `decipher.setAuthTag()`.
  - DEP0203: passing a WebCrypto `CryptoKey` to `node:crypto` functions → runtime-deprecated. Use `KeyObject`
    or stay in the WebCrypto API.
  - DEP0204: `KeyObject.from()` with a non-extractable `CryptoKey` → runtime-deprecated.

---

## 8. HTTP / Streams / Module removals & deprecations

- `http.Server.prototype.writeHeader()` → **removed**. Use `writeHead()`.
- Legacy internal modules `_stream_wrap`, `_stream_readable`, `_stream_writable`, `_stream_duplex`,
  `_stream_transform`, `_stream_passthrough` → **removed**. Use `node:stream` and `node:stream/*`.
- `Duplex.toWeb({ type })` (DEP0201) → runtime-deprecated. Use `node:stream` converters.
- `module.register()` → runtime-deprecated. Use `registerHooks()` from `node:module`.

---

## 9. Misc behavior changes

- `assert`: printf-style messages are now allowed in assertion error messages.
- `localStorage` returns `undefined` when no backing file is configured (previously threw).
- `QuotaExceededError` is now a `DOMException`-derived interface.
- `util.inspect` marks proxied objects as `Proxy` when inspecting.
- Build/toolchain (native-module maintainers): GCC ≥ 13.2, Python 3.9 support dropped, Windows SDK 11,
  IBM p8/z13 dropped.
