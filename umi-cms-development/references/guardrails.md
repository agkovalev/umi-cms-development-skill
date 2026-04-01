# Security and Safety Guardrails

## Absolute Never Dos

### 🔴 Never 1: Hardcode secrets or API keys

```php
// ❌ NEVER:
$apiKey = 'sk_live_abc123xyz';
curl_setopt($curl, CURLOPT_HTTPAUTH, [
    'username' => 'admin',
    'password' => 'password123'
]);
```

**Why:** Credentials will be committed to version control and leaked publicly.

**Do instead:**
```php
// ✅ CORRECT:
$apiKey = getenv('EXTERNAL_API_KEY');
if (!$apiKey) {
    throw new Exception('EXTERNAL_API_KEY not set in environment');
}
```

---

### 🔴 Never 2: Modify `classes/system/` directly without explicit request

```php
// ❌ NEVER without justification:
// Edit classes/system/bootstrap/ServiceContainer.php
// Modify the core request/response flow
```

**Why:** System internals power all modules. Changes cascade and break other features.

**Do instead:**
- Use Service facade (`Service::Request()`, etc.)
- Add custom event handlers or extension points
- Create custom class in `custom_classes/`

---

### 🔴 Never 3: Skip permission/capability checks

```php
// ❌ NEVER (even in custom code):
public function deleteProduct($productId) {
    // Directly delete without checking who's asking
    $db = Service::Database();
    $db->query("DELETE FROM products WHERE id = $productId");
}
```

**Why:** Anyone with code access can perform admin actions.

**Do instead:**
```php
// ✅ CORRECT:
public function deleteProduct($productId) {
    if (!$this->checkPermission('delete_product')) {
        throw new \Exception('Access denied');
    }
    
    $db = Service::Database();
    $db->query("DELETE FROM products WHERE id = ? ", [$productId]);
}
```

---

### 🔴 Never 4: Mix declaration and side effects in one file

```php
// ❌ NEVER (in same file):
<?php
// Declare class
class MyModule {
    // ...
}

// Side effect: include another file
include 'setup.php';  // ← VIOLATION

// Side effect: output HTML
echo '<div>test</div>';  // ← VIOLATION
?>
```

**Why:** Makes autoloading and testing impossible. Causes silent failures.

**Do instead:**
- File A (declaration only): `class MyModule { }`
- File B (side effects): includes, setup, output
- Separate concerns.

---

### 🔴 Never 5: Use unsanitized user input in queries or output

```php
// ❌ NEVER:
$search = $_GET['q'];
$results = $db->query("SELECT * FROM products WHERE name LIKE '%$search%'");
echo $results->name;  // Also vulnerable to XSS
```

**Why:** SQL injection and XSS attacks.

**Do instead:**
```php
// ✅ CORRECT:
$search = $_GET['q'];
$results = $db->query(
    "SELECT * FROM products WHERE name LIKE ?",
    ["%$search%"]
);
echo htmlspecialchars($results->name, ENT_QUOTES, 'UTF-8');
```

---

## High Caution Patterns

### ⚠️ Caution A: Event handlers that are slow

```php
// Event handler (runs on every change):
public function onProductUpdate($productId) {
    // ⚠️ This BLOCKS page render:
    $this->syncToExternalService($productId);
}
```

**How to fix:**
- Queue for async processing
- Return quickly, defer work to background job
- Add timeout + fallback

---

### ⚠️ Caution B: Changing existing macro parameters

```php
// OLD (existing):
<xsl:call-template name="macroProduct">
    <xsl:with-param name="id" select="$id"/>
</xsl:call-template>

// ❌ DON'T CHANGE TO:
// Reorder params, change count, change types
// → BREAKS all existing calls
```

**Why:** Templates calling this macro will break silently or produce wrong output.

**Do instead:**
- Add new params as optional
- Create new macro if behavior changes significantly
- Document any new params in PHPDoc

---

### ⚠️ Caution C: Direct database schema edits

```php
// ❌ RISKY:
// Edit tables directly without migration
$db->query("ALTER TABLE products ADD COLUMN new_field TEXT");
```

**Why:** Breaks consistency; other instances of same codebase fail.

**Do instead:**
- Document in upgrade routine
- Check for column existence before using
- Provide rollback path

---

## Verification Checklist Before Commit

- [ ] No hardcoded secrets or credentials
- [ ] No modifications to `classes/system/` unless justified
- [ ] All user input is sanitized before DB queries
- [ ] All output is escaped for HTML context
- [ ] No side effects in class declaration files
- [ ] Permissions checked for sensitive operations
- [ ] Event handlers return quickly (no blocking operations)
- [ ] New code follows UTF-8, proper namespaces, camelCase naming
- [ ] New methods/macros are documented with PHPDoc
- [ ] Backward compatibility maintained (no breaking changes to public APIs)
- [ ] Related test files (if exist) still pass
- [ ] No merge conflicts or unexpected file changes

---

## Common Failure Modes

| Failure | Cause | Fix |
|---------|-------|-----|
| Event handler not firing | Wrong module loaded, wrong event name, wrong class path | Check module registration, test with error_log in handler |
| Macro output broken/missing | Parameter mismatch, XSLT syntax error, data is null | Check template syntax, verify data source provides expected keys |
| Permission denied errors | Missing capability check, user not in right group | Add debug logging to show actual permission value |
| Database query hanging | Missing WHERE clause, N+1 queries in loop | Optimize SELECT, batch operations |
| Assets 404/broken paths | Template paths, relative vs absolute, incorrect asset directories | Verify static file paths relative to template root |
| External API failures | Timeout, auth failure, wrong endpoint | Add logging, implement retry + fallback, verify credentials |

---

## Escalation Criteria

Stop and ask the user if:

1. Task involves changes to `classes/system/` core code.
2. Task changes existing database schema without migration path.
3. Task involves customer-facing permission/security changes.
4. Two or more modules need coordinated changes.
5. External service integration requires credentials or infrastructure setup.
