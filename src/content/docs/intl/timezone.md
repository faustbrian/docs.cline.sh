---
title: TimeZone Value Object
description: Working with timezones using the TimeZone value object in Cline Intl
---

The `TimeZone` value object provides type-safe handling of timezone identifiers with localized names powered by Symfony Intl. It simplifies working with timezones in your Laravel application, especially for user preferences and scheduled tasks.

## Creating TimeZone Instances

### From String

Create a timezone instance from a timezone identifier:

```php
use Cline\Intl\ValueObjects\TimeZone;

$timezone = TimeZone::createFromString('America/New_York');
```

The factory method accepts standard timezone identifiers from the [IANA Time Zone Database](https://www.iana.org/time-zones):
- `America/New_York`
- `Europe/London`
- `Asia/Tokyo`
- `Australia/Sydney`
- `UTC`

## Properties

The `TimeZone` value object provides two read-only properties:

```php
$timezone = TimeZone::createFromString('America/New_York');

// Timezone identifier
echo $timezone->value;  // "America/New_York"

// Localized timezone name
echo $timezone->localized;  // "Eastern Time (New York)"
```

### Property Details

- **`value`** (string) - The IANA timezone identifier
- **`localized`** (string) - The localized timezone name based on your application's current locale

The `localized` property respects your application's locale:

```php
app()->setLocale('en');
$timezone = TimeZone::createFromString('America/New_York');
echo $timezone->localized;  // "Eastern Time (New York)"

app()->setLocale('fr');
$timezone = TimeZone::createFromString('America/New_York');
echo $timezone->localized;  // "heure de l'Est (New York)"
```

## String Representation

The `TimeZone` object implements `Stringable` and returns the timezone identifier:

```php
$timezone = TimeZone::createFromString('Europe/Paris');

echo (string) $timezone;  // "Europe/Paris"
echo $timezone->toString();  // "Europe/Paris"
echo $timezone;  // "Europe/Paris" (implicit string cast)
```

## Equality Comparison

Compare two timezone instances for equality:

```php
$timezone1 = TimeZone::createFromString('America/New_York');
$timezone2 = TimeZone::createFromString('America/New_York');
$timezone3 = TimeZone::createFromString('Europe/London');

$timezone1->isEqualTo($timezone2);  // true
$timezone1->isEqualTo($timezone3);  // false
```

## Eloquent Model Integration

### Using TimeZoneCast

Add timezone support to your Eloquent models using the `TimeZoneCast`:

```php
use Illuminate\Database\Eloquent\Model;
use Cline\Intl\Data\Cast\TimeZoneCast;

class User extends Model
{
    protected function casts(): array
    {
        return [
            'timezone' => TimeZoneCast::class,
        ];
    }
}
```

### Working with the Cast

```php
// Store as string, retrieve as TimeZone object
$user = new User();
$user->timezone = 'America/New_York';
$user->save();

// Automatically cast to TimeZone object
echo $user->timezone->value;       // "America/New_York"
echo $user->timezone->localized;   // "Eastern Time (New York)"

// Update with string
$user->timezone = 'Europe/London';
$user->save();
```

### Database Schema

Store the timezone as a string column:

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('timezone', 50)->default('UTC');
    $table->timestamps();
});
```

## Validation

Validate timezone input using the `TimeZoneRule`:

```php
use Cline\Intl\Rules\TimeZoneRule;

class UpdateProfileRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'timezone' => ['required', 'string', new TimeZoneRule()],
        ];
    }
}
```

The rule validates that the input is a valid IANA timezone identifier.

## Common Use Cases

### User Timezone Preferences

```php
use Cline\Intl\ValueObjects\TimeZone;
use Cline\Intl\Data\Cast\TimeZoneCast;
use Carbon\Carbon;

class User extends Model
{
    protected function casts(): array
    {
        return [
            'timezone' => TimeZoneCast::class,
        ];
    }

    public function localTime(?Carbon $time = null): Carbon
    {
        $time = $time ?? now();

        return $time->setTimezone($this->timezone->value);
    }

    public function formatTimeInUserTimezone(Carbon $time): string
    {
        return $this->localTime($time)->format('Y-m-d H:i:s T');
    }
}

// Usage
$user = auth()->user();
$event = Event::find(1);

// Display event time in user's timezone
echo $user->formatTimeInUserTimezone($event->starts_at);
// "2025-12-15 14:30:00 EST"
```

### Scheduled Tasks with User Timezones

```php
use Cline\Intl\ValueObjects\TimeZone;
use Cline\Intl\Data\Cast\TimeZoneCast;
use Carbon\Carbon;

class ScheduledReport extends Model
{
    protected function casts(): array
    {
        return [
            'timezone' => TimeZoneCast::class,
            'scheduled_time' => 'datetime',
        ];
    }

    public function shouldRunNow(): bool
    {
        $now = now()->setTimezone($this->timezone->value);
        $scheduledTime = $this->scheduled_time->setTimezone($this->timezone->value);

        return $now->hour === $scheduledTime->hour
            && $now->minute === $scheduledTime->minute;
    }

    public function getNextRunTime(): Carbon
    {
        return $this->scheduled_time
            ->setTimezone($this->timezone->value)
            ->addDay();
    }
}
```

### Meeting Scheduler

```php
use Cline\Intl\ValueObjects\TimeZone;
use Cline\Intl\Data\Cast\TimeZoneCast;

class Meeting extends Model
{
    protected function casts(): array
    {
        return [
            'timezone' => TimeZoneCast::class,
            'starts_at' => 'datetime',
            'ends_at' => 'datetime',
        ];
    }

    public function getStartTimeInTimezone(TimeZone $timezone): string
    {
        return $this->starts_at
            ->setTimezone($timezone->value)
            ->format('Y-m-d H:i:s T');
    }

    public function getTimezoneOffset(TimeZone $comparedTo): string
    {
        $meetingTz = new \DateTimeZone($this->timezone->value);
        $compareTz = new \DateTimeZone($comparedTo->value);

        $meetingOffset = $meetingTz->getOffset(new \DateTime());
        $compareOffset = $compareTz->getOffset(new \DateTime());

        $diffSeconds = $meetingOffset - $compareOffset;
        $diffHours = abs($diffSeconds / 3600);

        if ($diffSeconds > 0) {
            return "+{$diffHours} hours";
        } elseif ($diffSeconds < 0) {
            return "-{$diffHours} hours";
        }

        return "same timezone";
    }
}

// Usage
$meeting = Meeting::find(1);
$userTimezone = TimeZone::createFromString('America/Los_Angeles');

echo $meeting->getStartTimeInTimezone($userTimezone);
// "2025-12-15 11:00:00 PST"

echo $meeting->getTimezoneOffset($userTimezone);
// "+3 hours"
```

### Timezone Selector

```php
use Symfony\Component\Intl\Timezones;

class TimezoneController extends Controller
{
    public function index()
    {
        // Get all timezones grouped by region
        $timezones = collect(Timezones::getNames())
            ->map(fn($name, $id) => [
                'id' => $id,
                'name' => $name,
                'region' => explode('/', $id)[0] ?? 'Other',
            ])
            ->groupBy('region')
            ->map(fn($group) => $group->sortBy('name')->values());

        return view('timezones.index', compact('timezones'));
    }

    public function common()
    {
        // Common timezones for quick selection
        $common = [
            'UTC' => 'UTC',
            'America/New_York' => 'Eastern Time (US)',
            'America/Chicago' => 'Central Time (US)',
            'America/Denver' => 'Mountain Time (US)',
            'America/Los_Angeles' => 'Pacific Time (US)',
            'Europe/London' => 'London',
            'Europe/Paris' => 'Paris',
            'Asia/Tokyo' => 'Tokyo',
            'Australia/Sydney' => 'Sydney',
        ];

        return response()->json($common);
    }
}
```

### Displaying Times in Multiple Timezones

```php
use Cline\Intl\ValueObjects\TimeZone;
use Carbon\Carbon;

class TimeZoneConverter
{
    public function convertToMultipleTimezones(Carbon $time, array $timezoneIds): array
    {
        $results = [];

        foreach ($timezoneIds as $timezoneId) {
            $timezone = TimeZone::createFromString($timezoneId);
            $converted = $time->copy()->setTimezone($timezone->value);

            $results[] = [
                'timezone' => $timezone->localized,
                'time' => $converted->format('Y-m-d H:i:s'),
                'offset' => $converted->format('P'),
            ];
        }

        return $results;
    }
}

// Usage
$converter = new TimeZoneConverter();
$eventTime = Carbon::parse('2025-12-15 14:00:00', 'UTC');

$timezones = [
    'America/New_York',
    'Europe/London',
    'Asia/Tokyo',
    'Australia/Sydney',
];

$times = $converter->convertToMultipleTimezones($eventTime, $timezones);

/*
[
    ['timezone' => 'Eastern Time (New York)', 'time' => '2025-12-15 09:00:00', 'offset' => '-05:00'],
    ['timezone' => 'Greenwich Mean Time (London)', 'time' => '2025-12-15 14:00:00', 'offset' => '+00:00'],
    ['timezone' => 'Japan Time (Tokyo)', 'time' => '2025-12-15 23:00:00', 'offset' => '+09:00'],
    ['timezone' => 'Australian Eastern Time (Sydney)', 'time' => '2025-12-16 01:00:00', 'offset' => '+11:00'],
]
*/
```

### Notification Scheduling

```php
use Cline\Intl\ValueObjects\TimeZone;
use Cline\Intl\Data\Cast\TimeZoneCast;

class NotificationPreference extends Model
{
    protected function casts(): array
    {
        return [
            'timezone' => TimeZoneCast::class,
            'preferred_time' => 'datetime',
        ];
    }

    public function getNextNotificationTime(): Carbon
    {
        $now = now()->setTimezone($this->timezone->value);
        $preferred = $this->preferred_time->setTimezone($this->timezone->value);

        $next = $now->copy()
            ->setTime($preferred->hour, $preferred->minute);

        // If the preferred time has already passed today, schedule for tomorrow
        if ($next->isPast()) {
            $next->addDay();
        }

        return $next->setTimezone('UTC');
    }

    public function isNotificationTime(): bool
    {
        $now = now()->setTimezone($this->timezone->value);
        $preferred = $this->preferred_time->setTimezone($this->timezone->value);

        return $now->hour === $preferred->hour
            && $now->minute === $preferred->minute;
    }
}
```

## Integration with Carbon

The `TimeZone` value object works seamlessly with Laravel's Carbon:

```php
use Cline\Intl\ValueObjects\TimeZone;
use Carbon\Carbon;

$timezone = TimeZone::createFromString('America/New_York');

// Create Carbon instance in specific timezone
$time = Carbon::now($timezone->value);

// Convert existing time to timezone
$utcTime = Carbon::now('UTC');
$localTime = $utcTime->setTimezone($timezone->value);

// Format with timezone abbreviation
echo $localTime->format('Y-m-d H:i:s T');
// "2025-12-15 09:00:00 EST"
```

## Exception Handling

The `createFromString()` method throws a `MissingResourceException` if the timezone identifier is invalid:

```php
use Symfony\Component\Intl\Exception\MissingResourceException;
use Cline\Intl\ValueObjects\TimeZone;

try {
    $timezone = TimeZone::createFromString('Invalid/Timezone');
} catch (MissingResourceException $e) {
    // Handle invalid timezone identifier
    Log::error('Invalid timezone identifier', [
        'timezone' => 'Invalid/Timezone',
        'message' => $e->getMessage(),
    ]);
}
```

Always validate user input using the `TimeZoneRule` to prevent exceptions.

## Best Practices

1. **Always validate input** - Use `TimeZoneRule` in form requests before creating TimeZone objects
2. **Store UTC in database** - Store all timestamps in UTC and convert to user timezone for display
3. **Use IANA identifiers** - Avoid abbreviations like "EST" or "PST" - use full identifiers
4. **Consider DST** - IANA timezones automatically handle daylight saving time changes
5. **Default to UTC** - Provide UTC as a sensible default for users who don't set a timezone
6. **Test across timezones** - Test your application with users in different timezones
7. **Display timezone in UI** - Always show the timezone when displaying times to avoid confusion

## Common Timezone Identifiers

Here are some commonly used timezone identifiers:

### United States
- `America/New_York` - Eastern Time
- `America/Chicago` - Central Time
- `America/Denver` - Mountain Time
- `America/Los_Angeles` - Pacific Time
- `America/Anchorage` - Alaska Time
- `Pacific/Honolulu` - Hawaii Time

### Europe
- `Europe/London` - GMT/BST
- `Europe/Paris` - Central European Time
- `Europe/Berlin` - Central European Time
- `Europe/Rome` - Central European Time
- `Europe/Madrid` - Central European Time

### Asia
- `Asia/Tokyo` - Japan Standard Time
- `Asia/Shanghai` - China Standard Time
- `Asia/Hong_Kong` - Hong Kong Time
- `Asia/Singapore` - Singapore Time
- `Asia/Dubai` - Gulf Standard Time
- `Asia/Kolkata` - India Standard Time

### Australia
- `Australia/Sydney` - Australian Eastern Time
- `Australia/Melbourne` - Australian Eastern Time
- `Australia/Perth` - Australian Western Time

### Other
- `UTC` - Coordinated Universal Time
- `Pacific/Auckland` - New Zealand Time

## Related Resources

- [Symfony Intl Timezones Component](https://symfony.com/doc/current/components/intl.html)
- [IANA Time Zone Database](https://www.iana.org/time-zones)
- [PHP DateTimeZone](https://www.php.net/manual/en/class.datetimezone.php)
- [Laravel Carbon Documentation](https://carbon.nesbot.com/docs/)
- [Validation Rules](/intl/casts-and-rules)
