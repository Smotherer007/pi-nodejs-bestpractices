# Node.js Best Practices — Current (2026) Catalog

> Consolidated from the Node.js Best Practices project (2026 edition, 100+ items) plus
> 2026-era tooling defaults. These apply to Node 26 and remain valid "as of today."

## 0. Node-26-specific defaults (start here)

- Use **Temporal** for new date/time code (Instant for storage, ZonedDateTime for display, PlainDate for calendar-only).
- Run TypeScript via **native type-stripping** (`node app.ts`) but keep `tsc --noEmit` in CI; avoid `enum`/`namespace` for zero-build.
- Prefer **built-in `fetch`** (Undici 8) over `axios`/`node-fetch` for new code; set `allowH2: false` if you pin HTTP/1.1.
- Use **`node:sqlite`** for embedded SQLite instead of `better-sqlite3`/`sqlite3`.
- Use **`node --env-file=.env`** instead of `dotenv`.
- Use the **built-in test runner** `node --test` (with `--experimental-test-coverage`) instead of `jest`/`mocha` for greenfield.
- Import built-ins with the **`node:` prefix**: `import { readFile } from 'node:fs/promises'`.

---

## 1. Architecture

1. **Structure by business component** (bounded context), not by technical type. Each component owns its API, logic, data.
   ```
   apps/orders  apps/users  apps/payments
   libraries/logger  libraries/auth
   ```
2. **Layer each component in 3 tiers:** `entry-points` (controller/API, MQ consumer), `domain` (logic/services/DTOs),
   `data-access`. Never pass `req`/`res` into the domain layer.
3. **Wrap reusable utilities as packages** with their own `package.json` + `exports` (public interface via `exports`).
4. **Config:** environment-aware, secure, hierarchical, validated. Prefer `zod` (or `convict`, `env-var`) + `node --env-file`.
   Fail fast on missing required keys. Keep secrets out of code/repo.
5. **Choose the framework deliberately.** 2026 shortlist: **Nest.js** (large/OOP teams), **Fastify** (microservices/simple
   mechanics, ~3× Express throughput), **Express** (ubiquity), **Koa**. Prefer Fastify over Express for new APIs.
6. **Use TypeScript sparingly** — simple types for signatures/returns; reach for advanced features only when needed.
   Types catch ~20% of bugs; tests still catch the rest.

## 2. Error Handling

1. Use **async/await** (try/catch) — avoid callback style and `.then` chains.
2. **Extend the built-in `Error`** for app errors; add `code`, `isOperational`/`isCatastrophic`. Never `throw` a string/literal.
3. **Distinguish operational vs programmer errors.** Operational = expected (bad input, timeouts) → handle & continue.
   Programmer = bugs → crash + let the runtime/orchestrator restart.
4. **Handle errors centrally** (one error-handling object/middleware), not scattered.
5. **Document API errors** (OpenAPI/GraphQL).
6. **Exit gracefully** on unknown errors (close servers, flush logs, `SIGTERM`/`SIGINT` handling, `server.close()`).
7. **Use a mature structured logger** (`pino`) — log JSON to stdout.
8. **Test error flows** too (not just happy paths).
9. **APM** for discovery (Sentry/DataDog/New Relic/etc.).
10. **Catch unhandled rejections**: `process.on('unhandledRejection', ...)` and `uncaughtException` → log + exit.
11. **Validate arguments** at the boundary (dedicated lib).
12. **Always `await` promises before returning** (so the caller's stack trace includes the async frame).
13. **Subscribe to `'error'` events** on every EventEmitter/stream.

## 3. Code Style

1. **ESLint** + `eslint-plugin-node`/`eslint-plugin-n`; enforce `no-throw-literal`, security plugins.
2. `const` > `let`; **never `var`**. Use `===`/`!==`.
3. **async/await over callbacks**.
4. **Import modules at top level**, not inside functions (except lazy-loading large/optional deps).
5. **Explicit entry point** per module (`exports`, `main`, `type`).
6. **Name functions** (avoid anonymous callbacks in hot paths/debugging).
7. **Avoid side effects outside functions** (module top-level I/O).
8. Curly braces on the same line; consistent naming conventions.

## 4. Testing

1. **At minimum, API/component tests** (spin up the server, hit endpoints).
2. Test names: **3 parts** — unit, scenario, expectation.
3. **AAA pattern** (Arrange/Act/Assert).
4. **Unify the Node version** (`.nvmrc`/`engines`/`volta`) so dev/CI/prod match.
5. **Per-test data**, avoid global fixtures/seeds.
6. **Tag tests** (`--test-name-pattern`, `test('...', { skip/only })`).
7. **Track coverage** to find dead spots.
8. **Production-like environment for e2e**.
9. **Mock external HTTP** (`undici` `MockAgent`, `nock`, `msw`).
10. **Test middleware in isolation**.
11. Fixed port in prod, **random port in tests** (`server.listen(0)`).
12. Test the **five outcomes**: success, validation failure, not-found, auth failure, and the "unknown error".

## 5. Production

1. **Monitoring + observability** (metrics, tracing, APM).
2. **Smart structured logging** (JSON, correlation/transaction id, `pino`).
3. **Delegate gzip/TLS/static to a reverse proxy** (nginx/traefik/ALB).
4. **Lock dependencies** (commit `package-lock.json`); install with **`npm ci`** in CI/prod.
5. **Process manager / orchestrator** for uptime (systemd, PM2, Docker/K8s) — never bare `node server.js`.
6. **Use all CPU cores** (cluster/`worker_threads` or multiple replicas).
7. **Maintenance/health endpoint** (`/health`, `/ready`).
8. **Production-ready startup**: validate config/env at boot, exit if invalid.
9. **Guard memory** (`--max-old-space-size`, monitor RSS/heap).
10. **Keep frontend assets out of Node** (CDN/static host).
11. **Stay stateless** (sessions/state external).
12. **Auto-detect vulnerabilities** (`npm audit`, Dependabot/Renovate, `osv-scanner`).
13. **Transaction id** on every request/log entry.
14. **`NODE_ENV=production`**.
15. **Atomic zero-downtime deployments** (blue/green, rolling, readiness probes).
16. **Run an LTS release** in prod.
17. **Log to stdout** — never hardcode log file paths in the app.

## 6. Security

1. Linter **security rules** (`eslint-plugin-security`, `@typescript-eslint`).
2. **Rate-limit** concurrent requests (`@fastify/rate-limit`, `express-rate-limit`).
3. **Secrets** from env/secret manager, never in code or Docker args.
4. **ORM/parameterized queries** to prevent SQL/NoSQL injection.
5. **Security headers** (`helmet`), HTTPS, CSRF, cookie flags.
6. **Constantly inspect dependencies** for CVEs.
7. **Password hashing**: `bcrypt`/`scrypt`/`argon2` (use `node:crypto` `scrypt` built-in).
8. **Escape output** (HTML/JS/CSS) to prevent XSS.
9. **Validate incoming JSON** against a schema (zod/ajv).
10. **JWT blocklist** + short expiry + rotation.
11. **Brute-force protection** on auth endpoints.
12. **Run as non-root** (Docker `USER`, drop privileges).
13. **Limit payload size** (reverse proxy or `bodyLimit`).
14. **Avoid `eval`**/`new Function` with untrusted input.
15. **Guard against ReDoS** (evil regex) — use `node --eval` timeouts, safe regex.
16. **Avoid variable-based module loading** (`require(dynamic)`).
17. **Sandbox unsafe code** (`node:vm` with care, isolated worker).
18. **Care with child processes** (validate args, avoid shell interpolation).
19. **Hide error details** from clients.
20. **2FA on npm** and avoid publishing secrets to the registry (`.npmignore`, `files`).
21. **Inspect outdated packages** (`npm outdated`).

## 7. Performance

1. **Don't block the event loop** (no sync I/O in hot paths, no huge sync loops; offload CPU work to `worker_threads`).
2. **Prefer native JS methods** over userland utils (Lodash) — modern JS built-ins are fast and correct.

## 8. Docker

1. **Multi-stage builds** (build stage vs slim runtime).
2. **Bootstrap with `node` directly**, not `npm start` (`CMD ["node", "dist/main.js"]`).
3. Let the **orchestrator handle replication/restart**, not a process manager inside the container.
4. **`.dockerignore`** (exclude `node_modules`, secrets, `.git`).
5. **Clean dependencies** before production (only `--production` deps, or multi-stage copy).
6. **Graceful shutdown** (`STOPSIGNAL SIGTERM`, handle it in-app).
7. **Memory limits** via Docker + V8 flags (`--max-old-space-size`).
8. **Layer caching** (copy `package*.json` first, then `npm ci`, then source).
9. **Explicit image tags** (`node:26-alpine` / `node:26-slim`), never `latest`.
10. **Smaller base images** (alpine/slim/distroless).
11. **No build-time secrets** in `ARG`/layers (use BuildKit secrets).
12. **Scan images** (Trivy, `docker scan`).
13. **Lint the Dockerfile** (`hadolint`).
