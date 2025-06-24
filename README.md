# Claude Code Security Rules

This repository contains security rules converted from Cursor's security rules (https://github.com/matank001/cursor-security-rules) to be compatible with Claude Code. These rules help enforce secure coding practices and prevent the generation of unsafe code.

## What's Included

- **Core Security Principles**: Universal security practices for all code
- **Language-Specific Rules**: Security guidelines for Python, JavaScript, Java, PHP, Ruby, Rust, and C
- **Vulnerability Prevention**: Specific guidance for preventing common vulnerabilities (SQL injection, XSS, path traversal, etc.)
- **Advanced Topics**: Dangerous data flow analysis and MCP security

## Installation & Setup

### Method 1: Project-Level Rules (Recommended for Teams)

1. Copy the `CLAUDE.md` file and `security-rules` folder to your project root:
   ```bash
   cp -r claude-code-security-rules/CLAUDE.md your-project/
   cp -r claude-code-security-rules/security-rules your-project/
   ```

2. Claude Code will automatically load these rules when you open the project.

### Method 2: Global User Rules (Personal Use)

1. Copy the files to your Claude Code configuration directory:
   ```bash
   mkdir -p ~/.claude
   cp -r claude-code-security-rules/CLAUDE.md ~/.claude/
   cp -r claude-code-security-rules/security-rules ~/.claude/
   ```

2. These rules will apply to all your Claude Code sessions.

## How It Works

1. **Automatic Loading**: When Claude Code starts, it automatically loads the `CLAUDE.md` file
2. **Import System**: The `@` syntax imports additional rule files, allowing modular organization
3. **Global Application**: Unlike Cursor's file-specific rules, Claude Code applies these rules globally
4. **Context Awareness**: Claude Code uses these rules to guide code generation and provide security warnings

## Security Coverage

These rules help prevent:
- Injection attacks (SQL, Command, Code)
- Hardcoded secrets and credentials
- Path traversal vulnerabilities
- SSRF and XXE attacks
- Insecure authentication patterns
- Sensitive data exposure in logs
- Timing attacks on secret comparison
- Client-side trust issues

## Best Practices

1. **Review Before Implementing**: Always review the rules to ensure they align with your project's security requirements
2. **Customize as Needed**: Add project-specific security rules to your `CLAUDE.md`
3. **Keep Updated**: Periodically check for updates to security best practices
4. **Team Alignment**: Ensure all team members use the same security rules for consistency

## Disclaimer

While these rules help enforce secure coding practices, they are not a replacement for:
- Security audits
- Penetration testing
- Code reviews
- Security training
