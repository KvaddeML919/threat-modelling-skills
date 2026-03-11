# Threat Model Report Template

Use this exact structure when outputting the report. Fill in every section with findings from the scan. Replace placeholder text in `[brackets]` with actual content.

---

## Output Format

```markdown
# [Service/Repo Name] — Threat Model Report

## 1. System Overview

**Repository:** [repo name]
**Tech Stack:** [language, framework, build tool, runtime version]
**Purpose:** [1-2 sentence description from README or code analysis]
**Deployment:** [Docker/K8s/Lambda/EC2/unknown — from Dockerfile, k8s manifests, CI config]

---

## 2. Architecture and Data Flow

[Insert a mermaid diagram showing the main components, trust boundaries, and data flows.
Include: external clients, the application, databases, caches, message queues, external APIs.
Mark trust boundaries clearly.]

**Trust boundaries:**
1. [External clients → Application API (auth boundary)]
2. [Application → External services (credential boundary)]
3. [Application → Internal infrastructure (network boundary)]

---

## 3. Threat Analysis (STRIDE)

### 3.1 Spoofing (Identity)

| ID | Threat | Severity | Details |
|----|--------|----------|---------|
| S-1 | [threat description] | [CRITICAL/HIGH/MEDIUM/LOW] | [specific details, affected endpoints/files] |

[Repeat for each finding. If none found, state "No spoofing threats identified."]

### 3.2 Tampering

| ID | Threat | Severity | Details |
|----|--------|----------|---------|
| T-1 | [threat description] | [severity] | [details] |

### 3.3 Repudiation

| ID | Threat | Severity | Details |
|----|--------|----------|---------|
| R-1 | [threat description] | [severity] | [details] |

### 3.4 Information Disclosure

| ID | Threat | Severity | Details |
|----|--------|----------|---------|
| I-1 | [threat description] | [severity] | [details] |

### 3.5 Denial of Service

| ID | Threat | Severity | Details |
|----|--------|----------|---------|
| D-1 | [threat description] | [severity] | [details] |

### 3.6 Elevation of Privilege

| ID | Threat | Severity | Details |
|----|--------|----------|---------|
| E-1 | [threat description] | [severity] | [details] |

---

## 4. Vulnerable Dependencies

| Dependency | Current Version | Known CVE / Concern | Severity | Recommendation |
|------------|----------------|---------------------|----------|----------------|
| [lib name] | [version] | [CVE or "outdated"] | [severity] | [upgrade target] |

[If no vulnerable dependencies found, state that.]

---

## 5. Sensitive Data Inventory

| Data Type | Storage Location | Protection | Gap |
|-----------|-----------------|------------|-----|
| [e.g., passwords, tokens, PII, account numbers] | [DB table/column, config file, cache] | [encrypted/masked/hashed/plain text] | [what's missing] |

---

## 6. SQL Injection Assessment

[State whether raw SQL or native queries were found.
If found, list them and assess whether they use parameter binding.
Conclude with overall assessment.]

---

## 7. Encryption Assessment

| Layer | Status | Details |
|-------|--------|---------|
| In transit — external APIs | [Yes/No/Partial] | [HTTPS, TLS version, etc.] |
| In transit — message queues | [Yes/No/Partial] | [TLS, SASL, etc.] |
| In transit — database | [Yes/No/Partial] | [TLS, SSL, etc.] |
| At rest — sensitive fields | [Yes/No/Partial] | [KMS, AES, application-level, etc.] |
| At rest — tokens/credentials | [Yes/No/Partial] | [encrypted or plain text] |

---

## 8. Prioritized Recommendations

### Critical

| # | Finding | Recommendation |
|---|---------|----------------|
| 1 | [finding] | [specific actionable fix] |

### High

| # | Finding | Recommendation |
|---|---------|----------------|
| 1 | [finding] | [fix] |

### Medium

| # | Finding | Recommendation |
|---|---------|----------------|
| 1 | [finding] | [fix] |

### Low

| # | Finding | Recommendation |
|---|---------|----------------|
| 1 | [finding] | [fix] |

---

*This report is based on static analysis of the repository. Complement with dynamic testing (penetration testing), infrastructure-level review, and upstream/downstream service analysis for a complete threat model.*
```

## Guidelines for Filling the Template

1. **Be specific** — cite file paths, line ranges, class names, endpoint paths.
2. **Be actionable** — every recommendation should tell the developer exactly what to do.
3. **Don't pad** — if a STRIDE category has no findings, say so in one line. Don't invent threats.
4. **Use tables** — they're easier to scan than paragraphs.
5. **Mermaid diagrams** — use them for architecture. Keep node names in camelCase (no spaces).
6. **Severity consistency** — apply the severity criteria from SKILL.md uniformly.
7. **No false positives** — only report findings you can substantiate with evidence from the code.
