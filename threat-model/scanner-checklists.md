# Scanner Checklists

Detailed patterns and search queries for each scanning agent. The agent should adapt based on the detected tech stack.

---

## Agent 1: API Surface and Authentication

### Find Endpoints

**Java/Spring Boot:**
- Glob: `**/*Controller.java`, `**/*Resource.java`, `**/*Api.java`
- Grep: `@RestController`, `@Controller`, `@RequestMapping`, `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`
- Grep: `@PathVariable`, `@RequestParam`, `@RequestBody`

**Node.js/Express:**
- Glob: `**/routes/**`, `**/controllers/**`, `**/*.routes.ts`, `**/*.routes.js`
- Grep: `router.get`, `router.post`, `router.put`, `router.delete`, `app.get`, `app.post`
- Grep: `@Get(`, `@Post(`, `@Put(`, `@Delete(` (NestJS)

**Python/Django/Flask/FastAPI:**
- Glob: `**/views.py`, `**/urls.py`, `**/routes.py`, `**/*_view.py`
- Grep: `@app.route`, `@router.get`, `@router.post`, `def get(`, `def post(`
- Grep: `path(`, `re_path(`, `url(`

**Go:**
- Grep: `HandleFunc`, `Handle(`, `r.GET`, `r.POST`, `e.GET`, `e.POST`

### Find Authentication

**Spring Boot:**
- Glob: `**/*SecurityConfig*`, `**/*WebSecurityConfig*`, `**/*AuthConfig*`
- Grep: `WebSecurityConfigurerAdapter`, `SecurityFilterChain`, `@EnableWebSecurity`
- Grep: `@PreAuthorize`, `@Secured`, `@RolesAllowed`, `hasRole`, `hasAuthority`
- Grep: `OncePerRequestFilter`, `HandlerInterceptor`, `addInterceptors`
- Grep: `UsernamePasswordAuthenticationToken`, `JwtDecoder`, `OAuth2`

**Node.js:**
- Grep: `passport`, `jsonwebtoken`, `jwt.verify`, `jwt.sign`
- Grep: `middleware`, `isAuthenticated`, `isAuthorized`, `requireAuth`
- Grep: `express-jwt`, `passport-jwt`, `@UseGuards` (NestJS)

**Python:**
- Grep: `@login_required`, `@permission_required`, `IsAuthenticated`
- Grep: `@jwt_required`, `verify_jwt`, `decode_token`
- Grep: `AUTHENTICATION_BACKENDS`, `REST_FRAMEWORK.*authentication`

**General (any stack):**
- Grep: `Bearer`, `Authorization`, `X-API-KEY`, `api-key`, `apiKey`
- Grep: `token`, `session`, `cookie`

### Find Input Validation

**Spring Boot:**
- Grep: `@Valid`, `@Validated`, `@NotNull`, `@NotBlank`, `@NotEmpty`
- Grep: `@Size`, `@Min`, `@Max`, `@Pattern`, `@Email`, `@Range`
- Grep: `BindingResult`, `MethodArgumentNotValidException`
- Grep: `ConstraintValidator`

**Node.js:**
- Grep: `Joi.`, `yup.`, `zod.`, `.validate(`, `.parse(`
- Grep: `express-validator`, `class-validator`, `@IsString`, `@IsEmail`

**Python:**
- Grep: `serializer`, `Serializer`, `Form`, `ModelForm`
- Grep: `pydantic`, `BaseModel`, `Field(`, `validator`

### Find Rate Limiting

- Grep: `RateLimiter`, `rate.limit`, `throttle`, `Bucket4j`, `Resilience4j`
- Grep: `express-rate-limit`, `slowDown`, `@Throttle`
- Grep: `ratelimit`, `throttle_classes`

### Find CORS/CSRF

- Grep: `cors`, `CORS`, `CorsConfiguration`, `@CrossOrigin`, `Access-Control`
- Grep: `csrf`, `CSRF`, `csrfToken`, `_csrf`, `csurf`

### Find Endpoints — Frontend (Next.js / React / Vue / Svelte)

**Next.js (App Router):**
- Glob: `**/app/**/page.tsx`, `**/app/**/page.jsx`, `**/app/**/page.ts`
- Glob: `**/app/**/layout.tsx`, `**/app/**/loading.tsx`, `**/app/**/error.tsx`
- Glob: `**/app/**/route.ts`, `**/app/**/route.js` (API routes)
- Grep: `'use server'` (server actions)
- Grep: `export const runtime` (edge vs nodejs runtime)

**Next.js (Pages Router):**
- Glob: `**/pages/**/*.tsx`, `**/pages/**/*.jsx`
- Grep: `getServerSideProps`, `getStaticProps`, `getInitialProps`
- Glob: `**/pages/api/**` (API routes)

**React Router / Vue Router / SvelteKit:**
- Grep: `createBrowserRouter`, `createHashRouter`, `Route`, `Routes`
- Grep: `createRouter`, `defineRoute`
- Glob: `**/routes/**`, `**/+page.svelte`, `**/+server.ts`

### Find Authentication — Frontend

- Grep: `sessionToken`, `accessToken`, `access_token`, `authToken`, `idToken`
- Grep: `searchParams.get`, `useSearchParams`, `URLSearchParams` (tokens from URL)
- Grep: `Bearer`, `Authorization`
- Grep: `sessionStorage`, `localStorage` (token persistence)
- Grep: `useSession`, `useAuth`, `signIn`, `signOut` (auth hooks)
- Grep: `cookie`, `setCookie`, `getCookie`, `js-cookie`, `nookies`

### Find Input Validation — Frontend

- Grep: `searchParams.get`, `useSearchParams`, `router.query` (URL param usage without validation)
- Grep: `encodeURIComponent`, `decodeURIComponent`, `decodeURI` (encoding presence or absence)
- Grep: `JSON.parse` (unvalidated parse of external input)
- Grep: `Base64.decode`, `atob`, `btoa` (encoded data handling)

### Find Security Headers — Frontend

- Grep: `Content-Security-Policy`, `CSP`, `frame-ancestors`, `X-Frame-Options`
- Grep: `Strict-Transport-Security`, `HSTS`
- Grep: `X-Content-Type-Options`, `X-XSS-Protection`, `Referrer-Policy`
- Grep: `helmet`, `securityHeaders`, `headers()` (in next.config or middleware)
- Glob: `**/middleware.ts`, `**/middleware.js` (Next.js middleware)

---

## Agent 2: External Integrations and Secrets

### Find External Clients

**Java:**
- Grep: `RestTemplate`, `WebClient`, `FeignClient`, `@FeignClient`
- Grep: `HttpClient`, `OkHttpClient`, `CloseableHttpClient`
- Grep: `new URL(`, `HttpURLConnection`

**Node.js:**
- Grep: `axios`, `fetch(`, `node-fetch`, `got(`, `request(`
- Grep: `HttpService`, `@HttpModule`

**Python:**
- Grep: `requests.get`, `requests.post`, `httpx`, `aiohttp`
- Grep: `urllib`, `http.client`

**General:**
- Grep: `https://`, `http://` (in non-test, non-doc files)
- Grep: `baseUrl`, `base_url`, `BASE_URL`, `apiUrl`, `API_URL`

### Find Secrets and Credentials

**Hardcoded secrets:**
- Grep: `password\s*=\s*"`, `secret\s*=\s*"`, `api_key\s*=\s*"`
- Grep: `PASSWORD`, `SECRET_KEY`, `API_KEY`, `PRIVATE_KEY`, `ACCESS_KEY`
- Grep: `-----BEGIN`, `-----BEGIN RSA`, `-----BEGIN PRIVATE`

**Config files with defaults:**
- Grep: `\$\{.*:.*\}` (Spring Boot env var with fallback defaults)
- Grep: `default`, `changeme`, `password123`, `secret`, `TODO`
- Glob: `**/.env`, `**/.env.*`, `**/secrets.*`, `**/credentials.*`

**Secrets management:**
- Grep: `SecretsManager`, `Vault`, `vault`, `KMS`, `ParameterStore`
- Grep: `VAULT_`, `AWS_SECRET`, `SecretClient`
- Grep: `dotenv`, `config()`, `process.env`, `os.environ`, `System.getenv`

### Find Encryption

- Grep: `encrypt`, `decrypt`, `Cipher`, `AES`, `RSA`, `HMAC`
- Grep: `bcrypt`, `scrypt`, `argon2`, `PBKDF2`, `hash(`
- Grep: `KmsClient`, `aws-encryption-sdk`, `CryptoService`
- Grep: `AttributeConverter`, `@Convert`, `EncryptedString`
- Grep: `ssl`, `tls`, `truststore`, `keystore`, `certificate`

### Find PII Handling

- Grep: `mask`, `redact`, `sanitize`, `anonymize`, `obfuscate`
- Grep: `accountNumber`, `account_number`, `routingNumber`, `routing_number`
- Grep: `ssn`, `SSN`, `socialSecurity`, `social_security`
- Grep: `dateOfBirth`, `date_of_birth`, `dob`
- Grep: `rawData`, `raw_data`, `includeRawData`

### Find Client-Exposed Env Vars — Frontend

**Next.js:**
- Grep: `NEXT_PUBLIC_` across all source files (these are inlined into the client bundle)
- Grep: `NEXT_SERVER_` (should be server-only; verify they aren't in client code)

**Vite:**
- Grep: `VITE_` (client-exposed), `import.meta.env`

**Create React App:**
- Grep: `REACT_APP_` (client-exposed)

**General:**
- Grep: `process.env` in client-side files (may be replaced at build time)
- Check: are any `_TOKEN`, `_SECRET`, `_KEY`, `_PASSWORD` env vars exposed to the client via public prefix?
- Glob: `**/.env`, `**/.env.*` — check `.gitignore` covers them

### Find OAuth Flows — Frontend

- Grep: `client_id`, `client_secret`, `redirect_uri`, `response_type`, `authorization_code`
- Grep: `OAuth`, `oauth`, `openid`, `scope`
- Grep: `state` parameter usage in OAuth redirects — is it signed, encrypted, or just encoded?
- Check: does the OAuth `state` carry sensitive data (session tokens, user IDs)?
- Check: is `client_secret` kept server-side or exposed to the client?

### Find Third-Party SDK Integrations — Frontend

- Grep: `plaid`, `Plaid`, `usePlaidLink`, `react-plaid-link`
- Grep: `stripe`, `Stripe`, `loadStripe`, `@stripe/stripe-js`
- Grep: `finicity`, `Finicity`, `Connect.launch`, `connect-web-sdk`
- Grep: `akoya`, `Akoya`
- Grep: `firebase`, `Firebase`, `initializeApp`
- Grep: `auth0`, `Auth0`, `@auth0/auth0-react`
- Check: how are link tokens / connect URLs obtained? Are secrets kept server-side?

---

## Agent 3: Persistence and Data Flow

### Find Database Entities

**JPA/Hibernate:**
- Grep: `@Entity`, `@Table`, `@Column`, `@Id`
- Grep: `@ManyToOne`, `@OneToMany`, `@ManyToMany`, `@OneToOne`
- Glob: `**/entity/**`, `**/model/**`, `**/domain/**`

**Sequelize/TypeORM/Prisma (Node.js):**
- Grep: `@Entity()`, `@Column()`, `@PrimaryGeneratedColumn`
- Grep: `Model.init`, `sequelize.define`, `DataTypes.`
- Glob: `**/prisma/schema.prisma`, `**/models/**`

**Django:**
- Grep: `models.Model`, `models.CharField`, `models.TextField`
- Glob: `**/models.py`, `**/models/**`

**SQLAlchemy (Python):**
- Grep: `Base = declarative_base`, `Column(`, `relationship(`

### Find SQL Injection Risks

- Grep: `nativeQuery\s*=\s*true`, `@Query.*nativeQuery`
- Grep: `createNativeQuery`, `createSQLQuery`
- Grep: `execute(`, `raw(`, `rawQuery`, `query(`
- Grep: `String.format.*SELECT`, `"SELECT.*" \+`, `f"SELECT`, `f'SELECT`
- Grep: `connection.execute`, `cursor.execute`
- Look for string concatenation in any SQL query (vs parameterized queries)

### Find Message Queues

- Grep: `SQS`, `sqs`, `SqsClient`, `sendMessage`, `receiveMessage`
- Grep: `Kafka`, `kafka`, `KafkaTemplate`, `KafkaListener`, `@KafkaListener`
- Grep: `RabbitMQ`, `rabbit`, `amqp`, `@RabbitListener`, `AmqpTemplate`
- Grep: `SNS`, `sns`, `publish`, `subscribe`
- Grep: `EventBridge`, `eventbridge`
- Check: what data is in message payloads? Are tokens/PII included?

### Find Caching

- Grep: `@Cacheable`, `@CachePut`, `@CacheEvict`, `RedisTemplate`
- Grep: `redis`, `Redis`, `cache`, `Cache`, `memcached`
- Grep: `node-cache`, `lru-cache`, `ioredis`
- Check: what data is cached? Are secrets or tokens cached? TTLs?

### Find Logging Risks

- Grep: `log.info`, `log.debug`, `log.error`, `logger.info`, `logger.debug`
- Grep: `console.log`, `print(`, `println`
- Grep patterns near sensitive data: `token`, `password`, `secret`, `account`
- Grep: `MASKING_FIELDS`, `mask`, `redact`, `@JsonIgnore`
- Grep: logback config `logback.xml`, `logback-spring.xml`, `log4j2.xml`
- Check: are request/response bodies logged? Do audit tables store tokens?

### Find Client-Side State — Frontend

**Zustand:**
- Grep: `create(`, `zustand`, `useStore`, `persist`, `createJSONStorage`
- Check: what data is stored? Is `persist` used with `sessionStorage` or `localStorage`?

**Redux:**
- Grep: `createStore`, `configureStore`, `createSlice`, `redux-persist`
- Glob: `**/store/**`, `**/slices/**`, `**/reducers/**`

**MobX / Pinia / Context:**
- Grep: `makeObservable`, `defineStore`, `createContext`, `useContext`

**General:**
- Check: are session tokens, user IDs, or PII stored in client state?
- Check: can store state be accessed from browser DevTools or injected scripts?

### Find Browser Storage — Frontend

- Grep: `localStorage`, `sessionStorage`, `IndexedDB`, `openDatabase`
- Grep: `window.localStorage`, `window.sessionStorage`
- Grep: `setItem`, `getItem`, `removeItem` (direct storage API calls)
- Grep: `createJSONStorage`, `redux-persist` (framework-level persistence)
- Check: what sensitive data is persisted? Is it encrypted?

### Find postMessage Security — Frontend

- Grep: `postMessage`, `addEventListener.*message`, `window.addEventListener`
- Grep: `MessageEvent`, `event.origin`, `event.source`, `event.data`
- Grep: `document.referrer` (used as postMessage target origin)
- Check: do message listeners validate `event.origin` against an allowlist?
- Check: is postMessage sent with a specific target origin (not `*`)?
- Check: is `document.referrer` used to determine parent origin (can be spoofed or absent)?

### Find XSS Vectors — Frontend

- Grep: `dangerouslySetInnerHTML`, `innerHTML`, `outerHTML`
- Grep: `document.write`, `document.writeln`
- Grep: `eval(`, `new Function(`, `setTimeout` and `setInterval` with string arguments
- Grep: `v-html` (Vue), `[innerHTML]` (Angular), `{@html` (Svelte)
- Check: are user inputs or URL params rendered without React/framework escaping?
- Check: are API responses containing HTML rendered without sanitization?

### Find iframe Security — Frontend

- Grep: `createElement('iframe')`, `<iframe`, `iframe.src`
- Grep: `sandbox`, `allow=`, `allowfullscreen`
- Check: are iframes created with `sandbox` attribute restricting capabilities?
- Check: is the iframe `src` built from trusted sources only?
- Check: are `X-Frame-Options` or CSP `frame-ancestors` set to prevent clickjacking of the app itself?

### Find Open Redirect Risks — Frontend

- Grep: `window.location`, `window.location.href`, `window.location.replace`
- Grep: `router.push`, `router.replace`, `navigate(`, `redirect(`
- Grep: `window.open`
- Check: are redirect targets hardcoded/allowlisted or derived from user input?
- Check: does the callback/redirect page validate the target URL?

---

## Agent 4: Dependencies, Config, and Infrastructure

### Find Dependencies

**Maven:** Read `pom.xml` — list all `<dependency>` entries with `<groupId>`, `<artifactId>`, `<version>`
**Gradle:** Read `build.gradle` or `build.gradle.kts` — list all `implementation`, `api`, `compile` entries
**npm:** Read `package.json` — list `dependencies` and `devDependencies`
**pip:** Read `requirements.txt`, `pyproject.toml`, or `setup.py`
**Go:** Read `go.mod`

Flag libraries known to have frequent CVEs or that are very outdated:
- Java: commons-collections (<3.2.2), log4j (<2.17), Jackson, Guava, Spring
- Node: lodash, minimist, express (very old), jsonwebtoken
- Python: Django, Flask, requests, PyYAML, Jinja2

### Find Security Config

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

### Find Infrastructure

**Docker:**
- Glob: `**/Dockerfile*`, `**/docker-compose*`
- Check: base image (is it minimal? pinned version?), runs as root?, exposed ports, secrets in build args

**Kubernetes:**
- Glob: `**/*.yaml`, `**/*.yml` in k8s/, deploy/, charts/
- Check: securityContext, runAsNonRoot, resource limits, network policies

**CI/CD:**
- Glob: `.github/workflows/*.yml`, `Jenkinsfile`, `.gitlab-ci.yml`
- Check: secrets in env, skipped tests, force pushes, unreviewed deploys

### Find Error Handling

- Grep: `@ExceptionHandler`, `@ControllerAdvice`, `ErrorController`
- Grep: `getLocalizedMessage`, `getMessage`, `getStackTrace`, `printStackTrace`
- Grep: `errorHandler`, `error middleware`, `catch(`, `except`
- Check: do error responses expose internal details (SQL errors, stack traces, file paths)?

### Find Admin/Debug Endpoints

- Grep: `actuator`, `/health`, `/metrics`, `/info`, `/env`, `/heapdump`, `/threaddump`
- Grep: `swagger`, `/swagger-ui`, `/api-docs`, `/graphql`, `/graphiql`
- Grep: `backdoor`, `/debug`, `/admin`, `/internal`, `/qa`, `/test`
- Check: are these endpoints authenticated? Restricted by profile/environment?

### Find Framework Config — Frontend

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

### Find Edge/CDN Config — Frontend

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

### Find ESLint Security Rules — Frontend

- Read `.eslintrc.json` or `.eslintrc.js` or `eslint.config.js`
- Grep: `eslint-plugin-security`, `no-eval`, `no-implied-eval`, `no-new-func`
- Check: are security-focused rules enabled?

### Find Dev-Only Flags — Frontend

- Grep: `--inspect` (Node.js debugger, should be dev-only)
- Grep: `NODE_OPTIONS` in scripts
- Grep: `sourceMap`, `source-map` in build config (should be disabled in production)
- Check: are there development-only env vars or flags that could leak into production builds?

### Find Lockfile Issues — Frontend

- Check: are both `package-lock.json` AND `yarn.lock` present? (pick one to avoid drift)
- Check: is the lockfile committed to the repo?
