# VibeShield: Secure Vibe Coding Context Rules

## Project Plan & Implementation Guide

**Author:** Christopher Lusk  
**Date:** March 2026  
**Status:** Draft  
**Build Tool:** Claude Code

---

## 1. Problem Statement

AI coding assistants (Claude Code, Cursor, Copilot, Codex, Replit, Devin) generate functional code that consistently introduces predictable security vulnerabilities. Research shows:

- **45%** of AI-generated code contains security flaws (Veracode 2025)
- **69 vulnerabilities** found across 15 test apps built by 5 major tools (Tenzai, Dec 2025)
- **2,000+ vulnerabilities** and 400+ exposed secrets in 5,600 deployed vibe-coded apps (Escape.tech)
- **2.74x higher** security vulnerability rate in AI co-authored code vs human-written (CodeRabbit)
- **130 AI-linked CVEs** tracked by Vibe Security Radar across 8 tools (as of March 2026)

The failure modes are not random. They cluster around specific, well-documented vulnerability classes that share a common trait: they require contextual security understanding that LLMs lack by default. The most effective — and most underserved — intervention point is the **context layer**: the rules files, system prompts, and project instructions that shape what the AI generates in the first place.

Nobody has built a well-maintained, evidence-based, open-source set of security context rules specifically targeting the vulnerability patterns that AI coding tools actually produce. That's the gap.

---

## 2. Project Definition

**VibeShield** is an open-source collection of security-focused context rules files for AI coding assistants. It ships as drop-in configuration files (`.cursorrules`, `CLAUDE.md`, `.github/copilot-instructions.md`, etc.) that inject security constraints into the AI's generation context before code is written.

### Core Principles

1. **Prevention over detection.** Shift security left of the code — into the prompt context itself.
2. **Evidence-driven rules.** Every rule maps to a documented AI-generated vulnerability pattern, not theoretical risk.
3. **Zero friction.** Copy a file into your repo root. No installs, no dependencies, no CI changes required.
4. **Tool-agnostic core, tool-specific adapters.** One canonical ruleset, formatted for each major AI coding tool.
5. **Opinionated and narrow.** This is not a general secure coding guide. It targets specifically the mistakes AI agents make, not the mistakes humans make.

### What This Is Not

- Not a SAST scanner or linter
- Not a replacement for security testing or code review
- Not a comprehensive secure development lifecycle
- Not a compliance framework

---

## 3. Vulnerability Taxonomy

The following taxonomy is derived from Vibe Security Radar data, the Tenzai study, Escape.tech research, Invicti analysis, and Kaspersky/Unit 42 reporting. Rules are prioritized by **frequency in AI-generated code** and **exploitability**.

### Tier 1 — Near-Universal AI Failures (present in >80% of tested tools)

| ID | Pattern | Description | Evidence |
|---|---|---|---|
| V-01 | **Broken Authorization** | Auth checks verify identity but skip permission validation. IDOR via sequential/predictable IDs. Missing role checks on new endpoints. | Tenzai: most common failure across all 5 tools. Vibe Radar: multiple Critical CVEs. |
| V-02 | **Hardcoded Secrets** | JWT signing keys, API keys, DB passwords embedded in source. Each LLM has its own set of "favorite" placeholder secrets it reuses. | Invicti: each model reuses specific secrets across apps. Escape.tech: 400+ exposed secrets in 5,600 apps. |
| V-03 | **SSRF** | URL preview, webhook, image processing features make unrestricted outbound requests. No allowlist, no internal network protection. | Tenzai: all 5 tools introduced SSRF. No universal generic defense exists. |
| V-04 | **Client-Side Auth Logic** | Authentication and authorization logic implemented entirely in frontend JavaScript, trivially bypassable. | Kaspersky, Invicti: recurring pattern in vibe-coded web apps. |

### Tier 2 — High Frequency (present in >50% of tested tools)

| ID | Pattern | Description | Evidence |
|---|---|---|---|
| V-05 | **Insecure Credential Storage** | Passwords stored in plaintext or hashed with MD5/SHA-1 instead of bcrypt/Argon2. | Veracode, Vidoc, multiple blog analyses. |
| V-06 | **Missing Input Validation** | No sanitization on user input leading to XSS, SQL injection (less common now), command injection. | Kaspersky: 45% of AI code has OWASP Top 10 flaws. |
| V-07 | **Business Logic Bypass** | Negative quantities, negative prices, missing bounds checks on numerical inputs. | Tenzai: 4/5 tools allowed negative order quantities, 3/5 allowed negative prices. |
| V-08 | **Missing Security Headers** | No CSRF tokens, no Content-Security-Policy, no X-Frame-Options, no rate limiting. | Tenzai: zero apps built CSRF protection. Zero set security headers. |
| V-09 | **Secrets in Version Control** | `.env` files with real credentials committed to git. No `.gitignore` entries for sensitive files. | GitGuardian data, multiple incident reports. |
| V-10 | **Overly Permissive CORS** | Wildcard CORS (`Access-Control-Allow-Origin: *`) with credentials enabled. | Vibe Radar: GHSA-g9rg-8vq5-mpwm (mcp-memory-service). |

### Tier 3 — Significant but Context-Dependent

| ID | Pattern | Description | Evidence |
|---|---|---|---|
| V-11 | **Dangerous Deserialization** | `pickle.loads()`, `eval()`, `yaml.load()` on untrusted input. AI optimizes for brevity and picks the unsafe shortcut. | Kaspersky, Vidoc Security Lab. |
| V-12 | **Hallucinated Dependencies** | AI suggests packages that don't exist ("slopsquatting"). Attacker registers the name with malicious code. | TechRadar, SecurityWeek — documented supply chain vector. |
| V-13 | **Insufficient Logging** | No security event logging. Auth failures, admin actions, and access anomalies go unrecorded. | Ettenger analysis: excellent debug logging, terrible security logging. |
| V-14 | **Information Disclosure** | Verbose error messages, stack traces in production, health endpoints exposing system details. | Vibe Radar: CVE-2026-29787 (detailed system info on unauthenticated endpoint). |
| V-15 | **Weak Cryptography** | Timing-vulnerable HMAC comparison (`==` instead of `hmac.compare_digest`). Default/weak TLS configs. | ZeroPath: documented subtle HMAC timing attack in AI-generated code. |
| V-16 | **Path Traversal** | Unsanitized user input in file paths. Session IDs or filenames used directly in filesystem operations. | Vibe Radar: CVE-2026-28482 (path traversal via session ID). |
| V-17 | **Command Injection** | String interpolation of user input into shell commands via `exec()`, `execAsync()`, `os.system()`. | Vibe Radar: CVE-2026-31862 (Critical — execAsync with string interpolation). |

---

## 4. Project Structure

```
vibeshield/
├── README.md                          # Project overview, quick start, badges
├── LICENSE                            # Apache 2.0
├── CONTRIBUTING.md                    # Contribution guidelines
├── CHANGELOG.md                       # Release notes
│
├── core/
│   └── vibeshield-rules.md            # Canonical ruleset (tool-agnostic markdown)
│
├── rules/
│   ├── CLAUDE.md                      # Claude Code format
│   ├── .cursorrules                   # Cursor format
│   ├── .github/
│   │   └── copilot-instructions.md    # GitHub Copilot format
│   ├── .windsurfrules                 # Windsurf format
│   ├── .roo/
│   │   └── rules.md                   # Roo Code format
│   └── .aider.conf.yml               # Aider format (conventions section)
│
├── stacks/                            # Stack-specific rule supplements
│   ├── node-express.md                # Node.js / Express additions
│   ├── python-flask-django.md         # Python web framework additions
│   ├── react-nextjs.md                # Frontend framework additions
│   ├── supabase.md                    # Supabase-specific (RLS, service keys)
│   └── container-docker.md            # Container/Dockerfile additions
│
├── evidence/
│   ├── vulnerability-map.md           # V-ID → CVE/advisory cross-reference
│   └── sources.md                     # Research citations and data sources
│
├── tests/                             # Validation test prompts
│   ├── test-prompts.md                # Prompts that should trigger rules
│   └── expected-behaviors.md          # What compliant output looks like
│
└── docs/
    ├── taxonomy.md                    # Full vulnerability taxonomy (from §3)
    ├── how-it-works.md                # How context rules affect generation
    ├── faq.md                         # Common questions
    └── comparison.md                  # How this differs from SAST/linters
```

---

## 5. Core Rules File Design

The canonical rules file (`core/vibeshield-rules.md`) is the single source of truth. Tool-specific files in `rules/` are generated from it with formatting adjustments.

### Design Constraints

- **Under 4,000 tokens.** Context window budget is real. Every word competes with the user's actual prompt and codebase. Bloated rules files get ignored or truncated.
- **Imperative, not advisory.** "Never use `eval()` on user input" not "Consider avoiding `eval()` when possible."
- **Specific, not generic.** "Use `bcrypt` or `argon2` for password hashing. Never use MD5, SHA-1, or SHA-256 for passwords." Not "use strong hashing."
- **Negative examples where critical.** For the highest-risk patterns (V-01 through V-04), include a one-line "NEVER" example and a one-line "ALWAYS" example.
- **Grouped by generation phase.** Rules organized by when they matter during code generation: project setup → auth/identity → data handling → API design → deployment config.

### Canonical Rules Structure (Outline)

```markdown
# VibeShield Security Rules
# Drop this file into your project root to enforce secure defaults in AI-generated code.
# Source: https://github.com/<owner>/vibeshield | Version: X.Y.Z

## Project Setup
- Generate a .gitignore that excludes .env, *.pem, *.key, and credentials files BEFORE writing any code.
- Never hardcode API keys, JWT secrets, database passwords, or tokens. Use environment variables loaded at runtime.
- Pin all dependency versions. Do not install packages you cannot verify exist on the public registry.
- [...]

## Authentication & Authorization
- Every protected endpoint MUST check both authentication (who) AND authorization (permission to do this action).
- Never implement auth logic on the client side only. All auth decisions happen server-side.
- Use bcrypt or argon2 for password hashing. Never MD5, SHA-1, or SHA-256 for passwords.
- NEVER: `if (req.user) { /* allow */ }` — this checks auth but not authz.
- ALWAYS: `if (req.user && req.user.role === 'admin') { /* allow */ }` with resource ownership validation.
- [...]

## Data Handling & Input Validation
- Validate and sanitize ALL user input on the server side before use.
- Never pass user input to eval(), exec(), os.system(), execAsync(), or child_process without strict validation.
- Never use string interpolation/concatenation to build SQL queries, shell commands, or file paths from user input.
- [...]

## API & Network Security
- Never make HTTP requests to user-supplied URLs without an allowlist of permitted domains.
- Set CORS to specific allowed origins. Never use wildcard (*) with credentials.
- Add CSRF protection to all state-changing endpoints.
- Set security headers: Content-Security-Policy, X-Frame-Options, X-Content-Type-Options, Strict-Transport-Security.
- [...]

## Secrets & Configuration
- Never include real credentials in code, even as "examples" or "defaults."
- Generate unique, cryptographically random secrets for JWT signing, session keys, etc.
- Always use HTTPS. Never allow HTTP fallback for authenticated endpoints.
- [...]

## Error Handling & Logging
- Never expose stack traces, internal paths, or system details in production error responses.
- Log authentication events (success and failure), authorization failures, and admin actions.
- Never log secrets, tokens, passwords, or PII.
- [...]

## Business Logic
- Validate numerical inputs for bounds: quantities must be >= 0, prices must be > 0, etc.
- Never trust client-provided pricing, discount, or role information for server-side decisions.
- [...]
```

---

## 6. Implementation Plan

### Phase 1: Foundation (Week 1-2)

**Goal:** Ship the core ruleset and Claude Code + Cursor adapters. Get it usable immediately.

| Step | Task | Details |
|---|---|---|
| 1.1 | Initialize repo | Create GitHub repo, Apache 2.0 license, README with project overview and quick-start. |
| 1.2 | Write canonical rules | Draft `core/vibeshield-rules.md` from taxonomy. Target <4,000 tokens. Every rule traces to a V-ID. |
| 1.3 | Generate CLAUDE.md | Adapt canonical rules to Claude Code's CLAUDE.md format and conventions. |
| 1.4 | Generate .cursorrules | Adapt canonical rules to Cursor's .cursorrules format. |
| 1.5 | Write vulnerability map | Create `evidence/vulnerability-map.md` linking each V-ID to CVEs, advisories, and research. |
| 1.6 | Write README | Installation instructions (copy file → done), project philosophy, contributing guide. |
| 1.7 | Basic test prompts | Create 5-10 prompts in `tests/test-prompts.md` that exercise the highest-risk rules (V-01 through V-04). |
| 1.8 | Ship v0.1.0 | Tag release with CLAUDE.md and .cursorrules. Announce on GitHub. |

**Claude Code workflow for Phase 1:**

```bash
# Project scaffolding
claude "Create the vibeshield project directory structure as defined in the plan. 
       Initialize git, add Apache 2.0 license, create placeholder files."

# Core rules authoring — iterative
claude "Write the canonical vibeshield-rules.md based on the vulnerability taxonomy. 
       Rules must be imperative, specific, and under 4,000 tokens. 
       Each rule should reference its V-ID in a comment."

# Tool-specific adaptation
claude "Convert core/vibeshield-rules.md into rules/CLAUDE.md formatted for 
       Claude Code's CLAUDE.md conventions. Preserve all rules, adjust formatting only."

claude "Convert core/vibeshield-rules.md into rules/.cursorrules formatted for 
       Cursor's rules file conventions."

# Evidence documentation
claude "Create evidence/vulnerability-map.md that maps each V-ID (V-01 through V-17) 
       to its supporting CVEs, research papers, and data sources."
```

### Phase 2: Expand Coverage (Week 3-4)

**Goal:** Add remaining tool adapters, stack-specific supplements, and validation tests.

| Step | Task | Details |
|---|---|---|
| 2.1 | Copilot adapter | Generate `.github/copilot-instructions.md` from canonical rules. |
| 2.2 | Windsurf adapter | Generate `.windsurfrules` from canonical rules. |
| 2.3 | Aider adapter | Generate `.aider.conf.yml` conventions section from canonical rules. |
| 2.4 | Supabase stack supplement | Write `stacks/supabase.md` — RLS enforcement, service key handling, auth config. High priority given Supabase prevalence in vibe-coded apps. |
| 2.5 | Node/Express supplement | Write `stacks/node-express.md` — Express-specific middleware, helmet, rate limiting. |
| 2.6 | Python web supplement | Write `stacks/python-flask-django.md` — Django/Flask-specific security settings. |
| 2.7 | Container supplement | Write `stacks/container-docker.md` — non-root users, minimal base images, secret mounting. |
| 2.8 | Expand test suite | Add 20+ test prompts covering all tiers. Document expected behaviors. |
| 2.9 | Ship v0.2.0 | Full tool coverage, initial stack supplements. |

### Phase 3: Validation & Community (Week 5-8)

**Goal:** Empirically test rule effectiveness, build community, establish maintenance cadence.

| Step | Task | Details |
|---|---|---|
| 3.1 | Effectiveness testing | Run identical prompts with and without VibeShield rules across Claude Code, Cursor, Copilot. Document vulnerability reduction. |
| 3.2 | Write comparison doc | `docs/comparison.md` — how VibeShield differs from SAST, linters, and general secure coding guides. |
| 3.3 | Community outreach | Post to security-focused communities, Reddit r/netsec, Hacker News. Submit to awesome-security lists. |
| 3.4 | Contribution pipeline | Set up issue templates for new vulnerability patterns, new tool adapters, new stack supplements. |
| 3.5 | Vibe Radar integration | Reach out to Vibe Security Radar maintainers about cross-referencing. Their CVE data feeds the taxonomy; the rules are the mitigation layer. |
| 3.6 | Ship v1.0.0 | Stable release with validated effectiveness data. |

### Phase 4: Advanced (Post v1.0)

| Task | Description |
|---|---|
| CLI installer | `npx vibeshield init` or `pip install vibeshield` — detects tools in use, copies correct rules file. |
| Rule versioning & diffing | Track rule changes against new CVE data. Automated alerts when new AI-linked CVEs warrant rule updates. |
| MCP server (optional) | A lightweight MCP tool that AI agents can call to validate their own output against VibeShield rules mid-generation. |
| Compliance mapping | Map rules to OWASP Top 10, NIST 800-53 controls, CWE IDs for enterprise adoption. |

---

## 7. Differentiation & Positioning

| Existing Approach | Limitation | VibeShield Difference |
|---|---|---|
| SAST/SCA in CI/CD (Semgrep, CodeQL, Snyk) | Post-generation. Finds vulns after they exist. Requires pipeline setup. | Pre-generation. Prevents vulns from being written. Zero setup. |
| General `.cursorrules` security tips | Vague ("follow best practices"), not evidence-based, not maintained. | Specific, imperative, mapped to documented AI failure patterns. |
| Unit 42 SHIELD framework | High-level organizational guidance. Not executable. | Executable rules files you drop into a repo. |
| Vendor-specific security features | Locked to one tool. Opaque. Variable quality. | Tool-agnostic core. Transparent. Community-maintained. |
| "Prompt twice" technique (generate then review) | Relies on the same model catching its own mistakes. Inconsistent. | Encodes the security knowledge upfront so it generates correctly the first time. |

---

## 8. Success Metrics

- **Adoption:** GitHub stars, forks, and (more importantly) evidence of inclusion in real repos.
- **Effectiveness:** Measurable reduction in vulnerability count when running identical test prompts with vs. without rules.
- **Coverage:** Percentage of Vibe Radar CVE patterns that map to an active VibeShield rule.
- **Maintenance cadence:** Time between new AI-linked CVE disclosure and corresponding rule update.
- **Community:** External contributions (new tool adapters, stack supplements, vulnerability reports).

---

## 9. Open Questions

1. **Naming:** VibeShield is a working title. Alternatives: `vibe-armor`, `securevibe`, `vibeguard`, `shieldprompt`. Should be memorable, searchable, and not already taken on npm/PyPI/GitHub.
2. **Token budget tradeoffs:** 4,000 tokens is aggressive. May need a "full" and "compact" version, or a tiered approach where users opt into stack-specific supplements.
3. **Rule enforcement vs. guidance:** Some tools (Claude Code) respect `CLAUDE.md` strongly; others may treat rules as suggestions. Need to document observed compliance rates per tool.
4. **Overlap with tool-native features:** As AI coding tools improve their built-in security, some rules may become redundant. Need a deprecation process.
5. **Versioning against Vibe Radar:** Should the project formally track Vibe Radar's CVE feed, or maintain its own independent taxonomy?

---

## 10. References

- Vibe Security Radar: https://vibe-radar-ten.vercel.app/
- Tenzai Study (Dec 2025): 69 vulnerabilities across 5 AI coding tools
- Escape.tech (2025): 2,000+ vulnerabilities in 5,600 vibe-coded apps
- Veracode GenAI Code Security Report (2025): 45% of AI-generated code contains flaws
- CodeRabbit Analysis (Dec 2025): 2.74x higher vulnerability rate in AI co-authored PRs
- Carnegie Mellon: 61% functionally correct, 10.5% secure
- Unit 42 / Palo Alto Networks SHIELD Framework (Jan 2026)
- Kaspersky: Vibe Coding Security Risks (Oct 2025)
- Invicti: Security Issues in Vibe-Coded Web Apps
- OWASP Top 10 (2021)
- OWASP LLM Top 10 (2025)
