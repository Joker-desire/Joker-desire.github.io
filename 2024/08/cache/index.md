# Laravel 11.x 缓存


----

[原文](https://laravel.com/docs/11.x/cache)

----

## 介绍

Laravel 为各种缓存后端提供了一个富有表现力的统一API，可以利用它们极快的数据检索并加快Web应用程序。

## 配置

缓存配置文件位于`config/cache.php`。在此文件中，可以指定整个应用程序默认使用的缓存存储。

Laravel支持开箱即用的`Memcached`、`Redis`、`DynamoDB`和关系数据库等流行的缓存后端。

此外还提供基于文件的缓存驱动程序，而`array`和`”null“`缓存驱动程序为自动化测试提供了方便的缓存后端。

缓存文件还包含了其他各种选项。默认情况下，Laravel配置为使用`database`缓存驱动程序，该驱动程序将序列化的缓存对象存储在应用程序的数据库中。

### 驱动程序先决条件

#### Database

使用`database`高速缓存驱动程序时，需要一个数据库表来包含高速缓存数据。

通常，默认包含在`0001_01_01_000001_create_cache_table.php`数据库迁移中；

也可以使用`make:cache-table`Artisan命令来创建：

```bash
php artisan make:cache-table

php artisan migrate
```

#### Memcached

使用Memcached驱动程序需要安装[Memcached PECL 软件包](https://pecl.php.net/package/memcached)。

可以在`config/cache.php`配置文件中列出所有Memcached服务器。此文件已包含一个`memcached.servers`条目：

```php
'memcached' => [
    // ...

    'servers' => [
        [
            'host' => env('MEMCACHED_HOST', '127.0.0.1'),
            'port' => env('MEMCACHED_PORT', 11211),
            'weight' => 100,
        ],
    ],
],
```

如果需要，也可以将`host`选项设置为UNIX套接字路径。如果执行此操作，则`port`则应该设置为`0`：

```php
'memcached' => [
    // ...

    'servers' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],
],
```

#### Redis

使用Redis缓存时，需要通过PECL安装PhpRedis PHP扩展或者通过Composer安装`predis/predis`包（~2.0）

#### DynamoDB

在使用DynamoDB缓存之前，必须创建一个DynamoDB表来存储所有缓存的数据。

通过，表应该命名为`cache`。但是，根据缓存配置文件中`stores.dynamodb.table`值来命名极客，还可以通过`DYNAMODB_CACHE_TABLE`环境变量设置表名。

此表还应具有一个字符串分区键，其名称与应用程序的缓存配置文件中的`stores.dynamodb.attributes.key`配置项的值相对应。默认情况下，分区键应命名为key。

接下来，需要安装AWS开发工具包，以便Laravel应用程序与DynamoDB通信：

```bash
composer require aws/aws-sdk-php
```

此外，应确保为DynamoDB缓存存储配置选择提供值。通常，这些选项应在应用程序的`.env`配置文件中定义：

```php
'dynamodb' => [
    'driver' => 'dynamodb',
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => env('DYNAMODB_CACHE_TABLE', 'cache'),
    'endpoint' => env('DYNAMODB_ENDPOINT'),
],
```

## 缓存使用情况

### 获取 Cache 实例

要获取缓存存储实例，可以使用`Cache`Facade，它提供了对Laravel缓存合约的底层实现的方便、简洁的访问：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * Show a list of all users of the application.
     */
    public function index(): array
    {
        $value = Cache::get('key');

        return [
            // ...
        ];
    }
}
```

#### 访问多个缓存储存

可以通过`store`方法访问各种缓存存储。传递给`store`方法的key应该对应于缓存配置文件中`stores`配置数组中列出的其中之一：

```php
$value = Cache::store('file')->get('foo');

Cache::store('redis')->put('bar', 'baz', 600); // 10 Minutes
```

### 从缓存中检索项目

`get`方法用于从缓存中检索项目。如果缓存中不存在该项目，则返回`null`。也可以传递第二个参数，指定默认值：

```php
$value = Cache::get('key', function() {
    return DB::table(/*...*/)->get();
});
```

#### key是否存在

`has`方法可用于确定缓存中是否存在key。如果存在但值为null,则也返回`false`:

```php
if (Cache::has('key')) {
    // ...
}
```

#### 递增/递减值

`increment`和`decrement`方法可用于调整缓存中整数项的值。都可指定第二个参数，指定增加或减少项目值的数量：

```php
// 不存在则使用默认值
Cache::add('key', 0, now()->addHours(4));

// 递增/递减
Cache::increment('key');
Cache::increment('key', $amount);
Cache::decrement('key');
Cache::decrement('key', $amount);
```

#### 检索和存储

有时希望检索key时，如果不存在，则也存储一个默认的值。可以使用`Cache::remember`方法来处理：

```php
$value = Cache::remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

如果缓存中不存在该key，则将执行传递给`remember`方法的闭包，其结果将放置在缓存中。

也可以使用`rememberForever`方法从缓存中检索项目，或者如果它不存在，则永久存储：

```php
$value = Cache::rememberForever('users', function () {
    return DB::table('users')->get();
});
```

#### 检索和删除

如果需要从缓存中检索key，然后删除该key，则可以使用`pull`方法。与`get`方法一样，如果缓存不存在该key，则返回`null`：

```php
$value = Cache::pull('key');

$value = Cache::pull('key', 'default');
```

### 在缓存中存储项目

`put`方法用来存储缓存中的key：

```php
Cache::put('key', 'value', $seconds = 10);
```

如果存储时间没有传递，则该key将被无期限存储：

```php
Cache::put('key', 'value');
```

也可以传递一个`DateTime`实例，该实例表示key的过期时间：

```php
Cache::put('key', 'value', now()->addMinutes(10));
```

#### 不存在才存储

`add`方法只会在缓存存储中不存在key时才将其添加到缓存中。如果key实际添加到缓存中，该方法将返回`true`。否则，该方法将返回`false`。`add`方法是一个原子操作：

```php
Cache::add('key', 'value', $seconds);
```

#### 永久存储key

`forever`方法可用于将key永久存储在缓存中。由于这些key不会过期，因此必须使用`forget`方法从缓存中手动删除：

```php
Cache::forever('key', 'value');
```

***ps: 当使用的是Memcached驱动时，当缓存达到其大小限制时，可能会删除永久存储的key***

### 从缓存中删除项目

`forget`方法从缓存中删除key：

```php
Cache::forget('key');
```

还可以通过提供零或者负数的过期秒数来删除key：

```php
Cache::put('key', 'value', 0);

Cache::put('key', 'value', -5);
```

也可以使用`flush`方法清楚整个缓存：

```php
Cache::flush();
```

***ps: flush会从缓存中删除所有的key，谨慎使用***

### 缓存帮助程序

除了使用`Cache`门面外，还可以使用全局`cache`功能通过缓存来检索和存储数据。

当使用单个字符串参数调用`cache`函数时，它将返回给定键的值：

```php
$value = cache('key');

cache(['key' => 'value'], $seconds);

cache(['key' => 'value'], now()->addMinutes(10));
```

当调用不带任何参数时，则会返回`Illuminate\Contracts\Cache\Factory`实现的实例，允许调用其他缓存方法：

```php
cache()->remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

## 原子锁

### 管理锁

原子锁允许操作分布式锁，而无需担心争用条件。可以使用`Cache::lock`方法创建和管理锁：

```php
use Illuminate\Support\Facades\Cache;

$lock = Cache::lock('foo', 10);

if ($lock->get()) {
    // 获得锁

    $lock->release();
}
```

`get`方法也接受一个闭包。执行closure后，Laravel会自动释放锁：

```php
Cache::lock('foo', 10)->get(function () {
    // 获取锁，处理完毕后自动释放
});
```

如果在请求锁时锁不可用，则可以指定等待秒数。如果在指定的时限内无法获取锁，则抛出异常`Illuminate\Contracts\Cache\LockTimeoutException`：

```php
use Illuminate\Contracts\Cache\LockTimeoutException;

$lock = Cache::lock('foo', 10);

try {
    $lock->block(5);

    // 最多等待 5 秒后获取锁定...
} catch (LockTimeoutException $e) {
    // 无法获取锁...
} finally {
    $lock->release();
}
```

也可以通过将闭包传递给`block`方法来简化。当将闭包传递给此方法时，Laravel将尝试在指定的秒数内获取锁，并在执行闭包后自动释放锁：

```php
Cache::lock('foo', 10)->block(5, function () {
    // 最多等待 5 秒后获取锁定...
});
```

### 跨进程管理锁

有时，希望在一个进程中获取锁，并在另一个进程中释放它。例如，在Web请求期间获取一个锁，并希望在该请求触发的排队作业结束的时候释放该锁。在这种情况下，应该蒋所的范围所有者令牌传递给排队的作业，以便作业可以使用给定的令牌重新实例化锁。

在下面的示例中，如果成功获取了锁，将分派排队的作业。此外，将通过锁的`owner`方法将锁的owner token 传递给排队的作业：

```php
$podcast = Podcast::find($id);

$lock = Cache::lock('processing', 120);

if ($lock->get()) {
    ProcessPodcast::dispatch($podcast, $lock->owner());
}
```

在应用程序的`ProcessPodcast`作业中，可以使用owner令牌恢复和释放锁：

```php
Cache::restoreLock('processing', $this->owner)->release();
```

如果想在不考虑当前持有者的情况下释放锁，可以使用`forceRelease`方法。

```php
Cache::lock('processing')->forceRelease();
```

这个方法通常用于一些特殊场景，比如迫不得已需要立即释放一个锁，无论当前锁的持有者是谁。请谨慎使用 forceRelease 方法，因为它会绕过所有的持有者检查，可能会导致数据一致性问题或其他意外情况。

## 添加自定义缓存程序

### 编写驱动程序

要创建自定义缓存驱动程序，首先需要先实现`Illuminate\Contracts\Cache\Store` Contract：

```php
<?php

namespace App\Extensions;

use Illuminate\Contracts\Cache\Store;

class MongoStore implements Store
{
    public function get($key) {}
    public function many(array $keys) {}
    public function put($key, $value, $seconds) {}
    public function putMany(array $values, $seconds) {}
    public function increment($key, $value = 1) {}
    public function decrement($key, $value = 1) {}
    public function forever($key, $value) {}
    public function forget($key) {}
    public function flush() {}
    public function getPrefix() {}
}
```

一旦实现完成，就可以通过调用`cache`Facade的`extend`方法来完成自定义驱动程序注册：

```php
Cache::extend('mongo', function (Application $app) {
    return Cache::repository(new MongoStore);
});
```

### 注册驱动程序

在`cache`Facade的`extend`上使用`extend`进行注册自定义缓存驱动。

因为其他服务提供者可能会在它们的`boot`方法中尝试读取缓存值，所以我们将会在一个`booting`回调中注册我们的自定义驱动。通过使用`booting`回调，我们可以确保自定义驱动在应用程序的服务提供者的`boot`方法被调用之前注册，但在所有服务提供者的`register`方法被调用之后。我们将在应用程序的`App\Providers\AppServiceProvider`类的`register`方法中注册我们的`booting`回调：

```php
<?php

namespace App\Providers;

use App\Extensions\MongoStore;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->booting(function () {
             Cache::extend('mongo', function (Application $app) {
                 return Cache::repository(new MongoStore);
             });
         });
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        // ...
    }
}
```

传递给 `extend` 方法的第一个参数是驱动的名称。这将对应于 `config/cache.php` 配置文件中的 `driver` 选项。第二个参数是一个闭包，该闭包应该返回一个 `Illuminate\Cache\Repository` 实例。这个闭包将会接收一个 `$app` 参数，这是服务容器的一个实例。

一旦你的扩展注册完成，更新应用程序的 `config/cache.php` 配置文件中的 `CACHE_STORE` 环境变量或默认选项为你的扩展名称。

## 事件

要在每个缓存操作上执行代码，可以监听缓存调度的各种事件：

| Event Name                           | 事件名称           |
|--------------------------------------|--------------------|
| Illuminate\Cache\Events\CacheHit     | 缓存命中           |
| Illuminate\Cache\Events\CacheMissed  | 缓存未命中         |
| Illuminate\Cache\Events\KeyForgotten | 键被忘记           |
| Illuminate\Cache\Events\KeyWritten   | 键已写入           |

为了提高性能，你可以通过在应用程序的 `config/cache.php` 配置文件中将某个缓存存储的 `events` 配置选项设置为 `false` 来禁用缓存事件。

```php
'database' => [
    'driver' => 'database',
    // ...
    'events' => false,
],
```

