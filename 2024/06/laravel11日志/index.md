# Laravel 11.x 日志


----

[原文](https://laravel.com/docs/11.x/logging)

----

## 介绍

Laravel 日志记录基于“通道”。

每个通道代表一种特定的日志写入方式。

例如，`single` 通道将日志写入单个日志文件，而 `slack` 通道则将日志消息发送到 Slack。

日志消息可以根据其严重程度写入多个通道。

在内部，Laravel 使用 Monolog 库，该库支持多种强大的日志处理器。

Laravel 使得配置这些处理器变得非常简单，允许混合搭配它们以自定义应用程序的日志处理。

## 配置

控制应用程序日志记录行为的所有配置选项都位于 `config/logging.php` 配置文件中。

默认情况下，Laravel 将在记录消息时使用该 `stack` 通道。

通道 `stack` 用于将多个日志通道聚合到单个通道中。

### 可用通道驱动

| Name         | Description                                                  |
| :----------- | :----------------------------------------------------------- |
| `custom`     | 调用指定工厂以创建通道的驱动程序                             |
| `daily`      | `RotatingFileHandler` 基于 Monolog 的驱动程序，每天轮换      |
| `errorlog`   | `ErrorLogHandler` 基于 Monolog 驱动程序                      |
| `monolog`    | 可以使用任何受支持的 Monolog 处理程序的 Monolog 工厂驱动程序 |
| `papertrail` | `SyslogUdpHandler` 基于 Monolog 驱动程序                     |
| `single`     | 基于单个文件或路径的记录器通道 （ `StreamHandler` ）         |
| `slack`      | `SlackWebhookHandler` 基于 Monolog 驱动程序                  |
| `stack`      | 便于创建“多通道”通道的包装器                                 |
| `syslog`     | `SyslogHandler` 基于 Monolog 驱动程序                        |

#### 配置通道名称

默认情况下，Monolog 实例化时使用与当前环境（例如 production 或 local）匹配的“通道名称”。

要更改此值，可以在通道的配置中添加一个 `name` 选项：

```php
'stack' => [
    'driver' => 'stack',
    'name' => 'channel-name',
    'channels' => ['single', 'slack'],
],
```

### 通道先决条件

#### 配置单通道和每日通道

`single` 和 `daily` 通道有三个可选配置选项： `bubble` 、 `permission` 和 `locking` 。

| Name         | Description                          | Default |
| :----------- | :----------------------------------- | :------ |
| `bubble`     | 指示消息在处理后是否应冒泡到其他通道 | `true`  |
| `locking`    | 尝试在写入日志文件之前锁定日志文件   | `false` |
| `permission` | 日志文件的权限                       | `0644`  |

此外，可以通过 `LOG_DAILY_DAYS` 环境变量或设置 `days` 配置选项来配置 `daily` 通道的保留策略。

| Name   | Description              | Default |
| :----- | :----------------------- | :------ |
| `days` | 应保留每日日志文件的天数 | `7`     |

#### 配置papertrail通道

`papertrail` 通道需要 `host` 和 `port` 配置选项。

这些选项可以通过 `PAPERTRAIL_URL` 和 `PAPERTRAIL_PORT` 环境变量来定义。

可以从 Papertrail 获取这些值。

#### 配置 Slack 频道

`slack` 通道需要一个 `url` 配置选项。

这个值可以通过 `LOG_SLACK_WEBHOOK_URL` 环境变量来定义。

这个 URL 应该匹配为 Slack 团队配置的传入 webhook 的 URL。

默认情况下，Slack 只会接收等级为 `critical` 及以上的日志；

然而，可以通过 `LOG_LEVEL` 环境变量或修改 Slack 日志通道配置数组中的 `level` 配置选项来调整这一设置。

### 日记记录弃用警告

可以使用 `LOG_DEPRECATIONS_CHANNEL` 环境变量或在应用程序的 `config/logging.php` 配置文件中指定首选 `deprecations` 日志通道来记录一些弃用警告：

```php
'deprecations' => [
    'channel' => env('LOG_DEPRECATIONS_CHANNEL', 'null'),
    'trace' => env('LOG_DEPRECATIONS_TRACE', false),
],
 
'channels' => [
    // ...
]
```

还可以定义一个名为 `deprecations` 的日志通道。

如果存在具有此名称的日志通道，则始终使用它来记录弃用：

```php
'channels' => [
    'deprecations' => [
        'driver' => 'single',
        'path' => storage_path('logs/php-deprecation-warnings.log'),
    ],
],
```



## 构建日志堆栈

为了方便起见，该`stack`驱动程序允许将多个通道组合成一个日志通道。

```php
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['syslog', 'slack'], 
        'ignore_exceptions' => false,
    ],
 
    'syslog' => [
        'driver' => 'syslog',
        'level' => env('LOG_LEVEL', 'debug'),
        'facility' => env('LOG_SYSLOG_FACILITY', LOG_USER),
        'replace_placeholders' => true,
    ],
 
    'slack' => [
        'driver' => 'slack',
        'url' => env('LOG_SLACK_WEBHOOK_URL'),
        'username' => env('LOG_SLACK_USERNAME', 'Laravel Log'),
        'emoji' => env('LOG_SLACK_EMOJI', ':boom:'),
        'level' => env('LOG_LEVEL', 'critical'),
        'replace_placeholders' => true,
    ],
],
```

#### 日志级别

上面示例中 `syslog` 和 `slack` 通道配置上存在的 `level` 配置选项。

此选项确定通道记录消息必须达到的最低“级别”。

Monolog 为 Laravel 的日志记录服务提供支持，提供 RFC 5424 规范中定义的所有日志级别。

按严重性降序排列，这些日志级别为：紧急（emergency）、警报（alert）、严重（critical）、错误（error）、警告（warning）、通知（notice）、信息（info）和调试（debug）。



## 写入日志消息

```php
use Illuminate\Support\Facades\Log;
 
Log::emergency($message);
Log::alert($message);
Log::critical($message);
Log::error($message);
Log::warning($message);
Log::notice($message);
Log::info($message);
Log::debug($message);
```



### 上下文信息

上下文数据数组可以传递给日志方法。此上下文数据将被格式化，并与日志消息一起显示：

```php
use Illuminate\Support\Facades\Log;
 
Log::info('User {id} failed to login.', ['id' => $user->id]);
```

有时，可能希望指定一些上下文信息，这些信息应包含在特定通道中的所有后续日志条目中。

例如，需要将与应用程序的每个传入请求关联的请求 ID 记录下来。

为此，可以调用 `Log` Facade `withContext` 的方法：

```php
<?php
 
namespace App\Http\Middleware;
 
use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;
 
class AssignRequestId
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        $requestId = (string) Str::uuid();
 
        Log::withContext([
            'request-id' => $requestId
        ]);
 
        $response = $next($request);
 
        $response->headers->set('Request-Id', $requestId);
 
        return $response;
    }
}
```

如果要在所有日志记录通道之间共享上下文信息，可以调用该 `Log::shareContext()` 方法。

此方法将向所有创建的频道以及随后创建的任何频道提供上下文信息：

```php
<?php
 
namespace App\Http\Middleware;
 
use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;
 
class AssignRequestId
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        $requestId = (string) Str::uuid();
 
        Log::shareContext([
            'request-id' => $requestId
        ]);
 
        // ...
    }
}
```



### 写入特定通道

可以在 `Log` Facade 上使用该 `channel` 方法检索并记录到配置文件中定义的任何通道：

```php
use Illuminate\Support\Facades\Log;
 
Log::channel('slack')->info('Something happened!');
```

如果要创建由多个通道组成的按需日志记录堆栈，可以使用以下 `stack` 方法：

```php
Log::stack(['single', 'slack'])->info('Something happened!');
```

#### 点播通道

还可以通过在运行时提供配置来创建按需通道，而无需在应用程序的 `logging` 配置文件中显示该配置。为此，可以将配置数组传递给 `Log` Facade `build` 的方法：

```php
use Illuminate\Support\Facades\Log;
 
Log::build([
  'driver' => 'single',
  'path' => storage_path('logs/custom.log'),
])->info('Something happened!');
```

还希望在按需日志记录堆栈中包含按需通道。

这可以通过将点播频道实例包含在传递给方法的 `stack` 数组中来实现：

```php
use Illuminate\Support\Facades\Log;
 
$channel = Log::build([
  'driver' => 'single',
  'path' => storage_path('logs/custom.log'),
]);
 
Log::stack(['slack', $channel])->info('Something happened!');
```



## Monolog通道定制

### 自定义通道的Monolog

首先，在通道的配置上定义一个 `tap` 数组。

该 `tap` 数组应包含一个类列表，这些类在创建 Monolog 实例后应有机会对其进行自定义（或“点击”）。没有放置这些类的常规位置，因此可以在应用程序中自由创建一个目录来包含这些类：

```php
'single' => [
    'driver' => 'single',
    'tap' => [App\Logging\CustomizeFormatter::class],
    'path' => storage_path('logs/laravel.log'),
    'level' => env('LOG_LEVEL', 'debug'),
    'replace_placeholders' => true,
],
```

在通道上配置该 `tap` 选项后，就可以定义将自定义 Monolog 实例的类了。

此类只需要一个方法： `__invoke` ，该方法接收一个 `Illuminate\Log\Logger` 实例。

该 `Illuminate\Log\Logger` 实例将所有方法调用代理到底层 Monolog 实例：

```php
<?php
 
namespace App\Logging;
 
use Illuminate\Log\Logger;
use Monolog\Formatter\LineFormatter;
 
class CustomizeFormatter
{
    /**
     * Customize the given logger instance.
     */
    public function __invoke(Logger $logger): void
    {
        foreach ($logger->getHandlers() as $handler) {
            $handler->setFormatter(new LineFormatter(
                '[%datetime%] %channel%.%level_name%: %message% %context% %extra%'
            ));
        }
    }
}
```



### 创建Monolog处理程序通道

使用 `monolog` 驱动程序时， `handler` 配置选项用于指定将实例化的处理程序。可以使用 `with` 配置选项指定处理程序所需的任何构造函数参数：

```php
'logentries' => [
    'driver'  => 'monolog',
    'handler' => Monolog\Handler\SyslogUdpHandler::class,
    'with' => [
        'host' => 'my.logentries.internal.datahubhost.company.com',
        'port' => '10000',
    ],
],
```

#### 格式化程序

使用 `monolog` 驱动程序时，Monolog `LineFormatter` 将用作默认格式化程序。

但是，可以使用 `formatter` 和 `formatter_with` 配置选项自定义传递给处理程序的格式化程序的类型：

```php
'browser' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\BrowserConsoleHandler::class,
    'formatter' => Monolog\Formatter\HtmlFormatter::class,
    'formatter_with' => [
        'dateFormat' => 'Y-m-d',
    ],
],
```

如果使用的是能够提供自己的格式化程序的 Monolog 处理程序，则可以将 `formatter` 配置选项的值设置为 `default` ：

```php
'newrelic' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\NewRelicHandler::class,
    'formatter' => 'default',
],
```

#### 处理器

如果要自定义 `monolog` 驱动程序的处理器，请将 `processors` 配置值添加到频道的配置中：

```php
'memory' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\StreamHandler::class,
    'with' => [
        'stream' => 'php://stderr',
    ],
    'processors' => [
        // Simple syntax...
        Monolog\Processor\MemoryUsageProcessor::class,
 
        // With options...
        [
           'processor' => Monolog\Processor\PsrLogMessageProcessor::class,
           'with' => ['removeUsedContextFields' => true],
       ],
    ],
],
```



### 通过工厂创建自定义渠道

如果要定义一个完全自定义的通道，在其中可以完全控制 Monolog 的实例化和配置，则可以在 `config/logging.php` 配置文件中指定 `custom` 驱动程序类型。

配置应包含一个 `via` 选项，该选项包含工厂类的名称，该名称将被调用以创建 Monolog 实例：

```php
'channels' => [
    'example-custom-channel' => [
        'driver' => 'custom',
        'via' => App\Logging\CreateCustomLogger::class,
    ],
],
```

配置 `custom` 驱动程序通道后，即可定义将创建 Monolog 实例的类。

此类只需要一个 `__invoke` 方法，该方法应返回 Monolog 记录器实例。

该方法将接收通道配置数组作为其唯一参数：

```php
<?php
 
namespace App\Logging;
 
use Monolog\Logger;
 
class CreateCustomLogger
{
    /**
     * Create a custom Monolog instance.
     */
    public function __invoke(array $config): Logger
    {
        return new Logger(/* ... */);
    }
}
```



## 使用Pail跟踪日志消息

Laravel Pail 是一个软件包，可让直接从命令行轻松深入了解 Laravel 应用程序的日志文件。

与标准 `tail` 命令不同，Pail 旨在与任何日志驱动程序一起使用，包括 Sentry 或 Flare。

此外，Pail 还提供了一组有用的过滤器，可帮助您快速找到所需的内容。

### 安装

Laravel Pail 需要 PHP 8.2+ 和 PCNTL 扩展名。

```bash
composer require laravel/pail
```

### 用法

若要启动尾随日志，请运行以下 `pail` 命令：

```bash
php artisan pail
```

若要增加输出的详细程度并避免截断 （...），请使用以下 `-v` 选项：

```bash
php artisan pail -v
```

要获得最大详细程度并显示异常堆栈跟踪，请使用以下 `-vv` 选项：

```bash
php artisan pail -vv
```

要停止尾矿日志，请随时按 `Ctrl+C` 。

### 过滤日志

> `--filter`：选项按日志类型、文件、消息和堆栈跟踪内容筛选日志

```bash
php artisan pail --filter="QueryException"
```

> `--message`：仅按日志消息筛选日志

```bash
php artisan pail --message="User created"
```

> `--level`：选项可用于按日志级别筛选日志

```bash
php artisan pail --level=error
```

> `--user`：仅显示在对给定用户进行身份验证时写入的日志

```bash
php artisan pail --user=1
```


