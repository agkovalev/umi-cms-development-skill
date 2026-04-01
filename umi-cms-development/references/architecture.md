# UMI.CMS Architecture Overview

## Codebase Structure

```
classes/
  ├── components/         # Business modules (catalog, emarket, users, etc.)
  ├── system/             # Core (avoid direct edits)
  └── vendor/             # Composer dependencies

templates/
  └── <template-name>/
      ├── xslt/           # XSLT template views
      ├── js/             # Frontend JavaScript
      ├── css/            # Stylesheets and LESS
      ├── tpls/           # PHP view helpers (optional)
      └── images/         # Template assets

custom_classes/
  └── import/             # Custom import splitters

tests/
  ├── core/               # Core system tests
  ├── modules/            # Module-specific tests
  ├── PHPUnit/            # Unit tests (modern)
  └── simpletest/         # Legacy web tests
```

## Module Structure

Typical module in `classes/components/<module>/`:

```
<module>/
  ├── class.php           # Main module class (extends def_module)
  ├── admin.php           # Admin functionality
  ├── customAdmin.php     # Admin extensions (safe override point)
  ├── macros.php          # XSLT macros
  ├── customMacros.php    # Macro extensions (safe override point)
  ├── events.php          # Event handler registration
  ├── handlers.php        # Custom handlers
  ├── i18n.php            # Localization keys
  ├── manifest/           # Module metadata
  └── Classes/            # Additional classes
```

## Extension Points (Priority Order)

When extending module behavior, use these points in order:

### 1. Custom Macros (`customMacros.php`)
- Add new XSLT-callable functions for templates
- Isolated from core logic
- Safest for frontend-facing features

**Trigger:** Scope = display logic, data formatting, conditional rendering

### 2. Module Events (`events.php` → handlers)
- Subscribe to module lifecycle hooks
- Observe changes and trigger side effects
- Examples: `exchangeOnUpdateElement`, `beforeObjectCreate`, `afterObjectDelete`

**Trigger:** Scope = react to data changes, trigger external integrations, audit logging

### 3. Custom Handlers (`handlers.php`)
- Custom action handlers for forms/AJAX
- Extend module API without modifying core routes

**Trigger:** Scope = new user actions, form submissions, AJAX endpoints

### 4. Extension Override (`ext/` folder logic)
- Special loading hooks: `autoload_*.php`, `common_*.php`
- Used for pre/post hooks, monkey-patching

**Trigger:** Scope = low-level behavior modification when extension points insufficient

### 5. Direct Core Modification (LAST RESORT)
- Edit `class.php`, core methods
- High risk; only when no extension point exists
- Document reasoning in comments

**Trigger:** Scope = system internals, API contracts, no alternatives exist

## Service Container

Global access pattern:

```php
use UmiCms\Service;

$database = Service::Database();
$request  = Service::Request();
$cache    = Service::Cache();
$config   = Service::Configuration();
```

Common services:
- `Database()` — active DB connection
- `Request()` — current HTTP request
- `Response()` — response builder
- `Cache()` — cache engine
- `Template()` — template renderer
- `Session()` — user session
- `Configuration()` — config registry
- `Events()` — event dispatcher

## XSLT Data Flow

1. PHP module method called → returns array/object
2. Data passed to XSLT template via `$data` variable
3. XSLT macros transform data to HTML
4. Output rendered to client

Safe points:
- Add new macros in `customMacros.php`
- Add new data providers in `class.php` public methods
- Return new keys in existing methods (backward compatible)

Unsafe:
- Changing macro parameter order (breaks existing templates)
- Removing existing keys from returned data
- Changing data type signatures

## Template Integration Workflow

```
1. Static HTML/CSS/JS from designer
   ↓
2. Identify dynamic blocks (user-visible content)
   ↓
3. Map to existing module macros/data
   ↓
4. Create XSLT fragments for dynamic sections
   ↓
5. Include in main layout template
   ↓
6. Test on real CMS data
```

## Backward Compatibility Rules

- Never remove public methods or XSLT macros
- Never change macro parameter order/count
- Never change return data types
- Always add as `@deprecated` before removal (wait 2+ releases)
- Keep existing template variables if adding new ones

Reasoning: Live sites depend on exact API contracts. Breaking changes cascade across client projects.
