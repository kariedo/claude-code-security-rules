# Claude Code Security Rules

This directory contains comprehensive security guidelines for secure software development across multiple programming languages and vulnerability types.

## Language-Specific Security Rules

- [Python Security Rules](./python.md) - Security guidelines for Python development
- [JavaScript/Node.js Security Rules](./javascript.md) - Security guidelines for JavaScript and Node.js development
- [Java Security Rules](./java.md) - Security guidelines for Java development
- [PHP Security Rules](./php.md) - Security guidelines for PHP development
- [Ruby Security Rules](./ruby.md) - Security guidelines for Ruby development
- [Rust Security Rules](./rust.md) - Security guidelines for Rust development
- [C Security Rules](./c.md) - Security guidelines for C development

## Vulnerability Prevention Guidelines

- [Vulnerability Prevention Overview](./vulnerability-prevention.md) - Comprehensive overview of vulnerability prevention
- [Path Traversal Prevention](./path-traversal.md) - Preventing path traversal attacks
- [XXE Prevention](./xxe.md) - Preventing XML External Entity attacks
- [SSRF Prevention](./ssrf.md) - Preventing Server-Side Request Forgery
- [SQL Injection Prevention](./sql.md) - Secure SQL usage and injection prevention
- [Dangerous Flow Identification](./dangerous-flows.md) - Identifying and mitigating dangerous data flows
- [MCP Security](./mcp-security.md) - Secure Model Context Protocol usage

## How to Use These Rules

1. **For Code Generation**: When generating code, ensure it follows the security rules for the target language
2. **For Code Review**: Use these guidelines to identify potential security issues in existing code
3. **For Security Testing**: Test against the attack patterns described in each vulnerability prevention guide
4. **For Architecture Design**: Consider these security principles when designing new systems

## Key Security Principles

- **Input Validation**: Never trust user input; always validate and sanitize
- **Output Encoding**: Always encode output based on context (HTML, SQL, shell, etc.)
- **Least Privilege**: Grant minimum necessary permissions
- **Defense in Depth**: Apply multiple layers of security controls
- **Fail Securely**: Handle errors without exposing sensitive information
- **Keep Updated**: Regularly update dependencies and security patches