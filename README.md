# Threat Modelling Skills for Cursor

A Cursor Agent Skill that scans any open repository and generates a comprehensive STRIDE-based threat model report directly in chat.

## How It Works

1. Open any repo in Cursor
2. Ask the agent: "generate a threat model" or "scan this repo for security threats"
3. The agent launches 4 parallel exploration agents to scan the codebase
4. Findings are classified using STRIDE and output as a structured report

## Installation

### Personal (all projects)

Copy the `threat-model/` directory to your personal Cursor skills folder:

```bash
cp -r threat-model/ ~/.cursor/skills/threat-model/
```

### Project (shared with team via repo)

Copy into the project's `.cursor/skills/` directory and commit:

```bash
cp -r threat-model/ .cursor/skills/threat-model/
git add .cursor/skills/threat-model/
git commit -m "Add threat model skill"
```

## Skill Structure

```
threat-model/
  SKILL.md                            # Core instructions (discovery, scanning, classification)
  report-template.md                  # Output format template (STRIDE tables, recommendations)
  checklists/
    api-surface.md                    # Agent 1: endpoints, auth, input validation, SSRF
    integrations-secrets.md           # Agent 2: external clients, credentials, encryption, PII
    persistence-data-flow.md          # Agent 3: DB entities, data flow, deserialization, XSS
    dependencies-infra.md             # Agent 4: dependencies, config, Docker, CI/CD
```

## What It Scans

| Area | What It Looks For |
|------|-------------------|
| API Surface | Endpoints, authentication, authorization, input validation, rate limiting, CORS/CSRF |
| Integrations & Secrets | External clients, hardcoded credentials, secrets management, encryption, PII handling |
| Persistence & Data Flow | Database entities, SQL injection risks, message queues, caching, logging of sensitive data |
| Dependencies & Infra | Vulnerable libraries, security config, Dockerfiles, error handling, admin endpoints |

## Report Output

The generated report includes:

- System overview with architecture diagram
- STRIDE threat analysis (Spoofing, Tampering, Repudiation, Information Disclosure, DoS, Elevation of Privilege)
- Vulnerable dependencies table
- Sensitive data inventory
- Encryption assessment
- Prioritized recommendations (Critical / High / Medium / Low)

## Supported Frameworks

- Java / Spring Boot (Maven, Gradle)
- Node.js / Express / NestJS
- Python / Django / Flask / FastAPI
- Go
- Rust (basic)

## Trigger Phrases

- "generate a threat model"
- "scan this repo for security threats"
- "threat model this repo"
- "perform a threat assessment"
- "review the security posture"
