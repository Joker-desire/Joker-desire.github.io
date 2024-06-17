# Laravel 11.x 路由


----

[原文](https://laravel.com/docs/11.x/routing)

----

## 基本路由

最基本的路由接收 URI 和闭包，提供了一种非常简单且富有表现力的方法来定义路由和行为，而无需复杂的路由配置文件：

```php
use Illuminate\Support\Facades\Route;

Route::get('/greeting', function () {
    return 'Hello World';
});
```
### 默认路由文件

所有路由都在位于`routes`目录中的路由文件中定义。

这些文件由应用程序`bootstrap/app.php`文件中指定的配置自动加载。

#### web 路由

`routes/web.php`文件定义用于Web界面的路由，这些路由被分配为web中间件组，该组提供会话状态和CSRF保护等功能。

在`routes/web.php`中定义的路由，可以通过在浏览器中输入定义的路由的URL来访问。

例如：通过`http://example.com/user`URL访问以下路由：

```php
use App\Http\Controllers\UserController;
use Illuminate\Support\Facades\Route;

Route::get('/user', [UserController::class, 'index']);
```

#### api 路由

> `install:api` 命令用来生成无状态API，会创建`routes/api.php`文件
>
> 该命令将安装 Laravel Sanctum ，它提供了一个强大而简单的API令牌身份验证防护。

```bash
php artisan install:api
```

```php
use Illuminate\Support\Facades\Request;
use Illuminate\Support\Facades\Route;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```
> `routes/api.php`中的路由是无状态的，并且被分配到`api`中间件组。
>
> 会自动应用`/api`作为URI前缀。
>
> 可以通过`bootstrap/app.php`修改前缀

```php
->withRouting(
    api: __DIR__.'/../routes/api.php',
    apiPrefix: 'api/admin', // 自定义前缀
    // ...
)
```

#### 可用路由方法

```php
Route::get('/get', [UserController::class, 'index']);
Route::post('/post', [UserController::class, 'index']);
Route::put('/put', [UserController::class, 'index']);
Route::patch('/patch', [UserController::class, 'index']);
Route::delete('/delete', [UserController::class, 'index']);
Route::options('/options', [UserController::class, 'index']);
Route::match(['get', 'post'], '/match', [UserController::class, 'index']);
Route::any('/any', [UserController::class, 'index']);
```

#### 依赖注入

可以在路由的回调签名中进行类型提示，以声明路由所需的任何依赖项。被声明的依赖项将由服务容器自动解析并注入到回调中。

例如：可以对`Illuminate\Support\Facades\Request` 类进行类型提示，以便当前的HTTP请求自动注入到路由回调中：

```php
use Illuminate\Support\Facades\Request;
use Illuminate\Support\Facades\Route;

Route::get('/user', function (Request $request) {
    return $request->user(); // 可以使用 $request 对象来访问当前 HTTP 请求
});
```

#### CSRF 保护

在 web 路由文件中定义的指向 POST、PUT、PATCH 或 DELETE 路由的任何 HTML 表单都应包含一个 CSRF 令牌字段。否则，请求将被拒绝。

```html
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```

### 重定向路由

> `Route::redirect` 重定向路由

```php
Route::redirect('/here', '/there');
```

> `Route::redirect` 默认返回状态码是302代码，可以通过第三个参数进行自定义状态代码

```php
Route::redirect('/here', '/there', 301);
```

> `Route::permanentRedirect`重定向路由默认返回301状态代码

```php
Route::permanentRedirect('/here', '/there');
```

### 视图路由

> `Route:view` 视图路由。
>
> 该 `view` 方法接受 URI 作为其第一个参数，并接受视图名称作为其第二个参数。此外，还可以提供要传递给视图的数据数组作为可选的第三个参数：

```php
Route::view('/welcome', 'welcome');
 
Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

### 查看所有定义的路由

> `route:list` 命令可以查看应用程序定义的所有路由

```bash
php artisan route:list
```

> 默认情况下，分配给每个路由的路由中间件不会输出，但是可以通过`- ` 参数来显示路由中间件和中间件组名称

```bash
php artisan route:list -v

# 展开中间件组
php artisan route:list -vv
```

> `--path=xx` 指定显示以给定URI开头的路由

```bash
php artisan route:list --path=api
```

> `--except-vendor`选项来指定隐藏第三方软件包定义的任何路由

```bash
php artisan route:list --except-vendor
```

> `--only-vendor`选项来指定仅显示第三方软件包定义的路由

```bash
php artisan route:list --only-vendor
```

### 定制路由

默认情况，应用程序的路由由`bootstrap/app.php`进行配置和加载：

```php
use Illuminate\Foundation\Application;
 
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )->create();
```

> `then`闭包，用来注册应用程序所需要的任何附加路由

```php
use Illuminate\Support\Facades\Route;
 
->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
    then: function () {
        Route::middleware('api')
            ->prefix('webhooks')
            ->name('webhooks.')
            ->group(base_path('routes/webhooks.php'));
    },
)
```

> `using`闭包，用来完全控制路由注册。
>
> 当传递此参数时，框架将不会注册任何HTTP路由，需要手动注册所有路由

```php
use Illuminate\Support\Facades\Route;
 
->withRouting(
    commands: __DIR__.'/../routes/console.php',
    using: function () {
        Route::middleware('api')
            ->prefix('api')
            ->group(base_path('routes/api.php'));
 
        Route::middleware('web')
            ->group(base_path('routes/web.php'));
    },
)
```

## 路由参数

### 必需参数

> 路由参数适中使用`{}`括起来，并且应由字母字符组成。
>
> 下划线在路由参数名称中也是可以接受的。
>
> 路由参数根据其顺序注入到路由回调/控制器中 - 路由回调/控制器参数的名称无关紧要

```php
Route::get('/user/{id}', function (string $id) {
    return 'User ' . $id;
});

Route::get('posts/{post}/comments/{comment}', function (string $postId, string $commentId) {
    return 'Post ' . $postId . ' Comment ' . $commentId;
});
```

**参数和依赖注入**

依赖注入项应该在路由参数前

```php
Route::get('/user/{id}', function (Request $request, string $id) {
    return 'User ' . $id;
});
```

### 可选参数

> 通过在参数名称后放置标记`?`来表示可选参数。
>
> 需要确保相对于变量提供有默认值，不提供默认值又不传递参数，则会发生异常

```php
Route::get('/user2/{name?}', function (?string $name = null) {
    return $name;
});

Route::get('/user3/{name?}', function (?string $name = 'John') {
    return $name;
});
```

### 正则约束

> 使用路由实例上的方法`where`限制路由参数的格式。
>
> `where`方法接受参数的名称和正则表达式，用于定义参数应如何约束。

```php
Route::get('/user/{name}', function (string $name) {
    // ...
})->where('name', '[A-Za-z]+');

Route::get('/user/{id}', function (string $id) {
    // ...
})->where('id', '[0-9]+');

Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

> Laravel 提供了一些常用的正则表达式模式，可用于快速向路由添加模式约束

```php
Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->whereNumber('id')->whereAlpha('name');
 
Route::get('/user/{name}', function (string $name) {
    // ...
})->whereAlphaNumeric('name');
 
Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUuid('id');
 
Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUlid('id');
 
Route::get('/category/{category}', function (string $category) {
    // ...
})->whereIn('category', ['movie', 'song', 'painting']);
 
Route::get('/category/{category}', function (string $category) {
    // ...
})->whereIn('category', CategoryEnum::cases());
```

### 全局约束

> 使用`pattern` 方法，可以定义路由参数全局约束。
>
> 在`App\Providers\AppServiceProvider`类的`boot`方法中定义这些模式

```php
public function boot(): void
{
    // 定义全局路由参数约束
    Route::pattern('id', '[0-9]+');
}
```

### 编码正斜杠

> Laravel 路由组件允许路由参数值中包含除 / 之外的所有字符。
>
> 如果需要让 / 成为占位符的一部分，必须使用 `where` 条件正则表达式明确允许

```php
Route::get('search/{search}', function (string $search) {
    return $search;
})->where('search', '.*');
```

**注意：仅在最后一个路由段内受支持。**

## 命名路由

> 命名路由允许方便地为特定路由生成URL或进行重定向。
>
> 可以通过将 `name` 方法链接到路由定义上来为路由指定一个名称。

```php
Route::get('/user/profile', function () {
    // ...
})->name('profile');
```

**注意：路由名称应始终是唯一的。**

### 生成命名路由的URL

> 一旦为某个路由指定了名称，就可以在生成URL或进行重定向时，通过使用Laravel的 `route` 和 `redirect` 辅助函数来引用该路由的名称

```php
// Generating URLs...
$url = route('profile');
 
// Generating Redirects...
return redirect()->route('profile');
 
return to_route('profile');
```

> 如果命名路由定义了参数，可以将参数作为第二个参数传递给 `route` 函数。
>
> 传入的参数会自动插入到生成的 URL 中的正确位置

```php
Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');
 
$url = route('profile', ['id' => 1]);
```

> 如果在数组中传递了额外的参数，这些键值对将自动添加到生成的 URL 的查询字符串中：

```php
Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');
 
$url = route('profile', ['id' => 1, 'photos' => 'yes']);
 
// /user/1/profile?photos=yes
```

### 检查当前路由

> 可以在路由实例上使用`named`方法确定当前请求是否路由到给定的命名路由

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class RequestMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response) $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->route()->named('profile')) {
            print_r('Profile route');
        }
        return $next($request);
    }
}
```

## 路由组

路由组允许在大量路由之间共享路由属性（如中间件），而无需在每个单独的路由上定义这些属性。

### 中间件

> `middleware`方法可以将中间件分配给组内的所有路由。
>
> 中间件按它们在数组中列出的顺序执行

```php
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // Uses first & second Middleware
    });
    Route::get('/user', function () {
        // Uses first & second Middleware
    });
});
```

### 控制器

> `controller`方法为组内的所有路由定义公共控制器。
>
> 在定义路由时，只需提供它们调用的控制器方法。

```php
Route::controller(UserController::class)->group(function () {
    Route::get('user', 'show');
    Route::get('user/edit', 'edit');
});
```

### 子域路由

> `domain`指定子域路由。
>
> 子域可以像路由URI一样分配路由参数，允许捕获子域的一部分，以便在路由或控制器中使用。
>
> 注意：为确保子域路由可访问，应该在注册根域路由之前注册子域路由。防止根域路由覆盖具有相同URI路径的子域路由。

```php
Route::domain('{account}.myapp.com')->group(function () {
    Route::get('user/{id}', function (string $account, string $id) {
        //
    });
});
// GET|HEAD       {account}.myapp.com/api/user/{id}
```

### 路由前缀

> `prefix`方法可用于使用给定的URI 为组中的每个路由添加前缀

```php
Route::prefix('admin')->group(function () {
    Route::get('users', function () {
        // Matches The "/admin/users" URL
    });
});
// GET|HEAD       api/admin/users
```

### 路由名称前缀

> `name`方法可用于在组中的每个路由名称前面加上给定的字符串。
>
> 给定的字符串与指定的路由名称完全相同，因此我们确保在前缀中提供尾`.`字符

```php
Route::name('admin.')->group(function () {
    Route::get('users', function () {
        // Route assigned name "admin.users"...
    })->name('users');
});
// GET|HEAD       api/users ..... admin.users
```

## 路由模型绑定

### 隐式绑定

> Laravel 会自动解析在路由或控制器操作中定义的 Eloquent 模型，通过类型提示的变量名匹配路由段名称。

```php
use App\Models\User;
 
Route::get('/users/{user}', function (User $user) {
    return $user->email;
});
```

由于`$user`变量被类型提示为Eloquent模型，并且变量名称与`{user}`URI段匹配，因此，Laravel将自动注入具有与请求URI中相应值匹配的 ID 的模型实例。

如果在数据库中找不到匹配的模型实例，则会自动生成 404 HTTP 响应。

> 在使用控制器方法时，隐式绑定也是可用的

```php
use App\Http\Controllers\UserController;
use App\Models\User;
 
// Route definition...
Route::get('/users/{user}', [UserController::class, 'show']);
 
// Controller method definition...
public function show(User $user)
{
    return view('user.profile', ['user' => $user]);
}
```

#### 软删除模型

> `withTrashed`方法链接到路由的定义来指示隐式绑定检索已软删除的模式。

```php
use App\Models\User;
 
Route::get('/users/{user}', function (User $user) {
    return $user->email;
})->withTrashed();
```

#### 自定义键

> 有时希望使用除`id`以外的列来解析Eloquent模型。可以在路由参数定义中指定该列

```php
use App\Models\Post;
 
Route::get('/posts/{post:slug}', function (Post $post) {
    return $post;
});
```

>  如果希望模型绑定始终使用数据库列，但不是`id`，则可以在Eloquent模型上重写`getRouteKeyName`方法。

```php
/**
 * Get the route key for the model.
 */
public function getRouteKeyName(): string
{
    return 'slug';
}
```

#### 自定义键和作用域

当单个路由定义中隐式绑定多个Eloquent模型时，可能希望将第二个Eloquent模型的范围限定为必须是前一个Eloquent模型的子级。

例如：考虑以下这个根据用户特定的slug来检索博客文章的路由定义：

```php
use App\Models\Post;
use App\Models\User;
 
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
});
```

在将自定义键隐式绑定作为嵌套路由参数使用时，Laravel 会自动将查询范围限定在通过父模型来检索嵌套模型。Laravel 使用约定来猜测父模型上的关系名称。在这种情况下，将假设 `User` 模型有一个名为 `posts`（路由参数名称的复数形式）的关系，可以用来检索 `Post` 模型。

还可以指示 Laravel 限定“子”绑定的范围，即使未提供自定义键也是如此。为此，可以在定义路由时调用 `scopeBindings` 方法：

```php
use App\Models\Post;
use App\Models\User;
 
Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
})->scopeBindings();
```

还可以指示整个路由定义组使用作用域绑定：

```php
Route::scopeBindings()->group(function () {
    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    });
});
```

同样，可以通过调用 `withoutScopedBindings` 方法明确指示 Laravel 不限定绑定的范围：

```php
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
})->withoutScopedBindings();
```

#### 自定义缺失的模型行为

通常如果未找到隐式绑定的模型，则会生成 404 HTTP 响应。

可以通过在定义路由时调用`missing`方法来自定义此行为。

`missing`方法接受一个闭包，如果找不到隐式绑定的模型，则将调用该闭包

```php
use App\Http\Controllers\LocationsController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;
 
Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
        ->name('locations.view')
        ->missing(function (Request $request) {
            return Redirect::route('locations.index');
        });
```

#### 隐式枚举绑定

PHP 8.1 引入了对枚举（Enums）的支持。为了补充这一功能，Laravel 允许在路由定义中对一个有值的枚举进行类型提示，且 Laravel 仅在该路由段对应一个有效的枚举值时才会调用该路由。否则，将自动返回 404 HTTP 响应。

```php
<?php
 
namespace App\Enums;
 
enum Category: string
{
    case Fruits = 'fruits';
    case People = 'people';
}
```

```php
use App\Enums\Category;
use Illuminate\Support\Facades\Route;
 
Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
});
```

### 显示绑定

要注册显示绑定，使用路由器的model方法为给定参数指定类。

应该在AppServiceProvider 类的 boot 方法的开头定义显式模型绑定。

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;
 
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Route::model('user', User::class);
}
```

```php
use App\Models\User;
 
Route::get('/users/{user}', function (User $user) {
    // ...
});
```

#### 自定义解析逻辑

如果希望定义自己的模型绑定解析逻辑，可以使用`Route:bind`方法。

传递给 bind 方法的闭包将接收 URI 段的值，并应返回应注入到路由中的类的实例。

同样，这种自定义应该在应用程序的 AppServiceProvider 类的 boot 方法中进行。

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;
 
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Route::bind('user', function (string $value) {
        return User::where('name', $value)->firstOrFail();
    });
}
```

也可以在Eloquent 模型上重写 `resolveRouteBinding` 方法。这个方法将接收 URI 段的值，并应返回应注入到路由中的类的实例。

```php
/**
 * Retrieve the model for a bound value.
 *
 * @param  mixed  $value
 * @param  string|null  $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveRouteBinding($value, $field = null)
{
    return $this->where('name', $value)->firstOrFail();
}
```

如果路由使用隐式绑定作用域，将使用 `resolveChildRouteBinding` 方法来解析父模型的子绑定。

```php
/**
 * Retrieve the child model for a bound value.
 *
 * @param  string  $childType
 * @param  mixed  $value
 * @param  string|null  $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveChildRouteBinding($childType, $value, $field)
{
    return parent::resolveChildRouteBinding($childType, $value, $field);
}
```

## 回退路由

使用 `Route::fallback` 方法，可以定义一个在没有其他路由匹配传入请求时执行的路由。

通常，未处理的请求将通过应用程序的异常处理程序自动呈现一个 "404" 页面。

然而，由于会在 routes/web.php 文件中定义回退路由，因此 web 中间件组中的所有中间件都将应用于该路由。

也可以根据需要向该路由添加额外的中间件。

```php
Route::fallback(function () {
    // ...
});
```

**📢注意：回退路由应始终是应用程序注册的最后一个路由。**

## 速率限制

### 定义速率限制器

Laravel包含强大且可定制的速率限制服务，可以利用它们来限制给定路由或路由组的流量量。

要开始使用，应该定义满足应用程序需求的速率限制器配置。

速率限制器可以在应用程序的 `App\Providers\AppServiceProvider` 类的 `boot` 方法中定义。

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
 
/**
 * Bootstrap any application services.
 */
protected function boot(): void
{
    RateLimiter::for('api', function (Request $request) {
        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
    });
}
```

速率限制器是使用 `RateLimiter` Facade `for` 的方法定义的。

`for` 方法接受一个速率限制器名称和一个返回应用于分配给速率限制器的路由的限制配置的闭包。

限制配置是 `Illuminate\Cache\RateLimiting\Limit` 类的实例。

该类包含有用的 "构建器" 方法，因此可以快速定义限制。

速率限制器名称可以是任何字符串。

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
 
/**
 * Bootstrap any application services.
 */
protected function boot(): void
{
    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000);
    });
}
```

如果传入的请求超过了指定的速率限制，Laravel 将自动返回一个带有 429 HTTP 状态码的响应。如果希望定义自己的响应来处理速率限制，可以使用 `response` 方法。

```php
RateLimiter::for('global', function (Request $request) {
    return Limit::perMinute(1000)->response(function (Request $request, array $headers) {
        return response('Custom response...', 429, $headers);
    });
});
```

由于速率限制器回调接收传入的 HTTP 请求实例，可以根据传入的请求或经过身份验证的用户动态地构建适当的速率限制。

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->vipCustomer()
                ? Limit::none()
                : Limit::perMinute(100);
});
```

#### 分段速率限制

可以在构建速率限制时使用`by`方法对速率限制进行分段.

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->vipCustomer()
                ? Limit::none()
                : Limit::perMinute(100)->by($request->ip());
});
```

例：限制对路由的访问，每分钟对经过身份验证的用户ID限制为100次，对于未登录的访客，每分钟限制为10次

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()
                ? Limit::perMinute(100)->by($request->user()->id)
                : Limit::perMinute(10)->by($request->ip());
});
```

#### 多个速率限制

可以为给定的速率限制器配置返回一个速率限制数组。每个速率限制将根据它们在数组中的顺序进行路由评估。

```php
RateLimiter::for('login', function (Request $request) {
    return [
        Limit::perMinute(500),
        Limit::perMinute(3)->by($request->input('email')),
    ];
});
```

### 将速率限制器附加到路由

可以使用 `throttle` 中间件将速率限制器附加到路由或路由组。

`throttle` 中间件接受分配给路由的速率限制器的名称。

```php
Route::middleware(['throttle:uploads'])->group(function () {
    Route::post('/audio', function () {
        // ...
    });
 
    Route::post('/video', function () {
        // ...
    });
});
```

#### 使用Redis进行节流

默认情况下，`throttle` 中间件映射到 `Illuminate\Routing\Middleware\ThrottleRequests` 类。

然而，如果使用 Redis 作为应用程序的缓存驱动程序，并且指示 Laravel 使用 Redis 来管理速率限制，可以在应用程序的 `bootstrap/app.php` 文件中使用 `throttleWithRedis` 方法。

这个方法将 `throttle` 中间件映射到 `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis` 中间件类。

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->throttleWithRedis();
    // ...
})
```

## 表单方法欺骗

HTML表单不支持PUT、PATCH或DELETE操作。因此，当定义从HTML表单调用的PUT、PATCH或DELETE路由时，需要向表单添加一个隐藏的\_method字段。发送的\_method字段的值将被用作HTTP请求方法。

```php
<form action="/example" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```

为了方便起见，可以使用 Blade 指令 `@method` 来生成 `_method` 输入字段。

```php
<form action="/example" method="POST">
    @method('PUT')
    @csrf
</form>
```

## 访问当前路由

可以使用 `Route` Facade 上的 `current` 、 `currentRouteName` 和 `currentRouteAction` 方法来访问有关处理传入请求的路由的信息

```php
use Illuminate\Support\Facades\Route;
 
$route = Route::current(); // Illuminate\Routing\Route
$name = Route::currentRouteName(); // string
$action = Route::currentRouteAction(); // string
```

## 跨域资源共享（CORS）

Laravel可以自动使用您配置的值响应CORS OPTIONS HTTP请求。

OPTIONS请求将由HandleCors中间件自动处理，该中间件已自动包含在应用程序的全局中间件堆栈中。

有时，需要自定义应用程序的CORS配置值。可以使用 `config:publish` Artisan命令来发布CORS配置文件。

```bash
php artisan config:publish cors
```

此命令会将 `cors.php` 配置文件放在应用程序 `config` 的目录中。

## 路由缓存

在将应用程序部署到生产环境时，应该利用 Laravel 的路由缓存。

使用路由缓存可以大幅减少注册所有应用程序路由所需的时间。

要生成路由缓存，可以执行 `route:cache` Artisan 命令。

```bash
php artisan route:cache
```

运行此命令后，缓存路由文件将在每个请求时加载。

请记住，如果添加了任何新路由，需要生成新的路由缓存。

因此，在项目部署期间应该只运行 `route:cache` 命令。

使用以下 `route:clear` 命令清除路由缓存：

```bash
php artisan route:clear
```

