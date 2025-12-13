---
title: Country Value Object
description: Working with ISO 3166-1 country codes using the Country value object in Cline Intl
---

The `Country` value object provides type-safe handling of ISO 3166-1 country codes with support for both alpha-2 and alpha-3 formats, along with localized country names powered by Symfony Intl.

## Creating Country Instances

### From String

Create a country instance from an ISO 3166-1 alpha-2 country code:

```php
use Cline\Intl\ValueObjects\Country;

$country = Country::createFromString('US');
```

The factory method accepts alpha-2 codes (e.g., `US`, `GB`, `DE`) and automatically retrieves the corresponding alpha-3 code and localized name.

### From Enum

You can also use the `CountryCode` enum for type-safe country code handling:

```php
use Cline\Intl\Enums\CountryCode;

$countryCode = CountryCode::US;
```

## Properties

The `Country` value object provides three read-only properties:

```php
$country = Country::createFromString('US');

// Localized country name (based on application locale)
echo $country->localized;  // "United States"

// ISO 3166-1 alpha-2 code (2 letters)
echo $country->alpha2;  // "US"

// ISO 3166-1 alpha-3 code (3 letters)
echo $country->alpha3;  // "USA"
```

### Property Details

- **`localized`** (string) - The localized country name based on your application's current locale
- **`alpha2`** (string) - The ISO 3166-1 alpha-2 country code (2 characters)
- **`alpha3`** (string|null) - The ISO 3166-1 alpha-3 country code (3 characters)

## String Representation

The `Country` object implements `Stringable` and returns the alpha-2 code when cast to string:

```php
$country = Country::createFromString('GB');

echo (string) $country;  // "GB"
echo $country->toString();  // "GB"
echo $country;  // "GB" (implicit string cast)
```

## Equality Comparison

Compare two country instances for equality:

```php
$country1 = Country::createFromString('US');
$country2 = Country::createFromString('US');
$country3 = Country::createFromString('GB');

$country1->isEqualTo($country2);  // true
$country1->isEqualTo($country3);  // false
```

The comparison is based on the alpha-2 country code.

## Eloquent Model Integration

### Using CountryCast

Add country support to your Eloquent models using the `CountryCast`:

```php
use Illuminate\Database\Eloquent\Model;
use Cline\Intl\Data\Cast\CountryCast;

class User extends Model
{
    protected function casts(): array
    {
        return [
            'country' => CountryCast::class,
            'billing_country' => CountryCast::class,
            'shipping_country' => CountryCast::class,
        ];
    }
}
```

### Working with the Cast

```php
// Store as string, retrieve as Country object
$user = new User();
$user->country = 'US';
$user->save();

// Automatically cast to Country object
echo $user->country->localized;  // "United States"
echo $user->country->alpha2;     // "US"
echo $user->country->alpha3;     // "USA"

// Update with string
$user->country = 'GB';
$user->save();
```

### Database Schema

Store the country as a simple string column:

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('country', 2);  // alpha-2 code
    $table->string('billing_country', 2)->nullable();
    $table->string('shipping_country', 2)->nullable();
    $table->timestamps();
});
```

## Validation

Validate country input using the `CountryRule`:

```php
use Cline\Intl\Rules\CountryRule;
use Illuminate\Http\Request;

class UpdateProfileRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'country' => ['required', 'string', new CountryRule()],
            'billing_country' => ['nullable', 'string', new CountryRule()],
        ];
    }
}
```

The rule validates that the input is a valid ISO 3166-1 alpha-2 country code.

## Common Use Cases

### User Profile

```php
use Cline\Intl\ValueObjects\Country;
use Cline\Intl\Data\Cast\CountryCast;

class User extends Model
{
    protected function casts(): array
    {
        return [
            'country' => CountryCast::class,
        ];
    }

    public function getCountryDisplayName(): string
    {
        return $this->country?->localized ?? 'Not specified';
    }
}
```

### Order Shipping

```php
use Cline\Intl\ValueObjects\Country;
use Cline\Intl\Data\Cast\CountryCast;

class Order extends Model
{
    protected function casts(): array
    {
        return [
            'shipping_country' => CountryCast::class,
            'billing_country' => CountryCast::class,
        ];
    }

    public function isDomesticShipping(): bool
    {
        $shopCountry = Country::createFromString(config('shop.country'));

        return $this->shipping_country->isEqualTo($shopCountry);
    }

    public function getShippingZone(): string
    {
        return match ($this->shipping_country->alpha2) {
            'US' => 'domestic',
            'CA', 'MX' => 'north_america',
            default => 'international',
        };
    }
}
```

### Country Selection Dropdown

```php
use Symfony\Component\Intl\Countries;

class CountryController extends Controller
{
    public function index()
    {
        // Get all country codes and names for a dropdown
        $countries = collect(Countries::getNames())
            ->map(fn($name, $code) => [
                'code' => $code,
                'name' => $name,
            ])
            ->sortBy('name')
            ->values();

        return view('countries.index', compact('countries'));
    }
}
```

### Filtering by Region

```php
use Cline\Intl\ValueObjects\Country;

class Product extends Model
{
    protected function casts(): array
    {
        return [
            'available_countries' => 'array',
        ];
    }

    public function isAvailableInCountry(Country $country): bool
    {
        if (empty($this->available_countries)) {
            return true; // Available everywhere
        }

        return in_array($country->alpha2, $this->available_countries);
    }
}

// Usage
$product = Product::find(1);
$userCountry = Country::createFromString('US');

if ($product->isAvailableInCountry($userCountry)) {
    // Show product
}
```

## Exception Handling

The `createFromString()` method throws a `MissingResourceException` if the country code is invalid:

```php
use Symfony\Component\Intl\Exception\MissingResourceException;
use Cline\Intl\ValueObjects\Country;

try {
    $country = Country::createFromString('INVALID');
} catch (MissingResourceException $e) {
    // Handle invalid country code
    Log::error('Invalid country code provided', [
        'code' => 'INVALID',
        'message' => $e->getMessage(),
    ]);
}
```

Always validate user input using the `CountryRule` to prevent exceptions.

## Best Practices

1. **Always validate input** - Use `CountryRule` in form requests before creating Country objects
2. **Store alpha-2 codes** - Use 2-character columns in the database for consistency
3. **Use casts in models** - Let Eloquent handle the conversion automatically
4. **Leverage localization** - The `localized` property automatically respects your app's locale
5. **Consider nullable columns** - Not all records may require a country (e.g., optional billing address)

## Related Resources

- [Symfony Intl Countries Component](https://symfony.com/doc/current/components/intl.html)
- [ISO 3166-1 Standard](https://en.wikipedia.org/wiki/ISO_3166-1)
- [Validation Rules](/intl/casts-and-rules)
