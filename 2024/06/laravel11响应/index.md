# Laravel 11.x 响应


----

[原文](https://laravel.com/docs/11.x/responses)

----

## 创建响应

#### 字符串和数组

Laravel框架会自动将字符串转换为完整的HTTP相应：

```php
Route::get('/', function () {
    return 'Hello World';
});
```

会自动将数组转换为JSON响应：

```php
Route::get('/', function () {
    return [1, 2, 3];
});
```

#### 响应对象

通过返回完整`Response`实例，可以自定义相应的HTTP状态码和标头。

`Response`应继承自`Symfony\Component\HttpFoundation\Response`类

```php
Route::get('/home', function () {
    return response('Hello World', 200)
                  ->header('Content-Type', 'text/plain');
});
```

#### Eloquent模型和控制器

可以直接从路由和控制器返回Eloquent ORM模型和集合。会自动将模型和集合转换为JSON响应，同时遵循模型的隐藏属性：

```php
use App\Models\User;
 
Route::get('/user/{user}', function (User $user) {
    return $user;
});
```

### 将Headers附加到响应

> `header`方法为响应添加一系列标头

```php
return response($content)
            ->header('Content-Type', $type)
            ->header('X-Header-One', 'Header Value')
            ->header('X-Header-Two', 'Header Value');
```

> `withHeaders`方法指定要添加到响应中的标头数组

```php
return response($content)
            ->withHeaders([
                'Content-Type' => $type,
                'X-Header-One' => 'Header Value',
                'X-Header-Two' => 'Header Value',
            ]);
```

#### 缓存控制中间件

Laravel包含一个`cache.headers`中间件，可用于快速设置一组路由`Cache-Control`的标头。

指令应使用与相应缓存控制指令等效的“蛇形大小写”来提供，并应用分号分隔。

如果 `etag` 在指令列表中指定，则响应内容的 MD5 哈希值将自动设置为 ETag 标识符：

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



### 将Cookie附加到响应

> `cookie`方法将cookie附加到`Illuminate\Http\Response`实例

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

还接受一些不太频繁使用的参数。通常，这些参数与 PHP 的原生 setcookie 方法的参数具有相同的目的和含义：

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```

如果想确保在发送响应时附带一个 cookie，但还没有该响应的实例，可以使用 Cookie facade 来将 cookie "队列化"，以便在发送响应时附加到响应上。

queue 方法接受创建 cookie 实例所需的参数。

这些 cookie 将在发送到浏览器之前附加到传出的响应上：

```php
use Illuminate\Support\Facades\Cookie;
 
Cookie::queue('name', 'value', $minutes);
```

#### 生成Cookie实例

如果想生成一个可以稍后附加到响应实例的 `Symfony\Component\HttpFoundation\Cookie` 实例，可以使用全局 cookie 辅助函数。

除非该 cookie 被附加到响应实例，否则它不会发送回客户端：

```php
$cookie = cookie('name', 'value', $minutes);
 
return response('Hello World')->cookie($cookie);
```

#### 提前过期Cookie

可以通过传出响应 `withoutCookie` 的方法使 cookie 过期来删除 cookie：

```php
return response('Hello World')->withoutCookie('name');
```

如果还没有传出响应的实例，则可以使用 `Cookie` Facade `expire` 的方法使 cookie 过期：

```php
Cookie::expire('name');
```

### Cookie加密

默认情况下，由于 `Illuminate\Cookie\Middleware\EncryptCookies` 中间件的存在，Laravel 生成的所有 cookie 都经过加密和签名，因此客户端无法修改或读取它们。

如果想禁用应用程序生成的 cookie 子集的加密，可以使用应用程序 `bootstrap/app.php` 文件中的方法 `encryptCookies` ：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->encryptCookies(except: [
        'cookie_name',
    ]);
})
```

## 重定向

重定向响应是 `Illuminate\Http\RedirectResponse` 类的实例，包含将用户重定向到另一个 URL 所需的正确标头。有几种方法可以生成 `RedirectResponse` 实例。最简单的方法是使用全局 `redirect` 帮助程序：

```php
Route::get('/dashboard', function () {
    return redirect('home/dashboard');
});
```

有时，需要将用户重定向到其以前的位置，例如当提交的表单无效时。可以使用全局 `back` 帮助程序函数来执行此操作。由于此功能利用会话，因此请确保调用函数的 `back` 路由使用 `web` 中间件组：

```php
Route::post('/user/profile', function () {
    // Validate the request...
 
    return back()->withInput();
});
```



### 重定向到命名路由

当调用不带参数的 `redirect` 帮助程序时，将返回一个实例 `Illuminate\Routing\Redirector` ，允许调用 `Redirector` 实例上的任何方法。

例如，要生成 `RedirectResponse` 到命名路由，可以使用以下 `route` 方法：

```php
return redirect()->route('login');
```

如果路由具有参数，则可以将它们作为第二个参数传递给 `route` 方法：

```php
// For a route with the following URI: /profile/{id}
 
return redirect()->route('profile', ['id' => 1]);
```

#### 通过Eloquent模型填充参数

如果要重定向到具有“ID”参数的路由，该路由是从 Eloquent 模型填充的，则可以传递模型本身。将自动提取 ID：

```php
// For a route with the following URI: /profile/{id}
 
return redirect()->route('profile', [$user]);
```

如果要自定义放置在路由参数中的值，可以在路由参数定义 （ `/profile/{id:slug}` ） 中指定列，也可以在 Eloquent 模型上重写该 `getRouteKey` 方法：

```php
/**
 * Get the value of the model's route key.
 */
public function getRouteKey(): mixed
{
    return $this->slug;
}
```



### 重定向至控制器操作

还可以生成指向控制器操作的重定向。为此，请将控制器和操作名称传递给 `action` 方法：

```php
use App\Http\Controllers\UserController;
 
return redirect()->action([UserController::class, 'index']);
```

如果控制器路由需要参数，则可以将它们作为第二个参数传递给 `action` 方法：

```php
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```



### 重定向到外部域

可以通过调用该 `away` 方法重定向到应用程序外的域。

该方法将 `RedirectResponse` 创建一个无需任何其他 URL 编码、验证或验证的方法：

```php
return redirect()->away('https://www.google.com');
```



### 使用闪存的会话数量重定向

重定向到新 URL 和将数据刷入会话通常同时完成。

通常，这是在成功执行操作后，当向会话刷新成功消息时完成的。

为方便起见，可以创建一个 `RedirectResponse` 实例，并在单个流畅的方法链中将数据闪存到会话中：

```php
Route::post('/user/profile', function () {
    // ...
 
    return redirect('dashboard')->with('status', 'Profile updated!');
});
```

重定向用户后，可以显示会话中闪烁的消息。例如，使用 Blade 语法：

```php+HTML
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

#### 使用输入重定向

在将用户重定向到新位置之前，可以使用 `RedirectResponse` 实例提供 `withInput` 的方法将当前请求的输入数据刷新到会话。

如果用户遇到验证错误，通常会执行此操作。

将输入切换到会话后，可以在下一次请求重新填充表单时轻松检索它：

```php
return back()->withInput();
```



## 其他响应类型

### View 响应

```php
return response()
            ->view('hello', $data, 200)
            ->header('Content-Type', $type);
```

如果不需要传递自定义 HTTP 状态码或自定义标头，则可以使用全局 `view` 帮助程序函数。

### JSON 响应

该 `json` 方法会自动将 `Content-Type` 标头设置为 `application/json` ，并使用 `json_encode` PHP 函数将给定的数组转换为 JSON：

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA',
]);
```

如果要创建 JSONP 响应，可以将该 `json` 方法与以下 `withCallback` 方法结合使用

```php
return response()
            ->json(['name' => 'Abigail', 'state' => 'CA'])
            ->withCallback($request->input('callback'));
```



### 文件下载

该 `download` 方法可用于生成响应，强制用户的浏览器在给定路径下载文件。

该 `download` 方法接受文件名作为该方法的第二个参数，该参数将确定下载文件的用户看到的文件名。

最后，可以将 HTTP 标头数组作为第三个参数传递给该方法：

```php
return response()->download($pathToFile);
 
return response()->download($pathToFile, $name, $headers);
```

#### 流下载

该 `streamDownload` 方法可用于将给定操作的字符串响应转换为可下载的响应，而无需将操作的内容写入磁盘。

此方法接受回调、文件名和可选的标头数组作为其参数：

```php
use App\Services\GitHub;
 
return response()->streamDownload(function () {
    echo GitHub::api('repo')
                ->contents()
                ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');
```



### 文件响应

该 `file` 方法可用于直接在用户的浏览器中显示文件（如图像或 PDF），而不是启动下载。

此方法接受文件的绝对路径作为其第一个参数，并接受标头数组作为其第二个参数：

```php
return response()->file($pathToFile);
 
return response()->file($pathToFile, $headers);
```



## 响应宏

如果要定义可在各种路由和控制器中重复使用的自定义响应，则可以在 `Response` 外观上使用该 `macro` 方法。

通常，应从应用程序的服务提供商之一（如 `App\Providers\AppServiceProvider` 服务提供商） `boot` 的方法调用此方法：

```php
<?php
 
namespace App\Providers;
 
use Illuminate\Support\Facades\Response;
use Illuminate\Support\ServiceProvider;
 
class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Response::macro('caps', function (string $value) {
            return Response::make(strtoupper($value));
        });
    }
}
```

该 `macro` 函数接受名称作为其第一个参数，接受闭包作为其第二个参数。从 `ResponseFactory` 实现或 `response` 帮助程序调用宏名称时，将执行宏的闭包：

```php
return response()->caps('foo');
```


