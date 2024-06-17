# Laravel 路由


# 基本路由

## 基本路由

>  最基本的`Laravel`路由仅接受URI和一个闭包：

```php
Route::get("foo", function () {
    return 'Hello PHP';
});
```

### 可用的路由器方法

> 路由器允许注册响应任何HTTP动词的路由

```php
Route::get(uri, $callback);
Route::post(uri, $callback);
Route::put(uri, $callback);
Route::patch(uri, $callback);
Route::delete(uri, $callback);
Route::options(uri, $callback);
```

> 在Laravel中可以定义响应多个HTTP动作的路由

```php
/**
 * 注册可响应多个HTTP动作的路由
 */
Route::match(['get', 'post'], 'match', function () {
    return '使用match定义可响应多个HTTP动作的路由';
});

/**
 * 注册可响应全部HTTP动作的路由
 */
Route::any('any', function () {
    return '使用any定义全部HTTP动作的路由';
});
```
### 依赖注入

> 可以在路由的回调签名中键入提示路由所需要的任何依赖项。
>
> 声明的依赖项将自动解析，并由Laravel服务容器注入回调中。

```php
Route::get('/api', function (Request $request) {
    Log::info("api接口请求");
    print_r($request->all());
    return 'api接口请求';
});
```

## 重定向路由

> `Route::redirect`方法提供了一个方便快捷的方式进行重定向到另一个URI的路由

```php
Route::redirect('/here', '/api/first');
```

> 默认情况下，返回了`302`状态码，可以使用可选参数进行自定义状态码

```php
Route::redirect('/here', '/api/first', 301);
```

> 或者可以使用`Route::permanentRedirect`，此方法返回`301`状态码

```php
Route::permanentRedirect('/here2', '/api/api');
```



## 路由列表

```shell
# 查看全部路由
php artisan route:list

# 通过-v选项来显示路由中间件
php artisan route:list -v

# 通过 --path 选项仅显示给定URI开头的路由
php artisan route:list --path=api

# 通过 --except-vendor 选项隐藏第三方软件包定义的任何路由
php artisan route:list --except-vendor

# 通过 --only-vendor 选项只显示第三方软件包定义的任何路由
php artisan route:list --only-vendor
```

## 路由参数

### 必需参数

> 在路由中捕获URI的段
>
> 路由参数适中包含在`{}`大括号中，并应由字母字符组成，下划线`_`在路由参数名称中也是可以的。
>
> 路由参数根据其顺序注入路由回调/控制器，参数的名称无关紧要。

```php
// 从URL中获取用户ID
Route::get('/user/{id}', function (string $id) {
    return 'User ' . $id;
});

// URL：http://127.0.0.1:8000/api/user/1
// 结果：User 1 

Route::get('/posts/{post}/comments/{comment}', function (string $postId, string $commentId) {
    return 'Post id -> ' . $postId . ' Comment id -> ' . $commentId;
});
// url: http://127.0.0.1:8000/api/posts/1001/comments/3000
// 结果：Post id -> 1001 Comment id -> 3000
```

### 参数和依赖注入

> 如果路由具有依赖项，并希望Laravel服务容器自动注入路由的回调中，应该在依赖项之后列出路由参数

```php
use Illuminate\Http\Request;

Route::get('/user/{id}', function (Request $request, string $id) {
    return 'User ' . $id;
});
```

### 可选参数

> 通过放置一个`?`在参数名称后标记，则可设置一个可选参数。
>
> 注：确保给路由的相应变量一个默认值。

```php
Route::get('/user/{name?}', function (string $name = null) {
    return $name;
});

Route::get('/user/{name?}', function (string $name = 'Joker') {
    return $name;
});
```

### 正则表达式约束

> 使用路由实例上的`where`方法限制路由参数的格式
>
> `where`方法接受参数的名称和定义如何约束参数的正则表达式
>
> 如果传入的请求与路由模式约束不匹配，将返回`404`HTTP响应

```php
Route::get('/user/{name}', function (string $name) {
    // ...
})->where('name', '[A-Za-z]+');

Route::get('/user/{id}', function (int $id) {
    //...
})->where('id', '[0-9]+');

Route::get('/user/{id}/{name}', function (int $id, string $name) {
    // ...
})->where(['id' => '[0-9]+', 'name' => '[A-Za-z]+']);
```

> 为了方便起见，一些常用的正则表达式具有辅助方法，允许快速向路由添加模式约束

```php

/**
 * whereAlphaNumeric: 指定给定的路由参数必须是字母数字
 */
Route::get('/user/{name}', function (string $name) {
    // ...
})->whereAlphaNumeric('name');

/**
 * whereNumber: 指定给定的路由参数必须是数字
 */
Route::get('/user/{id}', function (int $id) {
    //...
})->whereNumber('id');

/**
 * 使用多个参数约束条件
 */
Route::get('/user/{id}/{name}', function (int $id, string $name) {
    // ...
})->whereNumber('id')->whereAlpha('name');

/**
 * whereUuid: 指定给定的路由参数必须是UUlDs。
 */
Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUuid('id');

/**
 * whereUlid: 指定给定的路由参数必须是ULIDs。
 */
Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUlid('id');

/**
 * whereIn: 指定给定的路由参数必须是给定值之一
 */
Route::get('/category/{category}', function (string $category) {
    // ...
})->whereIn('category', ['movie', 'song', 'painting']);

```

#### 全局约束条件

> 使用`patten`方法可以定义全局约束条件。
>
> 在`app/Providers/RouteServiceProvider.php`文件中的`RouteServiceProvider`类的`boot`方法中定义这些全局约束条件模式

```php
public function boot(): void
    {
        // 定义全局约束条件
        Route::pattern('id', '[0-9]+');
        // ...
    }
```

> 一旦定义了全局模式，将使用参数名称的自动应用于所有路由

```php
Route::get('/user/{id}', function (string $id) {
    return 'userid -> ' . $id;
});
```

###  编码的正斜杠

> `Laravel`路由组件允许除`/`之外的所有字符都在存在于路由参数值中。
>
> 必须使用`where`条件正则表达式明确允许`/`成为占位符的一部分。
>
> 注：编码的正斜杠仅在最后一个路由段内受支持。

```php
Route::get('/search/{search}', function (string $search) {
    return 'Search -> ' . $search;
})->where('search', '.*');

// URL：http://127.0.0.1:8000/api/search/Joker/19/Shenzhen
// 结果：Search -> Joker/19/Shenzhen
```

## 命名路线

> 命名路由允许为特定路由方便地生成URL或重定向。
>
> 通过`name`方法链接到路由定义上，可以指定路由的名称。
>
> 注：路由名称应始终是唯一的。

```php
Route::get('/user/profile', function () {
    // ...
})->name('profile');
```

### 生成命名路由的URL

一旦为给定的路由分配一个名字，可以通过`Laravel`的`route`和`redirect`辅助函数生成URL或重定向时使用该路由的名称：

```php
Route::get('/user/create', function () {
    // 生成完整的URL
    $url = route('profile');
    log::debug($url) // http://127.0.0.1:8000/api/user/profile
    // 生成重定向(两种方式)
	// return redirect()->route('profile');
    return to_route('profile');
});
```

如果命名路由定义了参数，可以将参数作为第二个参数传递给`route`函数。给定的参数将自动插入到生成的URL的正确位置：

```php
Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');


// 生成完整的URL
$url = route('profile', ['id' => 1]);
log::debug($url); // http://127.0.0.1:8000/api/user/1/profile
```

如果在数组中传递其他参数，这些键值对将自动添加到生成的URL的查询字符串中：

```php
$url = route('profile', ['id' => 1, 'photos' => 'yes']);
log::debug($url); // http://127.0.0.1:8000/api/user/1/profile?photos=yes
```

### 检查当前路由

如果想确定当前请求是否路由到给定的命名路由，可以在`Route`实例上使用`named`方法。例如，可以在路由中间件检查当前路由名称：

```php
if ($request->route()->named('profile')){
    // ...
}
```

## 路由组

路由组允许共享路由属性。例如中间件，而无需在每个独立的路由上定义这些属性。

嵌套组尝试智能地将属性与其父组"合并"。中间件和`where`条件合并，同时附加名称和前缀。URL前缀中的命名空间分隔符和斜杠会在适当的地方自动添加。

### 路由中间件

> 要将中间件分配给组内的所有路由，可以在定义组之前使用`middleware`方法。
>
> 中间件按照它们在数组中列出的顺序执行

```php
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // ...
    });
    Route::get('/user/profile', function () {
        // ...
    });
});
```

### 控制器

> 如果一组路由都使用相同的控制器，可以使用`controller`方法为组内的所有路由定义公共控制器。然后再定义路由时，只需要提供他们调用的控制器方法

```php
use \App\Http\Controllers\Api\OrderController;

Route::controller(OrderController::class)->group(function () {
    Route::get('/order/{id}', 'show');
    Route::get('/orders', 'store');
});
```

### 路由前缀

> `prefix`方法可以用给定的URI为组中的每个路由做前缀。

```php
Route::prefix('admin')->group(function () {
    Route::get('/users', function () {
        // 对应 '/admin/users' 的URL
    });
});
```

### 路由名称前缀

> `name`方法可以给定字符串作为组中的每个路由名的前缀
>
> 建议要提供末尾`.`字符在前缀中

```php
Route::name('admin.')->group(function () {
    Route::get('/users', function () {
        // 被分配的路由名为：'admin.users'
    })->name('users');
});
```





# 中间件

## 定义中间件

> 可以使用`make:middleware` Artisan命令

```bash
php artisan make:middleware EnsureTokenIsValid
```

此命令将在项目的 `app/Http/Middleware` 目录中放置一个新的 `EnsureTokenIsValid` 类。

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
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        // 判断传入的token是否一致
        if ($request->input('token') !== 'my-secret-token') {
            return redirect('home') // 如果不一致，则重定向到home路由
        }
        return $next($request);
    }
}

```

## 中间件和相应

> 中间件可以在请求更深入地传递到应用程序之前或之后执行任务

**请求之前执行任务**

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
     * @param \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response) $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        // 执行操作
        return $next($request);
    }
}

```

**请求后执行任务**

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
 
         // 执行操作
 
         return $response;
     }
 }
 
 ```

## 注册中间件

### 全局中间件

> 如果希望在对应程序的每个HTTP请求期间运行中间件，需要在`app/Http/Kernel.php`类的`$middleware`属性中列出中间件类。

```php
class Kernel extends HttpKernel
{
    /**
     * The application's global HTTP middleware stack.
     *
     * These middleware are run during every request to your application.
     *
     * @var array<int, class-string|string>
     */
    protected $middleware = [
        // \App\Http\Middleware\TrustHosts::class,
        \App\Http\Middleware\TrustProxies::class,
        \Illuminate\Http\Middleware\HandleCors::class,
        \App\Http\Middleware\PreventRequestsDuringMaintenance::class,
        \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
        \App\Http\Middleware\TrimStrings::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
    ];
```

### 将中间件分配给路由

> 如果要将中间件分配给特定路由，可以在定义路由时调用`middleware`方法

```php
use \App\Http\Middleware\BeforeMiddleware;

Route::get('/profile', function () {
    // ...
})->middleware(BeforeMiddleware::class);
```

> 通过向`middleware`方法传递一组中间件名称，可以为路由分配多个中间件

```php
use \App\Http\Middleware\BeforeMiddleware;
use \App\Http\Middleware\AfterMiddleware;

Route::get('/profile', function () {
    // ...
})->middleware([BeforeMiddleware::class, AfterMiddleware::class]);
```

### 定义中间件别名

> 为了方便起见，可以在应用程序的`app/Http/Kernel.php`文件中为中间件分配别名

```php
protected $middlewareAliases = [
    ...
    // 自定义中间价
    'before' => \App\Http\Middleware\BeforeMiddleware::class,
    'after' => \App\Http\Middleware\AfterMiddleware::class
];
```

在路由中使用中间件分配的别名

```php
Route::get('/profile', function () {
    // ...
})->middleware(['before', 'after']);
```

### 排除中间件

> 当将中间件分配给一组路由时，可能偶尔需要防止中间件应用于组内的单个路由。
>
> 可以使用`withoutMiddleware`方法完成此操作。

```ph
use \App\Http\Middleware\EnsureTokenIsValid;

Route::middleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/', function () {
        // ...
    });
    Route::get('/profile', function () {
        // ...
    })->withoutMiddleware([EnsureTokenIsValid::class]);
});
```

> 也可以从整个组路由定义中排除一组给定的中间件

```php
use \App\Http\Middleware\EnsureTokenIsValid;

Route::middleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/', function () {
        // ...
    });
    Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
        Route::get('/login', function () {
            // ...
        });
    });
});
```

### 中间件组

> 如果希望将多个中间件组合在一个键下，以使它们更容易分配给路由。可以使用HTTP内核的`$middlewareGroups`属性来完成此操作。
>
> Laravel包括预定义带有`web`和`api`中间件组，其中包含肯呢个希望应用于web和api路由的常见中间件。这些中间件组会由应用程序的`App\Providers\RouteServiceProvider`服务提供者自动应用于相应的`web`和`api`路由文件中的路由

```php
/**
 * The application's route middleware groups.
 *
 * @var array<string, array<int, class-string|string>>
 */
protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        //            \App\Http\Middleware\VerifyCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

    'api' => [
        // \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class . ':api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];
```

> 中间件组可以使用与单个中间件相同的语法分配给路由和控制器动作。同理，中间件组使一次将多个中间件分配给一个路由更加方便

```php
Route::get('/', function () {
    // ...
})->middleware('web');

Route::middleware(['web'])->group(function () {
    // ...
});
```

### 排序中间件

> 在特定情况下，可能需要中间件以特定的顺序执行，但当它们分配到路由时，是无法控制它们的顺序的。这种情况下，可以使用到`app/Http/Kernel.php`文件的`$middlewarePriority`属性指定中间件优先级。
>
> 注：默认情况下，HTTP内核中可能不存在此属性。如果不存在则新增即可。

```php
/**
 * 中间件的优先级排序列表
 *
 * 这迫使非全局中间件始终处于给定的顺序
 *
 * @var string[]
 */
protected $middlewarePriority = [
    \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
    \Illuminate\Cookie\Middleware\EncryptCookies::class,
    \Illuminate\Session\Middleware\StartSession::class,
    \Illuminate\View\Middleware\ShareErrorsFromSession::class,
    \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
    \Illuminate\Routing\Middleware\ThrottleRequests::class,
    \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
    \Illuminate\Contracts\Session\Middleware\AuthenticatesSessions::class,
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
    \Illuminate\Auth\Middleware\Authorize::class,
];
```

### 中间件参数

> 中间件也可以接收额外的参数，额外的中间件参数将在`$next`参数之后传递给中间件

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
    public function handle(Request $request, Closure $next, string $role): Response
    {
        if (!$request->user()->hasRole($role)) {
            // 重定向...
        }
        return $next($request);
    }
}

```

> 在定义路由时，可以指定中间件参数，方法是使用`:`分割中间件名称和参数。多个参数应以逗号分隔

```php
Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware('role:editor');
```

### 可终止的中间件

> 部分情况下，在将HTTP响应发送到浏览器之后，中间件可能需要做一些工作。
>
> 如果在中间件上定义了一个`terminate`方法，并且web服务器使用`FastCGI`，则在将响应发送到浏览器后会自动调用`terminate`方法

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class TerminatingMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response) $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        return $next($request);
    }

    /**
     * 在响应发送到浏览器后处理任务
     *
     * @param Request $request
     * @param Response $response
     * @return void
     */
    public function terminate(Request $request, Response $response): void
    {
        //...
    }
}

```

> `terminate`方法应该同时接收请求和响应。一旦定义了一个可终止的中间件，应该将它添加到`app/Http/Kernel.php`文件中的路由或全局中间件列表中。

> 当在中间件上调用`terminate`方法时，Laravel会从服务器容器解析一个新的中间件实例。如果想在调用`handle`和`terminate`方法时使用相同的中间件实例，可以使用容器的`singleton`方法向容器注册中间件。通常应该在项目的`AppServiceProvider`的`register`方法中完成

```php
<?php

namespace App\Providers;

use App\Http\Middleware\TerminatingMiddleware;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        //
        $this->app->singleton(TerminatingMiddleware::class);
    }

    ...
}

```

# 控制器

控制器可以将相关的请求处理逻辑分组到一个类中。

例如：一个`UserController`类可能会处理所有与用户相关的传入请求，包括显示、创建、更新和删除用户。默认情况下，控制器存储在`app/Http/Controller`目录中。

## 编写控制器

### 基本控制器

> 可以使用`make:controller`Artisan命令快速生成新控制器

```shell
php artisan make:controller UserController
```

> 默认情况下生成在`app/Http/Controller`目录中，可以通过指定路径子目录`Api/UserController`进行生成，这样生成后的目录在`app/Http/Controller/Api`目录下

```shell
php artisan make:controller Api/UserController
```

**示例**

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class UserController extends Controller
{
    public function show(string $id): string
    {
        return 'User' . $id;
    }
}

```

定义到控制器方法的路由：

```php
use App\Http\Controllers\Api\UserController;

Route::get('/user/{id}', [UserController::class, 'show']);

```

### 单动作控制器

如果控制器动作特别复杂，可能会发现将整个控制器专用于该单个动作很方便。为此，可以在控制器中定义一个`__invoke`方法。

> 可以使用`make:controller`Artisan命令的`--invokable`选项生成可调用控制器

```php
php artisan make:controller ProvisionServer --invokable
```

**生成的控制器：**

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class ProvisionServer extends Controller
{
    /**
     * Handle the incoming request.
     */
    public function __invoke(Request $request): array
    {
        //
        return $request->all();
    }
}

```

为单动作控制器注册路由时，不需要指定控制器方法。可以简单地将控制器的名称传递给路由器：

```php
use App\Http\Controllers\Api\ProvisionServer;

Route::get('/server', ProvisionServer::class);

```

## 控制器中间件

中间件可以在路由文件中分配给控制器的路由：

```php
Route::get('/user/{id}', [UserController::class, 'show'])->middleware('auth');
```

也可以在控制器的构造函数中指定中间件。使用控制器构造函数中的`middleware`方法将中间件分配给控制器的操作：

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Closure;
use Illuminate\Http\Request;

class UserController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');
        $this->middleware('log')->only('index');
        $this->middleware('subscribed')->except('store');

        // 控制器还允许使用闭包注册中间件
        $this->middleware(function (Request $request, Closure $next) {
            return $next($request);
        });
    }

    public function show(string $id): string
    {
        return 'User' . $id;
    }
}

```

## 资源控制器

如果将应用程序中的每个Eloquent模型都是为资源，那么通常对应用程序中的每个资源都执行相同的操作。

Laravel的资源路由通过单行代码即可将典型的增删改查(CURD)路由分配给控制器。首相，使用Artisan命令`make:controller`的`--resource`选项来快速创建一个控制器

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class PhotoController extends Controller
{
    /**
     * Display a listing of the resource.
     */
    public function index()
    {
        //
    }

    /**
     * Show the form for creating a new resource.
     */
    public function create()
    {
        //
    }

    /**
     * Store a newly created resource in storage.
     */
    public function store(Request $request)
    {
        //
    }

    /**
     * Display the specified resource.
     */
    public function show(string $id)
    {
        //
    }

    /**
     * Show the form for editing the specified resource.
     */
    public function edit(string $id)
    {
        //
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(Request $request, string $id)
    {
        //
    }

    /**
     * Remove the specified resource from storage.
     */
    public function destroy(string $id)
    {
        //
    }
}

```

给控制器注册一个资源路由：

```php
use App\Http\Controllers\Api\PhotoController;

Route::resource('photos', PhotoController::class);
```

这个单一的路由声明创建了多个路由来处理资源上的各种行为，可以通过`php artisan route:list`来快速了解：

![image-20230511221846162](https://raw.githubusercontent.com/XD825/picgo/main/img/202305112219104.png)

### 自定义缺失模型行为

通常如果未找到隐式绑定的资源模型，则会生成状态码为404的HTTP响应。但是可以通过在定义资源路由时调用`missing`的方法来自定义该行为。`missing`方法接受一个闭包，如果对任何资源的路由都找不到隐式绑定模型，则将调用该闭包：

```php
Route::resource('photos', PhotoController::class)
    ->missing(function (Request $request) {
        return Redirect::route('photos.index');
    });
```

### 软删除模型

通常情况下，隐式模型绑定将不会检索已经进行了软删除的模型，并且会返回一个404HTTP响应。但是，可以在定义资源路由时调用`withTrashed`方法来告诉框架允许软删除的模型：

```php
use App\Http\Controllers\Api\PhotoController;

Route::resource('photos', PhotoController::class)->withTrashed();
```

当不传递参数调用`withTrashed`时，将在`show`、`edit` 和 `update`资源路由中允许软删除的模型。可以通过一个数组指定这些路由的子集传递给`withTrashed`方法：

```php
Route::resource('photos', PhotoController::class)
    ->withTrashed(['show']);
```

### 指定资源模型

如果使用了路由模型的绑定并且想在资源控制器的方法中使用类型提示，可以在生成控制器的时候使用`--model`选项：

```bash
php artisan make:controller Api/PhotoController --model=Photo --resource
```

### 生成表单请求

可以在生成资源控制器时提供`--requests`选项来让Artisan位控制器的storage和update方法生成表单请求类：

```bash
php artisan make:controller Api/PhotoController --model=Photo --resource --requests
```

### 部分资源路由

当声明资源路由时，可以指定控制器处理的部分行为，而不是所有默认的行为：

```php
use App\Http\Controllers\Api\PhotoController;

Route::resource('photos', PhotoController::class)
    ->only(['index', 'show']);

Route::resource('photos', PhotoController::class)
    ->except(['create', 'store', 'update', 'destroy']);
```

### API资源路由

当声明用于API的资源路由时，通常需要排除显示HTML模板的路由，例如`create`和`edit`。为了方便，可以使用`apiResource`方法来排除这两个路由：

```php
use App\Http\Controllers\Api\PhotoController;

Route::apiResource('photos', PhotoController::class);

```

![image-20230512224640495](https://raw.githubusercontent.com/XD825/picgo/main/img/202305122246719.png)

也可以传递一个数组给`apiResources`方法来同时注册多个API资源控制器：

```php
Route::apiResources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

要快速生成API资源控制器，可以执行`make:controller`命令时使用`--api`参数：

```bash
php artisan make:controller PhotoController --api
```

### 嵌套资源

有时可能需要定义一个嵌套的资源型路由。例如，一个`photo`资源可能被添加了多个`comments`。那么可以在路由中使用`.`符号来声明资源型控制器：

```php
use App\Http\Controllers\Api\PhotoController;

Route::resource('photos.comments', PhotoController::class);

```

![image-20230512230451616](https://raw.githubusercontent.com/XD825/picgo/main/img/202305122304740.png)

### 浅层嵌套

通常，并不是在所有情况下都需要子啊URI中同时拥有父ID和子ID，因为子ID已经是唯一的标识符。当使用唯一标识符（如自动递增的主键）来表示URI中的模式时，可以选择使用「浅嵌套」的方式定义路由：

```php
use App\Http\Controllers\Api\PhotoController;

Route::resource('photos.comments', PhotoController::class)
    ->shallow();
```

![image-20230512231000873](https://raw.githubusercontent.com/XD825/picgo/main/img/202305122310007.png)

### 命名资源路由

默认情况下，所有的资源控制器行为都有一个路由名称，可以传入`names`数组来覆盖这些名称：

```php
use App\Http\Controllers\Api\PhotoController;

Route::resource('photos.comments', PhotoController::class)
    ->names(['create' => 'photos.build']);
```

![image-20230512231227788](https://raw.githubusercontent.com/XD825/picgo/main/img/202305122312922.png)

### 命名资源路由参数

默认情况下，`Route::resource`会根据资源名称的单数形式创建资源路由的路由参数。可以使用`parameters`方法来轻松地覆盖资源路由名称。传入`parameters`方法应该是自愿名称和参数名称的关联数组：

```php
use App\Http\Controllers\Api\PhotoController;

Route::resource('photos', PhotoController::class)
    ->parameters(['photos' => 'photo_id']);
```

![image-20230512231742069](https://raw.githubusercontent.com/XD825/picgo/main/img/202305122317219.png)

### 限定范围的资源路由

Laravel的作用域隐式模型绑定功能可以自动确定嵌套绑定的范围，以便确认已解析的子模型属于父模型。通过在定义嵌套资源时使用`scoped`方法，可以启用自动范围界定，并指示Laravel应该通过以下方式来检索子资源的哪个字段：

```php
use App\Http\Controllers\Api\PhotoController;

Route::resource('photos.comments', PhotoController::class)
    ->scoped(['comment' => 'slug']);
```

### 本地化资源URIs

默认情况下，`Route::resource`将会用英文动词创建资源URIs。如果需要自定义`create`和`edit`行为的动词，可以在`App\Providers\RouteServiceProvider` 的 `boot` 方法中使用 `Route::resourceVerbs` 方法实现：

```php
public function boot(): void
{
    ...
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
    ...
}
```

![image-20230512232706843](https://raw.githubusercontent.com/XD825/picgo/main/img/202305122327997.png)

### 补充资源控制器

如果需要向资源控制器添加超出默认资源路由集的其他路由，则应在调用`Route::resource`方法之前定义这些路由；否则，由`resource`方法定义的路由可能会无意中优先于补充路由：

```php
use App\Http\Controllers\Api\PhotoController;

Route::get('/photos/popular', [PhotoController::class, 'popular']);

```

![image-20230512233352814](https://raw.githubusercontent.com/XD825/picgo/main/img/202305122333947.png)

### 单例资源控制器

有时候，应用中的资源可能只有一个实例，这种情况下，可以注册成单例（signleton）资源控制器：

```php
use App\Http\Controllers\Api\PhotoController;

Route::singleton('photos', PhotoController::class);

```

单例资源中`create`路由没有被注册；并且注册的路由不接受路由参数，因为该资源中只有一个实例存在：

![image-20230512233615314](https://raw.githubusercontent.com/XD825/picgo/main/img/202305122336448.png)

单例资源也可以在标准资源内嵌套使用：

```php
Route::singleton('photos.comments', PhotoController::class);
```

上例中，`photo`资源将接收所有的标准资源路由；不过`comments`资源将会是个单例资源，它的路由如下所示：

![image-20230513000746483](https://raw.githubusercontent.com/XD825/picgo/main/img/202305130007650.png)

#### Creatable单例资源

有时，可能需要为单例资源定义`create`和`storage`路由。要实现这一功能，可以在注册单例资源路由时，调用`creatable`方法：

```php
Route::singleton('photos.comments', PhotoController::class)
    ->creatable();
```

![image-20230513001057564](https://raw.githubusercontent.com/XD825/picgo/main/img/202305130010713.png)

如果希望Laravel为单个资源注册`DETELE`路由，但不注册创建或存储路由，则可以使用`destroyable`方法：

```php
Route::singleton('photos.comments', PhotoController::class)
    ->destroyable();
```

![image-20230513001241365](https://raw.githubusercontent.com/XD825/picgo/main/img/202305130012543.png)

#### API单例资源

`apiSingleton`方法可用于注册将通过API操作的单例资源，从而不需要`create`和`edit`路由：

```php
Route::apiSingleton('photos', PhotoController::class);
```

![image-20230513001628209](https://raw.githubusercontent.com/XD825/picgo/main/img/202305130016298.png)

API单例资源也可以时`creatable`，它将注册`store`和`destroy`资源路由：

```php
Route::apiSingleton('photos', PhotoController::class)
    ->creatable();
```

![image-20230513001837389](https://raw.githubusercontent.com/XD825/picgo/main/img/202305130018473.png)

## 依赖注入和控制器

### 构造函数注入

Laravel服务容器用于解析所有Laravel控制器。因此，可以在其构造函数中对控制器可能需要的任何依赖项进行类型提示。声明的依赖项将自动解析并注入到控制器实例中：

```php
<?php

namespace App\Http\Controllers;

use App\Repositories\UserRepository;

class UserController extends Controller
{
    /**
     * 创建新控制器实例。
     */
    public function __construct(
        protected UserRepository $users,
    ) {}
}
```

### 方法注入

方法注入的一个常见用例时将`Illuminate\Http\Request`实例注入到控制器方法中：

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class UserController extends Controller
{
    public function __construct(Request $request)
    {
        $name = $request->name;

        // ...

        return redirect('/users');

    }
}
```

如果控制器方法也需要路由参数，那就在其他依赖项之后列出路由参数即可：

```php
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);
```

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * 更新给定用户。
     */
    public function update(Request $request, string $id): RedirectResponse
    {
        // 更新用户。。。

        return redirect('/users');
    }
}
```

# HTTP请求

Laravel的`Illuminate\Http\Request`类提供了一种面向对象的方式来与当前由应用程序处理的HTTP请求进行交互，并检索提交请求的输入内容、Cookie和文件。

## 与请求交互

### 访问请求

要通过依赖注入获取当前的HTTP请求实例，应该在路由闭包或控制器方法中导入`Illuminate\Http\Request`类。传入的请求实例将由Laravel服务容器自动注入：

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class UserController extends Controller
{
    public function __construct(Request $request)
    {
        $name = $request->name;

        // ...

        return redirect('/users');

    }

    public function show(string $id): string
    {
        return 'User' . $id;
    }
}

```

### 请求路径、主机和方法

`Illuminate\Http\Request`实例提供各种方法来检查传入的HTTP请求，并扩展了`Symfony\Component\HttpFoundation\Request`类。

#### 获取请求路径

`path`方法返回请求的路径信息。

```php
// 访问：http://127.0.0.1:8000/api/user/1
$uri = $request->path(); // api/user/1
```

#### 检查请求路径/路由信息

`is`方法允许验证传入请求路径是否与给定的模式匹配。当使用此方法时，可以使用`*`字符作为通配符：

```php
if ($request->is('admin/*')) {
    // ...
}
```

使用`routeIs`方法，可以确定传入的请求是否与命名路由匹配：

```php
if ($request->routeIs('admin.*')) {
    // ...
}
```

#### 获取请求URL

要获取传入请求的完整URL，可以使用`url`或`fullUrl`方法。

`url`方法将返回不带查询字符串的URL，而`fullUrl`方法将包括查询字符串：

```php
// 访问：http://127.0.0.1:8000/api/user/1?type=2

$url = $request->url(); // http://127.0.0.1:8000/api/user/1

$urlWithQueryString = $request->fullUrl(); // http://127.0.0.1:8000/api/user/1?type=2
```

如果想将查询字符串数据附加到当前URL，请调用`fullUrlWithQuery`方法。此方法将给定的查询字符串变量数组与当前查询字符串合并：

```php
$request->fullUrlWithQuery(['phone' => '11111']);
```

#### 获取请求Host

```php
Route::get('/getHost', function (Request $request) {
    $host = $request->host();
    $httpHost = $request->httpHost();
    $schemeAndHttpHost = $request->schemeAndHttpHost();
    return ['host' => $host, 'httpHost' => $httpHost, 'schemeAndHttpHost' => $schemeAndHttpHost];
});

// {
// "host": "127.0.0.1",
// "httpHost": "127.0.0.1:8000",
// "schemeAndHttpHost": "http://127.0.0.1:8000"
// }
```

#### 获取请求方法

`method`方法将返回请求的HTTP动词，可以使用`isMethod`方法来验证HTTP动词是否与给定的字符串匹配：

```php
$method = $request->method();

if ($request->isMethod('post')) {
    // ...
}
```

### 请求头

使用`header`方法从`Illuminate\Http\Request`实例中检索请求标头。如果请求中没有标头，则返回`null`。`header`方法接受两个可选参数，如果该标头在请求中不存在，则返回第二个参数：

```php
$header = $request->header('X-Header-Name'); // null

$header = $request->header('X-Header-Name', 'default'); // default
```

`hasHeader`方法可用于确定请求是否包含给定的标头：

```php
if ($request->hasHeader('X-Header-Name')) {
    // ...
}
```

`bearerToken`方法可用于从`Authorization`标头检索授权标记。如果不存在此类标头，将返回一个空字符串：

```php
$bearerToken = $request->bearerToken();
```

### 请求IP地址

`ip`方法可用于检索应用程序发出请求的客户端的IP地址：

```php
$ip = $request->ip();
```

### 内容协商

Laravel提供了几种方法，通过`Accept`标头检查传入请求的请求内容类型。首先，`getAcceptableContentTypes`方法将返回包含请求接受的所有内容类型的数组：

```php
$acceptableContentTypes = $request->getAcceptableContentTypes();

//    [
//        "text/html",
//        "application/xhtml+xml",
//        "image/avif",
//        "image/webp",
//        "image/apng",
//        "application/xml",
//        "*/*",
//        "application/signed-exchange"
//    ]
```

`accepts`方法接收一个内容类型数组，并在请求接受任何内容类型时返回`true`。否则将返回`false`：

```php
if ($request->accepts(['text/html', 'application/json'])) {
    // ...
}
```

`prefers`方法确定给定内容类型数组中的哪种内容类型由请求最具优势。如果请求未接受任何提供的内容类型，则返回`null`：

```php
$prefers = $request->prefers(['text/html', 'application/json']); // text/html
```

由于许多应用程序仅提供HTML或JSON，因此可以使用`expectsJson`方法快速确定传入请求是否期望获得JSON响应：

```php
if ($request->expectsJson()){
    // ...
}
```

### PSR-7请求

## 输入

### 检索输入

#### 检索所有输入数据

使用`all`方法将所有传入请求的输入数据作为`array`检索。无论传入请求是否来自于HTML表单或XHR请求：

```php
$all = $request->all();
```

使用`collect`方法，可以将所有传入请求的输入数据作为集合检索：

```php
$collection = $request->collect();
```

`collect`方法还允许将传入请求的子集作为集合检索：

```php
$request->collect('users')->each(function (string $user) {
    // ...
});
```

#### 检索输入值

`input`方法可用于检索用户输入：

```php
$name = $request->input('name');
```

可以将默认值作为第二个参数传递给`input`方法。如果请求中不存在所请求的输入值，则返回此值：

```php
$name = $request->input('name', 'desire');
```

处理包含数组输入的表单时，使用`.`符号访问数组：

```php
$name = $request->input('products.0.name');
$names = $request->input('products.*.name');
```

如果不带任何参数的`input`方法，以将所有输入值作为关联数组检索出来：

```php
$input = $request->input();
```

#### 从查询字符串检索输入

`query`方法仅从查询字符串检索值

```php
$queryName = $request->query('name');
```

如果请求的查询字符串值数据不存在，则将返回此方法的第二个参数：

```php
$queryName = $request->query('name', 'Desire');
```

如果调用不带任何参数的`query`方法，以将所有查询字符串值作为关联数组检索出来：

```php
$query = $request->query();
```

#### 检索可字符串化的输入值

可以使用`string`方法将请求的输入数据检索为`Illuminate\Http\Request`的实例，而不是将其作为基本`string`检索：

```php
$stringAble = $request->string('name')->trim();
```

#### 检索布尔值输入

可以使用`boolean`方法进行布尔值检索。

对于 1，「1」，true，「true」，「on」和「yes」，返回 `true`。所有其他值将返回 `false`：

```php
$archived = $request->boolean('archived');
```

#### 检索日期输入值

包含日期/时间的输入值可以使用`date`方法检索为`Carbon`实例。如果请求中不包含给定名称的输入值，则返回`null`：

```php
$birthday = $request->date('birthday');
```

`date`方法可接受的第二个和第三个参数可用于分别指定日期的格式和时区：

```php
$elapsed = $request->date('elapsed', '!H:i', 'Europe/Madrid');
```

如果输入值存在但格式无效，则会抛出一个`InvalidArgumentException`异常；因此，在调用`date`方法之前建议对输入进行验证。

#### 检索枚举输入值

`enum`方法接受输入值的名称和枚举类作为其第一个和第二个参数。

如果请求中不包含给定名称的输入值或枚举没有与输入值匹配的备份值，则返回`null`

```php
use App\Enums\Status;

$status = $request->enum('status', Status::class);
```

#### 通过动态属性检索输入

可以使用`Illuminate\Http\Request`实例上的动态属性访问用户输入：

```php
$name = $request->name;
```

使用动态属性时，Laravel首先会在请求负载中查找参数的值，如果不存在，则会在匹配路由的参数中搜索该字段。

#### 检索输入数据的一部分

如果需要检索输入数据的子集，则可以使用`only`和`except`方法。这两个方法都接受一个单一的`array`或动态参数列表：

```php
$only = $request->only(['username', 'password']);
$only1 = $request->only('username', 'password');
$except = $request->except(['credit_card']);
$except1 = $request->except('credit_card');
```

注：`only`方法返回请求的所有键值对；但是，不会返回请求中不存在的键值对。

### 判断输入是否存在

可以使用`has`方法来确定请求中是否存在某个值。如果请求中存在该值则返回`true`：

```php
if ($request->has('name')) {
    // ...
}
```

当给定一个数组时，`has`方法将确定所有指定的值是否都存在：

```php
if ($request->has(['name', 'email'])) {
    // ...
}
```

`whenHas`方法将在请求中存在一个值时执行给定的闭包：

```php
$request->whenHas('name', function (string $input) {
    // ...
});
```

可以通过向`whenHas`方法传递第二个闭包来执行在请求中没有指定值的情况：

```php
$request->whenHas('name', function (string $input) {
    // "name" 值存在...
}, function () {
    // "name" 值不存在...
});
```

`hasAny`方法放回`true`，如果任意指定的值存在，则返回`true`：

```php
if ($request->hasAny(['name', 'email'])) {
    // ...
}
```

如果想要确定请求中是否存在一个值且不是一个空字符串，则可以使用`fulled`方法：

```php
if ($request->filled('name')) {
    // ...
}
```

`whenFilled`方法将请求中存在一个值且不是空字符串时执行给定的闭包：

```php
$request->whenFilled('name', function (string $input) {
    // ...
});
```

可以通过向`whenFilled`方法传递第二个闭包来执行在请求中没有指定值的情况：

```php
$request->whenFilled('name', function (string $input) {
    // "name" 值已填写...
}, function () {
    // "name" 值未填写...
});
```

要确定给定的键是否不存在请求中，可以使用`missing`和`whenMissing`方法：

```php
if ($request->missing('name')) {
    // ...
}

$request->whenMissing('name', function (array $input) {
    // "name" 值缺失...
}, function () {
    // "name" 值存在...
});
```

### 合并其他输入

可以使用`merge`方法，手动将其他输入合并到请求的向右输入数据中。如果给定的输入键已经存在于请求中，它将被提供给`merge`方法的数据覆盖：

```php
$request->merge(['votes' => 0]);
```

如果请求的输入数据中不存在相应的键，则可以使用`mergeIfMissing`方法将输入合并到请求中：

```php
$request->mergeIfMissing(['votes' => 0]);
```



### 旧输入

Laravel允许在两次请求之间保留数据。这个特性在检测到验证错误后重新填充表单时特别有用。但是，如果使用Laravel的包含的表单验证，不需要自己手动调用这些方法，因为Laravel的一些内置验证功能将自动调用它们。

### 闪存

#### 闪存输入到Session

在`Illuminate\Http\Request`类上的`flash`方法将当前输入闪存到session，以便在下一次用户请求应用程序时使用：

```php
$request->flash();
```

还可以使用`flashOnly`和`flashExcept`方法闪存一部分请求数据到Session。这些方法对于将敏感信息（如密码）排除在Session外的情况下非常有用：

```php
$request->flashOnly(['username', 'email']);
$request->flashExcept('password');
```

#### 闪存输入后重定向

可以使用`withInput`方法将输入闪存到重定向中：

```php
return redirect('form')->withInput();

return redirect()->route('user.create')->withInput();

return redirect('form')->withInput(
    $request->except('password')
);
```

#### 检索旧输入值

若要获取上一次请求所保存的就输入数据，可以在`Illuminate\Http\Request`的实例上调用`old`方法，`old`方法会从session中检索先前闪存的输入数据：

```php
$username = $request->old('username');
```

此外，Laravel还提供了一个全局辅助函数`old`。如果在`Blade`模板中显示旧的输入，则更方便使用`old`辅助函数重新填充表单。如果给定字段没有就输入，则会返回`null`：

```php
<input type="text" name="username" value="{{ old('username') }}">
```

### Cookies

#### 检索请求中的Cookies

Laravel框架创建的所有cookies都经过加密并签名，这意味着如果客户端更改了cookie值，则这些cookie将被视为无效。要从请求中检索cookie值，在`Illuminate\Http\Request`实例上使用`cookie`方法：

```php
$cookie = $request->cookie('name');
```



## 输入过滤和规范化

默认情况下，Laravel在应用程序的全局中间件中包含`App\Http\Middleware\TrimStrings`和`Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull`中间件。这些中间件在`App\Http\Kernel`类的全局中间件中列出。这些中间件将自动修剪请求中的字符串字段，并将任何空字符串转换为`null`。

#### 禁用输入规范化

如果要禁用所有请求的该行为，可以从 `App\Http\Kernel` 类的 `$middleware` 属性中删除这两个中间件，从而将它们从应用程序的中间件栈中删除。

如果想要禁用应用程序的一部分请求的字符串修剪和空字符串转换，可以使用中间件提供的 `skipWhen` 方法。该方法接受一个闭包，该闭包应返回 `true` 或 `false`，以指示是否应跳过输入规范化。通常情况下，需要在应用程序的 `AppServiceProvider` 的 `boot` 方法中调用 `skipWhen` 方法。

```php
use App\Http\Middleware\TrimStrings;
use Illuminate\Http\Request;
use Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    TrimStrings::skipWhen(function (Request $request) {
        return $request->is('admin/*');
    });

    ConvertEmptyStringsToNull::skipWhen(function (Request $request) {
        // ...
    });
}
```

## 文件

### 检索上传的文件

可以使用`file`方法或动态属性从`Illuminate\Http\Request`实例中检索已上传的文件。`file`方法返回`Illuminate\Http\UploadedFile`类的实例，该类扩展了PHP的`SplFileInfo`类，并提供了各种用于与文件交互的方法：

```php
$file = $request->file('photo');

$file = $request->photo;
```

可以使用`hasFile`方法检查请求中是否存在文件：

```php
if ($request->hasFile('photo')) {
    // ...
}
```

#### 验证成功上传的文件

除了检查文件是否存在之外，还可以通过`isValid`方法验证上传文件时是否存在问题：

```php
if ($request->file('photo')->isValid()) {
    // ...
}
```

#### 文件路径和扩展名

`UploadedFile` 类还包含访问文件的完全限定路径及其扩展名的方法。`extension` 方法将尝试基于其内容猜测文件的扩展名。此扩展名可能与客户端提供的扩展名不同：

```php
$path = $request->photo->path();

$extension = $request->photo->extension();
```



### 存储上传的文件

要存储已上传的文件，通常会使用您配置的一个文件系统 。`UploadedFile` 类具有一个 `store` 方法，该方法将上传的文件移动到您的磁盘中的一个位置，该位置可以是本地文件系统上的位置，也可以是像 Amazon S3 这样的云存储位置。

`store` 方法接受存储文件的路径，该路径相对于文件系统的配置根目录。此路径不应包含文件名，因为将自动生成唯一的 ID 作为文件名。

`store` 方法还接受一个可选的第二个参数，用于指定应用于存储文件的磁盘的名称。该方法将返回相对于磁盘根目录的文件路径：

```php
$path = $request->photo->store('images');

$path = $request->photo->store('images', 's3');
```

如果不希望自动生成文件名，则可以使用`storeAs`方法，该方法接受路径、文件名和磁盘名称作为其参数：

```php
$path = $request->photo->storeAs('images', 'filename.jpg');

$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
```

## 配置受信任的代理

使用 `App\Http\Middleware\TrustProxies` 中间件，这个中间件已经包含在 Laravel 应用程序中，它允许快速定制应用程序应信任的负载均衡器或代理。信任的代理应该被列在此中间件的 `$proxies` 属性上的数组中。除了配置受信任的代理之外，还可以配置应该信任的代理 `$headers`：

```php
<?php

namespace App\Http\Middleware;

use Illuminate\Http\Middleware\TrustProxies as Middleware;
use Illuminate\Http\Request;

class TrustProxies extends Middleware
{
    /**
     * 此应用程序的受信任代理。
     *
     * @var string|array
     */
    protected $proxies = [
        '192.168.1.1',
        '192.168.1.2',
    ];

    /**
     * 应用于检测代理的标头。
     *
     * @var int
     */
    protected $headers = Request::HEADER_X_FORWARDED_FOR | Request::HEADER_X_FORWARDED_HOST | Request::HEADER_X_FORWARDED_PORT | Request::HEADER_X_FORWARDED_PROTO;
}
```

**注意**
如果正在使用 AWS 弹性负载平衡，请将 `$headers` 值设置为 `Request::HEADER_X_FORWARDED_AWS_ELB`

**信任所有代理**

```php
/**
 * 应用所信任的代理。
 *
 * @var string|array
 */
protected $proxies = '*';
```



## 配置可信任的Host

默认情况下，Laravel 将响应它接收到的所有请求，而不管 HTTP 请求的 `Host` 标头的内容是什么。此外，在 web 请求期间生成应用程序的绝对 URL 时，将使用 `Host` 头的值。

通常情况下，应该配置 Web 服务器（如 Nginx 或 Apache）仅向匹配给定主机名的应用程序发送请求。然而，如果没有直接自定义 Web 服务器的能力，需要指示 Laravel 仅响应特定主机名的请求，可以为应用程序启用` App\Http\Middleware\TrustHosts` 中间件。

`TrustHosts` 中间件已经包含在应用程序的 `$middleware` 堆栈中；但是，应该将其取消注释以使其生效。在此中间件的 `hosts` 方法中，可以指定应用程序应该响应的主机名。具有其他 `Host` 值标头的传入请求将被拒绝：

```php
/**
 * 获取应被信任的主机模式。
 *
 * @return array<int, string>
 */
public function hosts(): array
{
    return [
        'laravel.test',
        $this->allSubdomainsOfApplicationUrl(),
    ];
}
```

`allSubdomainsOfApplicationUrl` 帮助程序方法将返回与应用程序` app.url `配置值的所有子域相匹配的正则表达式。在构建利用通配符子域的应用程序时，这个帮助程序提供了一种方便的方法来允许所有应用程序的子域。

# 响应

## 创建响应

### 字符串 & 数组

最基本就是从路由或控制器返回一个简单的字符串，框架会自动将这个字符串转化为一个完整的HTTP响应：

```php
Route::get('/', function () {
    return 'Hello PHP!';
});
```

也可以返回数组，框架会自动将数组转换为JSON响应：

```php
Route::get('/', function () {
    return [1, 3, 4];
});
```

### Response对象

返回一个完整的`Illuminate\Http\Response`实力允许自定义返回的HTTP状态码和返回头信息。`Response`实例继承自`Symfony\Component\HttpFoundation\Response`类，此类提供了各种构建HTTP响应的方法：

```php
Route::get('/home', function () {
    return response('Hello PHP!', 200)
        ->header('Content-Type', 'text/plain');
});
```

### Eloquent模型和集合

可以直接从路由和控制器返回 [Eloquent ORM](https://learnku.com/docs/laravel/10.x/eloquent) 模型和集合。这样做时，Laravel 将自动将模型和集合转换为 JSON 响应，同时遵循模型的 [隐藏属性](https://learnku.com/docs/laravel/10.x/eloquent-serialization#hiding-attributes-from-json):

```php
use App\Models\User;

Route::get('/user/{user}', function (User $user) {
    return $user;
});
```

### 添加响应头

可以使用`header`方法将一系列头添加到响应中：

```php
return response($content)
    ->header('Content-Type', $type)
    ->header('X-Header-One', 'Header Value')
    ->header('X-Header-Two', 'Header Value');
```

也可以使用`withHeaders`方法指定要添加到响应的标头数组：

```php
return response($content)
    ->withHeaders([
        'Content-Type' => $type,
        'X-Header-One' => 'Header Value',
        'X-Header-Two' => 'Header Value',
    ]);
```

### 缓存控制中间件

Laravel包含一个`cache.headers`中间件，可用于快速设置一组路由的`CaChe-Control`标头。指令应使用相应缓存控制指定的蛇形命名法等效项提供，并应以分号分割。如果在指令列表中指定了`etag`，则响应内容的MD5哈希将自动设置为`ETag`标识符：

```php
Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function () {
    Route::get('/privacy', function () {
        // ...
    });

    Route::get('/terms', function () {
        // ...
    });
});
```

### 添加响应Cookies

可以使用`cookie`方法将cookie附加到传出的`Illumize\Http\Response`实例，应将cookie的名称、值和有效分钟数传递给此方法：

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

`cookie`方法还接受一些使用频率较低的参数。通常，这些参数的目的和意义与PHP原生setcookie的参数相同

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```

如果希望确保cookie与传出响应一起发送，但还没有该响应的实例，则可以使用`Cookie`facade将cookie加入队列，以便在发送响应时附加到响应中。`queue`方法接受创建cookie实例所需的参数。在发送到浏览器之前，这些cookies将附加到传出的响应中：

```php
use Illuminate\Support\Facades\Cookie;

Cookie::queue('name', 'value', $minutes);
```

#### 生成Cookie实例

如果要生成一个`Symfony\Component\HttpFoundation\Cookie`实例，打算稍后附加到响应实例中。可以使用全局`cookie`助手函数。此cookie将不会发送回客户端，除非它被附加到响应实例中：

```php
$cookie = cookie('name', 'value', $minutes);

return response('Hello PHP')->cookie($cookie)
```

#### 提前过期Cookies

可以通过响应中的`withoutCookie`方法使Cookie过期，用于删除Cookie：

```php
return response('Hello PHP')->withoutCookie('name');
```

如果尚未有创建响应的实例，则可以使用`Cookie`facade中的`expire`方法使Cookie过期：

```php
Cookie::expire('name');
```

### Cookies & 加密

默认情况下，由 Laravel 生成的所有 cookie 都经过了加密和签名，因此客户端无法篡改或读取它们。如果要对应用程序生成的部分 cookie 禁用加密，可以使用 `App\Http\Middleware\EncryptCookies` 中间件的 `$except` 属性，该属性位于 `App/Http/Middleware` 目录中：

```php
<?php

namespace App\Http\Middleware;

use Illuminate\Cookie\Middleware\EncryptCookies as Middleware;

class EncryptCookies extends Middleware
{
    /**
     * The names of the cookies that should not be encrypted.
     *
     * @var array<int, string>
     */
    protected $except = [
        //
        'cookie_name',
    ];
}

```

## 重定向

重定向响应是 `Illuminate\Http\RedirectResponse` 类的实例，包含将用户重定向到另一个 URL 所需的适当 HTTP 头。Laravel 有几种方法可以生成 `RedirectResponse` 实例。最简单的方法是使用全局 `redirect` 助手函数：

```php
Route::get('/dashboard', function () {
    return redirect('home/dashboard')
});
```

有时可能希望将用户重定向到以前的位置，例如当提交的表单无效时，可以使用全局`back`助手函数来执行操作。由于此功能使用session，请确保调用`back`函数的路由使用的是`web`中间件组：

```php
Route::post('/user/profile', function () {
    // 验证请求参数

    return back()->withInput();
});
```

### 重定向到指定名称的路由

当在没有传递参数的情况下调用`redirect`助手函数时，将返回`Illuminate\Routing\Redirector`的实例，允许调用`Redirector`实例上的任何方法。例如，要对命名路由生成`RedirectResponse`，可以使用`route`方法：

```php
return redirect()->route('login');
```

如果路由中有参数，可以将其作为第二个参数传递给`route`方法：

```php
// 对于具有以下URI的路由: /profile/{id}

return redirect()->route('profile', ['id' => 1]);
```

#### 通过Eloquent模型填充参数

如果要重定向到使用从Eloquent模型田中ID参数的路由，可以直接传递模型本身。ID将会被自动提取：

```php
// 对于具有以下URI的路由: /profile/{id}

return redirect()->route('profile', [$user]);
```

如果想要自定义路由参数，可以指定路由参数(`/profile/{id:slug}`)或者重写Eloquent模型上`getRouteKey`方法：

```php
/**
 * 获取模型的路由键值。
 */
public function getRouteKey(): mixed
{
    return $this->slug;
}
```

### 重定向到控制器方法

也可以生成重定向到`controller actions`。只要把控制器和`action`的名称传递给`action`方法：

```php
use App\Http\Controllers\UserController;

return redirect()->action([UserController::class, 'index']);
```

如果控制器路由有参数，可以将其作为第二个参数传递给`action`方法：

```php
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```

### 重定向到外部域名

有时候需要重定向到应用外的域名。可以通过调用`away`方法，它会创建一个不带有任何额外的URL编码、有效性校验和检查`RedirectResponse`实例：

```php
return redirect()->away('https://www.google.com');
```

### 重定向并使用闪存的Session数据

重定向到新的URL的同时传递数据给session是很常见的。通常在将消息发送给session后成功执行操作后完成的。为了方便，可以创建一个`RedirectResponse`实例并在链式方法调用中将数据传递给session：

```php
Route::post('/user/profile', function () {
    // ...

    return redirect('dashboard')->with('status', 'Profile updated!');
});
```



## 其他响应类型

`response`助手可以用于生成其他类型的响应实例。当不带参数调用`response`助手时，会返回`Illuminate\Contracts\Routing\ResponseFactory`contract的实现。该契约提供了几种有用的方法来生成响应。

### 视图响应

```php
return response()
            ->view('hello', $data, 200)
            ->header('Content-Type', $type);
```

如果不需要传递自定义HTTP状态代码或自定义标头，则可以使用全局`view`辅助函数。

### JSON响应

`json`方法会自动将`Content-Type`表头设置为`application/json`，并使用`json_encode`PHP函数将给定的数组转换为JSON：

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA',
]);
```

如果想创建一个JSONP响应，可以结合使用`json`方法和`withCallback`方法：

```php
return response()
            ->json(['name' => 'Abigail', 'state' => 'CA'])
            ->withCallback($request->input('callback'));
```

### 文件下载

`download`方法可用于生成强制用户浏览器在给定路径下载文件的响应。`download`方法接受文件名作为该方法的第二个参数，这将确定下载文件的用户看到的文件名。最后，可以将一组HTTP标头作为该方法的第三个参数传递：

```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);
```

#### 流式下载

有时候可能希望将给定操作的字符串响应转换为可下载的响应，而不必将操作的内容写入磁盘。在这种情况下，可以使用`streamDownload`方法。此方法接受回调、文件名和可选的标头数组作为其参数：

```php
use App\Services\GitHub;

return response()->streamDownload(function () {
    echo GitHub::api('repo')
                ->contents()
                ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');
```

### 文件响应

`file`方法可用于直接在用户的浏览器中显示文件，例如图像或PDF，而不是启动下载。这个方法接受文件的路径作为它的第一个参数和一个标头数组作为它的第二个参数：

```php
return response()->file($pathToFile);

return response()->file($pathToFile, $headers);
```

## 响应宏

如果想定义一个可以在各种路由和控制器中重复使用的自定义响应，可以使用`Response`facade上的`macro`方法。通常，应用从应用程序的服务提供者，如`App\Providers\AppServiceProvider`服务提供程序的`boot`方法调用此方法：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Response;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 启动一个应用的服务
     */
    public function boot(): void
    {
        Response::macro('caps', function (string $value) {
            return Response::make(strtoupper($value));
        });
    }
}
```

`macro`函数接受名称作为其第一个参数，并接受闭包作为其第二个参数。当从`ResponseFactory`实现或`response`助手函数调用宏名称时，将执行宏的闭包：

```php
return response()->caps('foo');
```

# 会话

## 简介

由于HTTP驱动的应用程序是无状态的，Session提供了一种在多个请求之间存储有关用户信息的方法，这类信息一般都存储在后续请求可以访问的持久存储/后端中。

Laravel通过同一个可读性强的API处理各种自带的后台驱动程序。支持诸如比较热门的`Memcached`、`Redis`和数据库。

### 配置

Session的配置文件存储在`config/session.php`文件中。默认情况下，Laravle为绝大多数应用程序配置的Session驱动为`file`驱动，它适用于大多数程序。如果应用程序需要再多个web服务器之间进行负载平衡，应该选择一个所有服务器都可以访问的几种式存储，例如Redis或数据库。

Session`driver`的配置预设了每个请求存储Session数据的位置。Laravel自带了几个不错而且开箱即用的驱动：

- `file`：Session存储在`storage/framework/sessions`
- `cookie`：Session存储在安全加密的cookie中
- `database`：Session储存在关系型数据库中
- `memcached`/`redis`：Session存储在基于高速缓存的存储系统中
- `dynamodb`：Session存储在AWS DynamoDB中
- `array`：Session存储在PHP数组中，不会被初九话

### 驱动先决条件

#### 数据库

使用`database`Session驱动时，需要创建一个记录Session的表。下面是`Schema`的声明示例：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::create('sessions', function (Blueprint $table) {
    $table->string('id')->primary();
    $table->foreignId('user_id')->nullable()->index();
    $table->string('ip_address', 45)->nullable();
    $table->text('user_agent')->nullable();
    $table->text('payload');
    $table->integer('last_activity')->index();
});
```

可以使用Artisan命令`session:table`生成这个迁移

```bash
php artisan session:table

php artisan migrate
```

#### Redis

在Laravel使用Redis Session驱动前，需要安装PhpReis PHP扩展，可以通过PECL/Composer安装`predis/predis`包。

## 使用Session

### 获取数据

在 Laravel 中有两种基本的 Session 使用方式：全局 `session` 助手函数和通过 `Request` 实例。首先看下通过 `Request` 实例访问 Session , 它可以隐式绑定路由闭包或者控制器方法。记住，Laravel 会自动注入控制器方法的依赖。

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * 显示指定用户个人资料。
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

当从Session获取数据时，也可以在`get`方法第二个参数里传递一个default默认值，如果Session里不存在键值对key的数据结果，这个默认值就会返回。如果传递给`get`方法一个闭包作为默认值，这个闭包会被执行并且返回结果：

```php
$value = $request->session()->get('key', 'default');

$value = $request->session()->get('key', function () {
    return 'default';
});
```

#### 全局Session助手函数

可以在Session里使用PHP全局`session`函数获取和存储数据。当这个`session`函数以一个单独的字符串形式被调用时，它将返回这个Session键值对的结果。当函数以key-value数组形式被调用时，这些值会被存储在Session中：

```php
Route::get('/home', function () {
    // 从 Session 获取数据 ...
    $value = session('key');

    // 设置默认值...
    $value = session('key', 'default');

    // 在Session 里存储一段数据 ...
    session(['key' => 'value']);
});
```

#### 获取所有Session数据

可以使用`all`方法从Session里获取所有数据：

```php
$data = $request->session()->all();
```

#### 判断Session里是否存在条目

可以使用`has`方法。如果条目存在返回`true`，否则返回`null`：

```php
if ($request->session()->has('users')) {
    // ...
}
```

判断 Session 里是否存在一个即使结果值为 `null` 的条目，可以使用 `exists` 方法：

```php
if ($request->session()->exists('users')) {
    // ...
}
```

要确定某个条目是否在会话中不存在，可以使用 `missing` 方法。如果条目不存在，`missing` 方法返回 `true`：

```php
if ($request->session()->missing('users')) {
    // ...
}
```

### 存储数据

Session 里存储数据，通常将使用 Request 实例中的 `put` 方法或者 `session` 助手函数：

```php
// 通过 Request 实例存储 ...
$request->session()->put('key', 'value');

// 通过全局 Session 助手函数存储 ...
session(['key' => 'value']);
```

#### Session存储数组

`push` 方法可以把一个新值推入到以数组形式存储的 session 值里。例如：如果 `user.teams` 键值对有一个关于团队名字的数组，可以推入一个新值到这个数组里：

```php
$request->session()->push('user.teams', 'developers');
```

#### 获取&删除条目

`pull` 方法会从 Session 里获取并且删除一个条目：

```php
$value = $request->session()->pull('key', 'default');
```

#### 递增、递减会话值

如果 Session 数据里有整形希望进行加减操作，可以使用 `increment` 和 `decrement` 方法：

```php
$request->session()->increment('count');

$request->session()->increment('count', $incrementBy = 2);

$request->session()->decrement('count');

$request->session()->decrement('count', $decrementBy = 2);
```

### 闪存数据

有时可能想在 Session 里为下次请求存储一些条目。可以使用 `flash` 方法。使用这个方法，存储在 Session 的数据将立即可用并且会保留到下一个 HTTP 请求期间，之后会被删除。闪存数据主要用于短期的状态消息：

```php
$request->session()->flash('status', 'Task was successful!');
```

如果需要为多次请求持久化闪存数据，可以使用 `reflash` 方法，它会为一个额外的请求保持住所有的闪存数据，如果仅需要保持特定的闪存数据，可以使用 `keep` 方法：

```php
$request->session()->reflash();

$request->session()->keep(['username', 'email']);
```

如果仅为了当前的请求持久化闪存数据，可以使用 `now` 方法：

```php
$request->session()->now('status', 'Task was successful!');
```

### 删除数据

`forget` 方法会从 Session 删除一些数据。如果想删除所有 Session 数据，可以使用 `flush` 方法：

```php
// 删除一个单独的键值对 ...
$request->session()->forget('name');

// 删除多个 键值对 ...
$request->session()->forget(['name', 'status']);

$request->session()->flush();
```



### 重新生成SessionID

重新生成 Session ID 经常被用来阻止恶意用户使用 session fixation 攻击你的应用。

如果正在使用入门套件或 Laravel Fortify 中的任意一种， Laravel 会在认证阶段自动生成 Session ID；然而如果需要手动重新生成 Session ID ，可以使用` regenerate` 方法：

```php
$request->session()->regenerate();
```

如果需要重新生成 Session ID 并同时删除所有 Session 里的数据，可以使用 `invalidate` 方法：

```php
$request->session()->invalidate();
```



## Session阻塞

默认情况下，Laravel 允许使用同一 Session 的请求并发地执行，举例来说，如果使用一个 JavaScript HTTP 库向你的应用执行两次 HTTP 请求，它们将同时执行。对多数应用这不是问题，然而 在一小部分应用中可能出现 Session 数据丢失，这些应用会向两个不同的应用端并发请求，并同时写入数据到 Session。

为了解决这个问题，Laravel 允许限制指定 Session 的并发请求。首先，可以在路由定义时使用 `block` 链式方法。在这个示例中，一个到 `/profile` 的路由请求会拿到一把 Session 锁。当它处在锁定状态时，任何使用相同 Session ID 的到 `/profile` 或 `/order` 的路由请求都必须等待，直到第一个请求处理完成后再继续执行：

```php
Route::post('/profile', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10)

Route::post('/order', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10)
```

`block` 方法接受两个可选参数。`block` 方法接受的第一个参数是 Session 锁释放前应该持有的最大秒数。当然，如果请求在此时间之前完成执行，锁将提前释放。

`block` 方法接受的第二个参数是请求在试图获得 Session 锁时应该等待的秒数。如果请求在给定的秒数内无法获得会话锁，将抛出 `Illuminate\Contracts\Cache\LockTimeoutException` 异常。

如果不传参，那么 Session 锁默认锁定最大时间是 10 秒，请求锁最大的等待时间也是 10 秒：

```php
Route::post('/profile', function () {
    // ...
})->block()
```



## 添加自定义Session驱动

#### 实现驱动

如果现存的 Session 驱动不能满足需求，Laravel 允许自定义 Session Handler。自定义驱动应实现 PHP 内置的 `SessionHandlerInterface`。这个接口仅包含几个方法。以下是 MongoDB 驱动实现的代码片段：

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

注：Laravel 没有内置存放扩展的目录，可以放置在任意目录下，这个示例里，创建了一个 `Extensions` 目录存放 `MongoSessionHandler`。

- `open` 方法通常用于基于文件的 Session 存储系统。因为 Laravel 附带了一个` file `Session 驱动。你无须在里面写任何代码。可以简单地忽略掉。
- `close` 方法跟 `open` 方法很像，通常也可以忽略掉。对大多数驱动来说，它不是必须的。
- `read` 方法应返回与给定的 `$sessionId` 关联的 Session 数据的字符串格式。在你的驱动中获取或存储 Session 数据时，无须作任何序列化和编码的操作，Laravel 会自动为你执行序列化。
- `write` 方法将与 `$sessionId` 关联的给定的 `$data` 字符串写入到一些持久化存储系统，如 MongoDB 或者其他你选择的存储系统。再次，你无须进行任何序列化操作，Laravel 会自动为你处理。
- `destroy` 方法应可以从持久化存储中删除与 `$sessionId` 相关联的数据。
- `gc` 方法应可以销毁给定的 `$lifetime`（UNIX 时间戳格式 ）之前的所有 Session 数据。对于像 Memcached 和 Redis 这类拥有过期机制的系统来说，本方法可以置空。

#### 注册驱动

在 Laravel 中添加额外的驱动到 Session 后端 ，可以使用 `Session` Facade 提供的 `extend` 方法。你应该在服务提供者中的 `boot` 方法中调用 extend 方法。可以通过已有的` App\Providers\AppServiceProvider` 或创建一个全新的服务提供者执行此操作：

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
     * 注册任意应用服务。
     */
    public function register(): void
    {
        // ...
    }

    /**
     * 启动任意应用服务。
     */
    public function boot(): void
    {
        Session::extend('mongo', function (Application $app) {
            // 返回一个 SessionHandlerInterface 接口的实现 ...
            return new MongoSessionHandler;
        });
    }
}
```

一旦 Session 驱动注册完成，就可以在 `config/session.php` 配置文件选择使用 `mongo` 驱动。

