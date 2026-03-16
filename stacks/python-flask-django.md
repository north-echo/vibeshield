# Python Flask & Django Security Rules

**Project:** VibeShield
**Stack:** Python Web Frameworks (Flask & Django)
**Version:** 0.1.0

## Overview

This document defines imperative security rules for Python web applications built with Flask and Django. AI code generators frequently introduce critical misconfigurations and insecure patterns in these frameworks.

---

## Django Security Configuration

### Debug Mode

Never set `DEBUG = True` in production. AI tools frequently generate this insecure default. [V-02, V-14]

Always set `DEBUG = False` in production environments. [V-02, V-14]

Always verify debug mode is disabled before deployment. [V-02, V-14]

NEVER: `DEBUG = True` in production settings [V-14]

ALWAYS: `DEBUG = False` or `DEBUG = os.environ.get('DEBUG', 'False') == 'True'` [V-02]

### Allowed Hosts

Never use `ALLOWED_HOSTS = ['*']` in production. This disables host header validation. [V-08]

Always set `ALLOWED_HOSTS` to an explicit list of permitted domains. [V-08]

Never deploy with an empty `ALLOWED_HOSTS` list. [V-08]

NEVER: `ALLOWED_HOSTS = ['*']` [V-08]

ALWAYS: `ALLOWED_HOSTS = ['example.com', 'www.example.com']` or load from environment [V-08]

### Secret Key Management

Never use the default Django-generated `SECRET_KEY` in production. [V-02]

Never commit `SECRET_KEY` to version control. [V-02, V-09]

Always load `SECRET_KEY` from environment variables. [V-02]

Always use a cryptographically random secret key (minimum 50 characters). [V-02]

NEVER: `SECRET_KEY = 'django-insecure-abc123...'` in committed code [V-02, V-09]

ALWAYS:
```python
import os
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')
if not SECRET_KEY:
    raise ValueError("DJANGO_SECRET_KEY environment variable must be set")
```

Generate with: `python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'` [V-02]

### HTTPS & Security Headers

Always enable `SECURE_SSL_REDIRECT = True` in production to enforce HTTPS. [V-08]

Always set `CSRF_COOKIE_SECURE = True` to prevent CSRF token transmission over HTTP. [V-08]

Always set `SESSION_COOKIE_SECURE = True` to prevent session hijacking. [V-08]

Always set `SECURE_HSTS_SECONDS` to at least 31536000 (1 year) for HSTS. [V-08]

Always set `SECURE_HSTS_INCLUDE_SUBDOMAINS = True` for comprehensive HSTS coverage. [V-08]

Always set `SESSION_COOKIE_HTTPONLY = True` to prevent JavaScript access to session cookies. [V-08]

Always set `CSRF_COOKIE_HTTPONLY = True` where possible. [V-08]

ALWAYS in production settings:
```python
SECURE_SSL_REDIRECT = True
CSRF_COOKIE_SECURE = True
SESSION_COOKIE_SECURE = True
SESSION_COOKIE_HTTPONLY = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_BROWSER_XSS_FILTER = True
X_FRAME_OPTIONS = 'DENY'
```

### CSRF Protection

Always use Django's built-in CSRF protection. [V-08]

Never use `@csrf_exempt` decorator without a specific, documented security reason. [V-08]

Always include `{% csrf_token %}` in all forms that perform state-changing operations. [V-08]

Always verify `CsrfViewMiddleware` is enabled in `MIDDLEWARE` settings. [V-08]

NEVER: `@csrf_exempt` on views handling user data without documented justification [V-08]

ALWAYS: Use CSRF tokens and only exempt third-party webhook endpoints with alternative authentication [V-08]

### Password Hashing

Always configure strong password hashers. Django uses PBKDF2 by default (acceptable) but Argon2 is preferred. [V-05]

Always set `PASSWORD_HASHERS` with Argon2 as the first choice. [V-05]

Never implement custom password hashing. Use Django's authentication system. [V-05]

ALWAYS:
```python
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
]
```

Install: `pip install argon2-cffi` [V-05]

---

## Flask Security Configuration

### Secret Key

Never hardcode `app.secret_key` in source code. [V-02, V-09]

Always load `app.secret_key` from environment variables. [V-02]

Always use a cryptographically random secret key (minimum 32 bytes). [V-02]

Never commit secret keys to version control. [V-02, V-09]

NEVER: `app.secret_key = 'dev'` or `app.secret_key = 'mysecret123'` [V-02]

ALWAYS:
```python
import os
app.secret_key = os.environ.get('FLASK_SECRET_KEY')
if not app.secret_key:
    raise ValueError("FLASK_SECRET_KEY environment variable must be set")
```

Generate with: `python -c 'import secrets; print(secrets.token_hex(32))'` [V-02]

### CSRF Protection

Never rely on Flask's default configuration. Flask has NO built-in CSRF protection. [V-08]

Always use Flask-WTF or Flask-SeaSurf for CSRF protection. [V-08]

Always enable CSRF protection globally, not per-form. [V-08]

ALWAYS with Flask-WTF:
```python
from flask_wtf.csrf import CSRFProtect

csrf = CSRFProtect(app)
```

Include in templates: `<input type="hidden" name="csrf_token" value="{{ csrf_token() }}"/>` [V-08]

### Security Headers

Always use Flask-Talisman for security headers (CSP, HSTS, etc.). [V-08]

Always configure Content Security Policy to restrict script sources. [V-08]

Always enable HTTPS enforcement in production. [V-08]

ALWAYS:
```python
from flask_talisman import Talisman

Talisman(app,
    force_https=True,
    strict_transport_security=True,
    strict_transport_security_max_age=31536000,
    content_security_policy={
        'default-src': "'self'",
        'script-src': "'self'",
        'style-src': "'self'",
    }
)
```

### Session Security

Always set `SESSION_COOKIE_SECURE = True` in production. [V-08]

Always set `SESSION_COOKIE_HTTPONLY = True` to prevent XSS attacks on sessions. [V-08]

Always set `SESSION_COOKIE_SAMESITE = 'Lax'` or `'Strict'` for CSRF protection. [V-08]

Always set `PERMANENT_SESSION_LIFETIME` to a reasonable duration (not indefinite). [V-08]

ALWAYS:
```python
app.config.update(
    SESSION_COOKIE_SECURE=True,
    SESSION_COOKIE_HTTPONLY=True,
    SESSION_COOKIE_SAMESITE='Lax',
    PERMANENT_SESSION_LIFETIME=3600  # 1 hour
)
```

---

## Common Python Security Rules

### Insecure Deserialization

Never use `pickle.loads()` on untrusted data. Pickle can execute arbitrary code. [V-11]

Never use `yaml.load()`. Always use `yaml.safe_load()`. [V-11]

Never use `eval()` or `exec()` on user input or external data. [V-11]

Always use JSON for data serialization when possible. [V-11]

NEVER:
```python
import pickle
data = pickle.loads(request.data)  # DANGEROUS

import yaml
config = yaml.load(user_file)  # DANGEROUS

result = eval(user_expression)  # DANGEROUS
```

ALWAYS:
```python
import json
data = json.loads(request.data)

import yaml
config = yaml.safe_load(user_file)
```

### Command Injection

Never use `os.system()` with user input. [V-17]

Never use `subprocess.call(shell=True)` or `subprocess.Popen(shell=True)` with user input. [V-17]

Always use `subprocess.run()` with list arguments (shell=False, the default). [V-17]

Always validate user input against strict allowlists before using in commands. [V-17]

NEVER:
```python
os.system(f"convert {user_file} output.png")
subprocess.call(f"ls {user_dir}", shell=True)
```

ALWAYS:
```python
import subprocess
subprocess.run(["convert", user_file, "output.png"], check=True)
subprocess.run(["ls", user_dir], check=True)
```

With validation:
```python
import re
if not re.match(r'^[a-zA-Z0-9_-]+\.png$', user_file):
    raise ValueError("Invalid filename")
subprocess.run(["convert", user_file, "output.png"], check=True)
```

### Cryptographic Operations

Never use `random` module for security-sensitive operations (tokens, keys, nonces). [V-02]

Always use `secrets` module for generating tokens and random values. [V-02]

Always use `hmac.compare_digest()` for constant-time comparison of secrets. [V-15]

Never use `==` to compare HMAC values, tokens, or password hashes. [V-15]

NEVER:
```python
import random
token = ''.join(random.choices('0123456789abcdef', k=32))  # WEAK

if user_hmac == computed_hmac:  # TIMING ATTACK
    pass
```

ALWAYS:
```python
import secrets
token = secrets.token_hex(32)
# or
token = secrets.token_urlsafe(32)

import hmac
if hmac.compare_digest(user_hmac, computed_hmac):  # CONSTANT TIME
    pass
```

### SQL Injection Prevention

Always use parameterized queries with `cursor.execute(sql, params)`. [V-06]

Never concatenate user input into SQL strings using f-strings or `+`. [V-06]

SQLAlchemy ORM is safe by default. Raw `text()` queries need bound parameters. [V-06]

Always use bound parameters, never string formatting. [V-06]

NEVER:
```python
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
cursor.execute("SELECT * FROM users WHERE name = '" + username + "'")

# SQLAlchemy
from sqlalchemy import text
session.execute(text(f"SELECT * FROM users WHERE id = {user_id}"))
```

ALWAYS:
```python
cursor.execute("SELECT * FROM users WHERE id = ?", [user_id])
cursor.execute("SELECT * FROM users WHERE name = %s", [username])

# SQLAlchemy ORM (safe by default)
user = session.query(User).filter_by(id=user_id).first()

# SQLAlchemy raw queries with bound parameters
from sqlalchemy import text
session.execute(text("SELECT * FROM users WHERE id = :id"), {"id": user_id})
```

### Path Traversal Prevention

Always validate and sanitize file paths from user input. [V-16]

Always use `pathlib.Path.resolve()` to canonicalize paths and check against base directory. [V-16]

Never construct file paths using string concatenation with user input. [V-16]

Always reject paths containing `..`, absolute paths, and null bytes. [V-16]

NEVER:
```python
file_path = f"/uploads/{user_filename}"
with open(file_path) as f:
    content = f.read()
```

ALWAYS:
```python
from pathlib import Path

base_dir = Path("/uploads").resolve()
user_path = (base_dir / user_filename).resolve()

if not user_path.is_relative_to(base_dir):
    raise ValueError("Invalid file path")

with open(user_path) as f:
    content = f.read()
```

Or with validation:
```python
import os

# Reject dangerous patterns
if '..' in user_filename or user_filename.startswith('/') or '\x00' in user_filename:
    raise ValueError("Invalid filename")

file_path = os.path.join('/uploads', user_filename)
real_path = os.path.realpath(file_path)

if not real_path.startswith('/uploads/'):
    raise ValueError("Path traversal detected")

with open(real_path) as f:
    content = f.read()
```

### Password Hashing (Flask & General)

Always use `bcrypt` or `argon2-cffi` for password hashing. [V-05]

Never use MD5, SHA-1, SHA-256, or plain hashing for passwords. [V-05]

Never store passwords in plaintext or reversible encoding. [V-05]

Always use a work factor/cost parameter: bcrypt cost 12+, Argon2 default parameters. [V-05]

NEVER:
```python
import hashlib
password_hash = hashlib.sha256(password.encode()).hexdigest()  # INSECURE
```

ALWAYS with bcrypt:
```python
import bcrypt

# Hashing
password_hash = bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt(rounds=12))

# Verification
if bcrypt.checkpw(password.encode('utf-8'), stored_hash):
    # Password correct
    pass
```

ALWAYS with Argon2:
```python
from argon2 import PasswordHasher

ph = PasswordHasher()

# Hashing
password_hash = ph.hash(password)

# Verification
try:
    ph.verify(password_hash, password)
    # Password correct
except:
    # Password incorrect
    pass
```

---

## Framework-Specific Vulnerabilities

### Django ORM

Always use Django ORM methods. They prevent SQL injection by default. [V-06]

Never use `.raw()` or `.extra()` with user input without parameterization. [V-06]

Always use parameterized queries with `.raw()`: `User.objects.raw('SELECT * FROM users WHERE id = %s', [user_id])` [V-06]

### Django Templates

Always use Django's template auto-escaping. It is enabled by default. [V-06]

Never use `mark_safe()` or `{% autoescape off %}` with user input. [V-06]

Never disable auto-escaping globally. [V-06]

### Flask Templates

Always use Jinja2 auto-escaping. It is enabled by default in Flask. [V-06]

Never use `| safe` filter with user input. [V-06]

Never render user input with `Markup()` without sanitization. [V-06]

### Django Mass Assignment

Always use `fields` or `exclude` in ModelForms to prevent mass assignment. [V-07]

Never allow unrestricted field assignment from request data. [V-07]

NEVER:
```python
class UserForm(forms.ModelForm):
    class Meta:
        model = User
        fields = '__all__'  # DANGEROUS if User has is_admin field
```

ALWAYS:
```python
class UserForm(forms.ModelForm):
    class Meta:
        model = User
        fields = ['username', 'email', 'first_name', 'last_name']
```

---

## Implementation Notes

### Django Security Checklist

Before deployment, verify:
- `DEBUG = False`
- `ALLOWED_HOSTS` configured
- `SECRET_KEY` from environment, not committed
- HTTPS enforcement enabled (`SECURE_SSL_REDIRECT`)
- Secure cookie flags set (`SESSION_COOKIE_SECURE`, `CSRF_COOKIE_SECURE`)
- HSTS configured (`SECURE_HSTS_SECONDS`)
- Strong password hashers configured (Argon2)
- CSRF protection enabled (default middleware)

### Flask Security Checklist

Before deployment, verify:
- `app.secret_key` from environment, not committed
- Flask-WTF or Flask-SeaSurf installed and configured
- Flask-Talisman installed and configured
- Secure cookie flags set (`SESSION_COOKIE_SECURE`, `SESSION_COOKIE_HTTPONLY`)
- HTTPS enforcement enabled
- CSRF protection enabled globally

### Testing

Always test authentication and authorization with automated tests. [V-01, V-04]

Always test CSRF protection on state-changing endpoints. [V-08]

Always test with security scanning tools (Bandit for Python, safety for dependencies). [V-12]

Run: `bandit -r .` and `safety check` before deployment.

---

**End of Python Flask & Django Security Rules v0.1.0**
