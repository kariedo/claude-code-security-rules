# Path Traversal Prevention

Path traversal attacks occur when user-controlled input is used to construct file paths, allowing attackers to access files outside the intended directory. This can lead to unauthorized file access, data leakage, or system compromise.

## 1. Do Not Use User Input in File Paths

**Rule:** Never use raw user input to construct file paths for read or write operations.

### ❌ Dangerous Examples

```python
# File Read - DANGEROUS
filename = request.args.get('file')
with open(f"/uploads/{filename}", 'r') as f:  # User can access ../../../etc/passwd
    content = f.read()

# File Write - DANGEROUS  
user_file = request.files['file']
filename = user_file.filename
user_file.save(f"/uploads/{filename}")  # User can write to ../../../sensitive/config.json
```

```javascript
// File Read - DANGEROUS
const filename = req.query.file;
fs.readFile(`./uploads/${filename}`, (err, data) => {  // ../../../etc/passwd
    res.send(data);
});

// File Write - DANGEROUS
const filename = req.body.filename;
fs.writeFile(`./uploads/${filename}`, data);  // ../../../system/config
```

```php
// File Read - DANGEROUS
$filename = $_GET['file'];
$content = file_get_contents("/uploads/" . $filename);  // ../../../etc/passwd

// File Write - DANGEROUS
$filename = $_POST['filename'];
file_put_contents("/uploads/" . $filename, $data);  // ../../../sensitive/data
```

### ✅ Secure Alternatives

```python
# File Read - SECURE
import os
from pathlib import Path

filename = request.args.get('file')
# Validate filename format
if not filename or not filename.replace('.', '').replace('_', '').replace('-', '').isalnum():
    abort(400, "Invalid filename")

# Use pathlib for safe path joining
upload_dir = Path("/uploads")
file_path = upload_dir / filename

# Ensure the final path is within the intended directory
if not str(file_path.resolve()).startswith(str(upload_dir.resolve())):
    abort(403, "Access denied")

with open(file_path, 'r') as f:
    content = f.read()
```

```javascript
// File Read - SECURE
const path = require('path');
const fs = require('fs');

const filename = req.query.file;
// Validate filename
if (!filename || !/^[a-zA-Z0-9._-]+$/.test(filename)) {
    return res.status(400).send('Invalid filename');
}

const uploadDir = path.resolve('./uploads');
const filePath = path.join(uploadDir, filename);

// Ensure path is within intended directory
if (!filePath.startsWith(uploadDir)) {
    return res.status(403).send('Access denied');
}

fs.readFile(filePath, (err, data) => {
    if (err) return res.status(404).send('File not found');
    res.send(data);
});
```

```php
// File Read - SECURE
$filename = $_GET['file'];

// Validate filename format
if (!preg_match('/^[a-zA-Z0-9._-]+$/', $filename)) {
    http_response_code(400);
    die('Invalid filename');
}

$uploadDir = realpath('/uploads');
$filePath = realpath('/uploads/' . $filename);

// Ensure path is within intended directory
if (!$filePath || strpos($filePath, $uploadDir) !== 0) {
    http_response_code(403);
    die('Access denied');
}

$content = file_get_contents($filePath);
```

## 2. Python Built-in Path Sanitization Functions

**Rule:** Use Python's built-in functions for path manipulation, but always validate the final result.

### os.path Functions

```python
import os

# os.path.normpath() - Normalizes path separators and resolves .. and .
filename = "../../../etc/passwd"
normalized = os.path.normpath(filename)  # Returns "../../../etc/passwd" (still dangerous!)

# os.path.abspath() - Returns absolute path
abs_path = os.path.abspath(filename)  # Returns full absolute path

# os.path.realpath() - Resolves symbolic links and returns absolute path
real_path = os.path.realpath(filename)  # Returns resolved absolute path

# os.path.join() - Safely joins path components
safe_path = os.path.join("/uploads", filename)  # Still dangerous if filename contains ../
```

### pathlib.Path (Recommended)

```python
from pathlib import Path

# Path.resolve() - Resolves all symbolic links and returns absolute path
base_dir = Path("/uploads")
user_file = Path(filename)
full_path = (base_dir / user_file).resolve()

# Check if final path is within intended directory
if not str(full_path).startswith(str(base_dir.resolve())):
    raise SecurityError("Path traversal detected")

# Path.parts - Check for dangerous components
if ".." in user_file.parts:
    raise ValueError("Path traversal attempt detected")
```

### urllib.parse (For URL-encoded paths)

```python
from urllib.parse import unquote

# Decode URL-encoded path traversal attempts
encoded_path = "%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd"
decoded_path = unquote(encoded_path)  # Returns "../../../etc/passwd"

# Always validate after decoding
if ".." in decoded_path:
    raise ValueError("Path traversal detected")
```

### ⚠️ Important Limitations

```python
# WARNING: These functions alone are NOT sufficient for security

# os.path.normpath() doesn't prevent path traversal
dangerous = "../../../etc/passwd"
normalized = os.path.normpath(dangerous)  # Still "../../../etc/passwd"

# os.path.join() doesn't prevent path traversal
joined = os.path.join("/uploads", "../../../etc/passwd")  # Still dangerous

# You MUST validate the final path
final_path = os.path.abspath(joined)
if not final_path.startswith("/uploads"):
    raise SecurityError("Path traversal detected")
```

### ✅ Secure Path Handling with Built-ins

```python
import os
from pathlib import Path
from urllib.parse import unquote

def secure_file_path(base_dir, user_filename):
    """
    Securely handle user-provided filenames using built-in functions.
    """
    # 1. Decode URL encoding if present
    decoded_filename = unquote(user_filename)
    
    # 2. Normalize the path
    normalized_filename = os.path.normpath(decoded_filename)
    
    # 3. Use pathlib for safe joining and resolution
    base_path = Path(base_dir).resolve()
    full_path = (base_path / normalized_filename).resolve()
    
    # 4. Validate the final path is within intended directory
    if not str(full_path).startswith(str(base_path)):
        raise SecurityError("Path traversal detected")
    
    # 5. Additional validation - check for dangerous patterns
    if ".." in Path(normalized_filename).parts:
        raise ValueError("Path traversal attempt detected")
    
    return str(full_path)

# Usage
try:
    safe_path = secure_file_path("/uploads", user_input)
    with open(safe_path, 'r') as f:
        content = f.read()
except (SecurityError, ValueError) as e:
    abort(400, str(e))
```

## 3. Use Safe Path Construction Methods

**Rule:** Always use language-specific safe path construction methods and validate the final path.

### Python Safe Methods
```python
from pathlib import Path
import os

# Safe path joining
base_dir = Path("/uploads")
file_path = base_dir / filename

# Validate final path
if not str(file_path.resolve()).startswith(str(base_dir.resolve())):
    raise SecurityError("Path traversal detected")
```

### JavaScript Safe Methods
```javascript
const path = require('path');

// Safe path joining
const uploadDir = path.resolve('./uploads');
const filePath = path.join(uploadDir, filename);

// Validate final path
if (!filePath.startsWith(uploadDir)) {
    throw new Error('Path traversal detected');
}
```

### PHP Safe Methods
```php
// Safe path construction
$uploadDir = realpath('/uploads');
$filePath = realpath('/uploads/' . $filename);

// Validate final path
if (!$filePath || strpos($filePath, $uploadDir) !== 0) {
    throw new Exception('Path traversal detected');
}
```

## 4. Implement Strict Input Validation

**Rule:** Validate filenames against strict allowlists rather than trying to block dangerous patterns.

### ✅ Good Validation Patterns

```python
# Allow only alphanumeric, dots, underscores, hyphens
import re
if not re.match(r'^[a-zA-Z0-9._-]+$', filename):
    raise ValueError("Invalid filename")

# Or use a strict allowlist
allowed_files = ['config.json', 'data.csv', 'report.pdf']
if filename not in allowed_files:
    raise ValueError("File not allowed")
```

```javascript
// Allow only safe characters
if (!/^[a-zA-Z0-9._-]+$/.test(filename)) {
    throw new Error('Invalid filename');
}

// Or use a strict allowlist
const allowedFiles = ['config.json', 'data.csv', 'report.pdf'];
if (!allowedFiles.includes(filename)) {
    throw new Error('File not allowed');
}
```

### ❌ Bad Validation Patterns

```python
# DON'T: Block specific patterns (can be bypassed)
if '..' in filename or '/' in filename:  # Can be bypassed with encoding
    raise ValueError("Invalid filename")
```

## 5. Use Secure File Upload Handling

**Rule:** When handling file uploads, generate safe filenames and never trust user-provided filenames.

```python
import uuid
import os
from werkzeug.utils import secure_filename

# Generate safe filename
def get_safe_filename(original_filename):
    # Use secure_filename to sanitize
    safe_name = secure_filename(original_filename)
    if not safe_name:
        # Generate random name if original is unsafe
        safe_name = f"{uuid.uuid4().hex}{os.path.splitext(original_filename)[1]}"
    return safe_name

# Usage
uploaded_file = request.files['file']
safe_filename = get_safe_filename(uploaded_file.filename)
file_path = Path("/uploads") / safe_filename
uploaded_file.save(file_path)
```

## 6. Common Path Traversal Payloads to Block

**Rule:** Be aware of common path traversal techniques and ensure your validation blocks them.

### Common Attack Payloads:
- `../../../etc/passwd`
- `..\..\..\windows\system32\config\sam`
- `....//....//....//etc/passwd` (double encoding)
- `%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd` (URL encoding)
- `..%252f..%252f..%252fetc%252fpasswd` (double URL encoding)

## 7. Additional Security Measures

### Use Chroot or Containerization
```python
# Run file operations in a chrooted environment
import os
os.chroot('/safe/uploads')
# Now all file operations are relative to this directory
```

### Implement File Type Restrictions
```python
# Only allow specific file extensions
ALLOWED_EXTENSIONS = {'.txt', '.pdf', '.jpg', '.png'}
file_ext = Path(filename).suffix.lower()
if file_ext not in ALLOWED_EXTENSIONS:
    raise ValueError("File type not allowed")
```

### Use Temporary File Handles
```python
import tempfile
import shutil

# Create temporary file with safe name
with tempfile.NamedTemporaryFile(delete=False, dir='/uploads') as tmp_file:
    tmp_filename = tmp_file.name
    # Process file safely
    shutil.move(tmp_filename, final_safe_path)
```

## 8. Testing for Path Traversal

**Rule:** Always test your file operations with path traversal payloads.

### Test Cases:
```python
# Test these inputs against your file operations
test_payloads = [
    "../../../etc/passwd",
    "..\\..\\..\\windows\\system32\\config\\sam",
    "....//....//....//etc/passwd",
    "%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd",
    "..%252f..%252f..%252fetc%252fpasswd"
]
```

Remember: **Never trust user input for file operations**. Always validate, sanitize, and verify that the final path is within your intended directory structure.