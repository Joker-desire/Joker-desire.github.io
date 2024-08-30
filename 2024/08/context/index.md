# Laravel 11.x Context


----

[原文](https://laravel.com/docs/11.x/context)

----

## 介绍

Laravel 的'上下文'功能使你能够在应用程序内捕获、检索和共享信息，这些信息可以贯穿于请求、作业和命令的执行过程中。这些捕获的信息也会包含在应用程序写入的日志中，使你能够更深入地了解在日志条目写入之前发生的代码执行历史，并允许你在分布式系统中跟踪执行流程。

### 它是如何运作的

了解Laravel上下文功能最佳方法是使用内置日志记录功能来查看它的实际情况。

首先，使用`Context`Facade将信息添加到上下文中；例如：在中间件中将请求url和唯一traceID添加到每个请求上下文中：

```php
<?php
 
namespace App\Http\Middleware;
 
use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;
 
class AddContext
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        Context::add('url', $request->url());
        Context::add('trace_id', Str::uuid()->toString());
 
        return $next($request);
    }
}
```

添加到上下文中的信息会自动作为元数据附加到整个请求期间写入的任何日志条目中。将上下文作为元数据附加允许将传递给单个日志条目的信息与通过上下文共享的信息区分开来。例如，假设我们写入以下日志条目：

```php
Log::info('User authenticated.', ['auth_id' => Auth::id()]);
```

写入的日志将包含传递给日志条目的 `auth_id`，但它还会包含上下文的 `url` 和 `trace_id` 作为元数据：

```
User authenticated. {"auth_id":27} {"url":"https://example.com/login","trace_id":"e04e1a11-e75c-4db3-b5b5-cfef4ef56697"}
```

添加到上下文中的信息也会提供给被派发到队列中的作业。例如，假设我们在添加一些信息到上下文后派发一个 `ProcessPodcast` 作业到队列：

```php
// In our middleware...
Context::add('url', $request->url());
Context::add('trace_id', Str::uuid()->toString());
 
// In our controller...
ProcessPodcast::dispatch($podcast);
```

当作业被派发时，当前存储在上下文中的所有信息都会被捕获并与作业共享。这些捕获的信息在作业执行期间会重新注入到当前上下文中。因此，如果我们的作业的 `handle` 方法写入日志：

```php
class ProcessPodcast implements ShouldQueue
{
    use Queueable;
 
    // ...
 
    /**
     * Execute the job.
     */
    public function handle(): void
    {
        Log::info('Processing podcast.', [
            'podcast_id' => $this->podcast->id,
        ]);
 
        // ...
    }
}
```

生成的日志条目将包含在最初派发作业的请求期间添加到上下文中的信息：

```
Processing podcast. {"podcast_id":95} {"url":"https://example.com/login","trace_id":"e04e1a11-e75c-4db3-b5b5-cfef4ef56697"}
```

## 捕捉上下文

可以使用`Context`Facade的`add`方法在当前上下文中存储信息：

```php
use Illuminate\Support\Facades\Context;
 
Context::add('key', 'value');
```

也可以将关联数组传递给`add`方法，以便一次性添加多个项目：

```php
Context::add([
    'first_key' => 'value',
    'second_key' => 'value',
]);
```

`add`方法将覆盖相同键的值。如果指向在键不存在的情况下新增，则可以使用`addIf`方法：

```php
Context::add('key', 'first');
 
Context::get('key');
// "first"
 
Context::addIf('key', 'second');
 
Context::get('key');
// "first"
```

### 条件上下文

`when`方法可用于根据给定条件将数据添加到上下文。如果给定的条件结果为`true`，则将调用提供给`when`方法的第一个闭包，而如果条件计算结果为`false`，则将调用第二个闭包：

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Context;
 
Context::when(
    Auth::user()->isAdmin(),
    fn ($context) => $context->add('permissions', Auth::user()->permissions),
    fn ($context) => $context->add('permissions', []),
);
```

### 堆栈

Context提供了创建“堆栈”的能力，“堆栈”是按添加顺序存储的数据列表。可以通过调用`push`方法向堆栈添加信息：

```php
use Illuminate\Support\Facades\Context;
 
Context::push('breadcrumbs', 'first_value');
 
Context::push('breadcrumbs', 'second_value', 'third_value');
 
Context::get('breadcrumbs');
// [
//     'first_value',
//     'second_value',
//     'third_value',
// ]
```

堆栈对于捕获请求的历史信息非常有用，例如应用程序中发生的事件。例如，你可以创建一个事件监听器，在每次执行查询时将信息压入堆栈，捕获查询的 SQL 和执行的时间作为一个元组：

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Facades\DB;
 
DB::listen(function ($event) {
    Context::push('queries', [$event->time, $event->sql]);
});
```

可以使用`stackContains`和`hiddenStackContains`方法确定某个值是否在堆栈中：

```php
if (Context::stackContains('breadcrumbs', 'first_value')) {
    //
}
 
if (Context::hiddenStackContains('secrets', 'first_value')) {
    //
}
```

`stackContains`和`hiddenStackContains`方法还接受闭包作为其第二个参数，从而允许对值比较操作进行更多控制：

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Str;
 
return Context::stackContains('breadcrumbs', function ($value) {
    return Str::startsWith($value, 'query_');
});
```

## 检索上下文

使用`Context`的`get`方法从上下文中检索信息：

```php
use Illuminate\Support\Facades\Context;
 
$value = Context::get('key');
```

`only`方法可用于检索上下文中的信息子集：

```php
$data = Context::only(['first_key', 'second_key']);
```

`pull`方法可用于从上下文中检索信息并立即将其从上下文中删除：

```php
$value = Context::pull('key');
```

`all`检索存储在上下文中的所有信息：

```php
$data = Context::all();
```

### 确定项目是否存在

`has`方法用来确定上下文是否存在给定键：

```php
use Illuminate\Support\Facades\Context;
 
if (Context::has('key')) {
    // ...
}
```

无论值是什么，`has`都会返回`true`。因此，`null`值也将被视为存在

## 删除上下文

`forget`方法用于从当前上下文中删除键值：

```php
use Illuminate\Support\Facades\Context;
 
Context::add(['first_key' => 1, 'second_key' => 2]);
 
Context::forget('first_key');
 
Context::all();
 
// ['second_key' => 2]
```

也可以向`forget`方法提供一个数组，一次性删除多个键值：

```php
Context::forget(['first_key', 'second_key']);
```

## 隐藏上下文

Context提供了存储“隐藏”数据的English。次隐藏信息不回附加到日志中，并且无法通过上面记录的数据检索方法访问。Context提供了一组不同的方法来与隐藏的上下文信息进行交互：

```php
use Illuminate\Support\Facades\Context;
 
Context::addHidden('key', 'value');
 
Context::getHidden('key');
// 'value'
 
Context::get('key');
// null
```

‘隐藏’方法的功能与上面文档中记录的非隐藏方法相同：

```php
Context::addHidden(/* ... */);
Context::addHiddenIf(/* ... */);
Context::pushHidden(/* ... */);
Context::getHidden(/* ... */);
Context::pullHidden(/* ... */);
Context::onlyHidden(/* ... */);
Context::allHidden(/* ... */);
Context::hasHidden(/* ... */);
Context::forgetHidden(/* ... */);
```

## Events

上下文派发了两个事件，这些事件允许你钩入上下文的注入和释放过程。

为了说明如何使用这些事件，假设在应用程序的一个中间件中，你根据传入 HTTP 请求的 Accept-Language 头设置 `app.locale` 配置值。上下文的事件允许你在请求期间捕获这个值并在队列上恢复它，从而确保在队列上发送的通知具有正确的 `app.locale` 值。我们可以使用上下文的事件和隐藏数据来实现这一点，下面的文档将对此进行说明。

### 释放

每当一个作业被派发到队列时，上下文中的数据会被'释放'并与作业的有效负载一起捕获。`Context::dehydrating` 方法允许你注册一个闭包，该闭包将在释放过程中被调用。在这个闭包中，你可以对将与队列作业共享的数据进行修改。

通常，你应该在应用程序的 `AppServiceProvider` 类的 `boot` 方法中注册 `dehydrating` 回调：

```php
use Illuminate\Log\Context\Repository;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Context;
 
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Context::dehydrating(function (Repository $context) {
        $context->addHidden('locale', Config::get('app.locale'));
    });
}
```

***ps：你不应该在 `dehydrating` 回调中使用 `Context` facade，因为那会改变当前进程的上下文。确保你只对传递给回调的仓库进行更改。***

### 注入

每当一个队列作业开始在队列上执行时，所有与作业共享的上下文将被重新‘注入’到当前上下文中。`Context::hydrated` 方法允许你注册一个闭包，该闭包将在注入过程中被调用。

通常，你应该在应用程序的 `AppServiceProvider` 类的 `boot` 方法中注册 `hydrated` 回调：

```php
use Illuminate\Log\Context\Repository;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Context;
 
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Context::hydrated(function (Repository $context) {
        if ($context->hasHidden('locale')) {
            Config::set('app.locale', $context->getHidden('locale'));
        }
    });
}
```

***ps：你不应该在 `hydrated` 回调中使用 `Context` facade，而应确保只对传递给回调的仓库进行更改。***

