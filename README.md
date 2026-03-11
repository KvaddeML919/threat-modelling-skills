# Threat Modelling Skills for Cursor

A Cursor Agent Skill that scans any open repository and generates a comprehensive STRIDE-based threat model report directly in chat.

## How It Works

1. Open any repo in Cursor
2. Ask the agent: "generate a threat model" or "scan this repo for security threats"
3. The agent launches 4 parallel exploration agents to scan the codebase
4. Findings are classified using STRIDE and output as a structured report

## Installation

Copy the `threat-model/` directory to your Cursor skills folder:

```bash
cp -r threat-model/ ~/.cursor/skills/threat-model/
```

## Skill Structure

```
threat-model/
  SKILL.md                  # Core instructions (discovery, scanning, classification)
  report-template.md        # Output format template (STRIDE tables, recommendations)
  scanner-checklists.md     # Framework-specific search patterns (Spring Boot, Node.js, Python, Go)
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
