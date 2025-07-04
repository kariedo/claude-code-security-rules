# Dangerous Flow Identification

A **Dangerous Flow** occurs when user input—or data derived from it—is used in a way that introduces **vulnerabilities, undefined behavior, or unwanted system interactions**. This can range from command injection, SSRF, and path traversal to logic bugs and broken access controls. The goal is to trace the lifecycle of untrusted inputs and assess whether their use is safe.

---

## Starting the Session
1. If the project is very large, you may ask the user for what specific flow he wishes to test or for where more specific area of the code he is interested in.

## Example of Dangerous Flow in JavaScript

```javascript
const express = require('express');
const fs = require('fs');
const app = express();

app.get('/read', (req, res) => {
  const filename = req.query.file; // User input
  fs.readFile(`/home/userdata/${filename}`, (err, data) => {  // Dangerous usage
    if (err) return res.status(500).send("Error reading file");
    res.send(data);
  });
});
```

> **Danger**: This allows for **Path Traversal** (e.g., `?file=../../etc/passwd`), enabling unauthorized file access.

---

## Identifying User Inputs

1. **Web/Backend Projects**

   * Look for where routes or API endpoints are defined (e.g., Express `app.get()`, Flask `@app.route`, FastAPI).
   * Search for request/response objects: `req`, `res`, `request`, `ctx`, etc.

2. **CLI/Native Applications**

   * Look for `process.argv`, `getopt`, or direct `stdin` usage.
   * Watch for file input, `env` variables, user config, or interactive prompts.

3. **Persisted Inputs**

   * Even if sanitized once, data stored in a DB, config, or file can be reused unsafely.
   * These create **second-order injection risks**.

---

## Following the Flow

1. **Start Point Detection**

   * Look for parameters from user input:

     * `req.body`, `req.query`, `input()`, `prompt()`, `fs.readFileSync(userInput)`, etc.
   * Example:

     ```python
     username = input("Enter username:")  # user input
     ```

2. **Follow Data Across Calls**

   * Trace how the input propagates:

     ```python
     def do_something(name):
         return f"{name}.txt"
     ```

3. **Input Saved in Resources**

   * Watch for:

     ```javascript
     db.save({ userInput })  // saved
     ...
     const val = db.find(...); fs.readFile(val); // reused dangerously
     ```

4. **Input Mutation**

   * Example of unsafe mutation:

     ```javascript
     const filename = `${req.query.name}.txt`;
     exec(`cat ${filename}`); // risky, even after mutation
     ```

   * AI should reason whether transformations strip dangerous content or just reshape it.

---

## Mitigating Risk

1. **Sanitization**

   * Apply proper validation/sanitization according to context (path, shell, SQL, etc.).
   * Examples:

     * `path.normalize`, `validator.escape`, `param.trim().match(/^[a-z0-9_-]+$/)`

2. **Allow-Lists Over Block-Lists**

   * Validate only known-good formats or values.

3. **Contextual Escaping**

   * Escape output depending on where it ends up: HTML, SQL, shell, etc.

4. **Least Privilege Access**

   * Don't give the code unnecessary access (e.g., limit file read paths, DB permissions).

5. **Avoid Dangerous APIs**

   * Avoid using APIs like `eval`, `exec`, or `Function()` when not absolutely necessary.

---

## When a Potential Risk is Found

1. **Document the Flow** using this template:
File and line number, what happens there
Next file and line number, what happens there
What is the potential risk

   **Additional Flow Templates:**

   * `[source] -> [saved to DB] -> [used in SQL query] => [SQL Injection]`
   * `[CLI param] -> [used in shell command] => [Command Injection]`

2. **List Dangerous Inputs**

   * Provide concrete examples of inputs that would exploit the issue.
   * Example: `?file=../../../../etc/passwd`

3. **Suggest Fix**

   * Concrete change, e.g., validate `filename` against a list or disallow `../` entirely.
   * In some cases, recommend using libraries that manage encoding/safety.


## Recognizing Dangerous Functions

Dangerous functions are operations that can cause unintended side effects, system compromise, or data exposure **when given untrusted input**. These functions are not always obviously labeled as "dangerous" — so the AI must reason **based on context** and the **type of behavior** involved.

### General Heuristics

Ask:

* Does this function **interact with the system** (shell, filesystem, network)?
* Does it **parse or render** something (e.g., HTML, SQL, templates)?
* Can untrusted input **change behavior** at runtime?
* Is the function **used to enforce access control, filtering, or logic**?

---

### Common Types of Dangerous Functions

#### 1. **Shell/Process Execution**

* These functions allow arbitrary commands to be run on the host system:

  * `exec`, `spawn`, `system`, `popen`, `ProcessBuilder`, `subprocess.run`, etc.

  > Example:

  ```js
  exec(`rm -rf ${userInput}`);  // Unvalidated input = command injection
  ```

#### 2. **File System Access**

* These functions read/write files or directories, potentially leaking or modifying unintended data:

  * `fs.readFile`, `fs.writeFile`, `open()`, `unlink()`, `path.join()`, `os.remove()`, etc.

  > Input like `../../../etc/passwd` can exploit path traversal.

#### 3. **Database Queries**

* Especially when **query strings** are manually constructed:

  * `SELECT * FROM users WHERE username = '${input}'`

  * Use of ORMs without proper parameterization can also be dangerous.

  > Risk: SQL Injection

#### 4. **Template Rendering / HTML Injection**

* Functions that render HTML, templates, or text:

  * `res.send()`, `render()`, `jinja2.render`, `ejs.render`, `Handlebars.compile()`, etc.

  > Input like `</script><script>alert(1)</script>` can result in XSS.

#### 5. **Dynamic Code Execution**

* Any function that executes arbitrary code or expressions:

  * `eval()`, `Function()`, `setTimeout(..., string)`, `exec()`, `pickle.load()`, `deserialize()`

  > Even `eval("user input")` that just returns a number is unsafe.

#### 6. **Insecure Cryptography or Auth Control**

* Functions that perform auth or security checks incorrectly:

  * `if (user.role == 'admin')` with user-controlled role
  * Weak crypto: `md5`, `sha1`, or hardcoded keys

#### 7. **Network Requests**

* Functions that can reach arbitrary URLs:

  * `fetch()`, `axios.get()`, `requests.get()`, `urllib`, `http.get()`, etc.

  > Risk: SSRF (Server-Side Request Forgery) when the URL is user-controlled

---

### Context-Sensitive Dangerous Behavior

A function may not be inherently dangerous, but **becomes dangerous when fed tainted input**.

> Safe: `fs.readFileSync('static.txt')`
> Dangerous: `fs.readFileSync(userInput)`

> Safe: `render({ title: 'About Us' })`
> Dangerous: `render({ title: userInput })` if not escaped

---
## Final Advice for the AI Coding Agent

* **Always trace input origin** → If it flows into any of the above, flag it.
* **Look for indirect usage** → e.g., input saved to file → read into `exec()`.
* **Trust no context unless proven** → even a `.bio` field might get rendered into a CLI or HTML string.
* Watch for input that is used in:

  * Paths
  * URLs
  * Query strings
  * Shell commands
  * HTML attributes or content
  * Template logic
* **Always reason based on context**: A string used in an HTML context poses different risks than one used in a system call.
* **Map out variable lifecycles**, including mutations and storage.
* **Ask "Can this input end up in an unsafe function?"** If yes, flag it.

---