# Framework Selection & Testing — Deep Dive (2026)

> Consolidated from `nodebestpractices/sections/projectstructre/choose-framework.md` and
> `sections/testingandquality/*`, reconciled to Node 26.

## Framework Selection

| Framework | Pros | Cons | Choose when |
|-----------|------|------|-------------|
| **Express** | Unmatched popularity; huge middleware ecosystem; every dev knows it | Minimal (only a router); no native async/await ergonomics; slow-ish; barely maintained | You need fine-grained control + an experienced architect |
| **Nest.js** | Most batteries (MQ, jobs, DI); OOP; great docs; vibrant | High abstraction hides Node conventions; TS-heavy; steeper learning curve; opinionated | OOP/Java-Spring-style team, large monolith, fast first delivery |
| **Fastify** | Lean, fast (~3× Express throughput), standards-based, official plugins | Smaller ecosystem than Express/Nest | Microservices, solid JS team, "Node spirit" |
| **Koa** | Simpler/nimbler than Express; modern async/await; better perf | Tiny ecosystem; minimal (fewer batteries) | You want Express-like minimalism with a modern API |

**Default recommendation (2026):** Fastify for new APIs; Nest.js for large/OOP teams and monoliths.

## Testing

### The five outcomes to test (not just the happy path)

1. **Response** — status code + body/schema correctness.
2. **New state** — the data was actually persisted/modified (don't only check the response).
3. **External calls** — HTTP/transport to outside services (email, SMS, payment) happened correctly.
4. **Message queues** — the right message was enqueued.
5. **Observability** — errors are handled, logged, and metric'd correctly (the "user" here is the SRE).

### Test structure

- **AAA**: Arrange → Act → Assert.
- **3-part names**: `unit under test → scenario → expectation` (e.g. `OrderService → when stock is 0 → rejects`).
- **Per-test data**: create data in each test; avoid global fixtures/seeds (they couple tests and hide state).

### Mocking external HTTP

Use a mock that intercepts at the client level (no network):

```js
import { MockAgent, setGlobalDispatcher } from 'undici';
const mock = new MockAgent();
mock.disableNetConnect(); // fail on any real network call
setGlobalDispatcher(mock);
const pool = mock.get('https://api.example.com');
pool.intercept({ path: '/users/1', method: 'GET' }).reply(200, { id: 1 });
```

### Ports & environment

- **Random port in tests** (`server.listen(0)` — let the OS pick), fixed port in production.
- **Production-like e2e** — run against a real (containerized) database/queue, not mocks.

### Built-in runner (Node 26)

```bash
node --test                              # auto-discovers test files
node --test --experimental-test-coverage # coverage report
node --test --test-name-pattern="order"  # run a subset (tags)
```

- Keep **`tsc --noEmit`** in CI even if you run `.ts` directly (type-stripping ≠ type-checking).
- Tag tests for grouping/filtering; track coverage to find dead spots (not as a vanity metric).

### Coverage is not a goal

Coverage identifies *untested* areas and wrong test patterns — it does not prove correctness.
Prioritize the five outcomes over hitting a percentage.
