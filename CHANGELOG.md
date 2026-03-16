# Changelog

All notable changes to VibeShield will be documented in this file.

## [0.2.0] - 2026-03-16

### Added
- GitHub Copilot adapter (`rules/.github/copilot-instructions.md`)
- Windsurf adapter (`rules/.windsurfrules`)
- Aider adapter (`rules/.aider.conf.yml`)
- Roo Code adapter (`rules/.roo/rules.md`)
- Supabase stack supplement (`stacks/supabase.md`)
- Node.js / Express stack supplement (`stacks/node-express.md`)
- Python (Django/Flask) stack supplement (`stacks/python-flask-django.md`)
- Docker / container stack supplement (`stacks/container-docker.md`)
- 15 additional test prompts (T-11 through T-25) covering all vulnerability tiers
- Expected behavior checklists for V-11 through V-17

### Changed
- Updated README with full tool adapter and stack supplement documentation
- Fixed GitHub repository URLs across all rules files

## [0.1.0] - 2026-03-16

### Added
- Core security ruleset (`core/vibeshield-rules.md`) covering 17 vulnerability patterns (V-01 through V-17)
- Claude Code adapter (`rules/CLAUDE.md`)
- Cursor adapter (`rules/.cursorrules`)
- Vulnerability evidence map (`evidence/vulnerability-map.md`)
- Test prompts for V-01 through V-10 (`tests/test-prompts.md`)
- Expected behavior reference (`tests/expected-behaviors.md`)
- Project documentation: README, CONTRIBUTING, LICENSE (Apache 2.0)
