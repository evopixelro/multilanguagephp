# Multi Language PHP

A small PHP helper that selects a language from the visitor's IP-derived country. It requests a country code from FreeIPAPI, looks it up in a local mapping and exposes the selected language as `$lang`.

The helper selects translation files; it does not translate text, read the browser's preferred language or store a language choice between requests.

## Development

Use PHP with cURL and JSON support, plus outbound HTTPS access. There are no Composer dependencies or build commands.

`example/` contains a working layout with English and Romanian translations. Serve that directory through PHP to check the integration. `lib/multilanguage.php` is the current implementation; `old/` is retained for reference and is not used by the current example.

## Integration

Place `lib/multilanguage.php` in your application's `libs/multilanguagephp/` directory. Copy `translations_list.php.example` to `lang/translations_list.php` and create a translation file for each language under `lang/translations/`.

Load the country mapping, language selector and selected translation in that order:

```php
<?php
require __DIR__.'/lang/translations_list.php';
require __DIR__.'/libs/multilanguagephp/multilanguage.php';
require __DIR__.'/lang/translations/'.$lang.'.php';

echo htmlspecialchars($text, ENT_QUOTES, 'UTF-8');
```

The translation files define the variables used by your templates. The included example uses `$lang_code` and `$text`; applications can define their own keys as long as they are consistent across languages.

## Configuration

The mapping associates country codes with local language filenames:

```php
<?php
$translations = [
	'RO' => 'ro',
	'MD' => 'ro'
];
```

Both countries use `lang/translations/ro.php`. Other countries use English. To add a language, add the appropriate country mappings and a matching PHP translation file, including every variable your templates need.

The default language is set to `en` inside `lib/multilanguage.php`. There is no separate default-language option; setting `$lang` before including the helper will not override it. Keep `en.php` available unless you deliberately change the helper's default.

Language codes come from the trusted local mapping. Do not replace them with unchecked query parameters when constructing a translation file path.

## Request behavior

The helper checks `HTTP_CLIENT_IP`, then `HTTP_X_FORWARDED_FOR`, then `REMOTE_ADDR`. It requests `https://freeipapi.com/api/json/<address>` and reads `country` from the JSON response.

The connection timeout is two seconds and the total request timeout is three seconds. Missing country data, unmapped countries and failed requests leave the language set to English. Each request that includes the helper performs a lookup; no result cache is implemented.

Country is only a language hint. VPNs, shared networks and travel can select a language different from the visitor's preference.

## Deployment

Keep the country mapping and translation files together, and check that every mapped language file exists. When updating the helper, synchronize the copy under `example/libs/multilanguagephp/` if the example is distributed too.

Configure the reverse proxy to strip or replace untrusted client-IP headers. The helper itself does not validate trusted proxies or parse a forwarded-address chain, and it must not be used to make access-control decisions.

The lookup sends the selected visitor IP address to an external service. Account for that request in the site's privacy configuration and avoid including the helper repeatedly in one page render.

## Diagnostics

If the page remains in English, check the country mapping, outbound HTTPS connectivity and PHP error log. A local or private address may not produce a usable country result. Missing translation variables usually indicate that the language files no longer have matching keys.

Verify mapped and unmapped countries, API failure behavior and all translated templates before deployment. There is no automated test suite configured in this repository.
