# Node.js Best Practices — citable rule index

The rule IDs below (`NBP-<section>.<item>`) mirror the numbering of the community **Node.js Best Practices** list at <https://github.com/goldbergyoni/nodebestpractices> — the de-facto canonical Node list. Cite them in findings the way the sibling `react-code-review` skill cites Vercel rule IDs: `NBP-6.10`, `NBP-2.10`, `NBP-5.15`.

Two rules about citation:

1. **Never invent an ID.** If nothing here fits the issue, write the finding grounded in the repository's own evidence and say which established practice it violates in words — a fabricated `NBP-9.3` destroys the report's credibility.
2. **The ID is grounding, not the finding.** Citing `NBP-6.10` is not a substitute for showing the unvalidated route in this repo. The ID says *which* practice; the evidence says *where*.

When it is useful to link the reader to the upstream write-up, the anchor is the item title slugified, e.g. `https://github.com/goldbergyoni/nodebestpractices#-610-validate-incoming-json-schemas`. Only include a link if you are confident in the anchor; the ID alone is sufficient.

---

## Section 1 — Project architecture (`ARCH`)

| ID | Practice | What to look for in the repo |
|---|---|---|
| NBP-1.1 | Structure the solution by business components | Top-level folders named for domains (`orders/`, `merchants/`) rather than one global `controllers/` + `services/` + `models/` split that every feature reaches into. |
| NBP-1.2 | Layer components with 3 tiers; keep the web layer within its boundaries | Express/Nest `req`/`res` objects passed down into services or data access; business rules written inside route handlers; ORM models used directly by controllers. |
| NBP-1.3 | Wrap common utilities as packages | Copy-pasted helpers (logger, http client, error classes) duplicated across services in a monorepo instead of a shared workspace package. |
| NBP-1.4 | Use environment-aware, secure, hierarchical config | `process.env.X` read ad hoc deep inside modules, no schema/validation of env at boot, no typed config object, missing `.env.example`. |
| NBP-1.5 | Weigh the consequences when choosing the main framework | Framework used against its grain (e.g. Nest modules bypassed by manual `new Service()`), or two frameworks/patterns mixed in one service. |
| NBP-1.6 | Use TypeScript sparingly and thoughtfully | `any` on public boundaries, `strict: false` in `tsconfig.json`, type assertions (`as X`) hiding unvalidated external data, `@ts-ignore` on real errors. |

## Section 2 — Error handling (`ERR`, `ASYNC`)

| ID | Practice | What to look for in the repo |
|---|---|---|
| NBP-2.1 | Use async/await or promises for async error handling | Callback-style error plumbing mixed with promises; `.then()` chains without `.catch()`. |
| NBP-2.2 | Extend the built-in Error object | Errors thrown as strings/objects (`throw { status: 400 }`), no `AppError`/`ApiError` base class carrying status + operational flag. |
| NBP-2.3 | Distinguish catastrophic errors from operational errors | Every error handled the same way; programmer errors swallowed and the process kept alive in an unknown state. |
| NBP-2.4 | Handle errors centrally, not within each middleware | Per-route `try/catch` blocks that each format their own response; no single error-handling middleware / exception filter. |
| NBP-2.5 | Document API errors (OpenAPI / GraphQL) | Error shapes undocumented; clients have to guess status codes and body shape. |
| NBP-2.6 | Exit the process gracefully when a stranger comes to town | No `process.on('uncaughtException')` strategy; process kept running after an unknown error; no process manager to restart it. |
| NBP-2.7 | Use a mature logger to increase error visibility | `console.log`/`console.error` in application code instead of `winston`/`pino`; no log levels; no structured (JSON) output. |
| NBP-2.8 | Test error flows | Tests cover only the happy path; no test asserts the 4xx/5xx response or the error body. |
| NBP-2.9 | Discover errors and downtime using APM | No error/APM reporting (Sentry, Datadog, OpenTelemetry) — errors visible only in local logs. |
| NBP-2.10 | Catch unhandled promise rejections | No `process.on('unhandledRejection')`; floating promises (an async call invoked without `await`, `.catch()`, or `void`). |
| NBP-2.11 | Fail fast — validate arguments with a dedicated library | Hand-rolled `if (!req.body.x)` checks instead of a schema library (Joi, Zod, `class-validator`, Fastify schemas); validation done deep in a service rather than at the boundary. |
| NBP-2.12 | Always await promises before returning | `return somePromise()` inside `try` — the `catch` never fires and the stack trace is truncated. |
| NBP-2.13 | Subscribe to `error` events on emitters and streams | Streams, sockets, DB clients, queue consumers piped/used without an `.on('error')` handler — an unhandled `error` event crashes the process. |

## Section 3 — Code patterns and style (`CQ`)

| ID | Practice | What to look for in the repo |
|---|---|---|
| NBP-3.1 | Use ESLint | No ESLint config, or lint not run in CI; `lint` script exists but the codebase has never passed it. |
| NBP-3.2 | Use Node-specific ESLint plugins | Missing `eslint-plugin-node` / `eslint-plugin-security` / `@typescript-eslint` rules like `no-floating-promises` (the single highest-value rule for Node correctness). |
| NBP-3.3 | Start a code block's curly braces on the same line | Style — only report if the repo's own config is violated. |
| NBP-3.4 | Separate statements properly | Style/ASI hazards — only report on a real defect. |
| NBP-3.5 | Name your functions | Anonymous functions everywhere, hurting stack traces and heap snapshots. |
| NBP-3.6 | Use naming conventions for variables, constants, functions, classes | Inconsistent casing across modules; `UPPER_SNAKE` constants that are mutable, classes not `PascalCase`. |
| NBP-3.7 | Prefer `const` over `let`, ditch `var` | Any remaining `var`; `let` for values never reassigned. |
| NBP-3.8 | Require modules first, not inside functions | `require()` inside a handler body (per-request resolution cost, hidden dependencies). |
| NBP-3.9 | Set an explicit entry point to a module/folder | Deep relative imports (`../../../services/foo/internal/bar`) reaching past a module's public surface; no `index.ts` barrel per component. |
| NBP-3.10 | Use `===` | `==` comparisons, especially against `null`/`undefined`/`0`. |
| NBP-3.11 | Use async/await, avoid callbacks | `callback(err, data)` plumbing in new code; nested callbacks. |
| NBP-3.12 | Use arrow function expressions | Relevant mainly where `this` binding is being worked around. |
| NBP-3.13 | Avoid side effects outside of functions | Module top-level code that connects to a DB, reads a file, or mutates global state at import time — makes the module untestable and import order significant. |

## Section 4 — Testing and quality (`TEST`)

| ID | Practice | What to look for in the repo |
|---|---|---|
| NBP-4.1 | At minimum, write API (component) tests | Only unit tests of pure helpers; no test that exercises a route end-to-end through the app. |
| NBP-4.2 | Include 3 parts in each test name (unit under test, scenario, expectation) | Test names like `it('works')`. |
| NBP-4.3 | Structure tests by AAA (Arrange, Act, Assert) | Tests with interleaved setup and assertions that are unreadable at a glance. |
| NBP-4.4 | Ensure the Node version is unified | `engines.node`, `.nvmrc`, CI matrix, and Dockerfile base image disagreeing with each other. |
| NBP-4.5 | Avoid global fixtures and seeds; add data per test | A shared seeded DB every test depends on — tests coupled and order-dependent. |
| NBP-4.6 | Tag your tests | No way to run a fast subset (`#sanity`, `#e2e`) separately in CI. |
| NBP-4.7 | Check test coverage | No coverage config/threshold; coverage collected but never enforced. |
| NBP-4.8 | Use a production-like environment for e2e | e2e run against mocks/sqlite while production is a different engine. |
| NBP-4.9 | Refactor regularly using static analysis | No type-check/lint/complexity gate in CI. |
| NBP-4.10 | Mock responses of external HTTP services | Tests hitting real third-party APIs (flaky, slow, sometimes billable). |
| NBP-4.11 | Test middlewares in isolation | Auth/validation middleware only ever covered incidentally. |
| NBP-4.12 | Specify a port in production, randomize in testing | Hardcoded port in tests causing parallel-run collisions. |
| NBP-4.13 | Test the five outcomes (response, state change, external call, message queue, observability) | Tests assert only the HTTP status and never that the row was written or the event emitted. |

## Section 5 — Going to production (`OPS`)

| ID | Practice | What to look for in the repo |
|---|---|---|
| NBP-5.1 | Monitoring | No health/readiness endpoint, no metrics, nothing to alert on. |
| NBP-5.2 | Increase observability with smart logging | Unstructured string logs; no request context; logs that can't be queried. |
| NBP-5.3 | Delegate gzip/SSL to a reverse proxy | TLS termination or compression done in-process where a proxy is available. |
| NBP-5.4 | Lock dependencies | No lockfile committed, or a lockfile for a package manager the repo doesn't use; `^` ranges installed fresh in CI. |
| NBP-5.5 | Guard process uptime with the right tool | No process manager/orchestrator restart policy (`pm2`, systemd, Kubernetes). |
| NBP-5.6 | Utilize all CPU cores | Single process on a multi-core box with no cluster/replica strategy. |
| NBP-5.7 | Create a maintenance endpoint | No safe way to inspect a live process. |
| NBP-5.8 | Discover the unknowns using APM | Same as NBP-2.9, from the performance angle. |
| NBP-5.9 | Make your code production-ready | Dev-only shortcuts left in (verbose stack traces to clients, seeded admin users, permissive CORS). |
| NBP-5.10 | Measure and guard memory usage | No memory limit and no leak detection on long-lived processes; unbounded in-memory caches/arrays. |
| NBP-5.11 | Get frontend assets out of Node | Static assets served by the app process where a CDN/proxy should. |
| NBP-5.12 | Strive to be stateless | In-memory session/cache/state that breaks the moment a second replica exists. |
| NBP-5.13 | Use tools that automatically detect vulnerabilities | No `npm audit`/Snyk/Dependabot in CI. |
| NBP-5.14 | Assign a transaction id to each log statement | No request/correlation ID threaded through logs (`AsyncLocalStorage`, `cls-hooked`, or an explicit context arg). |
| NBP-5.15 | Set `NODE_ENV=production` | Not set in Dockerfile/deploy config — silently costs performance and enables dev behavior in frameworks. |
| NBP-5.16 | Design automated, atomic, zero-downtime deployments | Manual deploy steps; no rollback path. |
| NBP-5.17 | Use an LTS release of Node.js | `engines`/base image pinned to an EOL or odd-numbered release. |
| NBP-5.18 | Log to stdout, not to a file destination in-app | Logger writing to app-managed files/rotation instead of stdout. |
| NBP-5.19 | Install packages with `npm ci` | CI or Dockerfile using `npm install`, which can drift from the lockfile. |

## Section 6 — Security (`SEC`) — with OWASP mapping

Cite both IDs for security findings: `NBP-6.4 / OWASP A03:2021 Injection`.

| ID | Practice | OWASP (2021) | What to look for in the repo |
|---|---|---|---|
| NBP-6.1 | Embrace linter security rules | A05 Misconfiguration | No `eslint-plugin-security` / `eslint-plugin-no-unsanitized`. |
| NBP-6.2 | Limit concurrent requests (rate limiting) | A04 Insecure Design | No `express-rate-limit`/proxy-level limiting, or limiter registered but not applied to auth routes. |
| NBP-6.3 | Extract secrets from config files; encrypt them | A02 Cryptographic Failures / A05 | Secrets, API keys, connection strings, or JWT secrets literal in source, committed `.env`, or defaulted in code (`process.env.SECRET \|\| 'dev-secret'`). |
| NBP-6.4 | Prevent query injection with ORM/ODM or parameterized queries | A03 Injection | String-concatenated SQL, `$queryRawUnsafe`, `sequelize.query` with interpolation, Mongo query objects built from raw `req.body`. |
| NBP-6.5 | Generic security best practices | — | Catch-all; prefer a more specific ID when one fits. |
| NBP-6.6 | Adjust HTTP response headers | A05 Misconfiguration | `helmet` (or equivalent) absent; permissive `Access-Control-Allow-Origin: *` on an authenticated API. |
| NBP-6.7 | Constantly inspect for vulnerable dependencies | A06 Vulnerable Components | No automated audit; known-vulnerable versions pinned. |
| NBP-6.8 | Protect passwords/secrets with bcrypt or scrypt | A02 Cryptographic Failures | Passwords hashed with md5/sha1/plain, low bcrypt cost factor, secrets compared with `===` instead of a timing-safe compare. |
| NBP-6.9 | Escape HTML, JS and CSS output | A03 Injection | Server-rendered templates or emails interpolating user input unescaped. |
| NBP-6.10 | Validate incoming JSON schemas | A04 Insecure Design | Routes reading `req.body`/`req.query`/`req.params` with no schema validation at the boundary; validation defined but not applied to every route. |
| NBP-6.11 | Support blocklisting JWTs | A07 Auth Failures | No revocation path; long-lived tokens with no refresh/rotation. |
| NBP-6.12 | Prevent brute-force attacks against authorization | A07 Auth Failures | Login/OTP/reset endpoints with no per-account or per-IP throttling or lockout. |
| NBP-6.13 | Run Node.js as a non-root user | A05 Misconfiguration | Dockerfile with no `USER node`. |
| NBP-6.14 | Limit payload size | A04 Insecure Design | `express.json()` with no `limit`; unbounded file uploads. |
| NBP-6.15 | Avoid `eval` | A03 Injection | `eval`, `new Function`, `vm` on user input. |
| NBP-6.16 | Prevent evil RegEx (ReDoS) | A04 Insecure Design | User input fed to a regex with nested quantifiers; regexes built from user input. |
| NBP-6.17 | Avoid module loading using a variable | A03 Injection | `require(userSuppliedPath)` / dynamic `import()` on untrusted values. |
| NBP-6.18 | Run unsafe code in a sandbox | A03 Injection | User-provided code/templates executed in-process. |
| NBP-6.19 | Take extra care with child processes | A03 Injection | `exec`/`execSync` with interpolated input where `execFile` with an args array belongs. |
| NBP-6.20 | Hide error details from clients | A05 Misconfiguration | Stack traces, SQL text, or internal paths returned in API error bodies. |
| NBP-6.21 | Configure 2FA for npm/Yarn | A08 Integrity Failures | Publish workflow without 2FA / trusted publishing (only if the repo publishes packages). |
| NBP-6.22 | Modify session middleware settings | A05 Misconfiguration | Default session cookie name; missing `httpOnly`/`secure`/`sameSite`. |
| NBP-6.23 | Explicitly set when a process should crash (avoid DoS) | A04 Insecure Design | Unbounded queues/concurrency with no backpressure. |
| NBP-6.24 | Prevent unsafe redirects | A01 Broken Access Control | `res.redirect(req.query.url)` with no allowlist. |
| NBP-6.25 | Avoid publishing secrets to the npm registry | A02 Cryptographic Failures | No `files`/`.npmignore` discipline in a published package. |
| NBP-6.26 | Inspect for outdated packages | A06 Vulnerable Components | Long-unmaintained or EOL dependencies (e.g. an abandoned auth or sanitizer library). |
| NBP-6.27 | Import built-in modules with the `node:` protocol | A08 Integrity Failures | `require('fs')` instead of `node:fs` — protects against dependency-confusion shadowing. |

**Access control is the category NBP under-covers and reviews most often miss.** Broken access control is OWASP A01 and the most common real backend defect: an authenticated route that never checks *this* user owns *that* record (IDOR), a role check done in the UI but not the API, an admin route protected only by an unguessable path. Report these as `SEC` findings citing **OWASP A01:2021 Broken Access Control** even though no NBP ID covers them, and check ownership on every route that takes an ID.

## Section 7 — Performance (`PERF`)

| ID | Practice | What to look for in the repo |
|---|---|---|
| NBP-7.1 | Don't block the event loop | Synchronous fs calls (`readFileSync`, `existsSync`) on the request path, CPU-heavy work (crypto, big JSON, image/PDF work, `JSON.parse` of megabytes) in a handler, long synchronous loops over large arrays. |
| NBP-7.2 | Prefer native JS methods over user-land utils | Lodash/underscore for what the language now does natively. |

Section 7 upstream is deliberately thin. Ground the rest of the `PERF` category in repository evidence and standard backend reasoning, naming the mechanism explicitly:

- **N+1 queries** — a query inside a `for`/`map` over rows; a Prisma/TypeORM call per item where an `include`/`in` batch would do.
- **Missing indexes / full scans** — filtering or sorting on an unindexed column; check the schema/migrations, not just the query.
- **Unbounded result sets** — `findMany()` with no `take`/pagination on a table that grows.
- **Missing connection pooling or a pool per request** — a client instantiated inside a handler rather than once per process.
- **No timeouts on outbound calls** — `fetch`/`axios` with no timeout, so one slow upstream exhausts the pool.
- **Sequential awaits that could be concurrent** — independent `await`s in series where `Promise.all` applies (and the converse: unbounded `Promise.all` over thousands of items with no concurrency limit).
- **Missing caching on hot, stable reads** — repeated identical lookups per request.
- **Over-fetching** — `SELECT *` / no `select` projection pulling large columns the handler discards.

## Section 8 — Docker & deployment (`DOCK`)

| ID | Practice | What to look for in the repo |
|---|---|---|
| NBP-8.1 | Use multi-stage builds | Single-stage image shipping devDependencies, source, and build toolchain. |
| NBP-8.2 | Bootstrap with `node`, avoid `npm start` | `CMD ["npm", "start"]` — npm swallows signals, breaking graceful shutdown. |
| NBP-8.3 | Let the runtime handle replication and uptime | `pm2` cluster inside a container the orchestrator already replicates. |
| NBP-8.4 | Use `.dockerignore` | Missing/incomplete `.dockerignore` — `.env`, `.git`, `node_modules` leaking into the image. |
| NBP-8.5 | Clean up dependencies before production | devDependencies present in the final image. |
| NBP-8.6 | Shut down smartly and gracefully | No `SIGTERM` handler closing the HTTP server, DB pool, and queue consumers; in-flight requests killed on deploy. |
| NBP-8.7 | Set memory limits (Docker and v8) | No container memory limit and no `--max-old-space-size`. |
| NBP-8.8 | Plan for efficient caching | `COPY . .` before `npm ci`, busting the dependency layer on every source change. |
| NBP-8.9 | Use explicit image references, avoid `latest` | `FROM node:latest` or a floating major. |
| NBP-8.10 | Prefer smaller base images | Full Debian base where slim/alpine would serve. |
| NBP-8.11 | Clean out build-time secrets | Secrets passed via `ARG`/`ENV` and baked into layers. |
| NBP-8.12 | Scan images for vulnerabilities | No image scanning in the pipeline. |
| NBP-8.13 | Clean the npm cache | Cache left in the final layer. |
| NBP-8.14 | Generic Docker practices | Catch-all. |
| NBP-8.15 | Lint your Dockerfile | No hadolint or equivalent. |

---

## Framework-specific review cues

Apply only the section matching the stack detected in Step 1. Rating an Express app against NestJS conventions produces findings the team will correctly ignore.

### Express

- **Async errors.** Express 4 does **not** forward rejections from async handlers — an unwrapped `async` route with a throwing `await` hangs the request. Confirm one of: Express 5 (which does forward), `express-async-errors` imported at bootstrap, or an `asyncHandler` wrapper applied to every async route. If none exists, that is a real `ASYNC`/`ERR` finding; if one does, don't report missing per-route `try/catch`.
- **Error middleware.** Exactly one terminal `(err, req, res, next)` middleware, registered **after** all routes, and it must not leak stack traces (`NBP-6.20`). A handler that formats its own error response is a symptom of its absence (`NBP-2.4`).
- **Middleware order.** `helmet` → cors → body parsers with limits → rate limiter → routes → 404 → error handler. Auth/validation middleware registered after the route it should protect silently protects nothing — check order, not just presence.
- **Router-level auth.** `router.use(auth)` placed below some routes leaves those routes public. Enumerate routes and confirm each is covered.
- **Response duplication.** Missing `return` before `res.status(...).json(...)` followed by more code — causes `ERR_HTTP_HEADERS_SENT` under specific inputs.

### Fastify

- Schema-first: every route should declare `schema` for body/query/params/response. Response schemas also serialize faster and prevent accidental field leakage.
- Plugin encapsulation: decorators/hooks registered in the wrong scope silently don't apply; shared decorators need `fastify-plugin`.
- `reply` must be returned or awaited — a handler that neither returns the payload nor calls `reply.send()` hangs.

### NestJS

- Layering: controllers thin, business logic in providers, data access behind a repository/service. A controller injecting `PrismaClient` directly crosses the layer boundary (`NBP-1.2`).
- Validation: a global `ValidationPipe` with `whitelist: true` and `forbidNonWhitelisted: true` plus DTOs with `class-validator` decorators. A DTO with no decorators validates nothing.
- Guards for authn/authz, interceptors for cross-cutting concerns, exception filters for error shaping — logic of these kinds inlined in controllers is a finding.
- Module boundaries: services exported and imported through modules rather than instantiated manually; watch for circular dependencies patched with `forwardRef`.
- Scopes: request-scoped providers injected into singletons silently change lifecycle and cost.

### Prisma

- One `PrismaClient` per process (a client per request exhausts connections). Check it is instantiated in a module-level singleton, and that hot-reload in dev doesn't create many.
- `$queryRawUnsafe` / `$executeRawUnsafe` with interpolated input is injection (`NBP-6.4`); the tagged-template `$queryRaw` is parameterized and safe.
- Multi-write flows that must be atomic need `$transaction` — check for sequences of writes where a mid-sequence failure leaves inconsistent rows.
- N+1: a `findUnique`/`findFirst` inside a loop over a prior result set, where `include`/`select` or a single `in` query would do.
- `findMany` with no `take`/cursor on an unbounded table; `select` omitted so every column (including large blobs) is fetched.
- Migrations committed and applied deterministically (`migrate deploy`), not `db push` against production.

### TypeORM / Sequelize / Mongoose

- Entity/model definitions carrying business logic (fat models crossing the layer boundary).
- Lazy relations triggering per-row queries (N+1), the ORM equivalent of the Prisma case above.
- Mongoose: queries built from raw `req.body` allow operator injection (`{ $gt: '' }`) — validate and cast at the boundary (`NBP-6.10`).
- Transaction scope: a transaction opened but not passed to every participating call, so some writes land outside it.
