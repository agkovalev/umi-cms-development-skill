# Common Extension Patterns

## Pattern A: Add Custom Macro

**Use when:** Need new data display logic in XSLT templates, or new computed field.

**File:** `templates/<template>/php/customMacros.php` or module `customMacros.php`

**Example:**
```php
<?php
namespace UmiCms\<Module>;

use UmiCms\Service;

class customMacros {
    /**
     * Format product price with currency and discount
     * @param float $price base price
     * @param float $discount percentage
     * @return string formatted HTML
     */
    public static function formatPriceWithDiscount($price, $discount = 0) {
        $final = $price * (1 - $discount / 100);
        return sprintf(
            '<span class="price">%s ₽</span><span class="discount">-%d%%</span>',
            number_format($final, 2, '.', ' '),
            (int) $discount
        );
    }
}
?>
```

**In XSLT:**
```xml
<xsl:value-of select="php:function('customMacros::formatPriceWithDiscount', $price, $discount)" />
```

**Risks:** None if macro is pure function. Medium risk if uses global state or errors.

---

## Pattern B: Subscribe to Module Event

**Use when:** Need to react to data changes (item created, updated, deleted), or log actions.

**File:** `classes/components/<module>/events.php`

**Example:**
```php
<?php
namespace UmiCms\Catalog;

class events {
    public static function registerHandlers() {
        // Subscribe to product update event
        $dispatcher = \UmiCms\Service::Events();
        $dispatcher->subscribe(
            'exchangeOnUpdateElement',
            ['UmiCms\Custom\ProductSync', 'onProductUpdate']
        );
    }
}

events::registerHandlers();
?>
```

**Handler in `custom_classes/ProductSync.php`:**
```php
<?php
namespace UmiCms\Custom;

class ProductSync {
    public static function onProductUpdate($elementId, $objectId) {
        // Log or sync to external service
        error_log("Product updated: $elementId");
    }
}
?>
```

**Risks:** Medium. Event handlers block page rendering if slow. Add async queue if needed.

---

## Pattern C: Extend Admin Functionality

**Use when:** Need new admin UI field, button, or setting.

**File:** `classes/components/<module>/customAdmin.php`

**Example:**
```php
<?php
namespace UmiCms\Catalog;

class customAdmin extends admin {
    /**
     * Add extra fields to product edit form
     */
    public function showEditPage() {
        // Call parent
        parent::showEditPage();
        
        // Add custom field rendering
        $this->renderCustomField('external_id', 'External ID');
    }
}
?>
```

**Risks:** Low if only adding new fields. High if manipulating core form logic.

---

## Pattern D: Add Custom Data Provider

**Use when:** Need computed/aggregated data for template without core logic change.

**File:** `classes/components/<module>/class.php` (safe if method is new)

**Example:**
```php
<?php
public function getProductsWithRating($limit = 10) {
    $products = $this->getProducts($limit);
    
    // Enrich with external rating data
    foreach ($products as &$product) {
        $rating = $this->fetchExternalRating($product['id']);
        $product['rating'] = $rating;
    }
    
    return $products;
}
?>
```

**Risks:** Low if method is new. Medium if called from XSLT (must handle errors gracefully).

---

## Pattern E: Integrate External Service

**Use when:** Need to call third-party API from module.

**File:** `custom_classes/ExternalService.php` (isolated client)

**Example:**
```php
<?php
namespace UmiCms\Custom;

class ExternalService {
    private $apiKey;
    private $endpoint;
    
    public function __construct() {
        $this->apiKey = getenv('EXTERNAL_API_KEY');
        $this->endpoint = 'https://api.example.com/v1';
    }
    
    public function syncProduct($productId) {
        try {
            $response = $this->call('POST', '/sync', ['id' => $productId]);
            error_log("Sync OK: $productId");
            return $response;
        } catch (\Exception $e) {
            error_log("Sync failed: " . $e->getMessage());
            return null; // Graceful degrade
        }
    }
    
    private function call($method, $path, $body) {
        // Isolated HTTP logic
    }
}
?>
```

**Trigger point:** Event handler or custom macro

**Risks:** Medium-High. Network failures, timeouts, rate limiting. Add fallback behavior.

---

## Pattern F: Add Custom Handler

**Use when:** Need new AJAX endpoint or form handler.

**File:** `classes/components/<module>/handlers.php` or event via routing

**Example (event-driven):**
```php
<?php
// In events.php:
$dispatcher->subscribe('beforeObjectCreate', [
    'UmiCms\Custom\ValidateProduct', 'onValidate'
]);

// In custom_classes/ValidateProduct.php:
class ValidateProduct {
    public static function onValidate($objectData) {
        if (empty($objectData['name'])) {
            throw new \Exception('Name required');
        }
    }
}
?>
```

**Risks:** Medium. Request validation errors should degrade gracefully, not crash.

---

## Backward Compatibility Checklist

- ✓ New method? Safe (non-breaking).
- ✓ Adding new return key to existing method? Usually safe (templates ignore unknown keys).
- ✗ Removing return key? **Breaking change.**
- ✗ Changing macro parameter order? **Breaking change.**
- ✗ Changing permission requirements? **May break admin workflows.**

When in doubt, add as **new method** instead of modifying existing.
