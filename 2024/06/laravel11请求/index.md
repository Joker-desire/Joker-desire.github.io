# Laravel 11.x 请求


----

[原文](https://laravel.com/docs/11.x/requests)

----

## 介绍

Laravel 的 `Illuminate\Http\Request` 类提供了一种面向对象的方式来与当前应用程序处理的 HTTP 请求进行交互，同时可以检索请求中提交的输入、Cookies 和文件。

## 请求交互

### 访问请求

通过依赖注入进行获取当前HTTP请求的实例：

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
        $name = $request->input('name');
 
        // Store the user...
 
        return redirect('/users');
    }
}
```

同样的也可以在闭包中使用：

```php
use Illuminate\Http\Request;
 
Route::get('/', function (Request $request) {
    // ...
});
```

#### 依赖注入和路由参数

如果控制器方法也需要来自路由参数的输入，则应该在其他依赖项之后列出路由参数。

例如，路由定义如下：

```php
use App\Http\Controllers\UserController;
 
Route::put('/user/{id}', [UserController::class, 'update']);
```

那么控制器就应该按照以下方式进行定义：

```php
<?php
 
namespace App\Http\Controllers;
 
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
 
class UserController extends Controller
{
    /**
     * Update the specified user.
     */
    public function update(Request $request, string $id): RedirectResponse
    {
        // Update the user...
 
        return redirect('/users');
    }
}
```

### 请求路由、主机和方法

#### 检索请求路径

> `path`方法返回请求的路径信息。

```php
$uri = $request->path();
```

#### 检查请求路径/路由

> `is`方法允许验证传入请求路径是否与给定模式匹配。
>
> 可以将`*`字符用作通配符

```php
if ($request->is('admin/*')) {
    // ...
}
```

> `routeIs`方法，可以确定出入请求是否与命名路由匹配：

```php
if ($request->routeIs('admin.*')) {
    // ...
}
```

#### 检索请求URL

> `url`或`fullUrl`方法可以检索传入请求的完整URL。
>
> `url`方法将返回不带查询字符串的URL
>
> `fullUrl`方法返回的URL包含查询字符串

```php
$url = $request->url();
 
$urlWithQueryString = $request->fullUrl();
```

> `fullUrlWithQuery`方法，可以将查询字符串数据追加到当前URL。
>
> 此方法将给定的查询字符串变量数组与当前查询字符串合并

```php
$request->fullUrlWithQuery(['type' => 'phone']);
```

> `fullUrlWithoutQuery`方法在没有给定查询字符串参数的情况下获取当前URL

```php
$request->fullUrlWithoutQuery(['type']);
```

#### 检索请求主机

> `host`、`httpHost`和`schemeAndHttpHost`方法可以检索传入请求的主机

```php
$request->host();
$request->httpHost();
$request->schemeAndHttpHost();
```

#### 检索请求方法

> `method`方法将返回请求的方式
>
> `isMethod`方法用来验证HTTP请求方式是否与给定字符串匹配

```php
$method = $request->method();
 
if ($request->isMethod('post')) {
    // ...
}
```



### 请求标头

> `header`方法从`Illuminate\Http\Request`实例中检索请求标头。
>
> 如果不存在，则返回`null`，`header`方法接受可选的第二个参数，如果不存在，则使用此参数

```php
$value = $request->header('X-Header-Name');
 
$value = $request->header('X-Header-Name', 'default');
```

> `hasHeader`方法用于确定请求是否包含给定的标头

```php
if ($request->hasHeader('X-Header-Name')) {
    // ...
}
```

> `bearerToken`方法可用于从`Authorization`标头中检索持有者令牌。
>
> 如果不存在，则返回空字符串

```php
$token = $request->bearerToken();
```



### 请求IP地址

> `ip`方法用于检索发出请求的IP地址

```php
$ipAddress = $request->ip();
```

> `ips`方法可以检索IP地址数组，包括代理转发的所有客户端IP地址
>
> 原始客户端IP地址将位于数组的末尾

```php
$ipAddresses = $request->ips();
```

### 内容协商

Laravel 提供了几种方法，用于通过 `Accept` 标头检查传入请求的请求内容类型。

> `getAcceptableContentTypes`方法返回一个数组，其中包含请求接受的所有内容类型。

```php
$contentTypes = $request->getAcceptableContentTypes();
```

> `accepts`方法接受内容类型的数组，
>
> 如果请求接受任何内容类型，则返回`true`，否则`false`

```php
if ($request->accepts(['text/html', 'application/json'])) {
    // ...
}
```

> `prefers`方法用来确定请求在给定的一组内容类型中最喜欢哪种内容类型。
>
> 如果请求不接受所提供的任何内容类型，则会返回`null`

```php
$preferred = $request->prefers(['text/html', 'application/json']);
```

> `expectsJson`方法用于快速确定传入请求是否需要JSON响应

```php
if ($request->expectsJson()) {
    // ...
}
```



### PRS-7 请求

PSR-7 标准指定了 HTTP 消息的接口，包括请求和响应。如果要获取 PSR-7 请求的实例而不是 Laravel 请求，则首先需要安装一些库。Laravel 使用 Symfony HTTP 消息桥接组件将典型的 Laravel 请求和响应转换为兼容 PSR-7 的实现：

```bash
composer require symfony/psr-http-message-bridge
composer require nyholm/psr7
```

安装后，就可以通过在路由闭包或控制器方法上键入提示请求接口来获取PSR-7请求：

```php
use Psr\Http\Message\ServerRequestInterface;
 
Route::get('/', function (ServerRequestInterface $request) {
    // ...
});
```



## 输入

### 检索输入

#### 检索所有输入数据

> `all`方法检索`array`所有传入入请求的输入数据。
>
> 无论请求来组HTML表单还是XHR请求，都适用。

```php
$input = $request->all();
```

> `collect`方法可以将传入请求的所有输入数据作为集合检索

```php
$input = $request->collect();
```

`collect`方法还允许将传入的输入的子集作为集合检索

```php
$request->collect('users')->each(function (string $user) {
    // ...
});
```

#### 检索输入值

> `input`方法用于检索用户输入
>
> 无参数调用，将所有输入值作为关联数组检索

```php
$name = $request->input('name');

$input = $request->input();

// 如果请求的输入值不存在，则可以设置默认值：
$name = $request->input('name', 'Sally');
```

> 使用`.`表示法访问数组

```php
$name = $request->input('products.0.name');
 
$names = $request->input('products.*.name');
```

#### 从查询字符串中检索输入

> `query`方法将仅从查询字符串中检索值
>
> 无参数调用，将所有查询字符串值检索为关联数组

```php
$name = $request->query('name');

$name = $request->query('name', 'Helen');

// 不存在则使用给定的默认值
$name = $request->query('name', 'Helen');
```

#### 检索JSON输入值

> 发送JSON请求时，只要请求`Content-Type=application/json`就可以通过`input`方法访问JSON数据。
>
> 也可以使用`.`语法来检索嵌套在JSON数组/对象中的值

```php
$name = $request->input('user.name');
```

#### 检索字符串输入值

> `string`方法可以将请求数据作为`Illuminate\Support\Stringable`实例进行检索

```php
$name = $request->string('name')->trim();
```

#### 检索布尔值输入值

> `boolean`方法将请求数据检索为布尔值

```php
$archived = $request->boolean('archived');
```

#### 检索日期输入值

> `date`方法将包含日期/时间的输入值作为Carbon实例进行检索。
>
> 如果不存在，则返回`null`

```php
$birthday = $request->date('birthday');
```

> `date`方法接受的第二三参数可以分别用于指定日期的格式和时区
>
> 如果格式无效，则会抛出`InvalidArgumentException`异常

```php
$elapsed = $request->date('elapsed', '!H:i', 'Europe/Madrid');
```

#### 检索枚举输入值

> `enum`方法可以从请求中检索与PHP枚举相对应的输入值。
>
> 如果请求不包括具有给定名称的输入值，或者枚举没有输入值匹配的支持值，则返回`null`
>
> `enum`方法接受输入值的名称和枚举类作为其第一二参数

```php
use App\Enums\Status;
 
$status = $request->enum('status', Status::class);
```

#### 通过动态属性检索输入

> 可以使用`Illuminate\Http\Request`实例上的动态属性访问用户输入

```php
$name = $request->name;
```

#### 检索部分输入数据

> `only`和`except`方法可以检索输入数据的子集
>
> 两个方法都接受单个array或动态参数列表
>
> `only`方法不会返回请求中不存在的键值对

```php
$input = $request->only(['username', 'password']);
 
$input = $request->only('username', 'password');
 
$input = $request->except(['credit_card']);
 
$input = $request->except('credit_card');
```

### 输入状态

> `has`方法可以确定请求中是否存在值。存在返回true

```php
if ($request->has('name')) {
    // ...
}
```

> `has`还可以接收一个数组，那么将确定是否存在所有指定的值

```php
if ($request->has(['name', 'email'])) {
    // ...
}
```

> `hasAny`如果存在任意指定值，则返回true

```php
if ($request->hasAny(['name', 'email'])) {
    // ...
}
```

> `whenHas`方法如果请求上存在值则将执行给定的闭包

```php
$request->whenHas('name', function (string $input) {
    // ...
});
```

> `whenHas`方法，如果请求中不存在指定的值，则可以将第二个闭包传递给将要执行的方法中

```php
$request->whenHas('name', function (string $input) {
    // The "name" value is present...
}, function () {
    // The "name" value is not present...
});
```

> `filled`方法用来确定请求中是否存在值并且不是空字符串

```php
if ($request->filled('name')) {
    // ...
}
```

> `anyFilled`方法如果任意一个指定的值不是空字符串，则返回true

```php
if ($request->anyFilled(['name', 'email'])) {
    // ...
}
```

> `whenFilled`如果请求上存在值并且不是空字符串，将执行给定的闭包

```php
$request->whenFilled('name', function (string $input) {
    // ...
});
```

> `whenFilled`如果不存在，则可以传递第二个闭包进行处理

```php
$request->whenFilled('name', function (string $input) {
    // The "name" value is filled...
}, function () {
    // The "name" value is not filled...
});
```

> `missing`和`whenMissing`方法用于确定请求中是否缺少给定的key

```php
if ($request->missing('name')) {
    // ...
}
 
$request->whenMissing('name', function () {
    // The "name" value is missing...
}, function () {
    // The "name" value is present...
});
```



### 合并其他输入

> `merge`方法，用于手动将其他输入合并到请求的现有输入数据中。
>
> 如果请求中已存在给定的输入key，则将会被覆盖掉

```php
$request->merge(['votes' => 0]);
```

> 如果请求的输入数据中尚不存在响应的key，则`mergeIfMissing`方法可用于将输入合并到请求中

```php
$request->mergeIfMissing(['votes' => 0]);
```

#### 刷新会话输入

> `flash`方法将刷新会话的当前输入。
>
> 以便在用户对应用程序的下一个请求期间可用

```php
$request->flash();
```

> `flashOnly` 和 `flashExcept` 方法用于将敏感信息排除在会话之外

```php
$request->flashOnly(['username', 'email']);
 
$request->flashExcept('password');
```

#### 刷新输入并重定向

> `withInput`方法可以刷新输入并进行重定向

```php
return redirect('form')->withInput();
 
return redirect()->route('user.create')->withInput();
 
return redirect('form')->withInput(
    $request->except('password')
);
```

#### 检索旧输入

> `old`方法将从会话中提取先前刷新的输入数据

```php
$username = $request->old('username');
```

### Cookies

#### 从请求中检索Cookie

> `cookie`方法将从请求中进行检索Cookie值

```php
$value = $request->cookie('name');
```



### 输入修整和归一化

默认情况下，Laravel将`Illuminate\Foundation\Http\Middleware\TrimStrings` and `Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull`中间件包含在应用程序的全局中间件中了。这些中间件将自动修剪请求上的所有输入字符串字段，并将任何空字符串转换为`null`

#### 禁用输入归一化

如果要对所有请求禁用此行为，可以通过调用应用程序 `bootstrap/app.php` 文件中 `$middleware->remove` 的方法从应用程序的中间件中删除两个中间件：

```php
use Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull;
use Illuminate\Foundation\Http\Middleware\TrimStrings;
 
->withMiddleware(function (Middleware $middleware) {
    $middleware->remove([
        ConvertEmptyStringsToNull::class,
        TrimStrings::class,
    ]);
})
```

如果想要在应用程序中禁用对一部分请求的字符串修剪和空字符串转换，可以在应用程序的 `bootstrap/app.php` 文件中使用 `trimStrings` 和 `convertEmptyStringsToNull` 中间件方法。这两个方法都接受一个闭包数组，闭包应返回 true 或 false，以指示是否应跳过输入的规范化处理：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->convertEmptyStringsToNull(except: [
        fn (Request $request) => $request->is('admin/*'),
    ]);
 
    $middleware->trimStrings(except: [
        fn (Request $request) => $request->is('admin/*'),
    ]);
})
```



## 文件

### 检索上传的文件

> `file`/`Illuminate\Http\Request`动态属性用于检索上传的文件
>
> `file`方法返回`Illuminate\Http\UploadedFile`类实例，

```php
$file = $request->file('photo');
 
$file = $request->photo;
```

> `hasFile`方法确定请求中是否存在文件

```php
if ($request->hasFile('photo')) {
    // ...
}
```

#### 验证成功上传

> `isValid`方法用于验证上传文件是否成功

```php
if ($request->file('photo')->isValid()) {
    // ...
}
```

#### 文件路径和扩展名

> 该 `UploadedFile` 类还包含用于访问文件的完全限定路径及其扩展名的方法。
>
> 该 `extension` 方法将尝试根据文件的内容猜测文件的扩展名。
>
> 此扩展可能与客户端提供的扩展不同

```php
$path = $request->photo->path();
 
$extension = $request->photo->extension();
```



### 存储上传的文件

> `UploadedFile`类的`store`方法将上传的文件移动到磁盘中，该磁盘可以是本地文件系统某一位置，也可以是 Amazon S3等云存储位置
>
> `store`方法接受存储文件的路径，此路径不包含文件名，自动生成唯一ID作为文件名

```php
$path = $request->photo->store('images');
 
$path = $request->photo->store('images', 's3');
```

> 如果不想自动生成文件名，则可以使用路径、文件名和磁盘名称作为其参数`storeAs`的方法

```php
$path = $request->photo->storeAs('images', 'filename.jpg');
 
$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
```

## 配置受信任的代理

启用Laravel应用程序中包含的`Illuminate\Http\Middleware\TrustProxies`中间件，可以快速自定义应用程序应信任的负载均衡或代理。

使用`bootstrap/app.php`文件中的`trustProxies`中间件方法指定受信任的代理：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->trustProxies(at: [
        '192.168.1.1',
        '192.168.1.2',
    ]);
})
```

也可以配置应受信任的代理标头：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->trustProxies(headers: Request::HEADER_X_FORWARDED_FOR |
        Request::HEADER_X_FORWARDED_HOST |
        Request::HEADER_X_FORWARDED_PORT |
        Request::HEADER_X_FORWARDED_PROTO |
        Request::HEADER_X_FORWARDED_AWS_ELB
    );
})
```

#### 信任所有代理

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->trustProxies(at: '*');
})
```



## 配置受信任的主机

启动`Illuminate\Http\Middleware\TrustHosts` 中间件来实现配置受信任的主机。

在 `bootstrap/app.php` 文件中调用 `trustHosts` 中间件方法。使用此方法的 `at` 参数，可以指定应用程序应响应的主机名。带有其他 `Host` 标头的传入请求将被拒绝：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->trustHosts(at: ['laravel.test']);
})
```

默认情况下，来自应用程序 URL 的子域的请求也会自动受信任。如果要禁用此行为，可以使用以下 `subdomains` 参数：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->trustHosts(at: ['laravel.test'], subdomains: false);
})
```


