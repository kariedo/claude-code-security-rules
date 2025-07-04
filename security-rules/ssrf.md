# SSRF Prevention

These rules apply to all code that performs outbound network requests, regardless of language or framework, including generated code.

All violations must include a clear explanation of which rule was triggered and why, to help developers understand and fix the issue effectively.  
Generated code must not violate these rules. If a rule is violated, a comment must be added explaining the issue and suggesting a correction.

## 1. Do Not Allow User Input to Control Target URLs
- **Rule:** Never use raw or unchecked user input as the destination for outbound HTTP requests.
- **Example (unsafe):**
  ```python
  requests.get(request.args["url"])
  ```

## 2. Block Access to Private/Internal IP Ranges
- **Rule:** Outbound requests must not be allowed to reach `localhost`, `127.0.0.1`, `169.254.0.0/16`, `192.168.0.0/16`, `10.0.0.0/8`, or other internal/reserved ranges.

## 3. Resolve and Validate Hostnames Before Use
- **Rule:** Perform DNS resolution and validate that the resolved IP is in an allowed range before initiating a request.

## 4. Restrict Allowed Protocols and Ports
- **Rule:** Only allow HTTP/HTTPS protocols and known-safe ports (e.g., 80, 443). Block access to file URLs, gopher, FTP, or custom handlers.

## 5. Do Not Forward Authorization Headers Automatically

- **Rule:** Never pass internal tokens, cookies, or auth headers when proxying or forwarding outbound requests unless explicitly scoped and audited.