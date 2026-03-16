# VibeShield Security Rules

**Project:** VibeShield
**Description:** Canonical security rules for AI-generated code validation
**Version:** 0.1.0
**Source:** https://github.com/north-echo/vibeshield

## Overview

This document defines imperative security rules for preventing common vulnerabilities in AI-generated applications. Rules are organized by development phase and reference vulnerability IDs (V-01 through V-17) from the VibeShield taxonomy.

---

## Project Setup

### Dependency Management

Never use packages that don't exist in the target ecosystem's registry. [V-12]

Always verify package names against official registries (npm, PyPI, RubyGems, Maven Central) before including dependencies. [V-12]

Never use `eval()`, `exec()`, or equivalent dynamic code execution on untrusted input. [V-11]

Never use `pickle.loads()`, `yaml.load()`, or `Marshal.load()` on untrusted data. Use `yaml.safe_load()` or JSON instead. [V-11]

NEVER: `pickle.loads(user_input)` or `eval(request.body)` [V-11]

### Version Control

Always add `.env`, `.env.local`, `secrets.yml`, and credential files to `.gitignore` before first commit. [V-09]

Never commit API keys, database passwords, JWT secrets, or private keys to version control. [V-09]

Always scan repository history for accidentally committed secrets before making repositories public. [V-09]

NEVER: Commit files containing `DATABASE_URL=postgres://user:password@host` [V-09]

---

## Authentication & Authorization

### Identity Verification

Always implement authentication checks on the server side, never in client-side JavaScript alone. [V-04]

Never rely on hidden form fields, disabled buttons, or frontend route guards as security controls. [V-04]

NEVER: `if (localStorage.getItem('isAdmin')) { showAdminPanel(); }` with no server validation [V-04]

### Permission Validation

Always verify that the authenticated user has permission to access the requested resource, not just that they are logged in. [V-01]

Always check resource ownership before allowing read, update, or delete operations. [V-01]

Never use predictable sequential IDs (1, 2, 3) for sensitive resources without authorization checks. Use UUIDs and verify ownership. [V-01]

Always validate user roles and permissions for privileged operations (admin functions, data exports, user management). [V-01]

NEVER: `DELETE FROM posts WHERE id = ?` without verifying `posts.user_id = current_user.id` [V-01]

ALWAYS: `DELETE FROM posts WHERE id = ? AND user_id = ?` with both parameters validated [V-01]

### Credential Storage

Always hash passwords with bcrypt (cost factor 12+), argon2id, or scrypt. [V-05]

Never store passwords in plaintext, base64, or using MD5/SHA-1/SHA-256. [V-05]

Never log passwords, tokens, or credentials, even in hashed form. [V-05]

Always use constant-time comparison for HMAC, tokens, and password hashes to prevent timing attacks. [V-15]

NEVER: `hash = sha256(password)` or `if hmac == computed_hmac:` [V-05, V-15]

ALWAYS: `hash = bcrypt.hashpw(password, bcrypt.gensalt(12))` and `hmac.compare_digest(hmac, computed_hmac)` [V-05, V-15]

---

## Data Handling & Input Validation

### Input Sanitization

Always validate and sanitize all user input on the server side. [V-06]

Always use parameterized queries or ORM methods for database operations. Never concatenate user input into SQL strings. [V-06]

Always escape or sanitize user input before rendering in HTML to prevent XSS. Use framework-provided auto-escaping. [V-06]

Never insert user input directly into `innerHTML`, `dangerouslySetInnerHTML`, or equivalent without sanitization. [V-06]

NEVER: `query = f"SELECT * FROM users WHERE id = {user_id}"` [V-06]

ALWAYS: `query = "SELECT * FROM users WHERE id = ?" with cursor.execute(query, [user_id])` [V-06]

### File Operations

Always validate and sanitize file paths. Never allow user input to control path traversal. [V-16]

Always use allowlists for file operations. Reject `..`, absolute paths, and null bytes in user-supplied filenames. [V-16]

Never construct file paths using string concatenation with user input. [V-16]

NEVER: `open(f"/uploads/{user_filename}")` where user_filename could be `../../etc/passwd` [V-16]

### Command Execution

Never pass user input to shell commands via string interpolation or concatenation. [V-17]

Always use parameterized command execution with array arguments when invoking system commands. [V-17]

Always validate user input against strict allowlists before using in commands. [V-17]

NEVER: `os.system(f"convert {user_file} output.png")` or `exec("ls " + user_dir)` [V-17]

ALWAYS: `subprocess.run(["convert", user_file, "output.png"], check=True)` with validation [V-17]

---

## API & Network Security

### Server-Side Request Forgery (SSRF)

Never make HTTP requests to URLs directly supplied by users without validation. [V-03]

Always validate URLs against an allowlist of permitted domains or protocols. [V-03]

Always block requests to private IP ranges (127.0.0.0/8, 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 169.254.0.0/16) and metadata endpoints (169.254.169.254). [V-03]

Never follow redirects from user-supplied URLs without revalidating the destination. [V-03]

NEVER: `requests.get(user_url)` without validation [V-03]

ALWAYS: Validate against allowlist, block private IPs, disable redirects or revalidate [V-03]

### Cross-Origin Resource Sharing (CORS)

Never use `Access-Control-Allow-Origin: *` with `Access-Control-Allow-Credentials: true`. [V-10]

Always specify exact allowed origins for credentialed requests. [V-10]

Always validate the Origin header against an allowlist when setting CORS headers dynamically. [V-10]

NEVER: `Access-Control-Allow-Origin: *` with cookies or Authorization headers [V-10]

### Security Headers

Always implement CSRF protection for state-changing operations (POST, PUT, DELETE). Use SameSite cookies and CSRF tokens. [V-08]

Always set security headers: `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Strict-Transport-Security`. [V-08]

Always implement a Content Security Policy (CSP) that restricts script sources and blocks inline scripts where possible. [V-08]

Always implement rate limiting on authentication endpoints, password reset, and public APIs. [V-08]

---

## Secrets & Configuration

### Secret Management

Never hardcode secrets in source code. Always load from environment variables or secure secret management systems. [V-02]

Never commit secrets to version control. Use `.env` files locally and secure vaults in production. [V-02]

Always rotate secrets immediately if they are exposed in code, logs, or version control. [V-02]

Always use cryptographically random values for JWT secrets, session keys, and API tokens (minimum 256 bits). [V-02]

NEVER: `JWT_SECRET = "mysecret123"` or `API_KEY = "12345"` in source code [V-02]

ALWAYS: `JWT_SECRET = os.environ.get('JWT_SECRET')` with secret generated via `openssl rand -hex 32` [V-02]

### Environment Configuration

Always use different secrets for development, staging, and production environments. [V-02]

Never include production credentials in code repositories, even in commented-out sections. [V-02]

Always validate that required environment variables are set at application startup. [V-02]

---

## Error Handling & Logging

### Error Disclosure

Never expose stack traces, database errors, or system paths to end users in production. [V-14]

Always log detailed errors server-side but return generic error messages to clients. [V-14]

Never include sensitive data (passwords, tokens, PII) in error messages. [V-14]

### Security Logging

Always log authentication events (login, logout, failed attempts, password changes). [V-13]

Always log authorization failures (attempted access to unauthorized resources). [V-13]

Always log security-relevant events (admin actions, permission changes, data exports). [V-13]

Never log credentials, tokens, or sensitive user data. [V-13]

Always include timestamps, user identifiers, IP addresses, and resource identifiers in security logs. [V-13]

---

## Business Logic

### Data Validation

Always validate business logic constraints server-side: positive quantities, valid price ranges, maximum order limits. [V-07]

Never trust client-side validation alone for business rules. [V-07]

Always validate that numeric inputs are within expected ranges and not negative where inappropriate. [V-07]

Always check for integer overflow when performing arithmetic on user-supplied values. [V-07]

NEVER: Accept `quantity = -5` or `price = -100` without validation [V-07]

ALWAYS: Validate `quantity > 0`, `price >= min_price`, `total = price * quantity < max_allowed` [V-07]

### State Management

Always validate state transitions in workflows (order status, payment status, approval flows). [V-07]

Never allow users to skip required steps by manipulating request parameters. [V-07]

Always verify prerequisites before allowing state changes (payment before shipping, approval before access). [V-07]

---

## Implementation Notes

### Rule Priority

Rules are categorized by vulnerability tier:
- **Tier 1 (V-01 to V-04):** Near-universal AI failures, highest priority
- **Tier 2 (V-05 to V-10):** High-frequency vulnerabilities, critical priority
- **Tier 3 (V-11 to V-17):** Context-dependent, important priority

### Language-Specific Adaptations

When implementing these rules in tooling:
- Translate examples to target language syntax
- Use language-specific secure functions (e.g., `PreparedStatement` in Java, `?` placeholders in Python sqlite3)
- Reference framework-specific protections (Django ORM, Rails Strong Parameters)
- Adapt file paths and environment variable access to platform conventions

### Coverage

These rules address the most common vulnerabilities in AI-generated code. They do not replace comprehensive security review, penetration testing, or defense-in-depth strategies.

---

**End of VibeShield Security Rules v0.1.0**
