# Community Outreach Drafts

These are draft posts for announcing VibeShield. Edit to fit each platform's tone and norms.

---

## Hacker News (Show HN)

**Title:** Show HN: VibeShield -- Drop-in security rules for AI coding assistants

**Text:**

I built VibeShield because AI coding tools consistently produce the same security vulnerabilities. Research shows 45% of AI-generated code contains flaws (Veracode), with specific patterns like hardcoded secrets, broken authorization, and SSRF appearing across every major tool tested (Tenzai found 69 vulnerabilities across 5 tools).

VibeShield is a set of drop-in config files (.cursorrules, CLAUDE.md, copilot-instructions.md, etc.) that inject security rules into the AI's context before it generates code. Prevention over detection.

The rules are evidence-based -- every one maps to a documented AI vulnerability pattern, not theoretical risk. 17 patterns organized by frequency, with stack supplements for Supabase, Node/Express, Django/Flask, and Docker.

Zero friction: copy one file into your project root. No installs, no dependencies, no CI changes.

Apache 2.0. Contributions welcome.

https://github.com/north-echo/vibeshield

---

## Reddit r/netsec

**Title:** VibeShield: Evidence-based security context rules for AI coding assistants (open source)

**Text:**

Released an open-source project targeting the security gap in AI-generated code.

**Problem:** AI coding tools (Claude Code, Cursor, Copilot, etc.) consistently introduce the same vulnerability patterns. The Tenzai study found 69 vulnerabilities across 15 apps built by 5 tools. Escape.tech found 2,000+ vulnerabilities in 5,600 deployed vibe-coded apps. The failure modes are predictable and well-documented.

**Approach:** VibeShield ships as drop-in configuration files that inject security constraints into the AI's generation context. The rules are imperative, specific, and mapped to 17 documented vulnerability patterns across 3 tiers. Each rule traces to CVEs, advisories, or research data.

**What it covers:** Broken authorization, hardcoded secrets, SSRF, client-side auth, insecure credential storage, input validation, business logic bypass, CSRF, CORS, deserialization, supply chain (hallucinated dependencies), path traversal, command injection, and more.

**What it's not:** Not a SAST scanner, not a replacement for security testing. It's one layer that reduces the volume of vulnerabilities generated. Use it alongside your existing security tooling.

Supports: Claude Code, Cursor, Copilot, Windsurf, Aider, Roo Code. Stack supplements for Supabase, Node/Express, Django/Flask, Docker.

GitHub: https://github.com/north-echo/vibeshield

Feedback and contributions welcome, especially effectiveness test results and new vulnerability pattern reports.

---

## Reddit r/programming or r/webdev

**Title:** VibeShield: Drop-in security rules that make AI coding assistants stop generating vulnerable code

**Text:**

If you use Cursor, Claude Code, Copilot, or any AI coding tool, you've probably noticed it generates insecure code by default -- hardcoded secrets, missing auth checks, SQL injection, etc.

I built VibeShield to fix this. It's a set of config files you drop into your project root that tell the AI what NOT to do. Rules like "never hardcode JWT secrets" and "always check resource ownership, not just authentication" -- specific, imperative instructions that the AI reads on every generation.

Copy one file. No installs. Apache 2.0.

17 vulnerability patterns covered, with stack-specific supplements for Supabase, Express, Django/Flask, and Docker.

https://github.com/north-echo/vibeshield

---

## Twitter/X thread

1/ Launched VibeShield -- open-source security rules for AI coding assistants.

Drop a single config file into your project, and your AI stops generating hardcoded secrets, broken auth, SSRF, and 14 other vulnerability patterns.

2/ The problem: 45% of AI-generated code has security flaws. The same patterns show up across every tool -- Cursor, Copilot, Claude Code, all of them. Broken auth. Hardcoded secrets. No CSRF. No input validation.

3/ The fix: shift security left of the code, into the AI's context. VibeShield injects specific, evidence-based rules that tell the AI what never to generate.

Every rule maps to a documented CVE or research finding, not theoretical risk.

4/ Zero friction: copy one file into your project root. Supports Claude Code, Cursor, Copilot, Windsurf, Aider, and Roo Code. Stack supplements for Supabase, Express, Django/Flask, Docker.

Apache 2.0. Contributions welcome.

https://github.com/north-echo/vibeshield
