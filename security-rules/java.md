# Secure Java Development

These rules apply to all Java code in the repository (including Spring, Jakarta EE, and standalone apps) and aim to prevent common security risks through disciplined use of input validation, safe APIs, and secure coding patterns.

All violations must include a clear explanation of which rule was triggered and why, to help developers understand and fix the issue effectively.  
Generated code must not violate these rules. If a rule is violated, a comment must be added explaining the issue and suggesting a correction.

## 1. Avoid Insecure Deserialization
- **Rule:** Never deserialize untrusted data using `ObjectInputStream`, `readObject()`, or similar APIs. Use safe serialization formats (e.g., JSON, protobuf) and validate all input before deserialization.
- **Unsafe:**
  ```java
  ObjectInputStream ois = new ObjectInputStream(inputStream);
  Object obj = ois.readObject(); // Untrusted input ➜ RCE risk
  ```
- **Safe:**
  ```java
  // Use a safe format and strict schema validation
  MyObject obj = objectMapper.readValue(jsonInput, MyObject.class);
  ```

## 2. Use Parameterized Queries for Database Access
- **Rule:** Never build SQL queries by concatenating user input. Use `PreparedStatement` or ORM parameterization.

## 3. Always Use Prepared Statements for SQL
- **Rule:** Always use `PreparedStatement` for SQL queries, never concatenate or interpolate user input into SQL strings. Prepared statements ensure user input is treated as data, not code, and are the most effective way to prevent SQL injection.
- **Unsafe:**
  ```java
  String query = "SELECT * FROM users WHERE username = '" + userInput + "'";
  Statement stmt = conn.createStatement();
  ResultSet rs = stmt.executeQuery(query);
  ```
- **Safe:**
  ```java
  String query = "SELECT * FROM users WHERE username = ?";
  PreparedStatement pstmt = conn.prepareStatement(query);
  pstmt.setString(1, userInput);
  ResultSet rs = pstmt.executeQuery();
  ```

## 4. Prevent Command Injection
- **Rule:** Never pass user input directly to `Runtime.exec()`, `ProcessBuilder`, or similar APIs. Use allow-lists, parameterization, and safe wrappers. Avoid shell invocation unless absolutely necessary.
- **Unsafe:**
  ```java
  Runtime.getRuntime().exec("ls " + userInput); // Command injection risk
  ```
- **Safe:**
  ```java
  // Use ProcessBuilder with static arguments or validated input
  ProcessBuilder pb = new ProcessBuilder("ls", validatedArg);
  pb.start();
  ```

## 5. Validate and Sanitize All Input
- **Rule:** Validate type, length, and format of all external input. Use allow-lists and built-in validation frameworks (e.g., Bean Validation, Hibernate Validator).

## 6. Avoid Insecure JNDI and LDAP Lookups
- **Rule:** Never perform JNDI, LDAP, or RMI lookups with untrusted input. Do not pass user-controlled data to lookup names or URLs.
- **Unsafe:**
  ```java
  ctx.lookup(userInput); // User input in JNDI ➜ RCE risk
  ```
- **Safe:**
  ```java
  // Only use allow-listed, static names for lookups
  ctx.lookup("java:comp/env/jdbc/MyDataSource");
  ```

## 7. Disable Dangerous Features in XML Parsers
- **Rule:** Disable DTDs, external entities, and XInclude in all XML parsers. Use `FEATURE_SECURE_PROCESSING` and block XXE.

## 8. Avoid Logging Sensitive Data
- **Rule:** Do not log credentials, tokens, or personal data. Mask or redact sensitive fields in logs.

## 9. Use Strong Cryptography
- **Rule:** Use strong, up-to-date cryptographic algorithms and libraries. Never use hardcoded keys or outdated algorithms (e.g., MD5, SHA-1, DES).

## 10. Limit Use of Reflection and Dynamic Code Execution
- **Rule:** Avoid reflection, dynamic class loading, or `Method.invoke()` with untrusted input. Never use `ScriptEngine` or similar APIs on user data.

## 11. Keep Dependencies Patched and Audited
- **Rule:** Regularly update all dependencies. Use tools like OWASP Dependency-Check to audit for known vulnerabilities.

## 12. Enforce Least Privilege

- **Rule:** Run Java processes with the minimum OS and JVM permissions required. Limit file, network, and system access.