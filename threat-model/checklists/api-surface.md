# API Surface and Authentication Checklist

Patterns and search queries for scanning API endpoints, authentication, authorization, input validation, and SSRF vectors. Adapt based on the detected tech stack.

---

## Find Endpoints

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

## Find Authentication

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

## Find Input Validation

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

## Find Rate Limiting

- Grep: `RateLimiter`, `rate.limit`, `throttle`, `Bucket4j`, `Resilience4j`
- Grep: `express-rate-limit`, `slowDown`, `@Throttle`
- Grep: `ratelimit`, `throttle_classes`

## Find CORS/CSRF

- Grep: `cors`, `CORS`, `CorsConfiguration`, `@CrossOrigin`, `Access-Control`
- Grep: `csrf`, `CSRF`, `csrfToken`, `_csrf`, `csurf`

## Find SSRF Vectors

SSRF occurs when endpoints accept URLs, hostnames, or IP addresses from user input and the server makes requests to those targets.

**Endpoint parameters accepting URLs:**
- Grep: `url`, `uri`, `endpoint`, `host`, `target`, `redirect`, `callback`, `webhook`, `proxy`, `fetch` in `@RequestParam`, `@RequestBody`, `req.body`, `req.query`, `request.args`
- Grep: `URL(`, `URI(`, `new URL`, `new URI`, `urllib.parse`, `urlparse`

**Server-side requests with user-controlled destinations:**
- Grep: `RestTemplate`, `WebClient`, `HttpClient`, `OkHttpClient` used with variable URLs (not hardcoded)
- Grep: `axios(`, `fetch(`, `got(`, `requests.get(`, `requests.post(` where the URL argument is a variable
- Grep: `redirect:`, `forward:` (Spring MVC server-side redirects)

**Checks:**
- Are URL inputs validated against a strict allowlist of domains/schemes?
- Can `file://`, `gopher://`, `dict://`, or `http://169.254.169.254` (cloud metadata) be reached?
- Are internal network addresses (10.x, 172.16.x, 192.168.x, 127.0.0.1) blocked?
- Do URL parsers handle DNS rebinding or redirect chains?

## Find Endpoints — Frontend (Next.js / React / Vue / Svelte)

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

## Find Authentication — Frontend

- Grep: `sessionToken`, `accessToken`, `access_token`, `authToken`, `idToken`
- Grep: `searchParams.get`, `useSearchParams`, `URLSearchParams` (tokens from URL)
- Grep: `Bearer`, `Authorization`
- Grep: `sessionStorage`, `localStorage` (token persistence)
- Grep: `useSession`, `useAuth`, `signIn`, `signOut` (auth hooks)
- Grep: `cookie`, `setCookie`, `getCookie`, `js-cookie`, `nookies`

## Find Input Validation — Frontend

- Grep: `searchParams.get`, `useSearchParams`, `router.query` (URL param usage without validation)
- Grep: `encodeURIComponent`, `decodeURIComponent`, `decodeURI` (encoding presence or absence)
- Grep: `JSON.parse` (unvalidated parse of external input)
- Grep: `Base64.decode`, `atob`, `btoa` (encoded data handling)

## Find Endpoints — Mobile BFF

**Route registration:**
- Read the main Express/Fastify app file (e.g. `express.ts`, `app.ts`) — find all `app.use()`, `expressApp.use()`, router mounts
- Glob: `**/controllers/**`, `**/*Controller.ts`, `**/*Controller.js`
- Grep: `router.get`, `router.post`, `router.put`, `router.delete`, `router.patch`
- Glob: `**/_generated/**`, `**/*.g.ts`, `**/*.g.js` — generated route schemas
- Check: generated schemas that auto-register endpoints from config (prefix, version, urls per env, endpoints array)

**Public / pre-auth endpoints:**
- Grep: `publicRoutes`, `PUBLIC_ROUTES`, `skipAuth`, `noAuth`
- Check: which endpoints skip JWT verification (registration, MFA challenge, push token cleanup, widget data, device sensors, affiliate/bidding callbacks)
- These are high-value targets — they bypass primary authentication

**IDOR (Insecure Direct Object Reference):**
- Grep: `req.params.userId`, `req.params.id` — where path params are used
- Grep: `res.locals.userId`, `req.userId` — where the authenticated user ID is available
- Check: are `req.params.userId` and `res.locals.userId` compared? If not, any authenticated user can access another user's data
- Check: does the generated controller code force `req.params.userId = res.locals.userId`? If so, which routes use this pattern and which don't?

## Find Authentication — Mobile BFF

**JWT verification:**
- Grep: `jwt.verify`, `jwt.sign`, `jsonwebtoken`, `express-auth`, `express-jwt`
- Find the main auth middleware — what library is used, what fields are set on `res.locals` or `req`
- Check: which token fields are extracted (userId, tokenSessionId, accessToken, roles, scopes)

**MFA middleware:**
- Grep: `mfa`, `MFA`, `mfaMiddleware`, `mfaToken`, `mfaSessionId`
- Check: MFA token subject validation, session ID matching, event type validation
- Check: does the MFA rejection response properly send a body (`.send()` / `.sendStatus()`) or just set status?

**Widget authentication:**
- Grep: `widget`, `WIDGET_TOKEN`, `widgetMiddleware`
- Check: widget JWT issuance endpoint, token subject, expiry, what data is in the payload

**Phone verification tokens:**
- Grep: `verifyPhoneNumber`, `phoneNumberJwt`, `VERIFY_PHONE_NUMBER_TOKEN`
- Check: what fields are validated (verifySessionId, phoneNumber, userId, eventType)

**Request signing (HMAC):**
- Grep: `x-signed-auth`, `HMAC`, `sha256`, `sha.js`, `crypto`
- Read the signature middleware fully — understand the canonical request format
- Check: what headers are included in the signature (method, path, query, body hash, device headers)
- Check: which endpoints are EXCLUDED from signature verification
- Check: is there a deprecated/legacy signing secret still accepted?

**Auth bypass:**
- Grep: `bypass`, `x-bypass-authorization`, `shouldBypass`, `performanceTest`
- Check: is auth bypass guarded by environment (production vs staging)?
- Check: can the bypass header be sent from outside the VPC?

## Find Rate Limiting — Mobile BFF

- Read the rate limiter middleware
- Check: key function — is it per-user (`userId`), per-IP, or per-device (`deviceId`)?
- Check: what happens when no key is available (e.g. public routes with no userId) — are these routes unprotected?
- Check: rate limit values (window, max requests) and whether they're appropriate
- Check: is rate limiting enabled by default or behind a feature flag?

## Find Security Headers — Frontend

- Grep: `Content-Security-Policy`, `CSP`, `frame-ancestors`, `X-Frame-Options`
- Grep: `Strict-Transport-Security`, `HSTS`
- Grep: `X-Content-Type-Options`, `X-XSS-Protection`, `Referrer-Policy`
- Grep: `helmet`, `securityHeaders`, `headers()` (in next.config or middleware)
- Glob: `**/middleware.ts`, `**/middleware.js` (Next.js middleware)
