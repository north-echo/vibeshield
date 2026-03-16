# VibeShield Vulnerability Taxonomy

This taxonomy catalogs the specific security vulnerability patterns that AI coding assistants consistently introduce into generated code. It is derived from empirical research including the Vibe Security Radar CVE database, the Tenzai study (69 vulnerabilities across 5 AI tools), Escape.tech analysis (2,000+ vulnerabilities in 5,600 deployed AI-coded apps), and security research from Veracode, Kaspersky, Invicti, and others.

The taxonomy is organized into three tiers based on frequency of occurrence across tested AI coding tools. Each vulnerability pattern is mapped to Common Weakness Enumeration (CWE) identifiers and supported by documented evidence from real-world AI-generated code.

## Tier 1 — Near-Universal AI Failures

Present in >80% of tested tools. These patterns represent the most common and critical security failures in AI-generated code.

| V-ID | Pattern | Description | CWE | Evidence |
|------|---------|-------------|-----|----------|
| V-01 | Broken Authorization | Auth checks verify identity but skip permission validation. IDOR via sequential/predictable IDs. Missing role checks on new endpoints. | CWE-862 (Missing Authorization), CWE-639 (IDOR) | Tenzai (most common across all 5 tools), Vibe Radar (multiple Critical CVEs) |
| V-02 | Hardcoded Secrets | JWT signing keys, API keys, DB passwords embedded in source. Each LLM has its own set of "favorite" placeholder secrets it reuses. | CWE-798 (Hardcoded Credentials) | Invicti (per-model placeholder secrets), Escape.tech (400+ exposed secrets) |
| V-03 | SSRF | URL preview, webhook, image processing features make unrestricted outbound requests. No allowlist, no internal network protection. | CWE-918 (SSRF) | Tenzai (all 5 tools introduced SSRF) |
| V-04 | Client-Side Auth Logic | Authentication and authorization logic implemented entirely in frontend JavaScript, trivially bypassable. | CWE-602 (Client-Side Enforcement of Server-Side Security) | Kaspersky, Invicti (recurring in vibe-coded web apps) |

## Tier 2 — High Frequency

Present in >50% of tested tools. These patterns are common enough to warrant universal mitigation rules.

| V-ID | Pattern | Description | CWE | Evidence |
|------|---------|-------------|-----|----------|
| V-05 | Insecure Credential Storage | Passwords stored in plaintext or hashed with MD5/SHA-1 instead of bcrypt/Argon2. | CWE-916 (Insufficient Password Hashing) | Veracode, Vidoc Security Lab |
| V-06 | Missing Input Validation | No sanitization on user input leading to XSS, SQL injection (less common now), command injection. | CWE-79 (XSS), CWE-89 (SQLi), CWE-77 (Command Injection) | Kaspersky (45% OWASP Top 10 flaws) |
| V-07 | Business Logic Bypass | Negative quantities, negative prices, missing bounds checks on numerical inputs. | CWE-20 (Improper Input Validation) | Tenzai (4/5 negative quantities, 3/5 negative prices) |
| V-08 | Missing Security Headers | No CSRF tokens, no Content-Security-Policy, no X-Frame-Options, no rate limiting. | CWE-352 (CSRF), CWE-693 (Protection Mechanism Failure) | Tenzai (zero CSRF, zero security headers) |
| V-09 | Secrets in Version Control | .env files with real credentials committed to git. No .gitignore entries for sensitive files. | CWE-540 (Source Code Secrets) | GitGuardian data |
| V-10 | Overly Permissive CORS | Wildcard CORS (Access-Control-Allow-Origin: *) with credentials enabled. | CWE-942 (Permissive CORS) | Vibe Radar GHSA-g9rg-8vq5-mpwm |

## Tier 3 — Context-Dependent

Significant vulnerability patterns that appear in specific contexts or with certain prompts. While less universal than Tier 1 and 2, these patterns are severe enough to include in the core ruleset.

| V-ID | Pattern | Description | CWE | Evidence |
|------|---------|-------------|-----|----------|
| V-11 | Dangerous Deserialization | pickle.loads(), eval(), yaml.load() on untrusted input. AI optimizes for brevity and picks the unsafe shortcut. | CWE-502 (Deserialization of Untrusted Data) | Kaspersky, Vidoc |
| V-12 | Hallucinated Dependencies | AI suggests packages that don't exist ("slopsquatting"). Attacker registers the name with malicious code. | CWE-829 (Untrusted Functionality) | TechRadar, SecurityWeek (slopsquatting) |
| V-13 | Insufficient Logging | No security event logging. Auth failures, admin actions, and access anomalies go unrecorded. | CWE-778 (Insufficient Logging) | Ettenger analysis |
| V-14 | Information Disclosure | Verbose error messages, stack traces in production, health endpoints exposing system details. | CWE-209 (Information Exposure via Error) | Vibe Radar CVE-2026-29787 |
| V-15 | Weak Cryptography | Timing-vulnerable HMAC comparison (== instead of hmac.compare_digest). Default/weak TLS configs. | CWE-208 (Timing Side-Channel) | ZeroPath (HMAC timing attack) |
| V-16 | Path Traversal | Unsanitized user input in file paths. Session IDs or filenames used directly in filesystem operations. | CWE-22 (Path Traversal) | Vibe Radar CVE-2026-28482 |
| V-17 | Command Injection | String interpolation of user input into shell commands via exec(), execAsync(), os.system(). | CWE-78 (OS Command Injection) | Vibe Radar CVE-2026-31862 |

## Methodology

This taxonomy is evidence-driven. Each vulnerability pattern (V-ID) is included only if it meets at least one of these criteria:

1. **Empirical frequency**: Documented in published research showing the pattern appears in multiple AI coding tools
2. **CVE evidence**: Linked to at least one real-world CVE in AI-generated code tracked by Vibe Security Radar
3. **Severity + reproducibility**: Critical or High severity AND reproducible across different prompts/tools

The taxonomy is maintained as a living document. As new AI-linked CVEs are disclosed and new research is published, patterns are added, refined, or deprecated. See the project CHANGELOG for update history.
