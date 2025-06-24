# Security Rules for Claude Code

These security rules enforce safe coding practices and prevent generation of unsafe code. All violations must include a clear explanation of which rule was triggered and why.

## Core Security Principles

@security-rules/core-principles.md

## Language-Specific Security Rules

When working with specific programming languages, apply these security rules:

- **Python**: @security-rules/python.md
- **JavaScript/Node.js**: @security-rules/javascript.md
- **Java**: @security-rules/java.md
- **PHP**: @security-rules/php.md
- **Ruby**: @security-rules/ruby.md
- **Rust**: @security-rules/rust.md
- **C**: @security-rules/c.md

## Vulnerability-Specific Prevention

@security-rules/vulnerability-prevention.md

## Advanced Security Topics

- **Dangerous Data Flows**: @security-rules/dangerous-flows.md
- **MCP Security**: @security-rules/mcp-security.md

## Implementation Guidelines

- Proactively apply these security rules when generating code
- Include security comments explaining why certain patterns are used
- Suggest secure alternatives when unsafe patterns are requested
- Educate users about security implications of their requests
- Never generate code that violates these rules without explicit user override