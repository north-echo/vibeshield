# Sources

Research and data sources used to build the VibeShield vulnerability taxonomy and ruleset.

## Primary Research

| Source | Date | Key Finding |
|---|---|---|
| Veracode GenAI Code Security Report | 2025 | 45% of AI-generated code contains security flaws |
| Tenzai AI Coding Tools Study | Dec 2025 | 69 vulnerabilities found across 15 test apps built by 5 major AI coding tools |
| Escape.tech Vibe-Coded Apps Analysis | 2025 | 2,000+ vulnerabilities and 400+ exposed secrets in 5,600 deployed vibe-coded apps |
| CodeRabbit AI Co-Authored PR Analysis | Dec 2025 | 2.74x higher security vulnerability rate in AI co-authored code vs human-written |
| Carnegie Mellon University Study | 2024 | AI-generated code is 61% functionally correct but only 10.5% secure |

## Industry Reports & Frameworks

| Source | Date | Relevance |
|---|---|---|
| Unit 42 / Palo Alto Networks SHIELD Framework | Jan 2026 | Organizational-level framework for securing AI-generated code |
| Kaspersky: Vibe Coding Security Risks | Oct 2025 | Analysis of common security patterns in vibe-coded applications |
| Invicti: Security Issues in Vibe-Coded Web Apps | 2025 | Identified per-model "favorite" placeholder secrets and recurring auth flaws |
| Vidoc Security Lab | 2025 | Analysis of credential storage and cryptographic failures in AI code |
| ZeroPath | 2025 | Documented subtle HMAC timing attack vulnerability in AI-generated code |

## Vulnerability Databases & Trackers

| Source | URL | Relevance |
|---|---|---|
| Vibe Security Radar | https://vibe-radar-ten.vercel.app/ | Tracks 130+ AI-linked CVEs across 8 tools (as of March 2026) |
| OWASP Top 10 | https://owasp.org/www-project-top-ten/ | Industry standard web application vulnerability classification (2021) |
| OWASP LLM Top 10 | https://owasp.org/www-project-top-10-for-large-language-model-applications/ | LLM-specific vulnerability classification (2025) |
| GitGuardian State of Secrets Sprawl | https://www.gitguardian.com/ | Data on secrets exposed in source code repositories |

## Supply Chain & Dependency Risks

| Source | Date | Relevance |
|---|---|---|
| TechRadar: Slopsquatting | 2025 | Documented AI hallucinating non-existent package names as attack vector |
| SecurityWeek: AI Package Hallucination | 2025 | Analysis of supply chain risks from AI-suggested phantom dependencies |
