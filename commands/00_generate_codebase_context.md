in our top level folder, create a comprehensive [./.claude/llms.txt] file that serves as a complete codebase reference. Structure it as follows:

## PROJECT OVERVIEW
- Project purpose and domain
- Tech stack and key dependencies
- Architecture pattern (MVC, microservices, etc.)

## FILE MANIFEST
For each file/module, provide:
- Primary responsibility (one-line purpose)
- Key functions/classes with signatures (name, params with types, return types)
- Critical dependencies (what it imports/requires)
- Public API surface (what other files can use)

## SYSTEM ARCHITECTURE
- ASCII diagram showing file relationships and data flow
- Entry points and execution flow
- State management approach
- External integrations

## DEVELOPMENT GUIDELINES
- Code style conventions (naming, formatting)
- Design patterns in use
- Common abstractions and utilities
- Error handling strategy
- Testing approach

## DATA CONTRACTS
- Input/output formats (JSON schemas, API contracts)
- Database schemas if applicable
- Configuration structure

## INSIGHTS & RECOMMENDATIONS
From a senior engineer's perspective:
- Architectural strengths and technical debt
- DRY violations and refactoring opportunities
- Security considerations
- Performance bottlenecks
- Scalability concerns

Keep descriptions concise. Prioritize actionable information over commentary. Use tables/lists for scanability. Assume the reader is an experienced developer joining the project.