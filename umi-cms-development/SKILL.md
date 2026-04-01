---
name: umi-cms-development
description: "UMI.CMS development workflow for template integration, frontend custom features, extending existing modules, and external API integrations. Use this skill whenever user mentions UMI.CMS, umi.cms.2, demomarket templates, XSLT/macros integration, custom module behavior in existing components, or service/API integration in UMI projects, even if request is phrased as generic PHP/CMS work."
compatibility: "Targets UMI.CMS 2 codebase structure with classes/components, templates/*, and XSLT-based frontend templates."
---

# UMI.CMS Development

## When to use

Use this skill when the task is about UMI.CMS project development, especially:

- integrating HTML/CSS/JS into UMI templates and XSLT rendering
- adding custom frontend-facing behavior in existing modules
- extending behavior in existing UMI modules without breaking core logic
- integrating external services via API (REST, webhook, XML/JSON import/export)

Out of scope for this MVP skill:

- full greenfield infrastructure automation and deployment architecture
- creating a brand new UMI module from zero with full installer/package lifecycle
- deep admin panel redesign system-wide (only targeted admin template tweaks are covered)

## Required inputs

Before edits, collect:

- target repository path and active template name
- concrete page flow/user scenario
- affected module(s) and public endpoints/macros
- integration contract (auth method, payload examples, retry/error policy)

Если пользователь не дал входные данные, сначала запроси минимальный набор, затем продолжай реализацию.

## Project map (quick orientation)

Use these paths first:

- `classes/components/` — system and business modules (catalog, emarket, users, etc.)
- `classes/system/` — core internals (avoid direct edits unless explicitly required)
- `templates/<template>/` — frontend template implementation
- `templates/<template>/xslt/` or `xslt/` folders — XSLT views and layout blocks
- `templates/<template>/js/`, `templates/<template>/css/` — static assets
- `custom_classes/` — custom import/extension logic
- `tests/` — unit/web tests and legacy test runners
- `CODINGSTANDARDS.md` — mandatory coding rules for PHP code style and file conventions

## Global workflow

Always follow this sequence:

1. Discover current implementation (search paths, handlers, macros, templates, data flow).
2. Define minimal change set with backward compatibility.
3. Implement in extension-friendly locations first (template/custom/module extension points).
4. Verify behavior on affected pages/flows and check regressions.
5. Summarize changed files, risks, and what was validated.

Не пропускай этап discovery: в UMI.CMS логика часто распределена между PHP, XSLT и конфигами.

## Scenario A: Template integration (layout -> UMI output)

Goal: integrate ready HTML/CSS/JS into UMI template and bind dynamic data.

### Procedure

1. Locate target template directory and page XSLT entry points.
2. Identify existing macros/data providers used by the page.
3. Split static markup into reusable template fragments where possible.
4. Map dynamic blocks to XSLT variables and module macros.
5. Keep JS/CSS asset inclusion consistent with current template conventions.
6. Validate rendered HTML structure and dynamic placeholders.

### Acceptance checklist

- all required blocks render with real CMS data
- no broken includes/asset paths
- no duplicated business logic between PHP and XSLT
- page works in desktop and mobile breakpoints used by current theme

## Scenario B: Custom frontend-facing functionality

Goal: add or extend non-standard behavior consumed by frontend pages.

### Procedure

1. Find existing module part responsible for data source.
2. Prefer extending module custom parts/macros over core replacement.
3. Add new macro/handler with clear input/output contract.
4. Keep response schema stable for existing templates.
5. Add graceful fallback for empty/error states.

### Acceptance checklist

- new behavior does not break old macro calls
- output format is documented in code comments or nearby docs
- edge cases covered (empty data, invalid filter, unavailable dependency)
- errors are handled without fatal output on user-facing pages

## Scenario C: Extend existing module behavior

Goal: customize existing module logic with minimal risk.

### Procedure

1. Locate module files in `classes/components/<module>/`.
2. Check for extension points (`custom*.php`, handlers/events, ext logic) before touching core methods.
3. Implement smallest safe override/extension.
4. Preserve permissions, validation, and existing side effects.
5. Re-check related methods that share the same data entities.

### Acceptance checklist

- no direct core rewrite if extension point exists
- permission checks remain intact
- existing admin/frontend methods still return expected data types
- related module workflows are smoke-tested

## Scenario D: External API integration

Goal: integrate third-party service into UMI flow safely.

### Procedure

1. Define integration boundary: transport, auth, timeout, retries.
2. Isolate API client logic from controller/macro presentation logic.
3. Validate request/response mapping and normalize external errors.
4. Add logging points for request id, endpoint, status, and failure reason.
5. Ensure idempotency for repeated sync/submit operations where relevant.

### Acceptance checklist

- secrets are not hardcoded and are not committed
- network failures degrade gracefully
- mapping between external payload and UMI entities is explicit
- retry policy avoids duplicate destructive operations

## Additional task groups (MVP limited coverage)

These groups are recognized and supported at routing/checklist level in MVP:

- new UMI site bootstrap and environment setup
- creating new modules from scratch
- admin template customization

Use this lightweight approach:

1. Confirm exact scope and boundaries of change.
2. Reuse existing project conventions and nearest working examples.
3. Implement only minimal vertical slice first.
4. Validate installation/runtime/admin impact before expanding.

Если задача относится к этим группам и требует большой архитектурный дизайн, сначала предложи phased plan, затем выполняй по этапам.

## Guardrails (must follow)

- Follow project coding standards from `CODINGSTANDARDS.md` (UTF-8, proper PHP tags, naming, side-effect separation).
- Do not perform broad refactoring unrelated to user task.
- Do not modify core internals in `classes/system/` unless user explicitly requests it and impact is explained.
- Prefer backward-compatible changes to public macros/templates.
- Never commit credentials, tokens, private keys, or client secrets.
- Keep edits minimal, predictable, and traceable.

Критично: если обнаружены неожиданные сторонние изменения в тех же файлах, остановись и уточни у пользователя как продолжать.

## Verification and reporting

Before finishing, verify:

1. main user scenario works end-to-end
2. no obvious regression in neighbor templates/macros
3. touched files pass syntax/basic checks available in project
4. file list and rationale are clear

Final response format should include:

- what was changed
- why this approach is safe for UMI.CMS architecture
- what was verified
- remaining risks or assumptions

## References

Use these docs during implementation when needed:

- UMI docs: https://docs.umi-cms.ru/
- UMI API docs: https://api.docs.umi-cms.ru/
- Repository (if user has access): https://github.com/Umisoft/umi.cms.2
