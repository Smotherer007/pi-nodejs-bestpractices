# Node.js Security — Deep Dive (2026)

> Consolidated from `nodebestpractices/sections/security/*` + OWASP, reconciled to Node 26.
> Use `helmet` (Express) / `@fastify/helmet` (Fastify) / `koa-helmet` (Koa) to set most headers.

## Security Headers

| Header | Recommended value | Protects against |
|--------|-------------------|------------------|
| `Strict-Transport-Security` (HSTS) | `max-age=31536000; includeSubDomains` | protocol downgrade, cookie hijack |
| `X-Content-Type-Options` | `nosniff` | MIME sniffing |
| `X-Frame-Options` | `deny` (or `SAMEORIGIN`) | clickjacking (prefer CSP `frame-ancestors`) |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | referrer leakage |
| `Content-Security-Policy` | `default-src 'self'; …` | XSS (defense-in-depth) |
| `Permissions-Policy` | restrict camera/mic/geolocation | feature abuse |
| `Cross-Origin-Opener-Policy` | `same-origin` | cross-origin isolation |

**Obsolete / deprecated — do NOT use:**
- `X-XSS-Protection` — ignored by modern browsers; CSP is the real control.
- `Public-Key-Pins` (HPKP) — removed from browsers; rely on Certificate Transparency instead.
- `Expect-CT` — obsolete since CT is now enforced by default.

```js
import helmet from 'helmet';
app.use(helmet());                       // sensible defaults
app.use(helmet.contentSecurityPolicy({
  directives: { defaultSrc: ["'self'"], scriptSrc: ["'self'"] }
}));
```

## Crypto Essentials

```js
import { timingSafeEqual, randomBytes, randomUUID, scrypt } from 'node:crypto';

// 1. Compare secrets/hashes in constant time (never ===)
const safe = timingSafeEqual(Buffer.from(a), Buffer.from(b));

// 2. Secure random tokens (never Math.random())
const token = randomBytes(32).toString('hex');   // or randomUUID()

// 3. Password hashing — scrypt (built-in), or argon2id / bcrypt (cost 10+)
const salt = randomBytes(16);
scrypt(password, salt, 64, (err, derivedKey) => { /* store salt:hash */ });
```

- `Math.random()` is **not** cryptographically secure — never for tokens/sessions.
- Never use reversible crypto for passwords (MD5/SHA-1/SHA-256 unsalted are wrong tools).

## Secrets

- Load from **env vars** / secret manager (HashiCorp Vault, AWS/GCP KMS). Node 26: `node --env-file=.env` (no `dotenv`).
- A good litmus test: *could you open-source the repo right now without leaking credentials?*
- Scan for accidental secrets: `gitleaks`, `trufflehog`, `git-secrets` (pre-commit + CI).
- Never log secrets; redact in log serializers.

## Input Validation

Validate **as early as possible** (edge middleware → 400 on invalid). Minimizes attack surface, prevents
DoS and insecure deserialization. TS-first: **zod**; JSON-schema: **ajv**; classic: **joi**/`validator.js`.

```js
import { z } from 'zod';
const Product = z.object({
  id: z.string(),
  name: z.string().min(1),
  price: z.number().positive(),
});
// app.post('/product', (req, res) => { const p = Product.parse(req.body); ... })
```

## Injection

- **SQL/NoSQL:** parameterized queries or an ORM/ODM — never string-concatenate user input into queries.
- **Command injection:** `child_process` with an args array and no shell interpolation:
  ```js
  import { execFile } from 'node:child_process';
  execFile('git', ['log', userInput], (err, out) => {}); // userInput is an arg, not shell
  ```

## XSS

- Use a templating engine/framework that **auto-escapes by default** (React, EJS, Pug, Angular). Escape
  untrusted output **context-sensitively** (HTML body vs attribute vs JS vs CSS vs URL).
- Defense-in-depth: CSP + cookies with `HttpOnly` (blocks JS cookie access).

## Authentication & Authorization

- Use **OAuth/OIDC** over basic auth; require **MFA**; strong password policy; no default credentials.
- **Rate-limit login** (and password reset) — e.g. max N attempts per time window.
- Return a **generic** auth error on login failure (don't reveal which of username/password was wrong).
- **Least privilege**: run as non-root, service accounts, assign permissions to groups not users.
- Short-lived **access token** + rotating **refresh token**.

### JWT

```js
// verify signature AND algorithm; keep payload small; short expiry; blocklist on logout/rotation
jwt.verify(token, secret, { algorithms: ['HS256'] });
```

## Rate Limiting & DoS

- `express-rate-limit` (Express) or `@fastify/rate-limit` (Fastify).
- **Limit payload size** (reverse proxy or `express.json({ limit })` / `bodyLimit`).
- **ReDoS**: avoid catastrophic regex; prefer safe patterns / `RE2` / timeouts. Don't run user regex unbounded.
- **Avoid `eval`/`new Function`** with untrusted input.
- **Sandbox** untrusted code (`node:vm` with care, isolated `worker_threads`).

## Sessions & Cookies

- `HttpOnly` (no JS access), `Secure` (HTTPS only), `SameSite=Lax|Strict`.
- Rotate session secrets; set sane `maxAge`; regenerate session on privilege change.

## Other Hardening

- **Hide error details from clients** (stack traces leak internals) — log full error server-side, return generic message.
- **Safe redirects**: validate destination against an allowlist; reject `//` (protocol-relative) and external URLs.
- **Run as non-root** in Docker (`USER node`), drop capabilities.
- **Dependency & image scanning**: `npm audit`, `osv-scanner`, Trivy/`docker scan`, Dependabot/Renovate.
- **`security.txt`** (production) at `/.well-known/security.txt` and **`SECURITY.md`** (open source) — give researchers a reporting channel.
- **PII**: encrypt at rest/in transit, follow GDPR/CCPA — don't log PII.

## OWASP Quick Checklist

- **Broken auth**: MFA, rotate keys, no default creds, rate-limit login, generic errors.
- **Broken access control**: least privilege, no root console, group-based perms.
- **Security misconfiguration**: cookie `Secure`/`SameSite`/`HttpOnly`, restrict internal network, STRIDE/DREAD threat modeling, DDoS protection at LB.
- **Sensitive data exposure**: TLS + HSTS only, vault for secrets, encrypt in transit, no secrets in logs.
- **Vulnerable components**: scan images, auto-patch, short-lived access tokens, audit API calls.
- **Insufficient logging/monitoring**: alert on logins/permission changes/login-failure spikes; timestamp+username on DB changes.
