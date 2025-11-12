# Searchable Multiselect With List

A searchable multiselect form component with a list view for Filament.

This component extends Filament's Select field to provide a better user experience for multiple selections, showing selected items in a clear list format below the select field.

## Version Compatibility

- **Filament 4**: Use `v4.0.0` branch or `^4.0` version
- **Filament 3**: Use `v3.0.0` branch or `^3.0` version

## Features

- **Searchable multiselect dropdown** - Built on Filament's Select component
- **Selected items list view** - Clear display of selected items with remove buttons
- **Async label loading** - Efficiently loads labels for selected items
- **Skeleton loading states** - Smooth loading experience
- **HTML content support** - Display rich HTML in option labels
- **Show search icon** - Optional search icon in the input field
- **Selection counter** - Shows number of selected items in list
- **Bulk deselect** - Remove all selections at once
- **Dark mode support** - Fully compatible with Filament's dark mode
- **Responsive design** - Works on all screen sizes

> **Note for Filament 3**: Due to Choices.js limitations in Filament 3, the placeholder cannot be dynamically updated to show selection count. The placeholder will always show the value set with `->placeholder()`. This feature is only available in Filament 4.

## Installation

Install the package via composer based on your Filament version:

### For Filament 4.x

```bash
composer require joost-meijerink/searchable-multiselect-with-list:^4.0
```

### For Filament 3.x

```bash
composer require joost-meijerink/searchable-multiselect-with-list:^3.0
```

> **Note:** The package will automatically install the correct version based on your Filament version constraints if you don't specify a version, but it's recommended to explicitly specify the version for clarity.

## Usage

The `SearchableMultiSelectWithList` component can be used just like any other Filament form field. It extends the standard Select field, so all Select methods are available.

### Basic Usage

```php
use JoostMeijerink\SearchableMultiselectWithList\Forms\Components\SearchableMultiSelectWithList;

SearchableMultiSelectWithList::make('users')
    ->label('Select Users')
    ->options(User::all()->pluck('name', 'id'))
    ->searchable()
```

### Advanced Usage with Dynamic Search

```php
SearchableMultiSelectWithList::make('entities')
    ->label('KvK Number')
    ->placeholder('Search entities')
    ->searchable()
    ->showSearchIcon() // Shows search icon in the field
    ->listLabel('Selected Entities')
    ->getSearchResultsUsing(function ($search) {
        return Entity::query()
            ->where('name', 'like', "%{$search}%")
            ->orWhere('kvk_number', 'like', "%{$search}%")
            ->limit(20)
            ->get()
            ->mapWithKeys(function ($entity) {
                return [
                    $entity->id => view('components.entity-option', [
                        'entity' => $entity,
                    ])->render(),
                ];
            })
            ->toArray();
    })
    ->getOptionLabelsUsing(function ($values) {
        return Entity::whereIn('id', $values)
            ->get()
            ->mapWithKeys(function ($entity) {
                return [
                    $entity->id => view('components.entity-label', [
                        'entity' => $entity,
                    ])->render(),
                ];
            })
            ->toArray();
    })
    ->allowHtml() // Enable HTML content in labels
    ->reactive()
```

### With Relationships

```php
SearchableMultiSelectWithList::make('categories')
    ->relationship('categories', 'name')
    ->listLabel('Selected Categories')
    ->showCountSelected(true)
    ->searchable()
    ->preload()
```

## Available Methods

All methods from Filament's Select component are available, plus:

### `listLabel(?string $label)`
Set a label for the selected items list section.

```php
->listLabel('Selected Items')
```

### `showCountSelected(bool $condition = true)`
Toggle the display of the selection count and "Deselect all" button.

```php
->showCountSelected(false) // Hide the count
```

### `showSearchIcon(bool $condition = true)`
Show a search icon in the input field to indicate search functionality.

```php
->showSearchIcon()
```

### `allowHtml()`
Allow HTML content in option labels (inherited from Select).

```php
->allowHtml()
```

## Tailwind CSS Configuration

**Important:** This plugin uses Tailwind CSS classes including theme-specific classes like `bg-primary-*`. Your main project needs to scan the plugin's views to compile these classes.

### Add Plugin Views to Tailwind Config

**For Tailwind v3** (tailwind.config.js):
```js
module.exports = {
    content: [
        // ... your existing paths
        './vendor/joost-meijerink/searchable-multiselect-with-list/resources/views/**/*.blade.php',
    ],
    // ... rest of your config
}
```

**For Tailwind v4** (resources/css/app.css):
```css
@import '../../vendor/filament/filament/resources/css/theme.css';

/* Content sources for Tailwind v4 */
@source '../views/**/*.blade.php';
@source '../../vendor/joost-meijerink/searchable-multiselect-with-list/resources/views/**/*.blade.php';
/* ... other sources */
```

After adding this configuration, rebuild your CSS:
```bash
npm run build
```

### Theme Classes Used

The component uses the following theme-dependent classes:
- `bg-primary-100/50`, `ring-primary-600/20` - Selected item styling
- `dark:bg-primary-400/10`, `dark:ring-primary-400/30` - Dark mode variants

Make sure your Filament panel has a primary color configured, or these will use Filament's default primary color.

## Customization

### Translations

The component comes with English translations. You can publish and customize them:

```bash
php artisan vendor:publish --tag="searchable-multiselect-with-list-translations"
```

This will publish the translation files to `resources/lang/vendor/searchable-multiselect-with-list/`.

### Styling

The component uses Filament's design system and is fully compatible with dark mode. The selected items are displayed with:
- Primary color background (light blue by default)
- Rounded corners
- Remove buttons using Filament's icon button component
- Smooth skeleton loading animations

## Examples

### Simple Options

```php
SearchableMultiSelectWithList::make('tags')
    ->options([
        'php' => 'PHP',
        'laravel' => 'Laravel',
        'filament' => 'Filament',
        'livewire' => 'Livewire',
        'alpine' => 'Alpine.js',
    ])
    ->listLabel('Selected Tags')
```

### With Custom Validation

```php
SearchableMultiSelectWithList::make('members')
    ->options(User::all()->pluck('name', 'id'))
    ->listLabel('Team Members')
    ->required()
    ->minItems(2)
    ->maxItems(10)
    ->searchable()
```

### Reactive with Dependencies

```php
SearchableMultiSelectWithList::make('cities')
    ->options(function (Get $get) {
        $countryId = $get('country_id');

        if (!$countryId) {
            return [];
        }

        return City::where('country_id', $countryId)
            ->pluck('name', 'id');
    })
    ->listLabel('Selected Cities')
    ->reactive()
    ->searchable()
    ->disabled(fn (Get $get) => !$get('country_id'))
```

## Requirements

### For v4.0.0 (Filament 4)
- PHP 8.1 or higher
- Filament 4.0 or higher
- Laravel 11.0 or higher

### For v3.0.0 (Filament 3)
- PHP 8.1 or higher
- Filament 3.0 or higher
- Laravel 10.0 or higher

## Testing

This package doesn't include tests yet. Contributions are welcome!

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

Please feel free to submit issues and pull requests.

## Credits

- [Joost Meijerink](https://github.com/joost-meijerink)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.