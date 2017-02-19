# Authentication

 - [Custom Middleware](#middleware)

SleepingOwl Admin doesn't use authentication by default, that means access to the Admin Panel is not restricted. You can set up restricted access using middleware.

The easiest way is to use [Laravel Authentication](https://laravel.com/docs/5.4/authentication#authentication-quickstart) provided by Laravel itself. You can get this done by running Artisan command:

```bash
$ php artisan make:auth
```

And then you should add `auth` to the middleware settings in the `config/sleeping_owl.php` configuration file.

**Example**
```php
/*
 | ...
 | see https://laravel.com/docs/5.2/authentication#authentication-quickstart
 |
 */
'middleware' => ['web', 'auth'],
...
```

Thereby you have an ability to configure access rules to the Admin Panel by creating your own middlewares.

---

<a name="middleware"></a>
## Custom Middleware

You can create custom middleware class, for example `App\Http\Middleware\AdminAuthenticate`

```php
<?php

namespace App\Http\Middleware;

use App\User;
use Closure;
use Illuminate\Support\Facades\Auth;
class AdminAuthenticate
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request $request
     * @param  \Closure $next
     * @param  string|null $guard
     *
     * @return mixed
     */
    public function handle($request, Closure $next, $guard = null)
    {
        $auth = Auth::guard($guard);

        if (Auth::guard($guard)->guest()) {
            if ($request->ajax() || $request->wantsJson()) {
                return response('Unauthorized.', 401);
            } else {
                return redirect()->guest('login');
            }
        }

        if (! $auth->user()->isAdmin()) {
            return response('Access denied.', 401);
        }
        
        return $next($request);
    }
}
```

And register this middleware in `App\Http\Kernel`

```php
 ...
 protected $routeMiddleware = [
     ...
     'admin' => \App\Http\Middleware\AdminAuthenticate::class,
     ...
 ];
 ...
```

And add this middleware `admin` to `config\sleeoping_owl.php`

```php
...
'middleware' => ['web', 'admin'],
...
```
