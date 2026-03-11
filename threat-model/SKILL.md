---
name: threat-model
description: Scan the currently open repository and generate a comprehensive STRIDE-based threat model report. Use when the user asks to generate a threat model, scan for security threats, perform a threat assessment, or review the security posture of a codebase.
---

# Threat Model Generator

Systematically scan the open repo and produce a structured threat model report in chat.

## Step 1: Discover the Tech Stack

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
- `README.md` → project purpose

Use Glob to find these files. Read the build file and README to understand the project's purpose, dependencies, and structure.

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

Ask it to return: a table of endpoints, auth mechanisms, validation gaps, and unprotected endpoints.

### Agent 2: External Integrations and Secrets

Tell the agent to find:
- External service clients (HTTP clients, SDKs, Feign, RestTemplate, axios, requests)
- How credentials are stored (env vars, config files, vaults, Secrets Manager, hardcoded)
- Default/fallback secrets in configuration
- `.env` files or config files with sensitive values
- Encryption mechanisms (at rest and in transit)
- PII handling (masking, redaction, encryption of sensitive fields)

Ask it to return: a list of integrations, secrets management approach, encryption status, PII handling patterns.

### Agent 3: Persistence and Data Flow

Tell the agent to find:
- Database entities/models and what sensitive data they store
- Raw SQL or native queries (SQL injection risk)
- Message queues (SQS, Kafka, RabbitMQ) and what data flows through them
- Caching (Redis, Memcached, in-memory) and what gets cached
- Logging patterns — is sensitive data logged?
- Audit tables that might store tokens or PII
- Data flow from external sources to DB to API consumers

Ask it to return: entity inventory with sensitive fields, SQL injection assessment, messaging/caching details, logging risks.

### Agent 4: Dependencies, Config, and Infrastructure

Tell the agent to find:
- All dependencies and their versions (from the build file)
- Known vulnerable libraries (check for outdated major versions)
- Security-relevant config (TLS, timeouts, actuator/debug endpoints, error handling)
- Dockerfile and k8s manifests (base images, exposed ports, privileges)
- Error handling — do error responses leak internal details?
- Health/debug/admin endpoints and their protection
- CI/CD pipeline security (secrets in workflows, skipped tests)

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
