Review the following code for security vulnerabilities, performance issues, and adherence to project patterns.

Focus on: $ARGUMENTS

Check against the Critical Constraints in CLAUDE.md, specifically:
- Security: No plain text passwords, validate all input with Pydantic, JWT in middleware
- Performance: No sync DB in async, no N+1 queries, use connection pooling
- Architecture: Domain has no dependencies on adapters

Provide a structured review with severity levels (Critical, Warning, Info).
