# Laravel-Vercel-Deploy-Guide
1. create api/lambda.php
   
```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Http\Request;

define('LARAVEL_START', microtime(true));

// Determine if the application is in maintenance mode...
if (file_exists($maintenance = __DIR__.'/../storage/framework/maintenance.php')) {
    require $maintenance;
}

// Register the Composer autoloader...
require __DIR__.'/../vendor/autoload.php';

// Bootstrap Laravel and handle the request...
/** @var Application $app */
$app = require_once __DIR__.'/../bootstrap/app.php';

$app->handleRequest(Request::capture());
```

2. bootstrap/app.php

```php
<?php

use App\Http\Middleware\HandleAppearance;
use App\Http\Middleware\HandleInertiaRequests;
use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;
use Illuminate\Http\Middleware\AddLinkHeadersForPreloadedAssets;
use Illuminate\Http\Request;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware): void {
        $middleware->encryptCookies(except: ['appearance', 'sidebar_state']);

        $middleware->web(append: [
            HandleAppearance::class,
            HandleInertiaRequests::class,
            AddLinkHeadersForPreloadedAssets::class,
        ]);

        $middleware->trustProxies(
            '*',
            Request::HEADER_X_FORWARDED_FOR |
            Request::HEADER_X_FORWARDED_HOST |
            Request::HEADER_X_FORWARDED_PROTO |
            Request::HEADER_X_FORWARDED_PORT
        );
    })
    ->withExceptions(function (Exceptions $exceptions): void {
        //
    })->create();
```

3. .vercelignore
```php
public/index.php
```

4. composer.json
```php
{
    "$schema": "https://getcomposer.org/schema.json",
    "name": "laravel/react-starter-kit",
    "type": "project",
    "description": "The skeleton application for the Laravel framework.",
    "keywords": [
        "laravel",
        "framework"
    ],
    "license": "MIT",
    "require": {
        "php": "^8.3",
        "inertiajs/inertia-laravel": "^3.0",
        "laravel/fortify": "^1.34",
        "laravel/framework": "^13.0",
        "laravel/tinker": "^3.0",
        "laravel/wayfinder": "^0.1.14"
    },
    "require-dev": {
        "fakerphp/faker": "^1.24",
        "laravel/boost": "^2.2",
        "laravel/pail": "^1.2.5",
        "laravel/pint": "^1.27",
        "laravel/sail": "^1.53",
        "mockery/mockery": "^1.6",
        "nunomaduro/collision": "^8.9.3",
        "nunomaduro/pao": "^1.0.3",
        "pestphp/pest": "^4.6",
        "pestphp/pest-plugin-laravel": "^4.1"
    },
    "autoload": {
        "psr-4": {
            "App\\": "app/",
            "Database\\Factories\\": "database/factories/",
            "Database\\Seeders\\": "database/seeders/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\": "tests/"
        }
    },
    "scripts": {
        "setup": [
            "composer install",
            "@php -r \"file_exists('.env') || copy('.env.example', '.env');\"",
            "@php artisan key:generate",
            "@php artisan migrate --force",
            "npm install",
            "npm run build"
        ],
        "dev": [
            "Composer\\Config::disableProcessTimeout",
            "npx concurrently -c \"#93c5fd,#c4b5fd,#fdba74\" \"php artisan serve\" \"php artisan queue:listen --tries=1\" \"npm run dev\" --names='server,queue,vite'"
        ],
        "lint": [
            "pint --parallel"
        ],
        "lint:check": [
            "pint --parallel --test"
        ],
        "ci:check": [
            "Composer\\Config::disableProcessTimeout",
            "npm run lint:check",
            "npm run format:check",
            "npm run types:check",
            "@test"
        ],
        "test": [
            "@php artisan config:clear --ansi",
            "@lint:check",
            "@php artisan test"
        ],
        "post-autoload-dump": [
            "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
            "@php artisan package:discover --ansi"
        ],
        "post-update-cmd": [
            "@php artisan vendor:publish --tag=laravel-assets --ansi --force",
            "@php artisan boost:update --ansi"
        ],
        "post-root-package-install": [
            "@php -r \"file_exists('.env') || copy('.env.example', '.env');\""
        ],
        "post-create-project-cmd": [
            "@php artisan key:generate --ansi",
            "@php -r \"file_exists('database/database.sqlite') || touch('database/database.sqlite');\"",
            "@php artisan migrate --graceful --ansi"
        ],
        "pre-package-uninstall": [
            "Illuminate\\Foundation\\ComposerScripts::prePackageUninstall"
        ],
        "vercel": [
            "npm run build",
            "mkdir -p vercel/output/static",
            "cp -r public/build /vercel/output/static/",
            "php artisan migrate --force"
        ]
    },
    "extra": {
        "laravel": {
            "dont-discover": []
        }
    },
    "config": {
        "optimize-autoloader": true,
        "preferred-install": "dist",
        "sort-packages": true,
        "allow-plugins": {
            "pestphp/pest-plugin": true,
            "php-http/discovery": true
        }
    },
    "minimum-stability": "stable"
}

```
5. create vercel.json
```php
{
    "$schema": "https://openapi.vercel.sh/vercel.json",
    "outputDirectory": "public",
    "functions": {
        "api/lambda.php": {
            "runtime": "vercel-php@0.7.4"
        }
    },
    "rewrites": [{"source": "/(.*)", "destination": "/api/lambda.php"}],
    "buildCommand": "echo skip..."
}
```


6. add this in the production .env
```php
APP_CONFIG_CACHE=/tmp/config.php
APP_ROUTES_CACHE=/tmp/routes.php
APP_EVENTS_CACHE=/tmp/events.php
APP_PACKAGES_CACHE=/tmp/packages.php
APP_SERVICES_CACHE=/tmp/services.php
VIEW_COMPILED_PATH=/tmp/views
ASSET_URL=/
CACHE_DRIVER=array




change
SESSION_DRIVER=cookie
```
