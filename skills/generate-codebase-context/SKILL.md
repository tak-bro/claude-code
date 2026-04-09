---
name: generate-codebase-context
model: sonnet
description: "Generate codebase context. Create .claude/llms.txt file. Triggers on 'generate context', 'codebase context', 'llms.txt', '코드베이스 분석', '프로젝트 분석', 'onboarding', '온보딩'."
tools: Read, Bash, Glob, Grep, Write
---

# Generate Codebase Context

Create `.claude/llms.txt` file in the project root directory and use it as a codebase reference.

## Project Onboarding Flow

Recommended order when starting a new project:
```
/generate-codebase-context  → Create .claude/llms.txt
         ↓
/explore                    → Read-only analysis of specific modules
         ↓
/{framework}-plan           → Reference llms.txt + explore results for planning
```

---

## llms.txt Structure

Write to `.claude/llms.txt` with the following structure:

### PROJECT OVERVIEW
- Project purpose and domain
- Tech stack and key dependencies
- Architecture pattern (MVC, microservices, etc.)

### FILE MANIFEST
For each file/module, provide:
- Primary responsibility (one-line purpose)
- Key functions/classes with signatures (name, params with types, return types)
- Critical dependencies (what it imports/requires)
- Public API surface (what other files can use)

### SYSTEM ARCHITECTURE
- ASCII diagram showing file relationships and data flow
- Entry points and execution flow
- State management approach
- External integrations

### DEVELOPMENT GUIDELINES
- Code style conventions (naming, formatting)
- Design patterns in use
- Common abstractions and utilities
- Error handling strategy
- Testing approach

### DATA CONTRACTS
- Input/output formats (JSON schemas, API contracts)
- Database schemas if applicable
- Configuration structure

### INSIGHTS & RECOMMENDATIONS
From a senior engineer's perspective:
- Architectural strengths and technical debt
- DRY violations and refactoring opportunities
- Security considerations
- Performance bottlenecks
- Scalability concerns

Keep descriptions concise. Prioritize actionable information over commentary. Use tables/lists for scanability. Assume the reader is an experienced developer joining the project.

---

## Post-Generation

After generation:
- `llms.txt` is automatically referenced by all plan skills (Knowledge Sources section)
- Re-run when project structure changes to keep it up-to-date

```
Context generated: `.claude/llms.txt`
→ Next: `/explore` for module-level analysis, or `/{framework}-plan` to start planning.
```
