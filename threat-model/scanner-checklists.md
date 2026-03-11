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
