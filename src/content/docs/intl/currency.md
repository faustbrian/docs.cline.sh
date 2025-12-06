---
title: Currency Value Object
description: Working with ISO 4217 currency codes using the Currency value object in Cline Intl
---

The `Currency` value object provides type-safe handling of ISO 4217 currency codes, including currency symbols, names, fraction digits, rounding increments, and numeric codes powered by Symfony Intl.

## Creating Currency Instances

### From String

Create a currency instance from an ISO 4217 currency code:

```php
use Cline\Intl\ValueObjects\Currency;

$currency = Currency::createFromString('USD');
```

The factory method accepts 3-letter ISO 4217 currency codes (e.g., `USD`, `EUR`, `GBP`) and automatically retrieves all currency metadata.

### From Enum

You can also use the `CurrencyCode` enum for type-safe currency code handling:

```php
use Cline\Intl\Enums\CurrencyCode;

$currencyCode = CurrencyCode::USD;
```

## Properties

The `Currency` value object provides comprehensive read-only properties:

```php
$currency = Currency::createFromString('USD');

// Currency code
echo $currency->code;  // "USD"

// Localized currency name
echo $currency->name;  // "US Dollar"

// Currency symbol
echo $currency->symbol;  // "$"

// Number of decimal places for standard transactions
echo $currency->fractionDigits;  // 2

// Rounding increment for standard transactions
echo $currency->roundingIncrement;  // 0

// Number of decimal places for cash transactions
echo $currency->cashFractionDigits;  // 2

// Rounding increment for cash transactions
echo $currency->cashRoundingIncrement;  // 0

// ISO 4217 numeric code (null if not available)
echo $currency->numericCode;  // 840
```

### Property Details

- **`code`** (string) - The ISO 4217 currency code (3 characters)
- **`name`** (string) - The localized currency name
- **`symbol`** (string) - The currency symbol (e.g., $, €, £)
- **`fractionDigits`** (int) - Default number of decimal places
- **`roundingIncrement`** (int) - Rounding increment for standard transactions
- **`cashFractionDigits`** (int) - Number of decimal places for cash transactions
- **`cashRoundingIncrement`** (int) - Rounding increment for cash transactions
- **`numericCode`** (int|null) - The ISO 4217 numeric code (if available)

## String Representation

The `Currency` object implements `Stringable` and returns the currency code when cast to string:

```php
$currency = Currency::createFromString('EUR');

echo (string) $currency;  // "EUR"
echo $currency->toString();  // "EUR"
echo $currency;  // "EUR" (implicit string cast)
```

## Equality Comparison

Compare two currency instances for equality:

```php
$currency1 = Currency::createFromString('USD');
$currency2 = Currency::createFromString('USD');
$currency3 = Currency::createFromString('EUR');

$currency1->isEqualTo($currency2);  // true
$currency1->isEqualTo($currency3);  // false
```

The comparison is based on the currency code.

## Understanding Fraction Digits

Different currencies have different decimal precision requirements:

```php
// Most currencies use 2 decimal places
$usd = Currency::createFromString('USD');
echo $usd->fractionDigits;  // 2 (e.g., $10.99)

$eur = Currency::createFromString('EUR');
echo $eur->fractionDigits;  // 2 (e.g., €10.99)

// Japanese Yen has no decimal places
$jpy = Currency::createFromString('JPY');
echo $jpy->fractionDigits;  // 0 (e.g., ¥1099)

// Bahraini Dinar uses 3 decimal places
$bhd = Currency::createFromString('BHD');
echo $bhd->fractionDigits;  // 3 (e.g., BD 10.995)
```

## Eloquent Model Integration

### Using CurrencyCast

Add currency support to your Eloquent models using the `CurrencyCast`:

```php
use Illuminate\Database\Eloquent\Model;
use Cline\Intl\Data\Cast\CurrencyCast;

class Product extends Model
{
    protected function casts(): array
    {
        return [
            'currency' => CurrencyCast::class,
        ];
    }
}
```

### Working with the Cast

```php
// Store as string, retrieve as Currency object
$product = new Product();
$product->currency = 'USD';
$product->price = 19.99;
$product->save();

// Automatically cast to Currency object
echo $product->currency->name;            // "US Dollar"
echo $product->currency->symbol;          // "$"
echo $product->currency->fractionDigits;  // 2

// Format price with currency
echo $product->currency->symbol . number_format(
    $product->price,
    $product->currency->fractionDigits
);  // "$19.99"

// Update with string
$product->currency = 'EUR';
$product->save();
```

### Database Schema

Store the currency as a simple string column:

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->decimal('price', 10, 2);
    $table->string('currency', 3);  // ISO 4217 code
    $table->timestamps();
});
```

## Validation

Validate currency input using the `CurrencyRule`:

```php
use Cline\Intl\Rules\CurrencyRule;
use Illuminate\Http\Request;

class CreateProductRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'currency' => ['required', 'string', new CurrencyRule()],
            'price' => ['required', 'numeric', 'min:0'],
        ];
    }
}
```

The rule validates that the input is a valid ISO 4217 currency code.

## Common Use Cases

### E-commerce Product Pricing

```php
use Cline\Intl\ValueObjects\Currency;
use Cline\Intl\Data\Cast\CurrencyCast;

class Product extends Model
{
    protected function casts(): array
    {
        return [
            'currency' => CurrencyCast::class,
        ];
    }

    public function getFormattedPriceAttribute(): string
    {
        return $this->currency->symbol . number_format(
            $this->price,
            $this->currency->fractionDigits
        );
    }

    public function convertTo(Currency $targetCurrency, float $exchangeRate): float
    {
        return round(
            $this->price * $exchangeRate,
            $targetCurrency->fractionDigits
        );
    }
}

// Usage
$product = Product::find(1);
echo $product->formatted_price;  // "$19.99"

$targetCurrency = Currency::createFromString('EUR');
$convertedPrice = $product->convertTo($targetCurrency, 0.85);
```

### Multi-Currency Support

```php
use Cline\Intl\ValueObjects\Currency;
use Cline\Intl\Data\Cast\CurrencyCast;

class Order extends Model
{
    protected function casts(): array
    {
        return [
            'currency' => CurrencyCast::class,
            'customer_currency' => CurrencyCast::class,
        ];
    }

    public function getTotalInCustomerCurrency(): string
    {
        // Use an exchange rate service
        $rate = app(ExchangeRateService::class)
            ->getRate($this->currency, $this->customer_currency);

        $convertedTotal = $this->total * $rate;

        return $this->customer_currency->symbol . number_format(
            $convertedTotal,
            $this->customer_currency->fractionDigits
        );
    }

    public function needsCurrencyConversion(): bool
    {
        return !$this->currency->isEqualTo($this->customer_currency);
    }
}
```

### Currency Selector

```php
use Symfony\Component\Intl\Currencies;

class CurrencyController extends Controller
{
    public function index()
    {
        // Get all currencies for a dropdown
        $currencies = collect(Currencies::getNames())
            ->map(fn($name, $code) => [
                'code' => $code,
                'name' => $name,
                'symbol' => Currencies::getSymbol($code),
            ])
            ->sortBy('name')
            ->values();

        return view('currencies.index', compact('currencies'));
    }
}
```

### Price Formatting Service

```php
use Cline\Intl\ValueObjects\Currency;

class PriceFormatter
{
    public function format(float $amount, Currency $currency): string
    {
        $formatted = number_format(
            $amount,
            $currency->fractionDigits,
            '.',
            ','
        );

        return $currency->symbol . $formatted;
    }

    public function formatWithCode(float $amount, Currency $currency): string
    {
        $formatted = number_format(
            $amount,
            $currency->fractionDigits,
            '.',
            ','
        );

        return $formatted . ' ' . $currency->code;
    }
}

// Usage
$formatter = new PriceFormatter();
$currency = Currency::createFromString('USD');

echo $formatter->format(1234.56, $currency);          // "$1,234.56"
echo $formatter->formatWithCode(1234.56, $currency);  // "1,234.56 USD"
```

### Handling Zero-Decimal Currencies

```php
use Cline\Intl\ValueObjects\Currency;

class PaymentProcessor
{
    public function convertToSmallestUnit(float $amount, Currency $currency): int
    {
        // Stripe and many payment processors require amounts in smallest units
        // USD: 19.99 -> 1999 (cents)
        // JPY: 1999 -> 1999 (yen, no conversion needed)

        return (int) round($amount * (10 ** $currency->fractionDigits));
    }

    public function convertFromSmallestUnit(int $amount, Currency $currency): float
    {
        return $amount / (10 ** $currency->fractionDigits);
    }
}

// Usage
$processor = new PaymentProcessor();
$usd = Currency::createFromString('USD');
$jpy = Currency::createFromString('JPY');

// USD has 2 fraction digits
$processor->convertToSmallestUnit(19.99, $usd);  // 1999 cents

// JPY has 0 fraction digits
$processor->convertToSmallestUnit(1999, $jpy);  // 1999 yen
```

## Exception Handling

The `createFromString()` method throws a `MissingResourceException` if the currency code is invalid:

```php
use Symfony\Component\Intl\Exception\MissingResourceException;
use Cline\Intl\ValueObjects\Currency;

try {
    $currency = Currency::createFromString('INVALID');
} catch (MissingResourceException $e) {
    // Handle invalid currency code
    Log::error('Invalid currency code provided', [
        'code' => 'INVALID',
        'message' => $e->getMessage(),
    ]);
}
```

Always validate user input using the `CurrencyRule` to prevent exceptions.

## Best Practices

1. **Always validate input** - Use `CurrencyRule` in form requests before creating Currency objects
2. **Store ISO codes** - Use 3-character columns in the database for consistency
3. **Use fraction digits** - Respect the currency's `fractionDigits` when formatting amounts
4. **Handle zero-decimal currencies** - JPY, KRW, and others have no decimal places
5. **Consider cash rounding** - Some currencies have different rounding for cash vs. electronic transactions
6. **Use with money libraries** - Consider pairing with libraries like `moneyphp/money` or `brick/money` for advanced calculations

## Integration with Money Libraries

### Using with Brick Money

```php
use Brick\Money\Money;
use Cline\Intl\ValueObjects\Currency;

$currency = Currency::createFromString('USD');

// Create Money instance
$money = Money::of(19.99, $currency->code);

echo $money->formatTo('en_US');  // "$19.99"
```

## Related Resources

- [Symfony Intl Currencies Component](https://symfony.com/doc/current/components/intl.html)
- [ISO 4217 Standard](https://en.wikipedia.org/wiki/ISO_4217)
- [Validation Rules](/intl/casts-and-rules)
