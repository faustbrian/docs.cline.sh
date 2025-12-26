---
title: Custom Handlers
description: Create custom postal code validators for specific needs.
---

Create custom postal code validators for specific needs.

## Creating a Custom Handler

```php
use Cline\PostalCode\Contracts\PostalCodeHandler;
use Cline\PostalCode\PostalCode;

class CustomCountryHandler implements PostalCodeHandler
{
    public function validate(string $postalCode): bool
    {
        // Custom validation logic
        return preg_match('/^[A-Z]{2}\d{4}$/', $postalCode) === 1;
    }

    public function format(string $postalCode): string
    {
        // Custom formatting
        return strtoupper($postalCode);
    }

    public function getPattern(): string
    {
        return '/^[A-Z]{2}\d{4}$/';
    }
}

// Register the handler
PostalCode::extend('XX', new CustomCountryHandler());

// Use it
PostalCode::isValid('AB1234', 'XX'); // true
```

## Handler Interface

```php
interface PostalCodeHandler
{
    /**
     * Validate a postal code.
     */
    public function validate(string $postalCode): bool;

    /**
     * Format a postal code.
     */
    public function format(string $postalCode): string;

    /**
     * Get the regex pattern for validation.
     */
    public function getPattern(): string;
}
```

## Extending Existing Handlers

```php
use Cline\PostalCode\Handlers\USPostalCodeHandler;

class StrictUSHandler extends USPostalCodeHandler
{
    public function validate(string $postalCode): bool
    {
        // Call parent validation
        if (!parent::validate($postalCode)) {
            return false;
        }

        // Add custom rules (e.g., check against database)
        return $this->existsInDatabase($postalCode);
    }

    private function existsInDatabase(string $postalCode): bool
    {
        // Check if ZIP exists in your database
        return DB::table('zip_codes')
            ->where('code', $postalCode)
            ->exists();
    }
}

// Register
PostalCode::extend('US', new StrictUSHandler());
```

## Multiple Patterns

```php
class MultiPatternHandler implements PostalCodeHandler
{
    private array $patterns = [
        '/^\d{5}$/',           // 5 digits
        '/^\d{5}-\d{4}$/',     // ZIP+4
        '/^\d{9}$/',           // 9 digits without hyphen
    ];

    public function validate(string $postalCode): bool
    {
        foreach ($this->patterns as $pattern) {
            if (preg_match($pattern, $postalCode)) {
                return true;
            }
        }
        return false;
    }

    public function format(string $postalCode): string
    {
        // Normalize to ZIP+4 format if 9 digits
        if (preg_match('/^\d{9}$/', $postalCode)) {
            return substr($postalCode, 0, 5) . '-' . substr($postalCode, 5);
        }
        return $postalCode;
    }

    public function getPattern(): string
    {
        return '/^\d{5}(-\d{4})?$/';
    }
}
```

## Conditional Handlers

```php
class ConditionalHandler implements PostalCodeHandler
{
    public function __construct(
        private PostalCodeHandler $primaryHandler,
        private PostalCodeHandler $fallbackHandler,
        private callable $condition
    ) {}

    public function validate(string $postalCode): bool
    {
        $handler = ($this->condition)($postalCode)
            ? $this->primaryHandler
            : $this->fallbackHandler;

        return $handler->validate($postalCode);
    }

    // ... other methods
}

// Usage: Different rules for different regions
PostalCode::extend('XX', new ConditionalHandler(
    primaryHandler: new RegionAHandler(),
    fallbackHandler: new RegionBHandler(),
    condition: fn($code) => str_starts_with($code, 'A')
));
```

## Handler Factory

```php
use Cline\PostalCode\PostalCode;

class PostalCodeHandlerFactory
{
    public static function make(string $country): PostalCodeHandler
    {
        return match($country) {
            'US' => new USPostalCodeHandler(),
            'GB' => new UKPostalCodeHandler(),
            'DE' => new GermanPostalCodeHandler(),
            default => new GenericPostalCodeHandler(),
        };
    }
}
```
