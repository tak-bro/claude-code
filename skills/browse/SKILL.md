---
name: browse
description: "Browser automation. Web testing with Playwright/Chrome. Triggers on 'browse', 'browser', 'browser test', 'playwright', 'UI test', 'visual test', 'screen test'."
tools: Read, Bash, Glob, Grep, Write
---

# Browse

Browser automation testing using Playwright or Chrome.

---

## Tool Selection

| Tool | Check | Action |
|------|-------|--------|
| Playwright MCP | `npx playwright --version` | Automated browser control |
| Puppeteer MCP | Check MCP availability | Fallback option |
| Neither | Ask user | Manual screenshot request |

---

## Workflow

### Phase 1: Setup
```bash
# Verify Playwright installation
npx playwright --version || npx playwright install
```

### Phase 2: Navigation
- Access target URL
- Handle necessary authentication
- Navigate to target page

### Phase 3: Verification
- Capture screenshots
- Verify UI elements
- Test interactions (click, input)
- Test responsiveness (different viewports)

### Phase 4: Report
- Attach screenshots
- List discovered issues
- Suggest fixes

---

## Output

```markdown
### 🌐 Browse Report: [Page/Feature]

**URL:** [tested URL]
**Viewport:** [sizes tested]

**Results:**
- [[PASS]/[FAIL]] [Element/Feature]: [Status]

**Screenshots:** [attached]

**Issues Found:**
- [Issue description + suggested fix]
```
