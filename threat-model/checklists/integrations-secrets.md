# External Integrations and Secrets Checklist

Patterns and search queries for scanning external service clients, credentials, secrets management, encryption, and PII handling. Adapt based on the detected tech stack.

---

## Find External Clients

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

## Find Secrets and Credentials

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

## Find Encryption

- Grep: `encrypt`, `decrypt`, `Cipher`, `AES`, `RSA`, `HMAC`
- Grep: `bcrypt`, `scrypt`, `argon2`, `PBKDF2`, `hash(`
- Grep: `KmsClient`, `aws-encryption-sdk`, `CryptoService`
- Grep: `AttributeConverter`, `@Convert`, `EncryptedString`
- Grep: `ssl`, `tls`, `truststore`, `keystore`, `certificate`

## Find PII Handling

- Grep: `mask`, `redact`, `sanitize`, `anonymize`, `obfuscate`
- Grep: `accountNumber`, `account_number`, `routingNumber`, `routing_number`
- Grep: `ssn`, `SSN`, `socialSecurity`, `social_security`
- Grep: `dateOfBirth`, `date_of_birth`, `dob`
- Grep: `rawData`, `raw_data`, `includeRawData`

## Find Client-Exposed Env Vars — Frontend

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

## Find Mobile BFF Credential Forwarding

**Backend service clients:**
- Glob: `**/services/**`, `**/*API.ts`, `**/*Api.ts`, `**/*Service.ts`
- Grep: `axios.create`, `axios.get`, `axios.post`, `axios.put`, `axios.delete`
- Check: how are credentials forwarded to backend services?
  - Custom headers (e.g. `X-User-Id`, `X-API-KEY`, `Authorization`)
  - Is the original user JWT re-sent, or is a service-to-service token used?
  - Are backend URLs hardcoded per environment or configurable?

**Internal vs external service URLs:**
- Grep: `http://` in service files (internal mesh services often use plain HTTP)
- Grep: `https://` (external-facing services)
- Check: are internal HTTP calls protected by service mesh TLS (Istio, Linkerd)?
- Check: do staging/dev URLs point to production services accidentally?

**Push notification tokens:**
- Grep: `push-notification`, `pushToken`, `push_token`, `fcm`, `apns`, `sendbird`
- Check: how are push tokens registered and removed
- Check: is token removal a public endpoint (for logout flows)?

**Payment provisioning:**
- Grep: `apple-pay`, `google-pay`, `push-provisioning`, `encryptedPassData`, `ephemeralPublicKey`
- Check: are payment provisioning tokens handled as passthrough or processed by the BFF?
- Check: are Apple/Google Pay activation data cached or logged?

**Widget JWT issuance:**
- Grep: `widget/token`, `WIDGET_TOKEN`, `widgetJwt`
- Check: what data goes into the widget JWT payload (userId, scopes, expiry)
- Check: are widget endpoints properly scoped to read-only operations?

## Find OAuth Flows — Frontend

- Grep: `client_id`, `client_secret`, `redirect_uri`, `response_type`, `authorization_code`
- Grep: `OAuth`, `oauth`, `openid`, `scope`
- Grep: `state` parameter usage in OAuth redirects — is it signed, encrypted, or just encoded?
- Check: does the OAuth `state` carry sensitive data (session tokens, user IDs)?
- Check: is `client_secret` kept server-side or exposed to the client?

## Find Third-Party SDK Integrations — Frontend

- Grep: `plaid`, `Plaid`, `usePlaidLink`, `react-plaid-link`
- Grep: `stripe`, `Stripe`, `loadStripe`, `@stripe/stripe-js`
- Grep: `finicity`, `Finicity`, `Connect.launch`, `connect-web-sdk`
- Grep: `akoya`, `Akoya`
- Grep: `firebase`, `Firebase`, `initializeApp`
- Grep: `auth0`, `Auth0`, `@auth0/auth0-react`
- Check: how are link tokens / connect URLs obtained? Are secrets kept server-side?
