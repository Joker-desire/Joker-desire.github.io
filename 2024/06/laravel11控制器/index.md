# Laravel 11.x 控制器


----

[原文](https://laravel.com/docs/11.x/controllers)

----

## 介绍

与其在路由文件中将所有请求处理逻辑定义为闭包，也可以使用"controller"类来组织这些行为。

控制器可以将相关的请求逻辑分组到一个类中。

默认情况下，控制器存储在`app/Http/Controllers`目录中。

## 编写控制器

### 基本控制器

> 使用`make:controller`命令快速生成新控制器

```bash
php artisan make:controller UserController
```

控制器可以有任意数量的公共方法，这些方法将响应传入的HTTP请求：

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\View\View;

class UserController extends Controller
{

    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }

}

```

编写控制器类和方法后，可以定义控制器方法的路由：

```php
use App\Http\Controllers\UserController;
 
Route::get('/user/{id}', [UserController::class, 'show']);
```

控制器不需要继承基类。然而，有时继承一个包含所有控制器通用方法的基控制器类会更方便。

### 单动作控制器

如果一个控制器动作特别复杂，为这个单一动作专门创建一个控制类会更方便。

可以在控制器中定义一个单一的`__invoke`方法。

可以使用`make:controller`命令`--invokable`选项生成单一动作控制器：

```bash
php artisan make:controller ProvisionServer --invokable
```

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class ProvisionServer extends Controller
{
    /**
     * Handle the incoming request.
     */
    public function __invoke(Request $request)
    {
        //
    }
}

```

在为单动作控制器注册路由时，不需要指定控制器方法。相反，可以简单地将控制器的名称传递给路由器：

```php
use App\Http\Controllers\ProvisionServer;
 
Route::post('/server', ProvisionServer::class);
```

## 控制器中间件

中间件可以在路由文件中分配给控制器的路由。

```php
Route::get('profile', [UserController::class, 'show'])->middleware('auth');
```

也可以在控制器类中指定中间件。只需控制器实现`HasMiddleware`接口，该接口规定控制器应具有静态`middleware`方法。从次方法中，可以返回一个中间件数组，该数组应用于控制器的操作：

```php
<?php
 
namespace App\Http\Controllers;
 
use App\Http\Controllers\Controller;
use Illuminate\Routing\Controllers\HasMiddleware;
use Illuminate\Routing\Controllers\Middleware;
 
class UserController extends Controller implements HasMiddleware
{
    /**
     * Get the middleware that should be assigned to the controller.
     */
    public static function middleware(): array
    {
        return [
            'auth',
            new Middleware('log', only: ['index']),
            new Middleware('subscribed', except: ['store']),
        ];
    }
 
    // ...
}
```

还可以讲控制器中间件定义成闭包，这提供了一种方便的形式来定义内联中间件，而无需编写整个中间件类：

```php
use Closure;
use Illuminate\Http\Request;
 
/**
 * Get the middleware that should be assigned to the controller.
 */
public static function middleware(): array
{
    return [
        function (Request $request, Closure $next) {
            return $next($request);
        },
    ];
}
```



## 资源控制器

使用`make:controller`命令的`--resource`选项，快速创建控制器的CRUD操作：

```php
php artisan make:controller PhotoController --resource
```
此命令将在`app/Http/Controllers/PhotoController.php` 生成一个控制器。该控制器将包含每个可用资源操作的方法。
```php
<?php

namespace App\Http\Controllers;

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

然后就可以注册一个指向该控制器的资源路由：

```php
use App\Http\Controllers\PhotoController;
 
Route::resource('photos', PhotoController::class);
```

![](https://raw.githubusercontent.com/XD825/picgo/main/img/202406161456066.png)

也可以通过将数组传递给`resources`方法来一次注册多个资源控制器：

```php
Route::resources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

#### 自定义缺失的模型行为

通常，如果未找到隐式绑定的资源模型，将生成一个 404 HTTP 响应。

然而，可以在定义资源路由时通过调用`missing` 方法来自定义此行为。

`missing` 方法接受一个闭包，如果在任何资源路由中找不到隐式绑定的模型，该闭包将被调用：

```php
use App\Http\Controllers\PhotoController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;
 
Route::resource('photos', PhotoController::class)
        ->missing(function (Request $request) {
            return Redirect::route('photos.index');
        });
```

#### 软删除模型

通常，隐式模型绑定不会检索已软删除的模型，而是返回 404 HTTP 响应。

然而，可以在定义资源路由时调用 `withTrashed` 方法来指示框架允许软删除的模型：

```php
use App\Http\Controllers\PhotoController;
 
Route::resource('photos', PhotoController::class)->withTrashed();
```

调用不带参数的 `withTrashed` 方法将允许 show、edit 和 update 资源路由处理软删除的模型。

可以通过传递一个数组给 `withTrashed` 方法来指定这些路由的子集。

```php
Route::resource('photos', PhotoController::class)->withTrashed(['show']);
```

#### 指定资源模型

如果使用路由模型绑定，并希望资源控制器的方法类型提示模型实例，那么可以在生成控制器时使用 

`--model` 选项。

```bash
php artisan make:controller PhotoController --model=Photo --resource
```

#### 生成表单请求

在生成资源控制器时，可以使用 `--requests` 选项生成表单请求类，以供控制器的存储（create）和更新（update）方法使用。

```php
php artisan make:controller PhotoController --model=Photo --resource --requests
```



### 部分资源路由

声明资源路由时，可以指定控制器应处理的操作子集，而不是完成的默认操作集：

```php
use App\Http\Controllers\PhotoController;
 
Route::resource('photos', PhotoController::class)->only([
    'index', 'show'
]);
 
Route::resource('photos', PhotoController::class)->except([
    'create', 'store', 'update', 'destroy'
]);
```

#### API 资源路由

在声明将由 API 使用的资源路由时，可以使用`apiResource`方法自动排除提供 HTML 模板（如 `create` 和 `edit` ）的路由：

```php
use App\Http\Controllers\PhotoController;
 
Route::apiResource('photos', PhotoController::class);
```

同样，也可以通过将数组传递给`apiResource`方法一次注册多个API资源控制器：

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\PostController;
 
Route::apiResources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

可以通过`make:controller`命令的`--api`快速生成API资源路由：

```php
php artisan make:controller PhotoController --api
```

### 嵌套资源

为了嵌套资源控制器，可以在路由声明中使用`.`符号：

```php
use App\Http\Controllers\PhotoCommentController;
 
Route::resource('photos.comments', PhotoCommentController::class);

// /photos/{photo}/comments/{comment}
```

#### 浅层嵌套

通常情况下，不必在 URI 中同时包含父资源和子资源的 ID，因为子资源的 ID 已经是唯一标识符。

当使用自增主键等唯一标识符来识别模型在 URI 段中时，可以选择使用 "浅嵌套"：

```php
use App\Http\Controllers\CommentController;
 
Route::resource('photos.comments', CommentController::class)->shallow();
```

![image-20240616151520793](https://raw.githubusercontent.com/XD825/picgo/main/img/202406161515846.png)

### 命名资源路由

默认情况下，所有资源控制器操作都具有路由名称，可以通过传递具有所需路由名称的`name`数据来覆盖这些名称：

```php
use App\Http\Controllers\PhotoController;
 
Route::resource('photos', PhotoController::class)->names([
    'create' => 'photos.build'
]);
```



### 命名资源路由参数

默认情况下，`Route::resource` 根据资源名称的单数形式创建资源路由的路由参数。

可以轻松地在每个资源基础上使用 `parameters` 方法覆盖这一行为。

传递给 `parameters` 方法的数组应该是资源名称和参数名称的关联数组。

```php
use App\Http\Controllers\AdminUserController;
 
Route::resource('users', AdminUserController::class)->parameters([
    'users' => 'admin_user'
]);

// URI
// /users/{admin_user}
```



### 确定资源路由的范围

Laravel 的作用域隐式模型绑定功能可以自动将嵌套绑定作用域化，以确保解析的子模型确实属于父模型。

通过在定义嵌套资源时使用 `scoped` 方法启用自动作用域化，并指示 Laravel 使用哪个字段来检索子资源：

```php
use App\Http\Controllers\PhotoCommentController;
 
Route::resource('photos.comments', PhotoCommentController::class)->scoped([
    'comment' => 'slug',
]);

// URI
// /photos/{photo}/comments/{comment:slug}
```



### 本地化资源URI

默认情况下，`Route::resource` 会使用英语动词和复数规则创建资源的 URI。

如果需要本地化 create 和 edit 动作的动词，可以使用 `Route::resourceVerbs` 方法。

这可以在应用程序的 `App\Providers\AppServiceProvider` 中的 boot 方法开始时完成：

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
}
```

![image-20240616152111623](https://raw.githubusercontent.com/XD825/picgo/main/img/202406161521686.png)

### 补充资源控制器

如果需要在资源控制器之外添加额外的路由，超出了默认的资源路由集合，应该在调用 `Route::resource`方法之前定义这些路由；否则，资源方法定义的路由可能会意外地优先于补充路由：

```php
use App\Http\Controller\PhotoController;
 
Route::get('/photos/popular', [PhotoController::class, 'popular']);
Route::resource('photos', PhotoController::class);
```



### 单例资源控制器

有时，应用程序会有一些资源只能有单个实例，那么可以注册一个`singleton`资源控制器：

```php
use App\Http\Controllers\ProfileController;
use Illuminate\Support\Facades\Route;
 
Route::singleton('profile', ProfileController::class);
```

![image-20240616152448895](https://raw.githubusercontent.com/XD825/picgo/main/img/202406161524969.png)

单例实例资源也可以嵌套在标准资源中：

```php
Route::singleton('photos.comments', CommentController::class);
```

![image-20240616152622105](https://raw.githubusercontent.com/XD825/picgo/main/img/202406161526161.png)

#### 可创建的单例资源

可以在注册单例实例资源路由时调用`creatable`方法：

```php
Route::singleton('photos.thumbnail', ThumbnailController::class)->creatable();
```

![image-20240616152754689](https://raw.githubusercontent.com/XD825/picgo/main/img/202406161527772.png)

使用`destroyable`方法注册单例资源的`DELETE`路由，但不注册创建或存储路由：

```php
Route::singleton('photos.comments', CommentController::class)->destroyable();
```

![image-20240616152958757](https://raw.githubusercontent.com/XD825/picgo/main/img/202406161529824.png)

#### API 单例资源

`apiSingleton`方法可用于注册API操作的单例资源：

```php
Route::apiSingleton('photos', PhotoController::class);
```

![image-20240616153144256](https://raw.githubusercontent.com/XD825/picgo/main/img/202406161531335.png)

也可以使用`creatable`将资源注册`store`和`destroy`路由：

```php
Route::apiSingleton('photos', PhotoController::class)->creatable();
```

![image-20240616153251938](https://raw.githubusercontent.com/XD825/picgo/main/img/202406161532998.png)

同样的`destroyable`方法依旧可用：

```php
Route::apiSingleton('photos', PhotoController::class)->destroyable();
```

![image-20240616153347425](https://raw.githubusercontent.com/XD825/picgo/main/img/202406161533488.png)

## 依赖注入和控制器

### 构造函数注入

Laravel的服务容器用于解析所有的Laravel控制器。

因此，可以在控制器的构造函数中对控制器可能需要的任何依赖项进行类型提示。

声明的依赖项将自动被解析并注入到控制器实例中。

```php
<?php
 
namespace App\Http\Controllers;
 
use App\Repositories\UserRepository;
 
class UserController extends Controller
{
    /**
     * Create a new controller instance.
     */
    public function __construct(
        protected UserRepository $users,
    ) {}
}
```

### 方法注入

除了构造函数注入外，还可以在控制器的方法中对依赖项进行类型提示。

一个常见的用例是在控制器方法中注入 `Illuminate\Http\Request` 实例：

```php
<?php
 
namespace App\Http\Controllers;
 
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
 
class UserController extends Controller
{
    /**
     * Store a new user.
     */
    public function store(Request $request): RedirectResponse
    {
        $name = $request->name;
 
        // Store the user...
 
        return redirect('/users');
    }
}
```

如果控制器方法还需要从路由参数中获取输入，请在其他依赖项之后列出路由参数。

例如，如果路由定义如下所示：

```php
use App\Http\Controllers\UserController;
 
Route::put('/user/{id}', [UserController::class, 'update']);
```

仍然可以对 `Illuminate\Http\Request` 进行类型提示，并通过以下方式访问 `id` 参数，定义控制器方法如下所示:

```php
<?php
 
namespace App\Http\Controllers;
 
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
 
class UserController extends Controller
{
    /**
     * Update the given user.
     */
    public function update(Request $request, string $id): RedirectResponse
    {
        // Update the user...
 
        return redirect('/users');
    }
}
```


