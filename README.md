# VibeShield

**Secure context rules for AI coding assistants**

## The Problem

AI coding tools generate code with predictable security vulnerabilities. Research shows that 45% of AI-generated code contains security flaws (Veracode), with a vulnerability rate 2.74x higher than human-written code (CodeRabbit). These failures cluster around specific, well-documented patterns like hardcoded credentials, broken authorization, and SSRF.

## What VibeShield Does

VibeShield is an open-source collection of security-focused context rules files for AI coding assistants. It ships as drop-in configuration files (`.cursorrules`, `CLAUDE.md`, etc.) that inject security constraints into the AI's generation context before code is written.

Prevention over detection — shift security left of the code, into the prompt context itself. Zero friction: copy a file, no installs required.

## Quick Start

### Claude Code
```bash
cp rules/CLAUDE.md your-project/CLAUDE.md
```

### Cursor
```bash
cp rules/.cursorrules your-project/.cursorrules
```

### GitHub Copilot
```bash
mkdir -p your-project/.github
cp rules/.github/copilot-instructions.md your-project/.github/copilot-instructions.md
```

### Windsurf
```bash
cp rules/.windsurfrules your-project/.windsurfrules
```

### Aider
```bash
cp rules/.aider.conf.yml your-project/.aider.conf.yml
```

### Roo Code
```bash
mkdir -p your-project/.roo
cp rules/.roo/rules.md your-project/.roo/rules.md
```

## Why VibeShield?

Other projects tell AI tools what secure code looks like. VibeShield tells them what they specifically get wrong, proves it with CVE data, and measures whether the rules actually work.

Three capabilities no competing project has shipped:

1. **Evidence-mapped rules** — every rule traces to a specific CVE, advisory, or published research finding, not best-practice intuition. See the [evidence map](evidence/vulnerability-map.md).
2. **Failure-mode organization** — rules organized by how AI fails (17 documented patterns), not by language or framework. See the [taxonomy](docs/taxonomy.md).
3. **Effectiveness testing** — reproducible before/after measurements of rule impact on AI-generated vulnerability rates. See the [methodology](tests/methodology.md).

## Evidence

Every VibeShield rule traces to documented vulnerabilities in AI-generated code. The [evidence map](evidence/vulnerability-map.md) cross-references each of the 17 V-IDs to specific CVEs, advisories, and research findings.

| Metric | Value |
|---|---|
| Total CVEs/advisories mapped | 7 |
| V-IDs with CVE/advisory evidence | 6 / 17 |
| V-IDs with research evidence | 17 / 17 |
| AI-linked CVEs tracked (Vibe Radar) | 130+ |

Key research:
- **69 vulnerabilities** found across 15 test apps built by 5 major tools (Tenzai, Dec 2025)
- **2,000+ vulnerabilities** and 400+ exposed secrets in 5,600 deployed vibe-coded apps (Escape.tech)
- **45%** of AI-generated code contains security flaws (Veracode 2025)
- **2.74x** higher vulnerability rate in AI co-authored pull requests (CodeRabbit, Dec 2025)

## Effectiveness Testing

VibeShield includes a [reproducible test framework](tests/methodology.md) for measuring whether the rules actually reduce vulnerabilities. The framework uses 8 standardized [test prompts](tests/prompts/) covering all 17 V-IDs across two tools (Claude Code and Cursor) with baseline vs. VibeShield conditions.

In preliminary v1 testing (single tool, single run per prompt, rules injected into agent context), VibeShield showed a **96% vulnerability reduction** across 10 prompts and 61 security checks. Full v1 results: [test-results-v0.3.md](docs/test-results-v0.3.md). These results are directionally useful but not yet reproducible — the v2 framework above adds multi-tool coverage, 3x repetitions, and standardized scoring to produce publishable results.

## What's Covered

VibeShield addresses 17 vulnerability patterns organized by frequency and impact:

### Tier 1 (Near-Universal)
- Broken authorization
- Hardcoded secrets
- Server-Side Request Forgery (SSRF)
- Client-side authentication/authorization

### Tier 2 (High Frequency)
- Insecure credential storage
- Missing input validation
- Business logic bypass
- Missing security headers
- Secrets in version control
- Permissive CORS

### Tier 3 (Context-Dependent)
- Dangerous deserialization
- Hallucinated dependencies
- Insufficient logging
- Information disclosure
- Weak cryptography
- Path traversal
- Command injection

## Project Structure

```
vibeshield/
├── core/
│   └── vibeshield-rules.md       # Canonical ruleset (tool-agnostic)
├── rules/
│   ├── CLAUDE.md                 # Claude Code adapter
│   ├── .cursorrules              # Cursor adapter
│   ├── .github/
│   │   └── copilot-instructions.md  # GitHub Copilot adapter
│   ├── .windsurfrules            # Windsurf adapter
│   ├── .aider.conf.yml           # Aider adapter
│   └── .roo/
│       └── rules.md              # Roo Code adapter
├── stacks/
│   ├── supabase.md               # Supabase (RLS, auth, keys)
│   ├── node-express.md           # Node.js / Express
│   ├── python-flask-django.md    # Python (Django / Flask)
│   └── container-docker.md       # Docker / containers
├── evidence/
│   ├── vulnerability-map.md      # V-ID to CVE/advisory cross-reference
│   └── sources.md                # Research citations
├── tests/
│   ├── prompts/                  # 8 standardized test prompts (P-01 to P-08)
│   ├── results/                  # Test results and reporting template
│   ├── methodology.md            # Effectiveness testing procedure
│   ├── test-prompts.md           # Additional test prompts
│   └── expected-behaviors.md     # What compliant output looks like
├── docs/
│   ├── comparison.md             # How VibeShield differs from SAST/linters
│   ├── how-it-works.md           # How context rules affect generation
│   ├── taxonomy.md               # Full vulnerability taxonomy
│   ├── faq.md                    # Common questions
│   └── effectiveness-testing.md  # Testing methodology
├── CONTRIBUTING.md
├── CHANGELOG.md
├── LICENSE                       # Apache 2.0
└── README.md
```

## What This Is NOT

- **Not a SAST scanner** — VibeShield works at generation time, not analysis time
- **Not a replacement for security testing** — Still run your scanners, penetration tests, and code reviews
- **Not a compliance framework** — This is developer tooling, not an audit checklist

## Stack Supplements

Copy a stack supplement alongside the core rules file for framework-specific guidance:

| Stack | File | Covers |
|---|---|---|
| Supabase | `stacks/supabase.md` | RLS enforcement, service key handling, auth config, storage policies |
| Node/Express | `stacks/node-express.md` | Helmet, rate limiting, CORS, sessions, input validation |
| Python (Django/Flask) | `stacks/python-flask-django.md` | Django settings, Flask-WTF, CSRF, ORM security |
| Docker | `stacks/container-docker.md` | Non-root users, multi-stage builds, secret management |

## Prior Art

Several projects ship security rules for AI coding assistants. VibeShield was built with awareness of these and designed to complement, not replace, them. Wiz's `secure-rules-files` covers breadth across many languages; CSA's R.A.I.L.G.U.A.R.D provides a reasoning framework; Secure Code Warrior and Pillar Security's `cursor-security-rules` offer additional coverage. VibeShield is narrower but evidence-backed, failure-mode-organized, and empirically tested. See [docs/comparison.md](docs/comparison.md) for the full comparison.

## Documentation

- [How It Works](docs/how-it-works.md) — How context rules affect AI code generation
- [Comparison](docs/comparison.md) — How VibeShield compares to prior art and other approaches
- [Vulnerability Taxonomy](docs/taxonomy.md) — Full taxonomy of 17 AI vulnerability patterns
- [Evidence Map](evidence/vulnerability-map.md) — V-ID to CVE/advisory cross-reference
- [Effectiveness Testing](tests/methodology.md) — Reproducible test framework and methodology
- [FAQ](docs/faq.md) — Common questions

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

Apache 2.0

## References

### Research
- [Vibe Security Radar](https://vibe-radar-ten.vercel.app/) — 130+ AI-linked CVEs tracked across 8 tools (March 2026)
- Veracode GenAI Code Security Report (2025) — 45% of AI-generated code contains flaws
- Tenzai Study (Dec 2025) — 69 vulnerabilities across 15 test apps built by 5 AI coding tools
- Escape.tech (2025) — 2,000+ vulnerabilities and 400+ exposed secrets in 5,600 vibe-coded apps
- CodeRabbit Analysis (Dec 2025) — 2.74x higher vulnerability rate in AI co-authored PRs
- Carnegie Mellon — AI-generated code is 61% functionally correct, 10.5% secure
- Unit 42 / Palo Alto Networks SHIELD Framework (Jan 2026)
- Kaspersky: Vibe Coding Security Risks (Oct 2025)
- Invicti: Security Issues in Vibe-Coded Web Apps
- OWASP Top 10 (2021), OWASP LLM Top 10 (2025)

### Prior Art
- [Wiz secure-rules-files](https://github.com/wiz-sec-public/secure-rules-files) — Language-organized security rules for AI tools
- [CSA R.A.I.L.G.U.A.R.D](https://github.com/brighton-labs/railguard-cursor-coding) — Cognitive security reasoning framework
- [Pillar Security cursor-security-rules](https://github.com/matank001/cursor-security-rules) — Cursor-specific security rules
