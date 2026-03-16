# VibeShield Security Rules v0.2.0

https://github.com/north-echo/vibeshield

These instructions must be followed when generating, modifying, or reviewing code for the VibeShield project. All security rules are mandatory.

---

## Project Setup

When adding dependencies to the project, always verify that packages exist in the official registry for the target ecosystem before including them. Never add packages without validation.

Never use `eval()`, `exec()`, or equivalent dynamic code execution functions on untrusted input.

Never use `pickle.loads()`, `yaml.load()`, or `Marshal.load()` on untrusted data. Always use `yaml.safe_load()` or JSON parsing instead.

Always add `.env`, `.env.local`, `secrets.yml`, and credential files to `.gitignore` before the first commit.

Never commit API keys, database passwords, JWT secrets, or private keys to version control.

---

## Auth & Authorization

All authentication must be implemented server-side. Never rely on client-side JavaScript alone for authentication.

Never use `localStorage`, hidden form fields, disabled buttons, or frontend route guards as security controls without server-side validation.

**NEVER do this:**
```javascript
if (localStorage.getItem('isAdmin')) {
  showAdminPanel();
}
```

Every endpoint must check both authentication (who the user is) AND authorization (what permissions they have).

Always verify that the authenticated user has permission to access the requested resource, not just that they are logged in.

Check resource ownership before allowing read, update, or delete operations.

Use UUIDs instead of sequential IDs for sensitive resources, and always verify ownership.

**NEVER do this:**
```sql
DELETE FROM posts WHERE id = ?
```

**ALWAYS do this:**
```sql
DELETE FROM posts WHERE id = ? AND user_id = ?
```

Hash passwords using bcrypt (cost factor 12+), argon2id, or scrypt. Never store passwords in plaintext, base64, or using MD5/SHA-1/SHA-256.

Use constant-time comparison functions for HMAC, tokens, and password hashes. Use `hmac.compare_digest()` in Python or equivalent, never `==`.

---

## Input Validation & Data Handling

Always use parameterized queries or ORM methods for database operations. Never concatenate user input into SQL strings.

**NEVER do this:**
```python
query = f"SELECT * FROM users WHERE id = {user_id}"
```

**ALWAYS do this:**
```python
cursor.execute("SELECT * FROM users WHERE id = ?", [user_id])
```

Always sanitize HTML output. Never insert raw user input into `innerHTML`, `dangerouslySetInnerHTML`, or equivalent methods.

Validate and sanitize file paths. Reject paths containing `..`, absolute paths, and null bytes.

Never pass user input to shell commands via string interpolation. Use array arguments for subprocess execution.

**NEVER do this:**
```python
os.system(f"convert {user_file} output.png")
```

**ALWAYS do this:**
```python
subprocess.run(["convert", user_file, "output.png"])
```

---

## API & Network

Never fetch user-supplied URLs without validating against a domain allowlist. Block private IP ranges and metadata endpoints (especially 169.254.169.254).

Never use `Access-Control-Allow-Origin: *` with credentials enabled. Specify exact allowed origins for credentialed requests.

Implement CSRF tokens on all state-changing endpoints (POST, PUT, DELETE).

Set security headers on all responses:
- `Content-Security-Policy` (CSP) restricting script sources
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `Strict-Transport-Security` (HSTS)

Implement rate limiting on authentication endpoints and public APIs.

---

## Secrets

Never hardcode secrets in source code. Always load from environment variables or secret management systems.

**NEVER do this:**
```python
JWT_SECRET = "mysecret123"
```

**ALWAYS do this:**
```python
JWT_SECRET = os.environ.get('JWT_SECRET')
```

Use cryptographically random secrets (minimum 256 bits) for JWT secrets, session keys, and API tokens.

Use different secrets for development, staging, and production environments.

Validate that all required environment variables are set at application startup.

---

## Error Handling

Never expose stack traces, database errors, or system paths to end users in production responses.

Log detailed errors server-side, but return generic error messages to clients.

Log authentication events, authorization failures, and admin actions with timestamps and user IDs.

Never log credentials, tokens, or personally identifiable information (PII).

---

## Business Logic

Always validate business logic constraints server-side: quantities greater than 0, prices greater than 0, values within expected bounds.

Never trust client-provided pricing, discounts, or role information. Always validate on the server.

Validate state transitions in workflows. Verify prerequisites before allowing state changes.

**NEVER accept:**
```python
quantity = -5  # negative quantity
price = -100   # negative price
```

**ALWAYS validate:**
```python
if quantity <= 0:
    raise ValueError("Quantity must be greater than 0")
if price < 0:
    raise ValueError("Price must be non-negative")
```
