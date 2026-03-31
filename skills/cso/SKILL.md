---
name: cso
description: "Security audit. OWASP + STRIDE analysis. Triggers on 'cso', 'security', 'security audit', 'OWASP', 'STRIDE', 'vulnerability'."
context: fork
tools: Read, Bash, Glob, Grep
---

# CSO (Chief Security Officer)

Security audit based on OWASP Top 10 + STRIDE framework.

**Context: fork** — Run in isolated context.

---

## Workflow

### Phase 1: Attack Surface Mapping
```bash
# Identify API endpoints
grep -r "app\.\(get\|post\|put\|delete\|patch\)" src/ --include="*.ts"
grep -r "@\(Get\|Post\|Put\|Delete\|Patch\)" src/ --include="*.ts"

# Check authentication/authorization
grep -r "auth\|token\|jwt\|session\|cookie" src/ --include="*.ts"

# Check user input handling
grep -r "req\.\(body\|query\|params\)" src/ --include="*.ts"
grep -r "innerHTML\|dangerouslySetInnerHTML" src/ --include="*.tsx"
```

### Phase 2: OWASP Top 10 Check
1. **Injection** — SQL, NoSQL, OS, LDAP injection
2. **Broken Authentication** — Session management, password policy
3. **Sensitive Data Exposure** — Encryption, transmission security
4. **XML External Entities** — XXE attack
5. **Broken Access Control** — Authorization validation
6. **Security Misconfiguration** — Default settings, error handling
7. **XSS** — User input escaping
8. **Insecure Deserialization** — Input validation
9. **Known Vulnerabilities** — Dependency vulnerabilities
10. **Insufficient Logging** — Audit trail

### Phase 3: STRIDE Analysis
- **S**poofing — Authentication bypass
- **T**ampering — Data modification
- **R**epudiation — Non-repudiation
- **I**nformation Disclosure — Information leakage
- **D**enial of Service — Service denial
- **E**levation of Privilege — Privilege escalation

### Phase 4: Dependency Audit
```bash
{pm} audit
# or
npx better-npm-audit audit
```

---

## Output

```markdown
### 🔒 Security Audit: [Project/Feature]

**Risk Level:** [Critical/High/Medium/Low]

**OWASP Findings:**
| # | Category | Risk | Finding | Fix |
|---|----------|------|---------|-----|
| 1 | Injection | [Critical] | [desc] | [fix] |

**STRIDE Findings:**
| Threat | Risk | Finding | Mitigation |
|--------|------|---------|------------|
| Spoofing | [Important] | [desc] | [fix] |

**Dependency Vulnerabilities:**
- [package@version]: [severity] — [description]

**Recommendations:**
1. [Highest priority fix]
2. [Next priority]

**Verdict:** [SECURE / NEEDS FIX / CRITICAL]
```

→ If NEEDS FIX, go to `/{framework}-fix`, if CRITICAL, respond immediately
