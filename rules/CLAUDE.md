# VibeShield Security Rules v1.0.0

This file contains security rules for the VibeShield project. You MUST follow these rules when generating, modifying, or reviewing code.

## Project Setup

Never use packages that don't exist in the target ecosystem's registry. Verify package names against official registries before including dependencies.

Never use eval(), exec(), or equivalent dynamic code execution on untrusted input.

Never use pickle.loads(), yaml.load(), or Marshal.load() on untrusted data. Use yaml.safe_load() or JSON instead.

Add .env, .env.local, secrets.yml, and credential files to .gitignore before first commit.

Never commit API keys, database passwords, JWT secrets, or private keys to version control.

## Authentication & Authorization

Implement authentication checks on the server side, never in client-side JavaScript alone.

Never rely on hidden form fields, disabled buttons, or frontend route guards as security controls.

NEVER: if (localStorage.getItem('isAdmin')) { showAdminPanel(); } with no server validation

Verify that the authenticated user has permission to access the requested resource, not just that they are logged in.

Check resource ownership before allowing read, update, or delete operations.

Never use predictable sequential IDs for sensitive resources without authorization checks. Use UUIDs and verify ownership.

Validate user roles and permissions for privileged operations.

NEVER: DELETE FROM posts WHERE id = ? without verifying posts.user_id = current_user.id

ALWAYS: DELETE FROM posts WHERE id = ? AND user_id = ? with both parameters validated

Hash passwords with bcrypt (cost factor 12+), argon2id, or scrypt.

Never store passwords in plaintext, base64, or using MD5/SHA-1/SHA-256.

Use constant-time comparison for HMAC, tokens, and password hashes.

## Data Handling & Input Validation

Validate and sanitize all user input on the server side.

Use parameterized queries or ORM methods for database operations. Never concatenate user input into SQL strings.

Escape or sanitize user input before rendering in HTML. Use framework-provided auto-escaping.

Never insert user input directly into innerHTML, dangerouslySetInnerHTML, or equivalent.

NEVER: query = f"SELECT * FROM users WHERE id = {user_id}"

ALWAYS: cursor.execute("SELECT * FROM users WHERE id = ?", [user_id])

Validate and sanitize file paths. Never allow user input to control path traversal.

Reject .., absolute paths, and null bytes in user-supplied filenames.

Never pass user input to shell commands via string interpolation.

Use parameterized command execution with array arguments.

NEVER: os.system(f"convert {user_file} output.png")

ALWAYS: subprocess.run(["convert", user_file, "output.png"], check=True)

## API & Network Security

Never make HTTP requests to URLs directly supplied by users without validation.

Validate URLs against an allowlist of permitted domains.

Block requests to private IP ranges and metadata endpoints (169.254.169.254).

Never follow redirects from user-supplied URLs without revalidating.

Never use Access-Control-Allow-Origin: * with Access-Control-Allow-Credentials: true.

Specify exact allowed origins for credentialed requests.

Implement CSRF protection for state-changing operations. Use SameSite cookies and CSRF tokens.

Set security headers: X-Content-Type-Options: nosniff, X-Frame-Options: DENY, Strict-Transport-Security.

Implement Content Security Policy (CSP) restricting script sources.

Implement rate limiting on auth endpoints, password reset, and public APIs.

## Secrets & Configuration

Never hardcode secrets in source code. Load from environment variables or secret management systems.

Use cryptographically random values for JWT secrets, session keys, API tokens (minimum 256 bits).

NEVER: JWT_SECRET = "mysecret123" in source code

ALWAYS: JWT_SECRET = os.environ.get('JWT_SECRET')

Use different secrets for dev, staging, and production.

Validate that required environment variables are set at startup.

## Error Handling & Logging

Never expose stack traces, database errors, or system paths to end users in production.

Log detailed errors server-side, return generic messages to clients.

Log authentication events, authorization failures, and admin actions.

Never log credentials, tokens, or sensitive user data.

Include timestamps, user identifiers, IP addresses, and resource identifiers in security logs.

## Business Logic

Validate business logic constraints server-side: positive quantities, valid price ranges, maximum limits.

Never trust client-side validation alone for business rules.

Validate numeric inputs are within expected ranges.

NEVER: Accept quantity = -5 or price = -100 without validation

ALWAYS: Validate quantity > 0, price >= min_price

Validate state transitions in workflows.

Verify prerequisites before allowing state changes.
