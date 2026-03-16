# Persistence and Data Flow Checklist

Patterns and search queries for scanning database entities, data flow, caching, logging, deserialization, and client-side state. Adapt based on the detected tech stack.

---

## Find Database Entities

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

## Find Deserialization Risks

Unsafe deserialization of untrusted input can lead to remote code execution.

**Java:**
- Grep: `ObjectInputStream`, `readObject`, `readUnshared`
- Grep: `XMLDecoder`, `XStream`, `fromXML`, `Unmarshaller`
- Grep: `@JsonTypeInfo`, `enableDefaultTyping`, `DefaultTyping` (Jackson polymorphic deserialization — allows arbitrary class instantiation)
- Grep: `SerializationUtils.deserialize`, `IOUtils`
- Check: is `ObjectInputStream` used on any user-controlled input (HTTP body, message queue payload, file upload)?

**Python:**
- Grep: `pickle.loads`, `pickle.load`, `cPickle`, `shelve`, `marshal.loads`
- Grep: `yaml.load` — must use `Loader=SafeLoader` or `yaml.safe_load`; bare `yaml.load` allows arbitrary code execution
- Grep: `jsonpickle.decode`, `dill.loads`
- Check: are any of these called on user-supplied data (request bodies, uploaded files, queue messages)?

**Node.js:**
- Grep: `node-serialize`, `serialize`, `unserialize`, `funcster`
- Grep: `js-yaml` — check if loaded with unsafe schema (`UNSAFE_SCHEMA`, `DEFAULT_FULL_SCHEMA`)
- Grep: `vm.runInNewContext`, `vm.runInThisContext`, `vm.Script` — code execution from strings
- Check: is `JSON.parse` followed by `eval` or `new Function` on parsed data?

**Ruby:**
- Grep: `Marshal.load`, `YAML.load`, `Oj.load`
- Check: use `YAML.safe_load` instead of `YAML.load`

**.NET/C#:**
- Grep: `BinaryFormatter`, `SoapFormatter`, `ObjectStateFormatter`
- Grep: `JavaScriptSerializer`, `DataContractSerializer`, `NetDataContractSerializer`
- Grep: `JsonConvert.DeserializeObject` with `TypeNameHandling` != `None`
- Check: `BinaryFormatter` is inherently unsafe and should never be used with untrusted data

**General checks:**
- Is the deserialized data sourced from user input, external APIs, or message queues?
- Are deserialization libraries configured with allowlists of permitted types?
- Are there integrity checks (signatures, MACs) on serialized data before deserializing?

## Find BFF Passthrough Risks — Mobile BFF

**Error response forwarding:**
- Grep: `error.response.data`, `error.data`, `error.status`
- Grep: `res.status(error`, `res.json(error`
- Check: are backend error responses forwarded directly to mobile clients?
- Check: could backend errors leak internal service URLs, stack traces, infrastructure details?
- Check: is there a global error handler that sanitizes responses for production?

**SQS / event queue data flow:**
- Grep: `SQS`, `sendMessage`, `UserTransaction`, `eventQueue`
- Read the transaction/event middleware fully
- Check payload fields:
  - `tokenId` / `token` — is the full JWT persisted? (high-severity if so)
  - `userId`, `tokenSessionId` — user identifiers
  - `headers` — is `authorization` stripped? What about other sensitive headers?
  - `req` / `resp` body — are these redacted before sending?
  - `ip`, `userAgent`, `deviceId` — PII fields
- Find the whitelist of endpoints that trigger event messages

**Token flow through BFF:**
- Trace: mobile sends Bearer JWT → middleware verifies → `res.locals` populated → controllers call services
- Check: do any service calls re-send the user's JWT as `Authorization: Bearer ${token}`?
- Check: are service-to-service calls authenticated separately from user tokens?

**File upload passthrough:**
- Grep: `multer`, `upload`, `multipart`, `FormData`, `formData`
- Check: file size limits per endpoint
- Check: file type / mimetype validation
- Check: filename sanitization (path traversal in uploaded filenames)
- Check: are uploaded files streamed to backend services or buffered in BFF memory?

## Find SQL Injection Risks

- Grep: `nativeQuery\s*=\s*true`, `@Query.*nativeQuery`
- Grep: `createNativeQuery`, `createSQLQuery`
- Grep: `execute(`, `raw(`, `rawQuery`, `query(`
- Grep: `String.format.*SELECT`, `"SELECT.*" \+`, `f"SELECT`, `f'SELECT`
- Grep: `connection.execute`, `cursor.execute`
- Look for string concatenation in any SQL query (vs parameterized queries)

## Find Message Queues

- Grep: `SQS`, `sqs`, `SqsClient`, `sendMessage`, `receiveMessage`
- Grep: `Kafka`, `kafka`, `KafkaTemplate`, `KafkaListener`, `@KafkaListener`
- Grep: `RabbitMQ`, `rabbit`, `amqp`, `@RabbitListener`, `AmqpTemplate`
- Grep: `SNS`, `sns`, `publish`, `subscribe`
- Grep: `EventBridge`, `eventbridge`
- Check: what data is in message payloads? Are tokens/PII included?

## Find Caching

- Grep: `@Cacheable`, `@CachePut`, `@CacheEvict`, `RedisTemplate`
- Grep: `redis`, `Redis`, `cache`, `Cache`, `memcached`
- Grep: `node-cache`, `lru-cache`, `ioredis`
- Check: what data is cached? Are secrets or tokens cached? TTLs?

## Find Logging Risks

- Grep: `log.info`, `log.debug`, `log.error`, `logger.info`, `logger.debug`
- Grep: `console.log`, `print(`, `println`
- Grep patterns near sensitive data: `token`, `password`, `secret`, `account`
- Grep: `MASKING_FIELDS`, `mask`, `redact`, `@JsonIgnore`
- Grep: logback config `logback.xml`, `logback-spring.xml`, `log4j2.xml`
- Check: are request/response bodies logged? Do audit tables store tokens?

## Find Logging Risks — Mobile BFF

- Grep: `sensitiveInfoReplacer`, `SENSITIVE`, `BLACKLIST`, `redact`
- Read the redaction utility fully — what fields are blacklisted?
- Grep: `JSON.stringify(error` — find all error serialization without the replacer function
- Check: are ALL controllers using the replacer when logging error data?
- Check: does the PII blacklist cover all relevant fields? Common gaps:
  - `dateOfBirth`, `dob`, `routingNumber`, `address`, `street`, `zipCode`, `city`, `state`
- Check: does the signature middleware log request bodies without redaction?
- Check: are SQS error callbacks (`console.log('SQS Error:', err)`) including sensitive data?
- Grep: `stringifyBody`, `stringifyQuery` — custom serializers that may bypass redaction

## Find Client-Side State — Frontend

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

## Find Browser Storage — Frontend

- Grep: `localStorage`, `sessionStorage`, `IndexedDB`, `openDatabase`
- Grep: `window.localStorage`, `window.sessionStorage`
- Grep: `setItem`, `getItem`, `removeItem` (direct storage API calls)
- Grep: `createJSONStorage`, `redux-persist` (framework-level persistence)
- Check: what sensitive data is persisted? Is it encrypted?

## Find postMessage Security — Frontend

- Grep: `postMessage`, `addEventListener.*message`, `window.addEventListener`
- Grep: `MessageEvent`, `event.origin`, `event.source`, `event.data`
- Grep: `document.referrer` (used as postMessage target origin)
- Check: do message listeners validate `event.origin` against an allowlist?
- Check: is postMessage sent with a specific target origin (not `*`)?
- Check: is `document.referrer` used to determine parent origin (can be spoofed or absent)?

## Find XSS Vectors — Frontend

- Grep: `dangerouslySetInnerHTML`, `innerHTML`, `outerHTML`
- Grep: `document.write`, `document.writeln`
- Grep: `eval(`, `new Function(`, `setTimeout` and `setInterval` with string arguments
- Grep: `v-html` (Vue), `[innerHTML]` (Angular), `{@html` (Svelte)
- Check: are user inputs or URL params rendered without React/framework escaping?
- Check: are API responses containing HTML rendered without sanitization?

## Find iframe Security — Frontend

- Grep: `createElement('iframe')`, `<iframe`, `iframe.src`
- Grep: `sandbox`, `allow=`, `allowfullscreen`
- Check: are iframes created with `sandbox` attribute restricting capabilities?
- Check: is the iframe `src` built from trusted sources only?
- Check: are `X-Frame-Options` or CSP `frame-ancestors` set to prevent clickjacking of the app itself?

## Find Open Redirect Risks — Frontend

- Grep: `window.location`, `window.location.href`, `window.location.replace`
- Grep: `router.push`, `router.replace`, `navigate(`, `redirect(`
- Grep: `window.open`
- Check: are redirect targets hardcoded/allowlisted or derived from user input?
- Check: does the callback/redirect page validate the target URL?
