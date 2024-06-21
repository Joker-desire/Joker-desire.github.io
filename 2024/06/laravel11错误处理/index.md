# Laravel 11.x 错误处理


----

[原文](https://laravel.com/docs/11.x/errors)

----

## 介绍

当启动一个新的 Laravel 项目时，错误和异常处理已经配置好了；

然而，在任何时候，都可以在应用程序的 `bootstrap/app.php` 文件中使用 `withExceptions` 方法来管理异常的报告和渲染方式。

传递给 `withExceptions` 闭包的 `$exceptions` 对象是 `Illuminate\Foundation\Configuration\Exceptions` 的一个实例，它负责管理应用程序中的异常处理。

## 配置

`config/app.php` 配置文件中的 `debug` 选项决定了实际向用户显示多少错误信息。

默认情况下，此选项设置为遵循存储在 `.env` 文件中的 `APP_DEBUG` 环境变量的值。

## 处理异常

### 报告异常

可以在应用程序 `bootstrap/app.php` 中使用 `report` 异常方法来注册一个闭包，当需要报告给定类型的异常时，应执行该闭包。Laravel 将通过检查闭包的类型提示来确定闭包报告的异常类型：

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->report(function (InvalidOrderException $e) {
        // ...
    });
})
```

当使用 `report` 方法注册自定义异常报告回调时，Laravel 仍然会使用应用程序的默认日志配置记录该异常。

如果希望阻止异常传播到默认的日志，则可以在定义报告回调时使用 `stop` 方法，或者从回调中返回 `false`：

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->report(function (InvalidOrderException $e) {
        // ...
    })->stop();
 
    $exceptions->report(function (InvalidOrderException $e) {
        return false;
    });
})
```

#### 全局日志上下文

如果可用，Laravel 会自动将当前用户的 ID 作为上下文数据添加到每个异常的日志消息中。

可以在应用程序的 `bootstrap/app.php` 文件中使用 `context` 异常方法定义全局上下文数据。

这些信息将被包含在应用程序写入的每条异常日志消息中：

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->context(fn () => [
        'foo' => 'bar',
    ]);
})
```

#### 异常日志上下文

虽然在每条日志消息中添加上下文信息是有用的，但有时特定的异常可能需要包含在日志中的独特上下文。

通过在应用程序的某个异常上定义一个 `context` 方法，可以指定任何与该异常相关的数据，该数据将被添加到该异常的日志条目中：

```php
<?php
 
namespace App\Exceptions;
 
use Exception;
 
class InvalidOrderException extends Exception
{
    // ...
 
    /**
     * Get the exception's context information.
     *
     * @return array<string, mixed>
     */
    public function context(): array
    {
        return ['order_id' => $this->orderId];
    }
}
```

#### `report`帮手

有时可能需要报告一个异常，但继续处理当前请求。

`report` 辅助函数允许快速报告一个异常，而不向用户呈现错误页面：

```php
public function isValid(string $value): bool
{
    try {
        // Validate the value...
    } catch (Throwable $e) {
        report($e);
 
        return false;
    }
}
```

#### 删除重复报告的异常

如果在整个应用程序中使用`report`函数，则有时可能会多次报告相同的异常，从而在日志中创建重复条目。

如果要确保某个异常实例仅被报告一次，则可以在应用程序的 `bootstrap/app.php` 文件中调用 `dontReportDuplicates` 异常方法：

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->dontReportDuplicates();
})
```

现在，当 `report` 辅助函数被调用同一个异常实例时，只有第一次调用会被报告：

```php
$original = new RuntimeException('Whoops!');
 
report($original); // reported
 
try {
    throw $original;
} catch (Throwable $caught) {
    report($caught); // ignored
}
 
report($original); // ignored
report($caught); // ignored
```





### 异常日志级别

可以在应用程序 `bootstrap/app.php` 文件中使用 `level` exception 方法。

此方法接收异常类型作为其第一个参数，并将日志级别作为其第二个参数：

```php
use PDOException;
use Psr\Log\LogLevel;
 
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->level(PDOException::class, LogLevel::CRITICAL);
})
```



### 按类型忽略异常

可以在应用程序 `bootstrap/app.php` 文件中使用 `dontReport` exception 方法进行忽略某些异常。

提供给此方法的任何类都不会被报告，但是可能仍具有自定义渲染逻辑：

```php
use App\Exceptions\InvalidOrderException;
 
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->dontReport([
        InvalidOrderException::class,
    ]);
})
```

在内部，Laravel 已经忽略了某些类型的错误，如由 404 HTTP 错误或无效 CSRF 令牌生成的 419 HTTP 响应所导致的异常。

如果想指示 Laravel 停止忽略某种类型的异常，可以在应用程序的 `bootstrap/app.php` 文件中使用 `stopIgnoring` 异常方法：

```php
use Symfony\Component\HttpKernel\Exception\HttpException;
 
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->stopIgnoring(HttpException::class);
})
```

### 渲染异常

默认情况下，Laravel 的异常处理器会将异常转换为 HTTP 响应。

但是，可以自由为特定类型的异常注册自定义渲染闭包。

可以通过在应用程序的 `bootstrap/app.php` 文件中使用 `render` 异常方法来实现这一点。

传递给 `render` 方法的闭包应该返回一个 `Illuminate\Http\Response` 实例，这个实例可以通过 `response` 辅助函数生成。

Laravel 会通过检查闭包的类型提示来确定该闭包渲染的异常类型：

```php
use App\Exceptions\InvalidOrderException;
use Illuminate\Http\Request;
 
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (InvalidOrderException $e, Request $request) {
        return response()->view('errors.invalid-order', [], 500);
    });
})
```

也可以使用 `render` 方法来覆盖内置的 Laravel 或 Symfony 异常（例如 `NotFoundHttpException`）的渲染行为。

如果传递给 `render` 方法的闭包没有返回值，Laravel 将使用默认的异常渲染：

```php
use Illuminate\Http\Request;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
 
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (NotFoundHttpException $e, Request $request) {
        if ($request->is('api/*')) {
            return response()->json([
                'message' => 'Record not found.'
            ], 404);
        }
    });
})
```

#### 将异常呈现为JSON

在渲染异常时，Laravel 会根据请求的 `Accept` 头自动确定是将异常渲染为 HTML 还是 JSON 响应。

如果自定义 Laravel 如何确定是渲染 HTML 还是 JSON 异常响应，可以使用 `shouldRenderJsonWhen` 方法：

```php
use Illuminate\Http\Request;
use Throwable;
 
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->shouldRenderJsonWhen(function (Request $request, Throwable $e) {
        if ($request->is('admin/*')) {
            return true;
        }
 
        return $request->expectsJson();
    });
})
```

#### 自定义异常响应

极少情况下，可能需要自定义 Laravel 异常处理器渲染的整个 HTTP 响应。

为此，可以使用 `respond` 方法注册一个响应自定义闭包：

```php
use Symfony\Component\HttpFoundation\Response;
 
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->respond(function (Response $response) {
        if ($response->getStatusCode() === 419) {
            return back()->with([
                'message' => 'The page expired, please try again.',
            ]);
        }
 
        return $response;
    });
})
```

### 可报告和可渲染的异常

与其在应用程序的 `bootstrap/app.php` 文件中定义自定义报告和渲染行为，不如直接在应用程序的异常类上定义 `report` 和 `render` 方法。当这些方法存在时，框架会自动调用它们：

```php
<?php
 
namespace App\Exceptions;
 
use Exception;
use Illuminate\Http\Request;
use Illuminate\Http\Response;
 
class InvalidOrderException extends Exception
{
    /**
     * Report the exception.
     */
    public function report(): void
    {
        // ...
    }
 
    /**
     * Render the exception into an HTTP response.
     */
    public function render(Request $request): Response
    {
        return response(/* ... */);
    }
}
```

如果异常继承自已具有渲染功能的异常（如内置的 Laravel 或 Symfony 异常），可以在异常的 `render` 方法中返回 `false` 以渲染该异常的默认 HTTP 响应：

```php
/**
 * Render the exception into an HTTP response.
 */
public function render(Request $request): Response|bool
{
    if (/** Determine if the exception needs custom rendering */) {
 
        return response(/* ... */);
    }
 
    return false;
}
```

如果异常包含仅在某些条件满足时才需要的自定义报告逻辑，那么可能需要指示 Laravel 有时使用默认的异常处理配置来报告该异常。

为此，可以在异常的 `report` 方法中返回 `false`：

```php
/**
 * Report the exception.
 */
public function report(): bool
{
    if (/** Determine if the exception needs custom reporting */) {
 
        // ...
 
        return true;
    }
 
    return false;
}
```



## 限制报告的异常

如果应用程序报告的异常数量非常多，则可能需要限制实际记录或发送到应用程序的外部错误跟踪服务的异常数量。

要对异常进行随机抽样率，可以在应用程序的 `bootstrap/app.php` 文件中使用 `throttle` 异常方法。

`throttle` 方法接收一个应返回 `Lottery` 实例的闭包：

```php
use Illuminate\Support\Lottery;
use Throwable;
 
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->throttle(function (Throwable $e) {
        return Lottery::odds(1, 1000);
    });
})
```

还可以根据异常类型有条件地进行采样。如果只想采样特定异常类的实例，可以仅为该类返回一个 `Lottery` 实例：

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Support\Lottery;
use Throwable;
 
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof ApiMonitoringException) {
            return Lottery::odds(1, 1000);
        }
    });
})
```

也可以通过返回一个 `Limit` 实例而不是 `Lottery` 实例来对记录的异常或发送到外部错误跟踪服务的异常进行速率限制。

这在想要防止异常突然激增而淹没日志时非常有用，例如，当应用程序使用的第三方服务宕机时：

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;
 
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof BroadcastException) {
            return Limit::perMinute(300);
        }
    });
})
```

默认情况下，限流会使用异常的类名作为限流键。可以通过在 `Limit` 上使用 `by` 方法指定自己的键来定制这一行为：

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;
 
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof BroadcastException) {
            return Limit::perMinute(300)->by($e->getMessage());
        }
    });
})
```

当然，可以为不同的异常返回 `Lottery` 和 `Limit` 实例的组合：

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Lottery;
use Throwable;
 
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->throttle(function (Throwable $e) {
        return match (true) {
            $e instanceof BroadcastException => Limit::perMinute(300),
            $e instanceof ApiMonitoringException => Lottery::odds(1, 1000),
            default => Limit::none(),
        };
    });
})
```



## HTTP异常

有些异常描述了服务器的 HTTP 错误码。

例如，这可能是一个“页面未找到”错误（404）、“未授权错误”（401）或甚至是开发人员生成的 500 错误。为了在应用程序的任何地方生成这样的响应，可以使用 `abort` 辅助函数：

```php
abort(404);
```

### 自定义HTTP错误页面

Laravel 使得为各种 HTTP 状态码显示自定义错误页面变得很容易。

例如，要自定义 404 HTTP 状态码的错误页面，可以创建一个 `resources/views/errors/404.blade.php` 视图模板。

该视图将为应用程序生成的所有 404 错误进行渲染。

此目录内的视图应命名为与其对应的 HTTP 状态码一致。

由 `abort` 函数引发的 `Symfony\Component\HttpKernel\Exception\HttpException` 实例将作为 `$exception` 变量传递给视图：

```php+HTML
<h2>{{ $exception->getMessage() }}</h2>
```

可以使用 `vendor:publish` Artisan 命令发布 Laravel 的默认错误页面模板。

模板发布后，可以根据你的喜好自定义它们：

```bash
php artisan vendor:publish --tag=laravel-errors
```

#### 回退HTTP错误页

还可以为一系列 HTTP 状态码定义一个“后备”错误页面。

如果没有对应特定 HTTP 状态码的页面发生，这个页面将被渲染。

为此，在应用程序的 `resources/views/errors` 目录下定义一个 `4xx.blade.php` 模板和一个 `5xx.blade.php` 模板。

在定义后备错误页面时，这些后备页面不会影响 404、500 和 503 的错误响应，因为 Laravel 为这些状态码提供了内部专用页面。

要自定义这些状态码的渲染页面，应该分别为每个状态码定义一个自定义错误页面。

