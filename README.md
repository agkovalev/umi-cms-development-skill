# UMI.CMS Development Skill

Open-source skill for AI coding agents working with UMI.CMS projects.

## Contents

- `umi-cms-development/SKILL.md` — main skill definition
- `umi-cms-development/references/` — deep-dive guides
  - `architecture.md` — codebase structure, module anatomy, service container, extension points
  - `patterns.md` — 6 safe implementation patterns (macros, events, admin UI, handlers, API integration)
  - `guardrails.md` — security guardrails, common failure modes, escalation criteria
- `umi-cms-development/scripts/evals/evals.json` — 10 test scenarios for evaluating skill quality

## Install (local)

1. Clone this repository.
2. Symlink the skill into your local skills directory:

```bash
ln -s "$(pwd)/umi-cms-development" ~/.agents/skills/umi-cms-development
```

## v1.1 Improvements

- Enhanced trigger phrases for better recognition of UMI.CMS tasks
- Architecture reference with extension point hierarchy (custom*.php, events, handlers, ext/ logic)
- 6 common implementation patterns with code examples
- Security guardrails with "Never Do" examples and escalation criteria
- 10 test cases covering all task groups

## Scope

The skill is optimized for:

- template integration (layout -> XSLT/UMI)
- frontend-facing custom functionality
- extending existing modules safely
- external API integrations

## References

- https://docs.umi-cms.ru/
- https://api.docs.umi-cms.ru/
- https://github.com/Umisoft/umi.cms.2
