# Laravle 11.x 会话


----

[原文](https://laravel.com/docs/11.x/session)

----

## 介绍

由于 HTTP 驱动的应用程序是无状态的，因此会话提供了一种跨多个请求存储有关用户的信息的方法。

该用户信息通常放置在可从后续请求访问的持久性存储/后端中。

### 配置

应用程序的会话配置文件存储在`config/session.php`。

默认情况下，Laravel配置使用`database`会话驱动程序。

会话`driver`配置选项定义每个请求的会话数据的存储位置。

Laravel包括多种驱动程序：

- `file`：会话存储在`storage/framework/sessions`
- `cookie`：会话存储在安全、加密的Cookie中
- `database`：会话存储在关系型数据库中
- `memcached`/`Redis`：会话存储在基于缓存的快速存储之一
- `dynamodb`：会话存储在AWS DynamoDB中
- `array`：会话存储在PHP数据中，不会被持久化

### 驱动程序先决条件

#### Database

需要确保有一个数据库表来包含会话数据。

通常，这包含在 Laravel 的默认 `0001_01_01_000000_create_users_table.php` 数据库迁移中;

但是，如果没有 `sessions` 表，则可以使用 `make:session-table` Artisan 命令生成此迁移：

```php
php artisan make:session-table
 
php artisan migrate
```

#### Redis

需要通过 PECL 安装 PhpRedis PHP 扩展或通过 Composer 安装 `predis/predis` 软件包 （~1.0）。

## 会话交互

### 检索数据

在 Laravel 中处理会话数据有两种主要方式：全局 `session` 帮助程序和通过 `Request` 实例。

通过 `Request` 实例访问会话，该实例可以在路由闭包或控制器方法上进行类型提示。

```php
<?php
 
namespace App\Http\Controllers;
 
use Illuminate\Http\Request;
use Illuminate\View\View;
 
class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     */
    public function show(Request $request, string $id): View
    {
        $value = $request->session()->get('key');
 
        // ...
 
        $user = $this->users->find($id);
 
        return view('user.profile', ['user' => $user]);
    }
}
```

从会话中检索项时，还可以将默认值作为第二个参数传递 `get` 给方法。

如果会话中不存在指定的key，则将返回此默认值。

如果将闭包作为默认值传递给方法， `get` 并且请求的键不存在，则将执行闭包并返回其结果：

```php
$value = $request->session()->get('key', 'default');
 
$value = $request->session()->get('key', function () {
    return 'default';
});
```

#### 全局会话助手

还可以使用全局 `session` PHP 函数在会话中检索和存储数据。

当使用单个字符串参数调用 `session` 帮助程序时，它将返回该会话键的值。

当使用键/值对数组调用帮助程序时，这些值将存储在会话中：

```php
Route::get('/home', function () {
    // Retrieve a piece of data from the session...
    $value = session('key');
 
    // Specifying a default value...
    $value = session('key', 'default');
 
    // Store a piece of data in the session...
    session(['key' => 'value']);
});
```

#### 检索所有会话数据

> `all`方法检索会话中的所有数据

```php
$data = $request->session()->all();
```

#### 检索会话数据的一部分

> `only` 和 `except` 方法可用于检索会话数据的子集

```php
$data = $request->session()->only(['username', 'email']);
 
$data = $request->session()->except(['username', 'email']);
```

#### 确定会话中是否存在项目

> `has`方法可以确定会话中是否存在某个项目。
>
> 如果项目存在并且不是`null`，则返回true

```php
if ($request->session()->has('users')) {
    // ...
}
```

要确定会话中是否存在某个项目，即使其值为 `null` ，也可以使用以下 `exists` 方法：

```php
if ($request->session()->exists('users')) {
    // ...
}
```

要确定会话中是否不存在某个项目，可以使用该 `missing` 方法。

如果项不存在，则该 `missing` 方法返回 `true` ：

```php
if ($request->session()->missing('users')) {
    // ...
}
```

### 存储数据

> `put`方法或者全局`session`帮助程序都可以将数据存储在会话中

```php
// Via a request instance...
$request->session()->put('key', 'value');
 
// Via the global "session" helper...
session(['key' => 'value']);
```

#### 推送到数组会话值

> `push`方法可用于将新值推送到作为数组的会话值上

```php
$request->session()->push('user.teams', 'developers');
```

#### 检索和删除项目

> `pull`方法将在单个语句中从会话中检索和删除项目

```php
$value = $request->session()->pull('key', 'default');
```

#### 递增和递减会话值

> 使用 `increment` 和 `decrement` 方法可以将会话数据中包含的整数进行自增或递减

```php
$request->session()->increment('count');
 
$request->session()->increment('count', $incrementBy = 2);
 
$request->session()->decrement('count');
 
$request->session()->decrement('count', $decrementBy = 2);
```



### 闪存数据

> `flash`方法可以在会话中存储项目以备下一个请求使用。
>
> 使用此方法存储在会话中的数据将立即可用，并在随后的 HTTP 请求期间可用。后续 HTTP 请求后，烧录的数据将被删除。
>
> 闪存数据主要用于短期状态消息。

```php
$request->session()->flash('status', 'Task was successful!');
```

> `reflash`方法为多个请求保留闪存数据。
>
> 该方法保留其他请求的所有闪存数据。

```php
$request->session()->reflash();
```

> 如果需要保留特定的闪存数据，则可以使用`keep`方法。
```php
$request->session()->keep(['username', 'email']);
```

> 要仅保留当前的闪存数据，可以使用`now`方法。

```php
$request->session()->now('status', 'Task was successful!');
```

### 删除数据

> `forget`方法将从会话中删除一段数据。
>
> 如果要删除所有数据，可以使用`flush`方法

```php
// Forget a single key...
$request->session()->forget('name');
 
// Forget multiple keys...
$request->session()->forget(['name', 'status']);
 
$request->session()->flush();
```



### 重新生成会话ID

重新生成会话 ID 通常是为了防止恶意用户利用对应用程序的会话固定攻击。

> 手动重新生成会话ID，可以使用`regenerate`方法

```php
$request->session()->regenerate();
```

> `invalidate`方法可以在单个语句中重新生成会话ID并从会话中删除所有数据

```php
$request->session()->invalidate();
```

## 会话阻止

若要利用会话阻止，应用程序必须使用支持原子锁的缓存驱动程序。目前，这些缓存驱动程序包括 `memcached` 、 `dynamodb` 、 `redis` 、 `database` `file` 和 `array` 驱动程序。此外，不能使用 `cookie` 会话驱动程序。

Laravel提供允许限制给定会话的并发请求的功能。只需将 `block` 方法链接到路由定义上即可。

在此示例中，对 `/profile` 终端节点的传入请求将获取会话锁。

在保持此锁时，对共享相同会话 ID 的 `/profile` 或 `/order` 终结点的任何传入请求都将等待第一个请求完成执行，然后再继续执行：

```php
Route::post('/profile', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10)
 
Route::post('/order', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10)
```

该 `block` 方法接受两个可选参数。该 `block` 方法接受的第一个参数是会话锁定在释放之前应保留的最大秒数。当然，如果请求在此时间之前完成执行，则锁将提前释放。

该 `block` 方法接受的第二个参数是请求在尝试获取会话锁时应等待的秒数。如果请求无法在给定的秒数内获得会话锁，则将抛出 `Illuminate\Contracts\Cache\LockTimeoutException`异常。

如果这两个参数均未传递，则在尝试获取锁时，将最多获取 10 秒，请求最多等待 10 秒：

```php
Route::post('/profile', function () {
    // ...
})->block()
```



## 自定义会话驱动程序

### 实现驱动程序

自定义会话驱动程序应实现PHP的`SessionHandlerInterface`内置接口。

```php
<?php
 
namespace App\Extensions;
 
class MongoSessionHandler implements \SessionHandlerInterface
{
    public function open($savePath, $sessionName) {}
    public function close() {}
    public function read($sessionId) {}
    public function write($sessionId, $data) {}
    public function destroy($sessionId) {}
    public function gc($lifetime) {}
}
```

Laravel 不附带包含扩展的目录。可以自由地将它们放置在任何地方。在此示例中，创建了一个 `Extensions` 目录来容纳 `MongoSessionHandler` 。

- `open`：通常用于基于文件的会话存储系统。由于 Laravel 附带了 `file` 会话驱动程序，因此很少需要在此方法中添加任何内容。可以简单地将此方法留空。
- `close`：与`open`方法一样，通常可以忽略不计
- `read`：应返回与给定`$sessionId`关联的会话数据的字符串版本。在驱动程序中检索或存储会话数据时，无需执行任何序列化或其他编码，因为Laravel将主动执行序列化。
- `write`：应将与关联的 `$sessionId` 给定 `$data` 字符串写入某些持久性存储系统，例如 MongoDB 或选择的其他存储系统。同样，不应该执行任何序列化 ，因为 Laravel 已经处理了。
- `destroy`：应从持久性存储中删除与 关联 `$sessionId` 的数据。
- `gc`：应销毁所有早于给定 `$lifetime` 的会话数据，即 UNIX 时间戳。对于像 Memcached 和 Redis 这样的自过期系统，此方法可能留空。

### 注册驱动程序

实现驱动程序后，即可将其注册到 Laravel。

要向 Laravel 的会话后端添加其他驱动程序，可以使用 `Session` Facade 提供 `extend` 的方法。

应从服务提供商 `boot` 的方法调用该 `extend` 方法。

可以从现有 `App\Providers\AppServiceProvider` 提供程序中执行此操作，也可以创建一个全新的提供程序：

```php
<?php
 
namespace App\Providers;
 
use App\Extensions\MongoSessionHandler;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Session;
use Illuminate\Support\ServiceProvider;
 
class SessionServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }
 
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Session::extend('mongo', function (Application $app) {
            // Return an implementation of SessionHandlerInterface...
            return new MongoSessionHandler;
        });
    }
}
```

注册会话驱动程序后，可以使用 `SESSION_DRIVER` 环境变量或在应用程序的 `config/session.php` 配置文件中将 `mongo` 驱动程序指定为应用程序的会话驱动程序。

