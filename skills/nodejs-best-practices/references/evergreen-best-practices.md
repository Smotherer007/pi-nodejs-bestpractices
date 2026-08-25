# Node.js Best Practices — Complete Catalog (2026)

> Full catalog curated from [goldbergyoni/nodebestpractices](https://github.com/goldbergyoni/nodebestpractices)
> (2026 edition, by Yoni Goldberg, MIT License) and reconciled to Node 26. Item numbers match the source.

## 1. Project Architecture

1. **Structure your solution by business components** (bounded contexts), not by technical type. Each component owns its API, logic, and data.
2. **Layer components in 3 tiers** — `entry-points` (controller/API/MQ), `domain` (logic/services/DTOs), `data-access` — keep the web layer within its boundaries (never pass `req`/`res` into domain).
3. **Wrap common utilities as packages** — own `package.json` + `exports` to define the public interface; consider publishing.
4. **Environment-aware, secure, hierarchical config** — keys from file *and* env, secrets out of code, validated (fail fast). Use `zod`/`convict`/`env-var` + `node --env-file`.
5. **Consider all consequences when choosing the main framework** — see `framework-and-testing.md` (Express / Nest.js / Fastify / Koa).
6. **Use TypeScript sparingly and thoughtfully** — simple types for signatures/returns; reach for advanced features only when needed.

## 2. Error Handling

1. **Use async/await (or promises)** for async error handling — avoid callback pyramids.
2. **Extend the built-in `Error`** — add `code`, `isOperational`; never throw strings/literals.
3. **Distinguish operational vs programmer errors** — operational = handle & continue; programmer = crash + restart.
4. **Handle errors centrally**, not within a middleware.
5. **Document API errors** using OpenAPI or GraphQL.
6. **Exit the process gracefully** on unknown errors (close servers, flush logs, handle `SIGTERM`/`SIGINT`).
7. **Use a mature logger** (`pino`) for structured, high-throughput logs.
8. **Test error flows** too.
9. **Discover errors/downtime using APM** products.
10. **Catch unhandled promise rejections** (`unhandledRejection`/`uncaughtException` → log + exit).
11. **Fail fast, validate arguments** with a dedicated library.
12. **Always await promises before returning** — keeps full stack traces.
13. **Subscribe to `'error'` events** on every emitter/stream.

## 3. Code Style

1. **Use ESLint.**
2. **Use Node.js eslint plugins** (`eslint-plugin-n`).
3. **Curly braces on the same line.**
4. **Separate statements properly.**
5. **Name your functions** (avoid anonymous callbacks in hot paths).
6. **Use naming conventions** (variables/constants/functions/classes).
7. **Prefer `const` over `let`; ditch `var`.**
8. **Import modules first**, not inside functions (except lazy-loading large/optional deps).
9. **Set an explicit entry point** to a module/folder (`exports`, `main`, `type`).
10. **Use `===`/`!==`.**
11. **Use async/await, avoid callbacks.**
12. **Use arrow function expressions.**
13. **Avoid side effects outside functions.**

## 4. Testing

1. **At minimum, write API/component tests.**
2. **Include 3 parts in each test name** — unit → scenario → expectation.
3. **Structure tests by AAA** (Arrange/Act/Assert).
4. **Ensure the Node version is unified** (`.nvmrc`/`engines`/`volta`).
5. **Avoid global fixtures/seeds; add per-test data.**
6. **Tag your tests** for grouping/filtering.
7. **Check coverage** to find wrong test patterns.
8. **Use a production-like environment for e2e.**
9. **Refactor regularly with static analysis tools.**
10. **Mock external HTTP services** (Undici `MockAgent`, `nock`, `msw`).
11. **Test middlewares in isolation.**
12. **Specify a port in production, randomize in testing** (`server.listen(0)`).
13. **Test the five possible outcomes** — response, new state, external calls, message queues, observability.

## 5. Production

1. **Monitoring.**
2. **Smart logging** — structured JSON, correlation/transaction id.
3. **Delegate gzip/SSL/static to a reverse proxy** (nginx/traefik/ALB).
4. **Lock dependencies** (commit the lockfile).
5. **Guard process uptime** with the right tool (systemd/PM2/Docker/K8s) — never bare `node server.js`.
6. **Utilize all CPU cores** (cluster/`worker_threads`/replicas).
7. **Create a maintenance/health endpoint** (`/health`, `/ready`).
8. **Discover unknowns with APM products.**
9. **Make code production-ready** — validate config/env at boot, exit if invalid.
10. **Measure and guard memory usage** (`--max-old-space-size`, monitor heap/RSS).
11. **Get frontend assets out of Node** (CDN/static host).
12. **Strive to be stateless** (external sessions/state).
13. **Auto-detect vulnerabilities** (`npm audit`, Dependabot/Renovate, `osv-scanner`).
14. **Assign a transaction id** to each log statement.
15. **Set `NODE_ENV=production`.**
16. **Automated, atomic, zero-downtime deployments** (blue/green, rolling, readiness probes).
17. **Use an LTS release** in production.
18. **Log to stdout** — don't hardcode log destinations in the app.
19. **Install with `npm ci`.**

## 6. Security

1. **Embrace linter security rules** (`eslint-plugin-security`).
2. **Limit concurrent requests** (rate limiting).
3. **Extract secrets** from config files; use a secret manager.
4. **Prevent query injection** via ORM/parameterized queries.
5. **Generic security best practices** (OWASP) — see `security-deep-dive.md`.
6. **Adjust HTTP response headers** (helmet).
7. **Constantly inspect for vulnerable dependencies.**
8. **Hash passwords** with `bcrypt`/`scrypt`/`argon2` (built-in `crypto.scrypt`).
9. **Escape HTML/JS/CSS output.**
10. **Validate incoming JSON schemas** (zod/ajv/joi).
11. **Support JWT blocklisting** + short expiry + rotation.
12. **Prevent brute-force attacks** on authorization endpoints.
13. **Run as non-root user.**
14. **Limit payload size** (reverse proxy or `bodyLimit`).
15. **Avoid `eval`**/`new Function` with untrusted input.
16. **Prevent evil regex** (ReDoS).
17. **Avoid module loading via a variable** (`require(dynamic)`).
18. **Run unsafe code in a sandbox** (`node:vm` with care, isolated worker).
19. **Take extra care with child processes** (args array, no shell interpolation).
20. **Hide error details from clients.**
21. **Configure 2FA for npm/yarn.**
22. **Modify session middleware settings** (HttpOnly, Secure, SameSite).
23. **Avoid DOS by explicitly setting when a process should crash.**
24. **Prevent unsafe redirects** (allowlist destinations).
25. **Avoid publishing secrets to the npm registry** (`files`, `.npmignore`).
26. **Inspect for outdated packages** (`npm outdated`).
27. **Import built-ins using the `node:` protocol.**

## 7. Performance

1. **Don't block the event loop** (no sync I/O in hot paths; offload CPU work to `worker_threads`).
2. **Prefer native JS methods** over userland utils like Lodash.

## 8. Docker

1. **Multi-stage builds** for leaner, more secure images.
2. **Bootstrap with `node` command**, avoid `npm start` (`CMD ["node", "dist/main.js"]`).
3. **Let the Docker runtime handle replication/uptime** (no process manager inside the container).
4. **Use `.dockerignore`** to prevent leaking secrets.
5. **Clean dependencies before production** (`--production` or multi-stage copy).
6. **Shutdown smartly and gracefully** (`STOPSIGNAL SIGTERM` + in-app handling).
7. **Set memory limits** using both Docker and V8.
8. **Plan for efficient caching** (copy `package*.json` first → `npm ci` → source).
9. **Use explicit image references**, avoid `latest`.
10. **Prefer smaller base images** (alpine/slim/distroless).
11. **Clean out build-time secrets** (BuildKit secrets, no `ARG`).
12. **Scan images for vulnerabilities** (Trivy, `docker scan`).
13. **Clean the NODE_MODULE cache.**
14. **Generic Docker practices** (healthchecks, non-root, minimal layers).
15. **Lint your Dockerfile** (`hadolint`).
