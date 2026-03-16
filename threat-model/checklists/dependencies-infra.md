# Dependencies, Config, and Infrastructure Checklist

Patterns and search queries for scanning dependencies, security configuration, infrastructure, and error handling. Adapt based on the detected tech stack.

---

## Find Dependencies

**Maven:** Read `pom.xml` — list all `<dependency>` entries with `<groupId>`, `<artifactId>`, `<version>`
**Gradle:** Read `build.gradle` or `build.gradle.kts` — list all `implementation`, `api`, `compile` entries
**npm:** Read `package.json` — list `dependencies` and `devDependencies`
**pip:** Read `requirements.txt`, `pyproject.toml`, or `setup.py`
**Go:** Read `go.mod`
**Rust:** Read `Cargo.toml` — list all `[dependencies]`
**.NET:** Read `*.csproj` — list all `<PackageReference>` entries

Flag libraries known to have frequent CVEs or that are very outdated:
- Java: commons-collections (<3.2.2), log4j (<2.17), Jackson, Guava, Spring
- Node: lodash, minimist, express (very old), jsonwebtoken
- Python: Django, Flask, requests, PyYAML, Jinja2
- Go: older versions of `golang.org/x/crypto`, `golang.org/x/net`
- Rust: `hyper` (<1.0 for certain CVEs), `regex` (ReDoS in older versions)
- .NET: `Newtonsoft.Json` with `TypeNameHandling`, `System.Web` (legacy)

**Flag unsafe deserialization libraries (any version):**
- Java: `commons-collections` (gadget chains), `XStream` (default config allows RCE), `SnakeYAML` (arbitrary constructors)
- Python: `PyYAML` (if `yaml.load` used without SafeLoader), `jsonpickle`, `dill`
- Node: `node-serialize` (inherently unsafe), `js-yaml` (check schema config)
- Ruby: `Oj` (check mode), `Psych` (check safe_load usage)
- .NET: `BinaryFormatter` (deprecated, inherently unsafe), `JavaScriptSerializer`

## Find Security Config

**Spring Boot:**
- Glob: `**/application.yml`, `**/application.properties`, `**/application-*.yml`, `**/bootstrap.yml`
- Grep: `management.endpoints`, `actuator`, `exposure.include`
- Grep: `server.ssl`, `server.port`, `server.error`
- Grep: `spring.security`, `spring.datasource`

**Node.js:**
- Glob: `**/config/**`, `**/.env.example`
- Grep: `helmet`, `hpp`, `csp`, `contentSecurityPolicy`

**General:**
- Grep: `debug`, `DEBUG`, `trace`, `TRACE`, `development`, `dev`
- Grep: `swagger`, `openapi`, `api-docs`
- Grep: `actuator`, `health`, `metrics`, `info`, `env`, `heapdump`

## Find Infrastructure

**Docker:**
- Glob: `**/Dockerfile*`, `**/docker-compose*`
- Check: base image (is it minimal? pinned version?), runs as root?, exposed ports, secrets in build args

**Kubernetes:**
- Glob: `**/*.yaml`, `**/*.yml` in k8s/, deploy/, charts/
- Check: securityContext, runAsNonRoot, resource limits, network policies

**CI/CD:**
- Glob: `.github/workflows/*.yml`, `Jenkinsfile`, `.gitlab-ci.yml`
- Check: secrets in env, skipped tests, force pushes, unreviewed deploys

## Find Error Handling

- Grep: `@ExceptionHandler`, `@ControllerAdvice`, `ErrorController`
- Grep: `getLocalizedMessage`, `getMessage`, `getStackTrace`, `printStackTrace`
- Grep: `errorHandler`, `error middleware`, `catch(`, `except`
- Check: do error responses expose internal details (SQL errors, stack traces, file paths)?

## Find Mobile BFF Security Config

**Express-level security:**
- Grep: `helmet`, `hpp`, `csp`, `contentSecurityPolicy` — are security header middlewares present?
- Grep: `express.json`, `express.urlencoded` — are body size `limit` options set?
- Grep: `trust proxy` — is `trust proxy` enabled? (needed for correct `req.ip` behind load balancers)
- Grep: `x-powered-by` — is it disabled?
- Grep: `server.timeout`, `server.requestTimeout`, `server.keepAliveTimeout` — are server-level timeouts set?

**Timeout configuration:**
- Grep: `NETWORK_SERVER_TIMEOUT`, `NETWORK_HANDSHAKE_TIMEOUT`, `timeout`
- Check: default timeout values and per-service overrides
- Check: are there endpoints with very long timeouts that could be abused?

**APM / tracing:**
- Grep: `dd-trace`, `datadog`, `newrelic`, `opentelemetry`, `apm`
- Check: does tracing capture request/response bodies? Are sensitive headers excluded from traces?

**Generated routes:**
- Glob: `**/_generated/**`, `**/*.g.ts`
- Check: are generated route schemas treated as trusted? Is there validation on the generated config?
- Check: do generated routes properly set userId from token (e.g. `req.params.userId = res.locals.userId`)?

## Find Admin/Debug Endpoints

- Grep: `actuator`, `/health`, `/metrics`, `/info`, `/env`, `/heapdump`, `/threaddump`
- Grep: `swagger`, `/swagger-ui`, `/api-docs`, `/graphql`, `/graphiql`
- Grep: `backdoor`, `/debug`, `/admin`, `/internal`, `/qa`, `/test`
- Check: are these endpoints authenticated? Restricted by profile/environment?

## Find Framework Config — Frontend

**Next.js:**
- Read `next.config.mjs` or `next.config.js`:
  - `reactStrictMode` — should be `true`; `false` weakens dev-time safety checks
  - `headers()` — are security headers (CSP, HSTS, X-Frame-Options) configured?
  - `removeConsole` — is it enabled for production?
  - `images.remotePatterns` — are allowed image domains appropriate?
  - `poweredByHeader` — should be `false` to hide `X-Powered-By: Next.js`
- Glob: `**/middleware.ts` — does middleware set security headers or validate auth?

**Vite:**
- Read `vite.config.ts` — `server.proxy`, `define` (env replacement), plugins

**Angular:**
- Read `angular.json` — `budgets`, `outputHashing`, `sourceMap` (should be false in prod)

## Find Edge/CDN Config — Frontend

**Cloudflare:**
- Glob: `**/wrangler*.toml`
- Check: are security headers set in `[env.production.vars]` or via Workers?
- Check: compatibility flags, pages config

**Vercel:**
- Read `vercel.json` — `headers`, `rewrites`, `redirects`
- Check: are security headers configured?

**Netlify:**
- Read `netlify.toml` — `[[headers]]` blocks
- Read `_headers` file

## Find ESLint Security Rules — Frontend

- Read `.eslintrc.json` or `.eslintrc.js` or `eslint.config.js`
- Grep: `eslint-plugin-security`, `no-eval`, `no-implied-eval`, `no-new-func`
- Check: are security-focused rules enabled?

## Find Dev-Only Flags — Frontend

- Grep: `--inspect` (Node.js debugger, should be dev-only)
- Grep: `NODE_OPTIONS` in scripts
- Grep: `sourceMap`, `source-map` in build config (should be disabled in production)
- Check: are there development-only env vars or flags that could leak into production builds?

## Find Lockfile Issues — Frontend

- Check: are both `package-lock.json` AND `yarn.lock` present? (pick one to avoid drift)
- Check: is the lockfile committed to the repo?
