# Laravel 11.x 中间件


----

[原文](https://laravel.com/docs/11.x/middleware)

----

## 介绍

中间件为检查和过滤进入应用程序的HTTP请求提供了一种便捷的机制。

## 定义中间件

> `make:middleware`命令用于创建新的中间件。

```bash
php artisan make:middleware EnsureTokenIsValid
```

此命令将在`app/Http/Middleware`目录中放置一个新`EnsureTokenIsValid`类。

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureTokenIsValid
{
    /**
     * Handle an incoming request.
     *
     * @param \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response) $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->input('token') !== 'my-secret-token') {
            return response('This action is unauthorized.', 401);
        }
        return $next($request);
    }
}

```

### 中间件和响应

中间件可以在将请求传递到应用程序更深处之前或之后执行任务。

例如，以下中间件会在请求被应用程序处理之前执行某些任务

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class BeforeMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        // Perform action
        
        return $next($request);
    }
}

```

以下中间件将在应用程序处理请求后执行其任务：

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class AfterMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response) $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        $response = $next($request);

        // Perform action

        return $response;
    }
}

```



## 注册中间件

### 全局中间件

如果需要中间件在每个HTTP请求到达应用程序时都运行，那么可以将它附加到应用程序的全局中间件中，在应用程序的`bootstrap/app.php`文件中进行配置

```php
use App\Http\Middleware\EnsureTokenIsValid;
 
->withMiddleware(function (Middleware $middleware) {
     $middleware->append(EnsureTokenIsValid::class);
})
```

在 `withMiddleware` 闭包中提供的 `$middleware` 对象是 `Illuminate\Foundation\Configuration\Middleware` 的实例，负责管理分配给应用程序路由的中间件。`append` 方法将中间件添加到全局中间件列表的末尾。

如果需要将中间件添加到列表的开头，应该使用 `prepend` 方法。

#### 手动管理Laravel默认全局中间件

可以使用`use`方法对Laravel提供的默认全局中间件进行手动管理，根据所需调整中间件使用：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->use([
        // \Illuminate\Http\Middleware\TrustHosts::class,
        \Illuminate\Http\Middleware\TrustProxies::class,
        \Illuminate\Http\Middleware\HandleCors::class,
        \Illuminate\Foundation\Http\Middleware\PreventRequestsDuringMaintenance::class,
        \Illuminate\Http\Middleware\ValidatePostSize::class,
        \Illuminate\Foundation\Http\Middleware\TrimStrings::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
    ]);
})
```

### 将中间件分配给路由

在定义路由时调用`middleware`方法将中间件分配给路由

```php
use App\Http\Middleware\EnsureTokenIsValid;
 
Route::get('/profile', function () {
    // ...
})->middleware(EnsureTokenIsValid::class);
```

也可以将中间件名称数组传递给`middleware`方法，将多个中间件分配给路由：

```php
Route::get('/', function () {
    // ...
})->middleware([First::class, Second::class]);
```

#### 排除中间件

> 当将中间件分配给一组路由时，需要防止某个中间件应用于该组中的某个单独路由。
>
> 可以使用 `withoutMiddleware` 方法来实现。

```php
use App\Http\Middleware\EnsureTokenIsValid;
 
Route::middleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/', function () {
        // ...
    });
 
    Route::get('/profile', function () {
        // ...
    })->withoutMiddleware([EnsureTokenIsValid::class]);
});
```

还可以从整个路由组定义中排除给定的一组中间件

```php
use App\Http\Middleware\EnsureTokenIsValid;
 
Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/profile', function () {
        // ...
    });
});
```

**注意：`withoutMiddleware` 方法只能移除路由中的中间件，不能用于全局中间件。**

### 中间件组

将多个中间件分组到一个键下，以便更轻松分配给路由，可以使用应用程序`bootstrap/app.php` 文件中的方法 `appendToGroup` 完成此操作

```php
use App\Http\Middleware\First;
use App\Http\Middleware\Second;
 
->withMiddleware(function (Middleware $middleware) {
    $middleware->appendToGroup('group-name', [
        First::class,
        Second::class,
    ]);
 
    $middleware->prependToGroup('group-name', [
        First::class,
        Second::class,
    ]);
})
```

中间件组可以使用与单个中间件相同的语法分配给路由和控制器动作

```php
Route::get('/', function () {
    // ...
})->middleware('group-name');
 
Route::middleware(['group-name'])->group(function () {
    // ...
});
```

#### Laravel默认的中间件组

| `web` 中间件组                                            |
| :-------------------------------------------------------- |
| `Illuminate\Cookie\Middleware\EncryptCookies`             |
| `Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse` |
| `Illuminate\Session\Middleware\StartSession`              |
| `Illuminate\View\Middleware\ShareErrorsFromSession`       |
| `Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` |
| `Illuminate\Routing\Middleware\SubstituteBindings`        |

| `api` 中间件组                                     |
| :------------------------------------------------- |
| `Illuminate\Routing\Middleware\SubstituteBindings` |

如果向这些组附加或前置中间件，可以在应用程序的 `bootstrap/app.php` 文件中使用 `web` 和 `api` 方法。`web` 和 `api` 方法是向组添加中间件的便捷替代方法，相当于 `appendToGroup` 方法。

```php
use App\Http\Middleware\EnsureTokenIsValid;
use App\Http\Middleware\EnsureUserIsSubscribed;
 
->withMiddleware(function (Middleware $middleware) {
    $middleware->web(append: [
        EnsureUserIsSubscribed::class,
    ]);
 
    $middleware->api(prepend: [
        EnsureTokenIsValid::class,
    ]);
})
```

也可以将Laravel的默认中间件条目之一替换为自定义中间件：

```php
use App\Http\Middleware\StartCustomSession;
use Illuminate\Session\Middleware\StartSession;
 
$middleware->web(replace: [
    StartSession::class => StartCustomSession::class,
]);
```

也可以完全删除中间件：

```php
$middleware->web(remove: [
    StartSession::class,
]);
```

##### 手动管理Laravel的默认中间件组

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->group('web', [
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        // \Illuminate\Session\Middleware\AuthenticateSession::class,
    ]);
 
    $middleware->group('api', [
        // \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        // 'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ]);
})
```



### 中间件别名

可以在应用程序的 `bootstrap/app.php` 文件中为中间件分配别名。

中间件别名允许您为给定的中间件类定义一个简短的别名，这对于具有较长类名的中间件特别有用

```php
use App\Http\Middleware\EnsureUserIsSubscribed;
 
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'subscribed' => EnsureUserIsSubscribed::class
    ]);
})
```

一旦在应用程序的 `bootstrap/app.php` 文件中定义了中间件别名，就可以在将中间件分配给路由时使用这个别名

```php
Route::get('/profile', function () {
    // ...
})->middleware('subscribed');
```

#### Laravel 内置中间件默认使用的别名

| Alias              | Middleware                                                   |
| :----------------- | :----------------------------------------------------------- |
| `auth`             | `Illuminate\Auth\Middleware\Authenticate`                    |
| `auth.basic`       | `Illuminate\Auth\Middleware\AuthenticateWithBasicAuth`       |
| `auth.session`     | `Illuminate\Session\Middleware\AuthenticateSession`          |
| `cache.headers`    | `Illuminate\Http\Middleware\SetCacheHeaders`                 |
| `can`              | `Illuminate\Auth\Middleware\Authorize`                       |
| `guest`            | `Illuminate\Auth\Middleware\RedirectIfAuthenticated`         |
| `password.confirm` | `Illuminate\Auth\Middleware\RequirePassword`                 |
| `precognitive`     | `Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests` |
| `signed`           | `Illuminate\Routing\Middleware\ValidateSignature`            |
| `subscribed`       | `\Spark\Http\Middleware\VerifyBillableIsSubscribed`          |
| `throttle`         | `Illuminate\Routing\Middleware\ThrottleRequests` or `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis` `Illuminate\Routing\Middleware\ThrottleRequests` 或 `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis` |
| `verified`         | `Illuminate\Auth\Middleware\EnsureEmailIsVerified`           |

### 中间件优先级

以使用应用程序 `bootstrap/app.php` 文件中的方法 `priority` 指定中间件优先级

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->priority([
        \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
        \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ]);
})
```



## 中间件参数

中间件还可以接收其他参数

```php
<?php
 
namespace App\Http\Middleware;
 
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;
 
class EnsureUserHasRole
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next, string $role): Response
    {
        if (! $request->user()->hasRole($role)) {
            // Redirect...
        }
 
        return $next($request);
    }
 
}
```

在定义路由时，可以通过将中间件名称和参数以`:`分隔来指定中间件参数：

```php
Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware('role:editor');
```

多个参数可以用逗号进行分隔：

```php
Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware('role:editor,publisher');
```

## 终止中间件

有时候，中间件可能需要在HTTP响应已发送到浏览器后进行一些工作。如果在中间件上定义了 `terminate` 方法，并且 Web 服务器使用 FastCGI，那么 `terminate` 方法将会在响应发送到浏览器后自动被调用

```php
<?php
 
namespace Illuminate\Session\Middleware;
 
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;
 
class TerminatingMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        return $next($request);
    }
 
    /**
     * Handle tasks after the response has been sent to the browser.
     */
    public function terminate(Request $request, Response $response): void
    {
        // ...
    }
}
```

`terminate` 方法应接收请求和响应两个参数。一旦定义了一个终止中间件，需要将其添加到路由列表或全局中间件中，即在应用程序的 `bootstrap/app.php` 文件中。

在调用中间件的 `terminate` 方法时，Laravel 将会从服务容器中解析一个新的中间件实例。

如果需要在调用 `handle` 和 `terminate` 方法时使用同一个中间件实例，可以使用容器的 `singleton` 方法将中间件注册为单例。

通常情况下，这需要在 `AppServiceProvider` 的 `register` 方法中完成。

```php
use App\Http\Middleware\TerminatingMiddleware;
 
/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(TerminatingMiddleware::class);
}
```


