# Secure PHP Development

These rules apply to all PHP code in the repository and aim to prevent common security vulnerabilities through strict handling of input, output, and execution.

All violations must include a clear explanation of which rule was triggered and why, to help developers understand and fix the issue effectively.

if code is in violation add a comment explaining why. 
don't generate code that violates these rules.

## 1. Do Not Use User Input in File Paths
- **Rule:** Never use `$_GET`, `$_POST`, or any unsanitized input directly in file paths or file operations.

## 2. Always Sanitize User Input Before Use
- **Rule:** All external input (e.g., `$_GET`, `$_POST`, `$_COOKIE`, `$_SERVER`) must be sanitized before being used in logic, queries, or output.

## 3. Escape Output to Prevent XSS
- **Rule:** Always escape output when rendering user-controlled data into HTML.

## 4. Do Not Log Sensitive Input or Secrets
- **Rule:** Avoid logging passwords, tokens, or personal user data.

## 5. Use Constant-Time Comparison for Secrets
- **Rule:** Secrets such as tokens, signatures, or passwords must be compared using constant-time functions (e.g., `hash_equals`) to prevent timing attacks.

## 6. Use Strong Defaults for Encryption
- **Rule:** Default encryption settings must use secure algorithms and sufficient key lengths (e.g., RSA ≥ 2048 bits, AES-256).

## 7. Do Not Use `unserialize()` on Untrusted Input
- **Rule:** The `unserialize()` function must not be used on data from user input or external sources. Use `json_decode()` as a safer alternative.

## 8. Restrict File Includes to Known Values
- **Rule:** `include`, `require`, `include_once`, and `require_once` must only be used with hardcoded or explicitly whitelisted file paths, not values derived from user input.

## 9. Do Not Create Files Based on Untrusted Input
- **Rule:** File creation functions (e.g., `fopen`, `file_put_contents`) must not use filenames derived from untrusted input without strict validation.

## 10. Do Not Use Dangerous Execution Functions
- **Rule:** Avoid using `eval`, `shell_exec`, `exec`, `system`, or similar functions, especially with variable input. Use safer APIs or libraries instead.

## 11. Avoid Type-Juggling Comparisons
- **Rule:** Do not use loose comparison operators (`==`, `!=`). Always use strict comparison (`===`, `!==`) to prevent type coercion vulnerabilities.