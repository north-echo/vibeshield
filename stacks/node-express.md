# VibeShield Security Rules: Node.js / Express Stack

**Project:** VibeShield
**Description:** Node.js and Express-specific security rules for AI-generated code validation
**Version:** 0.1.0
**Supplement to:** vibeshield-rules.md

## Overview

This document provides Node.js and Express-specific security rules that complement the core VibeShield rules. Express is a minimal framework, and AI tools frequently generate insecure configurations by omitting critical security middleware and using dangerous defaults.

---

## Security Middleware

### Helmet Configuration

Always use `helmet()` middleware for security headers. [V-08]

Never rely on default helmet configuration alone. Configure Content Security Policy explicitly. [V-08]

Always place helmet middleware early in the middleware chain, before route handlers. [V-08]

NEVER: Missing helmet or default-only configuration

```javascript
const express = require('express')
const app = express()

// No helmet middleware at all
app.get('/api/data', (req, res) => { /* ... */ })
```

ALWAYS: Explicit helmet configuration

```javascript
const express = require('express')
const helmet = require('helmet')
const app = express()

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],  // Only if necessary
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  },
}))

app.get('/api/data', (req, res) => { /* ... */ })
```

### Rate Limiting

Always use `express-rate-limit` on authentication endpoints and public APIs. [V-08]

Never use default rate limit values in production. Set specific limits based on use case. [V-08]

Always apply stricter rate limits to authentication, password reset, and registration endpoints. [V-08]

NEVER: No rate limiting or default limits

```javascript
// No rate limiting at all
app.post('/api/login', async (req, res) => {
  // Vulnerable to brute force
})
```

ALWAYS: Explicit rate limiting

```javascript
const rateLimit = require('express-rate-limit')

// General API rate limit
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,  // Limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP',
  standardHeaders: true,
  legacyHeaders: false,
})

// Strict auth rate limit
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,  // Only 5 login attempts per 15 minutes
  message: 'Too many login attempts, please try again later',
  skipSuccessfulRequests: true,
})

app.use('/api/', apiLimiter)
app.post('/api/login', authLimiter, async (req, res) => { /* ... */ })
app.post('/api/register', authLimiter, async (req, res) => { /* ... */ })
app.post('/api/password-reset', authLimiter, async (req, res) => { /* ... */ })
```

---

## Request Handling

### Body Parsing

Always set size limits for request bodies. Never accept unbounded input. [V-08]

Never use `express.json()` or `express.urlencoded()` without a limit parameter. [V-08]

Always set appropriate limits based on use case (smaller for APIs, larger only for file uploads). [V-08]

NEVER: Unbounded body parsing

```javascript
app.use(express.json())  // No limit - vulnerable to DoS
app.use(express.urlencoded({ extended: true }))  // No limit
```

ALWAYS: Size-limited body parsing

```javascript
app.use(express.json({ limit: '10kb' }))  // 10kb for API endpoints
app.use(express.urlencoded({ extended: true, limit: '10kb' }))

// Larger limit only for specific upload endpoints
app.post('/api/upload', express.json({ limit: '5mb' }), uploadHandler)
```

### CORS Configuration

Always use the `cors` package with explicit origin configuration. [V-10]

Never use `cors()` with no arguments. This allows all origins. [V-10]

Never use `Access-Control-Allow-Origin: *` with credentials. [V-10]

Always validate origins against an allowlist when setting CORS headers dynamically. [V-10]

NEVER: Permissive CORS

```javascript
const cors = require('cors')

app.use(cors())  // Allows ALL origins - dangerous

// Or even worse:
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*')
  res.header('Access-Control-Allow-Credentials', 'true')  // CRITICAL VULNERABILITY
  next()
})
```

ALWAYS: Explicit origin allowlist

```javascript
const cors = require('cors')

const allowedOrigins = [
  'https://example.com',
  'https://app.example.com',
  process.env.NODE_ENV === 'development' ? 'http://localhost:3000' : null
].filter(Boolean)

const corsOptions = {
  origin: function (origin, callback) {
    // Allow requests with no origin (mobile apps, Postman, etc.)
    if (!origin) return callback(null, true)

    if (allowedOrigins.indexOf(origin) === -1) {
      return callback(new Error('Not allowed by CORS'), false)
    }
    return callback(null, true)
  },
  credentials: true,
  optionsSuccessStatus: 200
}

app.use(cors(corsOptions))
```

---

## Session Management

### Session Configuration

Always use `express-session` with secure cookie settings. [V-08]

Always set `httpOnly: true`, `secure: true`, and `sameSite: 'strict'` for session cookies. [V-08]

Never use the default MemoryStore in production. Always use a persistent store (Redis, database). [V-08]

Always set a strong, random session secret from environment variables. [V-02]

NEVER: Insecure session configuration

```javascript
const session = require('express-session')

app.use(session({
  secret: 'keyboard cat',  // Hardcoded, weak secret
  resave: false,
  saveUninitialized: true,
  // Missing cookie configuration - uses insecure defaults
  // Using MemoryStore (default) - sessions lost on restart
}))
```

ALWAYS: Secure session configuration

```javascript
const session = require('express-session')
const RedisStore = require('connect-redis')(session)
const redis = require('redis')

const redisClient = redis.createClient({
  url: process.env.REDIS_URL
})

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,  // From environment, generated via openssl rand -hex 32
  resave: false,
  saveUninitialized: false,
  name: 'sessionId',  // Don't use default 'connect.sid'
  cookie: {
    secure: process.env.NODE_ENV === 'production',  // true in production (HTTPS only)
    httpOnly: true,  // Prevents XSS access to cookie
    sameSite: 'strict',  // CSRF protection
    maxAge: 1000 * 60 * 60 * 24,  // 24 hours
  },
  proxy: process.env.NODE_ENV === 'production'  // Trust proxy in production
}))
```

### Session Security

Always regenerate session IDs after login to prevent session fixation. [V-08]

Always destroy sessions on logout. [V-08]

Always implement session timeout and idle timeout. [V-08]

NEVER: No session regeneration

```javascript
app.post('/api/login', async (req, res) => {
  const user = await authenticateUser(req.body)
  if (user) {
    req.session.userId = user.id  // Session ID not regenerated - fixation risk
    res.json({ success: true })
  }
})
```

ALWAYS: Regenerate session on login

```javascript
app.post('/api/login', async (req, res) => {
  const user = await authenticateUser(req.body)
  if (user) {
    req.session.regenerate((err) => {
      if (err) return res.status(500).json({ error: 'Login failed' })

      req.session.userId = user.id
      req.session.loginTime = Date.now()
      res.json({ success: true })
    })
  } else {
    res.status(401).json({ error: 'Invalid credentials' })
  }
})

app.post('/api/logout', (req, res) => {
  req.session.destroy((err) => {
    if (err) return res.status(500).json({ error: 'Logout failed' })
    res.clearCookie('sessionId')
    res.json({ success: true })
  })
})
```

---

## Input Validation

### Request Validation

Always use `express-validator`, `zod`, or equivalent for request validation. [V-06]

Always validate params, query, body, and headers. [V-06]

Never trust any input without validation, even from authenticated users. [V-06]

Always sanitize input to prevent injection attacks. [V-06]

NEVER: Unvalidated input

```javascript
app.post('/api/users/:id/update', async (req, res) => {
  const userId = req.params.id  // No validation
  const { email, age } = req.body  // No validation

  await db.query('UPDATE users SET email = ?, age = ? WHERE id = ?',
    [email, age, userId])  // Vulnerable to injection if db.query isn't properly parameterized
})
```

ALWAYS: Comprehensive validation

```javascript
const { body, param, validationResult } = require('express-validator')

app.post('/api/users/:id/update',
  // Validation rules
  param('id').isUUID(),
  body('email').isEmail().normalizeEmail(),
  body('age').isInt({ min: 0, max: 120 }),

  async (req, res) => {
    // Check validation results
    const errors = validationResult(req)
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() })
    }

    const userId = req.params.id
    const { email, age } = req.body

    // Validated and sanitized input
    await db.query('UPDATE users SET email = ?, age = ? WHERE id = ?',
      [email, age, userId])

    res.json({ success: true })
  }
)
```

Using Zod:

```javascript
const { z } = require('zod')

const UpdateUserSchema = z.object({
  params: z.object({
    id: z.string().uuid(),
  }),
  body: z.object({
    email: z.string().email(),
    age: z.number().int().min(0).max(120),
  }),
})

app.post('/api/users/:id/update', async (req, res) => {
  try {
    const validated = UpdateUserSchema.parse({
      params: req.params,
      body: req.body,
    })

    await db.query('UPDATE users SET email = ?, age = ? WHERE id = ?',
      [validated.body.email, validated.body.age, validated.params.id])

    res.json({ success: true })
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({ errors: error.errors })
    }
    throw error
  }
})
```

---

## Error Handling

### Error Middleware

Always use a centralized error handler. [V-14]

Never send `err.stack` or detailed `err.message` directly to clients in production. [V-14]

Always set `NODE_ENV=production` in production. [V-14]

Always log full errors server-side but return generic messages to clients. [V-14]

NEVER: Exposed error details

```javascript
app.get('/api/data', async (req, res) => {
  try {
    const data = await fetchData()
    res.json(data)
  } catch (err) {
    res.status(500).json({
      error: err.message,  // Exposes internal errors
      stack: err.stack  // Exposes code structure
    })
  }
})
```

ALWAYS: Centralized error handler

```javascript
// Error handling middleware (must be last)
app.use((err, req, res, next) => {
  // Log full error server-side
  console.error('Error:', {
    message: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method,
    ip: req.ip,
    userId: req.session?.userId,
  })

  // Send generic error to client in production
  if (process.env.NODE_ENV === 'production') {
    res.status(err.status || 500).json({
      error: 'An error occurred processing your request'
    })
  } else {
    // Development: send detailed error
    res.status(err.status || 500).json({
      error: err.message,
      stack: err.stack
    })
  }
})

// Route handlers
app.get('/api/data', async (req, res, next) => {
  try {
    const data = await fetchData()
    res.json(data)
  } catch (err) {
    next(err)  // Pass to error handler
  }
})
```

### Async Error Handling

Always handle promise rejections in async route handlers. [V-14]

Always use `try-catch` or wrapper functions for async routes. [V-14]

Never leave async routes without error handling. Unhandled rejections crash the server. [V-14]

ALWAYS: Async wrapper or try-catch

```javascript
// Option 1: Async wrapper
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next)
}

app.get('/api/data', asyncHandler(async (req, res) => {
  const data = await fetchData()
  res.json(data)
}))

// Option 2: Explicit try-catch
app.get('/api/data', async (req, res, next) => {
  try {
    const data = await fetchData()
    res.json(data)
  } catch (err) {
    next(err)
  }
})
```

---

## Dependency Security

### Dependency Management

Always run `npm audit` regularly. [V-12]

Never ignore high or critical vulnerabilities. [V-12]

Always pin dependency versions in `package-lock.json`. Commit it to version control. [V-12]

Always use `npm ci` in production instead of `npm install` to ensure reproducible builds. [V-12]

NEVER: Ignored vulnerabilities

```bash
npm audit
# 12 vulnerabilities (3 low, 5 moderate, 4 high)
# "We'll fix it later" - never gets fixed
```

ALWAYS: Regular audits and updates

```bash
# Check for vulnerabilities
npm audit

# Fix automatically fixable issues
npm audit fix

# For breaking changes, update manually
npm update package-name

# In CI/CD
npm ci  # Uses package-lock.json exactly
npm audit --audit-level=high  # Fail build on high/critical
```

### Package Verification

Always verify package names against official registries before installing. [V-12]

Never install packages with typo-squatted names (e.g., `expresss` instead of `express`). [V-12]

Always review package source and maintainers for critical dependencies. [V-12]

---

## File Upload Security

### Upload Configuration

Always use `multer` with file size limits, type validation, and sanitized filenames. [V-16]

Never write uploads to a publicly accessible directory without validation. [V-16]

Always validate file MIME types and extensions. [V-16]

Always sanitize filenames to prevent path traversal. [V-16]

NEVER: Unsafe file uploads

```javascript
const multer = require('multer')
const upload = multer({ dest: 'public/uploads/' })  // Public directory, no validation

app.post('/api/upload', upload.single('file'), (req, res) => {
  res.json({ filename: req.file.filename })  // No validation, stored in public dir
})
```

ALWAYS: Secure file upload configuration

```javascript
const multer = require('multer')
const path = require('path')
const crypto = require('crypto')

const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, '/var/uploads/temp')  // Private directory, not public
  },
  filename: function (req, file, cb) {
    // Sanitized, random filename
    const randomName = crypto.randomBytes(16).toString('hex')
    const ext = path.extname(file.originalname)
    cb(null, `${randomName}${ext}`)
  }
})

const fileFilter = (req, file, cb) => {
  // Allowlist of permitted MIME types
  const allowedMimes = ['image/jpeg', 'image/png', 'image/gif']

  if (allowedMimes.includes(file.mimetype)) {
    cb(null, true)
  } else {
    cb(new Error('Invalid file type. Only JPEG, PNG, and GIF allowed.'), false)
  }
}

const upload = multer({
  storage: storage,
  limits: {
    fileSize: 5 * 1024 * 1024,  // 5MB limit
    files: 1  // Only one file
  },
  fileFilter: fileFilter
})

app.post('/api/upload', upload.single('file'), async (req, res, next) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: 'No file uploaded' })
    }

    // Additional validation: verify file content matches extension
    // Move to permanent storage, scan for malware, etc.

    res.json({
      success: true,
      fileId: req.file.filename  // Random name, not original
    })
  } catch (err) {
    next(err)
  }
})
```

---

## Database Security

### Query Parameterization

Always use parameterized queries with `pg`, `mysql2`, or any raw SQL library. [V-06]

Never concatenate user input into SQL strings. [V-06]

With Prisma, Sequelize, or Knex, always use ORM methods. Never use raw string interpolation. [V-06]

NEVER: SQL injection vulnerability

```javascript
const mysql = require('mysql2')
const connection = mysql.createConnection(process.env.DATABASE_URL)

app.get('/api/user/:id', (req, res) => {
  const query = `SELECT * FROM users WHERE id = ${req.params.id}`  // INJECTION!
  connection.query(query, (err, results) => {
    res.json(results)
  })
})
```

ALWAYS: Parameterized queries

```javascript
const mysql = require('mysql2/promise')
const connection = await mysql.createConnection(process.env.DATABASE_URL)

app.get('/api/user/:id', async (req, res) => {
  const [results] = await connection.query(
    'SELECT * FROM users WHERE id = ?',
    [req.params.id]  // Parameterized
  )
  res.json(results)
})

// With Prisma
const { PrismaClient } = require('@prisma/client')
const prisma = new PrismaClient()

app.get('/api/user/:id', async (req, res) => {
  const user = await prisma.user.findUnique({
    where: { id: req.params.id }  // ORM method, automatically safe
  })
  res.json(user)
})
```

---

## Environment Configuration

### Environment Variables

Always use `dotenv` to load environment variables in development. [V-02]

Never commit `.env` files to version control. Add to `.gitignore`. [V-09]

Always validate required environment variables at startup. [V-02]

ALWAYS: Environment validation

```javascript
require('dotenv').config()

const requiredEnvVars = [
  'DATABASE_URL',
  'SESSION_SECRET',
  'JWT_SECRET',
  'NODE_ENV'
]

for (const envVar of requiredEnvVars) {
  if (!process.env[envVar]) {
    console.error(`Missing required environment variable: ${envVar}`)
    process.exit(1)
  }
}

// Validate specific values
if (!['development', 'production', 'test'].includes(process.env.NODE_ENV)) {
  console.error('NODE_ENV must be development, production, or test')
  process.exit(1)
}
```

---

## Implementation Notes

### Priority Rules

Node.js/Express-specific high-priority rules:
1. **Use helmet with explicit CSP** (most commonly missing security middleware)
2. **Set body size limits** (frequent DoS vector)
3. **Configure CORS with allowlist** (frequent CORS misconfiguration)
4. **Use persistent session store** (MemoryStore is default but wrong)
5. **Implement centralized error handling** (stack traces frequently leaked)

### Common AI Failures

AI tools frequently:
- Generate Express apps without helmet
- Use `cors()` with no arguments
- Use default MemoryStore for sessions
- Forget to set body size limits
- Expose error stacks in production
- Use `express.json()` without limits
- Omit rate limiting entirely
- Create insecure session configurations
- Use hardcoded session secrets
- Skip input validation

### Integration with Core Rules

These Express rules extend core VibeShield rules:
- V-02 (Hardcoded Secrets): Session secret, JWT secret management
- V-06 (Injection): Parameterized queries, input validation
- V-08 (Security Headers): Helmet, CORS, rate limiting, session security
- V-10 (CORS): Explicit origin configuration
- V-12 (Dependency Confusion): npm audit, package verification
- V-14 (Error Disclosure): Centralized error handler
- V-16 (Path Traversal): File upload security

---

**End of VibeShield Node.js/Express Security Rules v0.1.0**
