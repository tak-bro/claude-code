---
name: security-scan
model: sonnet
description: "Deep security scan. Multi-angle vulnerability detection with auto-fix suggestions. Triggers on 'security scan', '보안 스캔', 'security', '보안', 'vulnerability', '취약점', 'cso', 'security audit', 'OWASP', 'STRIDE'."
context: fork
tools: Read, Bash, Glob, Grep, Write, Agent
---

# Security Scan

Scan the codebase for vulnerabilities from multiple angles. Identifies issues, suggests patches, and can auto-fix. Supersedes basic OWASP checklists with concrete code-level detection.

---

## Scan Angles

### 1. Input Validation
```bash
# Unsanitized user input
grep -rn "req\.body\|req\.query\|req\.params" --include="*.ts" | grep -v "validate\|sanitize\|zod\|yup\|joi\|class-validator"

# innerHTML / dangerouslySetInnerHTML
grep -rn "innerHTML\|dangerouslySetInnerHTML\|v-html" --include="*.ts" --include="*.tsx" --include="*.vue"

# SQL/NoSQL injection patterns
grep -rn "raw(\|rawQuery\|\$where\|eval(" --include="*.ts"

# Template literal in queries
grep -rn '`.*\$\{.*\}.*`' --include="*.ts" | grep -i "query\|sql\|where\|find"
```

### 2. Authentication & Authorization
```bash
# Missing auth middleware on routes
grep -rn "router\.\(get\|post\|put\|delete\)" --include="*.ts" | grep -v "auth\|guard\|middleware\|protect"

# Hardcoded secrets
grep -rn "password.*=.*['\"]" --include="*.ts" | grep -v "\.test\.\|\.spec\.\|schema\|type\|interface\|mock"
grep -rn "secret.*=.*['\"]" --include="*.ts" | grep -v "\.test\.\|\.spec\.\|process\.env"
grep -rn "api[_-]key.*=.*['\"]" --include="*.ts" | grep -v "\.test\.\|\.spec\."

# JWT issues
grep -rn "algorithm.*none\|verify.*false" --include="*.ts"
```

### 3. Injection & Code Execution
```bash
# eval / Function constructor
grep -rn "eval(\|new Function(" --include="*.ts" --include="*.tsx"

# Command injection
grep -rn "exec(\|execSync(\|spawn(" --include="*.ts" | grep -v "node_modules"

# Dynamic require/import with user input
grep -rn "require(\$\|import(\$" --include="*.ts"
```

### 4. Endpoint Exposure
```bash
# Debug/admin routes in production
grep -rn "\/debug\|\/admin\|\/test\|\/internal" --include="*.ts" | grep -i "route\|router\|app\."

# CORS wildcard
grep -rn "origin.*\*\|Access-Control-Allow-Origin.*\*" --include="*.ts"

# Missing rate limiting
grep -rn "router\.\(post\|put\|delete\)" --include="*.ts" | grep -v "rateLimit\|throttle"
```

### 5. Secret Management
```bash
# .env files tracked in git
git ls-files | grep -i "\.env" | grep -v "\.example\|\.sample\|\.template"

# process.env scattered (should be centralized)
grep -rn "process\.env\." --include="*.ts" | grep -v "config\|\.test\.\|\.spec\.\|node_modules" | wc -l

# Secrets in logs
grep -rn "console\.log.*password\|console\.log.*secret\|console\.log.*token" --include="*.ts"
```

### 6. Dependency Vulnerabilities
```bash
# npm/yarn/pnpm audit
{pm} audit 2>&1 | tail -30

# Check for known vulnerable packages
grep -E "lodash@[0-3]\.|axios@0\.[0-9]\." package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null
```

### 7. Platform-Specific

**Android (Kotlin):**
```bash
# Insecure SharedPreferences
grep -rn "getSharedPreferences\|MODE_WORLD" --include="*.kt"
# Missing certificate pinning
grep -rn "TrustManager\|X509TrustManager" --include="*.kt"
# WebView JavaScript enabled
grep -rn "setJavaScriptEnabled(true)" --include="*.kt"
```

**iOS (Swift):**
```bash
# Keychain vs UserDefaults for secrets
grep -rn "UserDefaults.*password\|UserDefaults.*token\|UserDefaults.*secret" --include="*.swift"
# ATS exceptions
grep -rn "NSAppTransportSecurity\|NSAllowsArbitraryLoads" --include="*.plist"
# Insecure URLSession
grep -rn "didReceive challenge.*completionHandler(.useCredential" --include="*.swift"
```

---

## Workflow

### Phase 1: Run all scan angles above
### Phase 2: Classify findings by severity (Critical / High / Medium / Low)
### Phase 3: Generate fix suggestions for each finding
### Phase 4: Present report

**User can then say "fix all" or "fix #N" to apply patches.**

---

## Output

```markdown
### 🔒 Security Scan: [Project]

**Scanned:** [N files]

| # | Angle | Severity | File:Line | Finding | Fix |
|---|-------|----------|-----------|---------|-----|
| 1 | Input Validation | 🔴 Critical | `api.ts:42` | Unsanitized query param in SQL | Use parameterized query |
| 2 | Secrets | 🟠 High | `.env` tracked | `.env` committed to git | Add to .gitignore, rotate secrets |
| 3 | Auth | 🟡 Medium | `routes.ts:15` | POST without auth middleware | Add auth guard |

**Summary:**
- 🔴 Critical: [N]
- 🟠 High: [N]
- 🟡 Medium: [N]
- 🟢 Low: [N]

**Dependency audit:** [N vulnerabilities]

**Verdict:** [SECURE ✅ / NEEDS FIX ⚠️ / CRITICAL 🚨]

Fix all? (y/n) or specify: fix #1, #2
```
