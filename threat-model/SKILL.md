---
name: threat-model
description: Scan the currently open repository and generate a comprehensive STRIDE-based threat model report. Use when the user asks to generate a threat model, scan for security threats, perform a threat assessment, or review the security posture of a codebase.
---

# Threat Model Generator

Systematically scan the open repo and produce a structured threat model report in chat.

## Step 1: Discover the Tech Stack and Repo Type

Read these files (whichever exist) to determine language, framework, and build tool:

```
pom.xml, build.gradle, build.gradle.kts    → Java/Kotlin (Maven or Gradle)
package.json                                → Node.js / TypeScript
requirements.txt, pyproject.toml, setup.py  → Python
go.mod                                      → Go
Cargo.toml                                  → Rust
Gemfile                                     → Ruby
```

Also check for:
- `Dockerfile`, `docker-compose.yml` → containerization
- `.github/workflows/`, `Jenkinsfile`, `.gitlab-ci.yml` → CI/CD
- `terraform/`, `*.tf`, `cloudformation/` → IaC
- `wrangler.toml`, `vercel.json`, `netlify.toml` → edge/CDN deployment
- `README.md` → project purpose

Use Glob to find these files. Read the build file and README to understand the project's purpose, dependencies, and structure.

### Classify the repo type

After reading the build file and source structure, classify the repo as one of:

| Type | Indicators |
|------|------------|
| **Backend** | REST controllers, database entities, server-only frameworks (Spring Boot, Express API, Django REST, Go HTTP servers) |
| **Frontend** | React/Next.js/Vue/Angular/Svelte, `src/app/` or `src/pages/` with page components, no database entities, client-side state management (Zustand, Redux, MobX) |
| **Full-stack** | Both backend endpoints AND frontend pages in the same repo (e.g., Next.js with API routes + pages, Django with templates) |
| **Mobile BFF** | Backend-for-Frontend pattern serving mobile apps. Indicators: no direct DB access, proxies to multiple backend microservices, mobile-specific headers (`x-device-id`, `x-app-version`, `x-user-agent-mobile`), request signing (HMAC), device/emulator detection, push notification endpoints, payment provisioning (Apple Pay/Google Pay), widget auth, MFA middleware. Typically Express/Fastify with 50+ service clients and generated route schemas. |

This classification determines which patterns each agent should prioritize. Backend, frontend, and mobile BFF checklists exist in [scanner-checklists.md](scanner-checklists.md) — agents should use the ones relevant to the detected repo type. Include the repo type classification in each agent prompt.

## Step 2: Parallel Scanning

Launch **4 explore subagents concurrently** using the Task tool. Each agent scans a different area. Provide the detected tech stack in each prompt so the agent knows what patterns to look for.

**IMPORTANT**: Set `subagent_type: "explore"` and include "very thorough" in each prompt. Tell each agent exactly what to search for and what to return.

### Agent 1: API Surface and Authentication

Tell the agent to find:
- All API endpoints (REST controllers, route handlers, GraphQL resolvers)
- Authentication mechanisms (Spring Security, Passport.js, custom interceptors, middleware)
- Authorization checks (role-based, scope-based, missing auth on endpoints)
- Input validation (Bean Validation, Joi, Zod, custom validators)
- Rate limiting / throttling
- CORS and CSRF configuration
- Endpoints with weak or missing auth

**Additional for frontend repos:**
- All pages/routes (Next.js `page.tsx`/`page.js`, React Router routes, Vue Router, SvelteKit routes)
- Server actions or server components (`'use server'` directives, API route handlers)
- How session tokens / auth credentials are received (URL params, cookies, headers, postMessage)
- Client-side token validation (or lack thereof)
- URL query parameter handling — are params validated or used raw?
- Security headers: CSP, X-Frame-Options, HSTS (in framework config, middleware, or edge config)

**Additional for mobile BFF repos:**
- Mobile-specific auth middleware: JWT verification, MFA token validation, widget token auth, phone number verification tokens
- Request signing: HMAC-SHA256 signature check via custom headers (`x-signed-auth`, `x-ml-date`, `x-ml-time`)
- Endpoints excluded from signature verification
- Public (pre-auth) endpoints: registration, MFA challenge, push token removal, widget data, device sensors
- IDOR risks: path parameters like `:userId` not validated against the authenticated user from JWT (`res.locals.userId`)
- Performance test bypass headers (`x-bypass-authorization`) that skip auth in non-production
- Device-specific headers: `x-device-id`, `x-app-version`, `x-user-agent-mobile`, `x-is-emulator`, `x-source`
- Generated route schemas that auto-wire endpoints from config files
- Rate limiting gaps: public routes that skip rate limiting because there's no userId key

Ask it to return: a table of endpoints/routes, auth mechanisms, validation gaps, and unprotected endpoints.

### Agent 2: External Integrations and Secrets

Tell the agent to find:
- External service clients (HTTP clients, SDKs, Feign, RestTemplate, axios, requests)
- How credentials are stored (env vars, config files, vaults, Secrets Manager, hardcoded)
- Default/fallback secrets in configuration
- `.env` files or config files with sensitive values
- Encryption mechanisms (at rest and in transit)
- PII handling (masking, redaction, encryption of sensitive fields)

**Additional for frontend repos:**
- `NEXT_PUBLIC_*` / `VITE_*` / `REACT_APP_*` env vars — these are bundled into client code and visible in the browser. Flag any that contain secrets, tokens, or API keys.
- OAuth flows — are client IDs, redirect URIs, or state parameters exposed? Is the OAuth `state` signed or just encoded?
- Third-party SDK integrations (Plaid, Stripe, Finicity, Akoya, etc.) — how are link tokens obtained? Are secrets kept server-side?
- `.gitignore` coverage — are `.env` files (not just `.env*.local`) properly ignored?
- Hardcoded URLs pointing to internal services or staging environments

**Additional for mobile BFF repos:**
- How the BFF forwards credentials to backend microservices (custom headers like `X-MoneyLion-User-Id`, API keys, Bearer tokens)
- Mobile request signing secrets (current + deprecated/rotated secrets)
- Push notification token handling (Sendbird, FCM, APNs)
- Payment provisioning credentials (Apple Pay, Google Pay — encrypted pass data, ephemeral keys)
- Widget JWT issuance — how widget tokens are signed and what data they contain
- Whether the BFF stores/caches secrets in memory and how rotation works
- Internal service URLs (HTTP vs HTTPS) and whether service mesh TLS is assumed

Ask it to return: a list of integrations, secrets management approach, encryption status, PII handling patterns, and client-exposed env vars.

### Agent 3: Persistence and Data Flow

**For backend repos**, tell the agent to find:
- Database entities/models and what sensitive data they store
- Raw SQL or native queries (SQL injection risk)
- Message queues (SQS, Kafka, RabbitMQ) and what data flows through them
- Caching (Redis, Memcached, in-memory) and what gets cached
- Logging patterns — is sensitive data logged?
- Audit tables that might store tokens or PII
- Data flow from external sources to DB to API consumers

**For frontend repos**, tell the agent to find:
- Client-side state management (Zustand, Redux, MobX, Pinia, Context) — what sensitive data (tokens, userId, PII) is held in stores?
- Browser storage — is `localStorage`, `sessionStorage`, or `IndexedDB` used to persist sensitive data?
- postMessage handling — all `postMessage` senders and `addEventListener('message', ...)` listeners. Is `event.origin` validated? Is the target origin specified (not `*`)?
- XSS vectors — `dangerouslySetInnerHTML`, `innerHTML`, `outerHTML`, `document.write`, `eval()`, `new Function()`, `setTimeout/setInterval` with string args. Are user inputs or URL params rendered without sanitization?
- iframe security — are iframes created with `sandbox` and `allow` attributes? Can the app itself be framed (clickjacking)?
- Session tokens in URLs — tokens passed via query params are exposed in browser history, Referrer headers, and server logs
- Open redirect risks — are redirect targets validated? Are `window.location`, `router.push`, or `redirect()` targets controlled by user input?
- Logging — `console.log/error/warn` calls that include tokens, PII, or raw error objects

**For full-stack repos**, combine both checklists.

**For mobile BFF repos**, tell the agent to find:
- BFF passthrough patterns — how backend responses are transformed (or not) before returning to mobile clients
- Whether backend error responses are forwarded directly (leaking internal service URLs, stack traces, infrastructure details)
- SQS/event queue payloads — what data is included (user IDs, tokens, request/response bodies, IP addresses, device IDs)
- Whether full JWT tokens are persisted in message queues or audit logs
- PII redaction in logging — which controllers use `sensitiveInfoReplacer` and which don't
- File upload handling (multer) — size limits, type validation, path traversal on filenames
- In-memory caching of secrets, config, or user data
- Redis usage — what's stored (rate limit counters, tokens, session data?), TLS configuration
- How auth tokens flow: received from mobile → verified → forwarded to backends (is the original JWT re-used or a service token substituted?)

Ask it to return: entity/state inventory with sensitive fields, data flow assessment, XSS/postMessage/iframe findings, storage risks, logging risks.

### Agent 4: Dependencies, Config, and Infrastructure

Tell the agent to find:
- All dependencies and their versions (from the build file)
- Known vulnerable libraries (check for outdated major versions)
- Security-relevant config (TLS, timeouts, actuator/debug endpoints, error handling)
- Dockerfile and k8s manifests (base images, exposed ports, privileges)
- Error handling — do error responses leak internal details?
- Health/debug/admin endpoints and their protection
- CI/CD pipeline security (secrets in workflows, skipped tests)

**Additional for frontend repos:**
- Framework config security: `next.config.mjs` (headers, CSP, strict mode, removeConsole), `vite.config.ts`, `angular.json`
- Edge/CDN config: Wrangler TOML (Cloudflare), `vercel.json`, `netlify.toml` — are security headers set?
- Error pages — do they expose internal details (API URLs, stack traces, env values)?
- Console logging — does the build strip `console.*` in production? Are raw error objects logged in non-production?
- ESLint/linter security rules — is `eslint-plugin-security` or equivalent configured?
- Dev-only flags — `--inspect`, debug modes, development-only env vars leaking into production builds
- Dual lockfiles — both `package-lock.json` and `yarn.lock` present (dependency drift risk)

**Additional for mobile BFF repos:**
- Express-level security: `helmet`, body size limits on `express.json()` and `urlencoded()`, server request timeouts
- `trust proxy` configuration and `x-powered-by` header
- Mobile-specific timeouts: network handshake timeout, server response timeout, per-service overrides
- Datadog/APM tracing configuration (`dd-trace`) — does it capture sensitive data?
- Dual lockfile check (package-lock.json + yarn.lock)
- Docker: base image pinning, non-root user, init process (tini), multi-stage build
- CI/CD: staging auto-deploy on push to master (no approval gate?), secrets in workflows, OIDC for AWS
- Generated code: are generated route schemas validated or trusted blindly?
- TypeScript strict mode, `noImplicitAny`, ESLint security rules

Ask it to return: dependency table with version concerns, config issues, infra findings, error handling assessment.

## Step 3: Classify Findings

After all 4 agents return, classify every finding using STRIDE:

| Category | Question |
|----------|----------|
| **Spoofing** | Can an attacker pretend to be someone else? (missing/weak auth, forged tokens) |
| **Tampering** | Can data be modified without detection? (missing integrity checks, unsigned tokens, no input validation) |
| **Repudiation** | Can actions be performed without traceability? (missing audit logs, tokens in logs) |
| **Information Disclosure** | Can sensitive data leak? (unencrypted storage, verbose errors, PII in logs) |
| **Denial of Service** | Can the service be disrupted? (no rate limiting, long timeouts, resource exhaustion) |
| **Elevation of Privilege** | Can an attacker gain higher access? (broken authorization, header-only auth, missing RBAC) |

Assign severity to each finding:

| Severity | Criteria |
|----------|----------|
| **CRITICAL** | Exploitable without authentication, leads to full compromise or data breach |
| **HIGH** | Significant security gap, requires minimal prerequisites to exploit |
| **MEDIUM** | Real risk but requires specific conditions or has partial mitigations |
| **LOW** | Minor concern, defense-in-depth improvement |

## Step 4: Generate the Report

Output the report directly in chat following the template in [report-template.md](report-template.md).

Fill in every section. Use tables for structured data. Use mermaid diagrams for architecture/data flow where helpful.

For the recommendations section, sort by severity (Critical first) and provide actionable fixes.

## Detailed Scanning Checklists

For the complete list of patterns, file names, and grep queries each agent should use, see [scanner-checklists.md](scanner-checklists.md).
