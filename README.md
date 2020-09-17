# Nova Menu Builder 4.0

[![Latest Version on Packagist](https://img.shields.io/packagist/v/optimistdigital/nova-menu-builder.svg?style=flat-square)](https://packagist.org/packages/optimistdigital/nova-menu-builder)
[![Total Downloads](https://img.shields.io/packagist/dt/optimistdigital/nova-menu-builder.svg?style=flat-square)](https://packagist.org/packages/optimistdigital/nova-menu-builder)

This [Laravel Nova](https://nova.laravel.com/) package allows you to create and manage menus and menu items.

## 4.0 Major Release

- Reworked locale system
- Reworked menu types system (previously `MenuLinkable`)
- Custom fields support instead of JSON parameters
- `menubuilder:type` command that greatly simplifies custom menu types creation
- Greatly improved UI (the menu editor is no longer just on the "Detail" view)

## Features

- Menu management
- Menu items management
  - Simple drag-and-drop nesting and re-ordering
- Custom menu item types support
  - Ability to easily add select types
  - Ability to add custom fields

## Screenshots

![Menu Detail View](docs/menu-detail.png)

![Menu Item Edit](docs/menu-item-edit.png)

## Installation

Install the package in a Laravel Nova project via Composer, edit the configuration file and run migrations.

```bash
# Install the package
composer require optimistdigital/nova-menu-builder

# Publish the configuration file and edit it to your preference
# NB! If you want custom table names, configure them before running the migrations.
php artisan vendor:publish --tag=nova-menu-builder-config

# Run automatically loaded migrations
php artisan migrate
```

Register the tool with Nova in the `tools()` method of the `NovaServiceProvider`:

```php
// in app/Providers/NovaServiceProvider.php

public function tools()
{
    return [
        // ...
        new \OptimistDigital\MenuBuilder\MenuBuilder,
    ];
}
```

### Optionally publish migrations

This is only useful if you want to overwrite migrations and models. If you wish to use the menu builder as it comes out of the box, you don't need them.

```bash
# Publish migrations to overwrite them (optional)
php artisan vendor:publish --tag=nova-menu-builder-migrations
```

## Usage

### Locales configuration

You can define the locales for the menus in the config file, as shown below.

```php
// in config/nova-menu.php

return [
  // ...
  'locales' => [
    'en' => 'English',
    'et' => 'Estonian',
  ],

  // or using a closure (not cacheable):

  'locales' => function() {
    return nova_lang_get_locales();
  }

  // or if you want to use a function, but still be able to cache it:

  'locales' => '\App\Configuration\NovaMenuConfiguration@getLocales',

  // or

  'locales' => 'nova_lang_get_locales',
  // ...
];
```

### Custom menu item types

Menu builder allows you create custom menu item types with custom fields.

Create a class that extends the `OptimistDigital\MenuBuilder\OptimistDigital\MenuBuilder\MenuItemTypes\BaseMenuItemType` class and register it in the config file.

```php
// in config/nova-menu.php

return [
  // ...
  'menu_item_types' => [
    \App\MenuItemTypes\CustomMenuItemType::class,
  ],
  // ...
];
```

In the created class, overwrite the following methods:

```php
/**
 * Get the menu link identifier that can be used to tell different custom
 * links apart (ie 'page' or 'product').
 *
 * @return string
 **/
public static function getIdentifier(): string {
    // Example usecase
    // return 'page';
    return '';
}

/**
 * Get menu link name shown in  a dropdown in CMS when selecting link type
 * ie ('Product Link').
 *
 * @return string
 **/
public static function getName(): string {
    // Example usecase
    // return 'Page Link';
    return '';
}

/**
 * Get list of options shown in a select dropdown.
 *
 * Should be a map of [key => value, ...], where key is a unique identifier
 * and value is the displayed string.
 *
 * @return array
 **/
public static function getOptions($locale): array {
    // Example usecase
    // return Page::all()->pluck('name', 'id')->toArray();
    return [];
}

/**
 * Get the subtitle value shown in CMS menu items list.
 *
 * @param string $value
 * @param array $parameters The JSON parameters added to the item.
 * @return string
 **/
public static function getDisplayValue($value = null, array $parameters = null, array $data = null) {
    // Example usecase
    // return 'Page: ' . Page::find($value)->name;
    return $value;
}

/**
 * Get the value of the link visible to the front-end.
 *
 * Can be anything. It is up to you how you will handle parsing it.
 *
 * This will only be called when using the nova_get_menu()
 * and nova_get_menus() helpers or when you call formatForAPI()
 * on the Menu model.
 *
 * @param string $value The key from options list that was selected.
 * @param array $parameters The JSON parameters added to the item.
 * @return any
 **/
public static function getValue($value = null, array $parameters = null) {
    // Example usecase
    // return Page::find($value);
    return $value;
}

/**
 * Get the fields displayed by the resource.
 *
 * @return array An array of fields.
 */
public static function getFields(): array
{
    return [];
}

/**
 * Get the rules for the resource.
 *
 * @return array A key-value map of attributes and rules.
 */
public static function getRules(): array
{
    return [];
}

/**
 * Get data of the link visible to the front-end.
 *
 * Can be anything. It is up to you how you will handle parsing it.
 *
 * This will only be called when using the nova_get_menu()
 * and nova_get_menus() helpers or when you call formatForAPI()
 * on the Menu model.
 *
 * @param null $data Field values
 * @param array|null $parameters The JSON parameters added to the item.
 * @return any
 */
public static function getData($data = null, array $parameters = null)
{
    return $data;
}
```

### Returning the menus in a JSON API

#### nova_get_menus()

A helper function `nova_get_menus` is globally registered in this package which returns all the menus including their menu items in an API friendly format.

```php
public function getMenus(Request $request) {
    $menusResponse = nova_get_menus();
    return response()->json($menusResponse);
}
```

#### nova_get_menu_by_slug(\$menuSlug, \$locale = null)

To get a single menu, you can use the helper function `nova_get_menu('slug', 'en')`. Returns null if no menu with the slug is found or returns the menu if it is found. If no locale is passed, the helper will automatically choose the first configured locale.

## Credits

- [Tarvo Reinpalu](https://github.com/Tarpsvo)
- [Eric Lagarda (original nova-menu-builder)](https://github.com/Krato)
- [Ralph Huwiler (vue-nestable)](https://github.com/rhwilr/vue-nestable)

## License

Nova Menu Builder is open-sourced software licensed under the [MIT license](LICENSE.md).
