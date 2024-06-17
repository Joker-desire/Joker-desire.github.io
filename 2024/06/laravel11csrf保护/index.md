# Laravel 11.x CSRF保护


----

[原文](https://laravel.com/docs/11.x/csrf)

----

## 介绍

跨站请求伪造（CSRF）是一种恶意利用方式，通过该方式未经授权的命令会代表已验证用户执行。

Laravel 提供了简单的方法来保护应用程序免受跨站请求伪造（CSRF）攻击的影响。

## 阻止 CSRF 请求

Laravel自动为应用程序管理的每个活动用户会话生成一个CSRF“token”。

此令牌用于验证已验证用户实际上是正在向应用程序发出请求的人。

由于此令牌存储在用户的会话中，并且每次会话重新生成时都会更改，因此恶意应用程序无法访问它。

可以通过请求的会话或者通过 `csrf_token` 辅助函数来访问当前会话的CSRF令牌

```php
use Illuminate\Http\Request;
 
Route::get('/token', function (Request $request) {
    $token = $request->session()->token();
 
    $token = csrf_token();
 
    // ...
});
```

每当您定义一个 "POST"、"PUT"、"PATCH" 或 "DELETE" 的 HTML 表单时，都应该包含一个隐藏的 CSRF `_token` 字段，以便CSRF保护中间件可以验证请求。

为了方便起见，可以使用 `@csrf` Blade 指令来生成隐藏的令牌输入字段：

```php+HTML
<form method="POST" action="/profile">
    @csrf
 
    <!-- Equivalent to... -->
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```

`Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` 中间件默认包含在 web 中间件组中，它会自动验证请求输入中的令牌是否与会话中存储的令牌匹配。当这两个令牌匹配时，就可以确定已验证的用户是发起请求的用户。

### 从CSRF保护中排除URI

有时需要从CSRF保护中排除一组URI。

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->validateCsrfTokens(except: [
        'stripe/*',
        'http://example.com/foo/bar',
        'http://example.com/foo/*',
    ]);
})
```

为了方便起见，在运行tests时，所有路由都会自动禁用 CSRF 中间件。

## X-CSRF-Token

除了检查作为 POST 参数的 CSRF 令牌外，默认包含在 `web` 中间件组中的 `Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` 中间件还会检查 `X-CSRF-TOKEN` 请求头。

例如，将令牌存储在一个 HTML 元标记中。

```php+HTML
<meta name="csrf-token" content="{{ csrf_token() }}">
```

然后，在发送 AJAX 请求时，可以从元标记中获取令牌，并将其包含在请求头中：

```javascript
let token = document.querySelector('meta[name="csrf-token"]').getAttribute('content');

fetch('/example', {
    method: 'POST',
    headers: {
        'X-CSRF-TOKEN': token,
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({ key: 'value' })
});

```



## X-XSRF-Token

Laravel 将当前的 CSRF 令牌存储在一个加密的 `XSRF-TOKEN` cookie 中，该 cookie 包含在框架生成的每个响应中。可以使用这个 cookie 的值来设置 `X-XSRF-TOKEN` 请求头。

例如，在发送 AJAX 请求时，可以从 `XSRF-TOKEN` cookie 中获取令牌，并将其包含在请求头中：

```javascript
let token = document.cookie.split('; ')
    .find(row => row.startsWith('XSRF-TOKEN='))
    .split('=')[1];

fetch('/example', {
    method: 'POST',
    headers: {
        'X-XSRF-TOKEN': token,
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({ key: 'value' })
});
```

这样，您的请求将包含 `X-XSRF-TOKEN` 请求头，用于验证 CSRF 令牌。

