---
title: Casts & Validation Rules
description: Complete guide to using Eloquent casts and validation rules with Cline Intl value objects
---

Cline Intl provides both Eloquent model casts and Laravel validation rules for all internationalization value objects. This page provides a comprehensive guide to using both features together in your Laravel applications.

## Overview

Each value object in Cline Intl comes with two essential features:

1. **Eloquent Casts** - Automatically convert database values to/from value objects
2. **Validation Rules** - Validate user input before it reaches your models

This separation ensures that invalid data never enters your database while providing convenient object access in your application code.

## Eloquent Casts

### Available Casts

```php
use Cline\Intl\Data\Cast\CountryCast;
use Cline\Intl\Data\Cast\CurrencyCast;
use Cline\Intl\Data\Cast\LanguageCast;
use Cline\Intl\Data\Cast\LocaleCast;
use Cline\Intl\Data\Cast\TimeZoneCast;
use Cline\Intl\Data\Cast\PhoneNumberCast;
use Cline\Intl\Data\Cast\PostalCodeCast;
```

### Basic Usage

Define casts in your Eloquent model's `casts()` method:

```php
use Illuminate\Database\Eloquent\Model;
use Cline\Intl\Data\Cast\CountryCast;
use Cline\Intl\Data\Cast\CurrencyCast;
use Cline\Intl\Data\Cast\LanguageCast;
use Cline\Intl\Data\Cast\TimeZoneCast;

class User extends Model
{
    protected function casts(): array
    {
        return [
            'country' => CountryCast::class,
            'currency' => CurrencyCast::class,
            'language' => LanguageCast::class,
            'timezone' => TimeZoneCast::class,
        ];
    }
}
```

### How Casts Work

Casts automatically handle conversion between database strings and value objects:

```php
// Setting values (accepts strings)
$user = new User();
$user->country = 'US';          // Stored as "US" in database
$user->currency = 'USD';        // Stored as "USD" in database
$user->language = 'en';         // Stored as "en" in database
$user->timezone = 'UTC';        // Stored as "UTC" in database
$user->save();

// Retrieving values (returns value objects)
$user = User::find(1);
echo $user->country->localized;     // "United States"
echo $user->currency->symbol;       // "$"
echo $user->language->localized;    // "English"
echo $user->timezone->localized;    // "Coordinated Universal Time"
```

### Null Handling

All casts gracefully handle null values:

```php
class User extends Model
{
    protected function casts(): array
    {
        return [
            'country' => CountryCast::class,
        ];
    }
}

$user = new User();
$user->country = null;
$user->save();

$user->fresh();
var_dump($user->country);  // null
```

Ensure your database columns are nullable if you want to allow null values:

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('country', 2)->nullable();
});
```

## Validation Rules

### Available Rules

```php
use Cline\Intl\Rules\CountryRule;
use Cline\Intl\Rules\CurrencyRule;
use Cline\Intl\Rules\LanguageRule;
use Cline\Intl\Rules\LocaleRule;
use Cline\Intl\Rules\TimeZoneRule;
use Cline\Intl\Rules\PhoneNumberRule;
```

### Basic Usage

Use validation rules in form request classes:

```php
use Illuminate\Foundation\Http\FormRequest;
use Cline\Intl\Rules\CountryRule;
use Cline\Intl\Rules\CurrencyRule;
use Cline\Intl\Rules\LanguageRule;
use Cline\Intl\Rules\TimeZoneRule;

class UpdateProfileRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'country' => ['required', 'string', new CountryRule()],
            'currency' => ['required', 'string', new CurrencyRule()],
            'language' => ['required', 'string', new LanguageRule()],
            'timezone' => ['required', 'string', new TimeZoneRule()],
        ];
    }
}
```

### Controller Usage

```php
use Cline\Intl\Rules\CountryRule;

class UserController extends Controller
{
    public function update(UpdateProfileRequest $request)
    {
        $user = auth()->user();

        // Values are already validated by the form request
        $user->update($request->validated());

        return redirect()->back()->with('success', 'Profile updated');
    }

    // Or validate directly in the controller
    public function updateQuick(Request $request)
    {
        $validated = $request->validate([
            'country' => ['required', 'string', new CountryRule()],
        ]);

        auth()->user()->update($validated);
    }
}
```

## Complete Integration Example

Here's a complete example showing casts and validation working together:

### Database Migration

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->string('country', 2)->nullable();
            $table->string('currency', 3)->default('USD');
            $table->string('language', 2)->default('en');
            $table->string('timezone', 50)->default('UTC');
            $table->string('phone', 20)->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};
```

### Eloquent Model

```php
use Illuminate\Database\Eloquent\Model;
use Cline\Intl\Data\Cast\CountryCast;
use Cline\Intl\Data\Cast\CurrencyCast;
use Cline\Intl\Data\Cast\LanguageCast;
use Cline\Intl\Data\Cast\TimeZoneCast;
use Cline\Intl\Data\Cast\PhoneNumberCast;

class User extends Model
{
    protected $fillable = [
        'name',
        'email',
        'country',
        'currency',
        'language',
        'timezone',
        'phone',
    ];

    protected function casts(): array
    {
        return [
            'country' => CountryCast::class,
            'currency' => CurrencyCast::class,
            'language' => LanguageCast::class,
            'timezone' => TimeZoneCast::class,
            'phone' => PhoneNumberCast::class,
        ];
    }
}
```

### Form Request

```php
use Illuminate\Foundation\Http\FormRequest;
use Cline\Intl\Rules\CountryRule;
use Cline\Intl\Rules\CurrencyRule;
use Cline\Intl\Rules\LanguageRule;
use Cline\Intl\Rules\TimeZoneRule;
use Cline\Intl\Rules\PhoneNumberRule;

class UpdateUserProfileRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'country' => ['nullable', 'string', new CountryRule()],
            'currency' => ['required', 'string', new CurrencyRule()],
            'language' => ['required', 'string', new LanguageRule()],
            'timezone' => ['required', 'string', new TimeZoneRule()],
            'phone' => ['nullable', 'string', new PhoneNumberRule()],
        ];
    }

    public function messages(): array
    {
        return [
            'country.required' => 'Please select a country.',
            'currency.required' => 'Please select a currency.',
            'language.required' => 'Please select a language.',
            'timezone.required' => 'Please select a timezone.',
        ];
    }
}
```

### Controller

```php
use App\Http\Requests\UpdateUserProfileRequest;

class ProfileController extends Controller
{
    public function edit()
    {
        $user = auth()->user();

        return view('profile.edit', compact('user'));
    }

    public function update(UpdateUserProfileRequest $request)
    {
        $user = auth()->user();

        // All values are validated and will be automatically cast
        $user->update($request->validated());

        // Access as value objects
        logger()->info('Profile updated', [
            'user_id' => $user->id,
            'country' => $user->country?->alpha2,
            'currency' => $user->currency->code,
            'language' => $user->language->value,
            'timezone' => $user->timezone->value,
        ]);

        return redirect()
            ->route('profile.edit')
            ->with('success', 'Profile updated successfully');
    }
}
```

## Advanced Validation: PhoneNumberRule

The `PhoneNumberRule` is more sophisticated and supports additional options for phone number validation:

### Basic Phone Number Validation

```php
use Cline\Intl\Rules\PhoneNumberRule;

$request->validate([
    'phone' => ['required', 'string', new PhoneNumberRule()],
]);
```

### Validation with Region Code

Provide a specific region code for validation:

```php
use Cline\Intl\Rules\PhoneNumberRule;

$request->validate([
    'phone' => ['required', 'string', new PhoneNumberRule('US')],
]);
```

### Validation with Dynamic Region Code

Use another field's value as the region code:

```php
use Cline\Intl\Rules\PhoneNumberRule;

$request->validate([
    'country' => ['required', 'string', new CountryRule()],
    'phone' => [
        'required',
        'string',
        new PhoneNumberRule(regionCodeReference: 'country'),
    ],
]);
```

This validates that the phone number is valid for the selected country.

### Strict Validation

By default, the rule checks if a phone number is "possible" (has valid length and format). Use strict mode to validate that the number is fully valid:

```php
use Cline\Intl\Rules\PhoneNumberRule;

$request->validate([
    'phone' => [
        'required',
        'string',
        new PhoneNumberRule(
            regionCode: 'US',
            shouldBeStrict: true
        ),
    ],
]);
```

### Complete Phone Number Example

```php
class CreateContactRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'country' => ['required', 'string', new CountryRule()],
            'phone' => [
                'required',
                'string',
                new PhoneNumberRule(
                    regionCodeReference: 'country',
                    shouldBeStrict: true
                ),
            ],
        ];
    }

    public function messages(): array
    {
        return [
            'phone.required' => 'Please provide a phone number.',
        ];
    }
}
```

## Custom Error Messages

Customize validation error messages in your form request:

```php
class UpdateProfileRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'country' => ['required', 'string', new CountryRule()],
            'currency' => ['required', 'string', new CurrencyRule()],
            'language' => ['required', 'string', new LanguageRule()],
        ];
    }

    public function messages(): array
    {
        return [
            'country.required' => 'Please select your country.',
            'currency.required' => 'Please select your preferred currency.',
            'language.required' => 'Please select your language.',
        ];
    }

    public function attributes(): array
    {
        return [
            'country' => 'country',
            'currency' => 'preferred currency',
            'language' => 'language',
        ];
    }
}
```

## Working with Multiple Models

### Related Models with Casts

```php
use Cline\Intl\Data\Cast\CountryCast;
use Cline\Intl\Data\Cast\CurrencyCast;

class User extends Model
{
    protected function casts(): array
    {
        return [
            'country' => CountryCast::class,
            'currency' => CurrencyCast::class,
        ];
    }

    public function orders()
    {
        return $this->hasMany(Order::class);
    }
}

class Order extends Model
{
    protected function casts(): array
    {
        return [
            'shipping_country' => CountryCast::class,
            'billing_country' => CountryCast::class,
            'currency' => CurrencyCast::class,
        ];
    }

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function isInternationalShipping(): bool
    {
        return !$this->shipping_country->isEqualTo($this->user->country);
    }
}
```

### Validation Across Models

```php
class CreateOrderRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'currency' => ['required', 'string', new CurrencyRule()],
            'shipping_country' => ['required', 'string', new CountryRule()],
            'billing_country' => ['required', 'string', new CountryRule()],
            'shipping_phone' => [
                'required',
                'string',
                new PhoneNumberRule(regionCodeReference: 'shipping_country'),
            ],
            'billing_phone' => [
                'required',
                'string',
                new PhoneNumberRule(regionCodeReference: 'billing_country'),
            ],
        ];
    }
}
```

## API Resources with Casts

When using API resources, the value objects are automatically serialized:

```php
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'country' => [
                'code' => $this->country->alpha2,
                'name' => $this->country->localized,
            ],
            'currency' => [
                'code' => $this->currency->code,
                'symbol' => $this->currency->symbol,
                'name' => $this->currency->name,
            ],
            'language' => $this->language->value,
            'timezone' => $this->timezone->value,
        ];
    }
}
```

## Testing with Casts and Validation

### Feature Tests

```php
use Tests\TestCase;
use Cline\Intl\ValueObjects\Country;

class ProfileTest extends TestCase
{
    public function test_user_can_update_profile_with_valid_data()
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->put('/profile', [
            'country' => 'US',
            'currency' => 'USD',
            'language' => 'en',
            'timezone' => 'America/New_York',
        ]);

        $response->assertRedirect();
        $response->assertSessionHas('success');

        $user->fresh();
        $this->assertEquals('US', $user->country->alpha2);
        $this->assertEquals('USD', $user->currency->code);
    }

    public function test_validation_fails_with_invalid_country()
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->put('/profile', [
            'country' => 'INVALID',
            'currency' => 'USD',
        ]);

        $response->assertSessionHasErrors(['country']);
    }
}
```

### Model Factories

```php
use Illuminate\Database\Eloquent\Factories\Factory;

class UserFactory extends Factory
{
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'country' => fake()->randomElement(['US', 'GB', 'DE', 'FR', 'CA']),
            'currency' => fake()->randomElement(['USD', 'EUR', 'GBP', 'CAD']),
            'language' => fake()->randomElement(['en', 'fr', 'de', 'es']),
            'timezone' => fake()->randomElement([
                'America/New_York',
                'Europe/London',
                'Europe/Paris',
                'UTC',
            ]),
        ];
    }
}
```

## Best Practices

1. **Always validate before casting** - Use validation rules in form requests to prevent invalid data from reaching your models
2. **Use nullable appropriately** - Make database columns nullable only when the field is truly optional
3. **Provide defaults** - Set sensible defaults in migrations (e.g., `'UTC'` for timezone, `'en'` for language)
4. **Consistent validation** - Use the same validation rules everywhere you accept user input
5. **Leverage value object methods** - Use methods like `isEqualTo()`, `toString()`, and property access for business logic
6. **Test validation** - Write tests for both valid and invalid input scenarios
7. **Handle exceptions** - While validation should prevent most errors, handle exceptions gracefully in edge cases
8. **Use type hints** - Leverage PHP type hints with value objects in your application code

## Troubleshooting

### Cast Not Working

If casts aren't working as expected:

```php
// Check that the property is fillable or not guarded
protected $fillable = ['country', 'currency'];

// Or use guarded
protected $guarded = ['id'];

// Verify the cast is defined
protected function casts(): array
{
    return [
        'country' => CountryCast::class,
    ];
}
```

### Validation Passing Invalid Data

Ensure you're using the validation rules correctly:

```php
// Correct
new CountryRule()

// Not correct (missing parentheses)
new CountryRule

// Ensure string type is specified
'country' => ['required', 'string', new CountryRule()],
```

### Null Values Not Handled

Make sure your database column is nullable:

```php
$table->string('country', 2)->nullable();  // Correct
$table->string('country', 2);              // Will fail on null
```

## Related Resources

- [Country Value Object](/intl/country)
- [Currency Value Object](/intl/currency)
- [Language & Locale](/intl/language-locale)
- [TimeZone Value Object](/intl/timezone)
- [Laravel Validation Documentation](https://laravel.com/docs/validation)
- [Laravel Eloquent Casting Documentation](https://laravel.com/docs/eloquent-mutators#attribute-casting)
