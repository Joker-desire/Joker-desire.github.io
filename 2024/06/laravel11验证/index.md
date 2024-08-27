# Laravel 11.x 验证


----

[原文](https://laravel.com/docs/11.x/validation)

----

## 介绍

Laravel提供了多种不同的方法来验证应用程序的传入数据。最常用的方法是在所有传入的HTTP请求上使用`validate`方法。

## 验证快速入门

### 定义路由

```php
use App\Http\Controllers\PostController;
 
Route::get('/post/create', [PostController::class, 'create']);
Route::post('/post', [PostController::class, 'store']);
```

`GET` 路由将显示一个表单，供用户创建新的博客文章，而 `POST` 路由将新博客文章存储在数据库中。

### 创建控制器

```php
<?php
 
namespace App\Http\Controllers;
 
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\View\View;
 
class PostController extends Controller
{
    /**
     * Show the form to create a new blog post.
     */
    public function create(): View
    {
        return view('post.create');
    }
 
    /**
     * Store a new blog post.
     */
    public function store(Request $request): RedirectResponse
    {
        // Validate and store the blog post...
 
        $post = /** ... */
 
        return to_route('post.show', ['post' => $post->id]);
    }
}
```

### 编写验证逻辑

使用 `Illuminate\Http\Request` 对象提供 `validate` 的方法。

如果验证规则通过，代码将继续正常执行;但是，如果验证失败，将引发异常 `Illuminate\Validation\ValidationException` ，并且将自动将正确的错误响应发送回用户。

如果在传统 HTTP 请求期间验证失败，将生成对上一个 URL 的重定向响应。

如果传入请求是 XHR 请求，则将返回包含验证错误消息的 JSON 响应。

```php
/**
 * Store a new blog post.
 */
public function store(Request $request): RedirectResponse
{
    $validated = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);
 
    // The blog post is valid...
 
    return redirect('/posts');
}
```

也可以将验证规则指定为规则数组，而不是单个 `|` 分隔字符串：

```php
$validatedData = $request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

此外，还可以使用该 `validateWithBag` 方法验证请求并将任何错误消息存储在命名的错误包中：

```php
$validatedData = $request->validateWithBag('post', [
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

#### 首次验证失败时停止

> `bail`规则在第一次验证失败后停止对属性运行验证其余规则

```php
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

在上面示例中，如果属性`unique`上的规则失败，则不会进行后续的`max`规则检查了。

#### 关于嵌套属性的说明

如果传入的HTTP请求包含嵌套字段数据，则可以使用`.`语法在验证规则中指定这些字段：

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```

如果字段名称包含文字句号，则可以通过使用反斜杠转义来明确防止`.`语法异常：

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'v1\.0' => 'required',
]);
```

### 显示验证错误

一个 `$errors` 变量通过 `Illuminate\View\Middleware\ShareErrorsFromSession` 中间件共享给应用程序的所有视图，该中间件由 `web` 中间件组提供。

当应用此中间件时，一个 `$errors` 变量将始终在视图中可用，允许假设 `$errors` 变量总是定义的并且可以安全地使用。

 `$errors` 变量将是 `Illuminate\Support\MessageBag` 的一个实例。

在上面示例中，当验证失败时，用户将被重定向到控制器`create`的方法，从而允许在视图中显示错误消息：

```php
<!-- /resources/views/post/create.blade.php -->
 
<h1>Create Post</h1>
 
@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
 
<!-- Create Post Form -->
```

#### 自定义错误消息

Laravel 的内置验证规则每个规则都有一个错误消息，该错误消息位于应用程序 `lang/en/validation.php` 的文件中。

如果应用程序没有 `lang` 目录，则可以指示 Laravel 使用 `lang:publish` Artisan 命令创建它。

```bash
php artisan lang:publish
```

#### XHR请求和验证

在 XHR 请求期间使用该 `validate` 方法时，Laravel 不会生成重定向响应。

相反，Laravel 会生成包含所有验证错误的 JSON 响应。

此 JSON 响应将使用 422 HTTP 状态代码发送。

#### @error指令

可以使用 `@error` Blade 指令快速确定给定属性是否存在验证错误消息。

在 `@error` 指令中，可以回显 `$message` 变量以显示错误消息：

```php+HTML
<!-- /resources/views/post/create.blade.php -->
 
<label for="title">Post Title</label>
 
<input id="title"
    type="text"
    name="title"
    class="@error('title') is-invalid @enderror">
 
@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

如果使用命名的错误包，可以将错误包的名称作为第二个参数传递给 `@error` 指令：

```php+HTML
<input ... class="@error('title', 'post') is-invalid @enderror">
```

### 重新填充表单

当 Laravel 由于验证错误而生成重定向响应时，框架会自动将请求的所有输入刷新到会话中。

这样做是为了在下一个请求期间方便地访问输入，并重新填充用户尝试提交的表单。

若要从上一个请求中检索刷新输入，可以在 `Illuminate\Http\Request` 实例上调用 `old` 方法。

该 `old` 方法将从会话中提取先前刷新的输入数据：

```php
$title = $request->old('title');
```

Laravel 还提供了一个全局 `old` 。如果要在 Blade 模板中显示旧输入，则使用 `old` 帮助程序重新填充表单会更方便。

如果给定字段不存在旧输入， `null` 则将返回：

```php+HTML
<input type="text" name="title" value="{{ old('title') }}">
```

### 可选字段的说明

默认情况下，Laravel将 `TrimStrings` and `ConvertEmptyStringsToNull` 中间件包含在应用程序的全局中间件中。

因此，如果不希望验证器将空值视为无效值，通常需要将“可选的”请求字段标记为可空。比如：

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
    'publish_at' => 'nullable|date', // 可以是有效的日期也可以是null
]);
```

### 验证错误响应格式

当应用程序抛出一个 `Illuminate\Validation\ValidationException` 异常并且传入的 HTTP 请求期望得到一个 JSON 响应时，Laravel 会自动进行格式化错误信息并返回一个 422 Unprocessable Entity（无法处理的实体）HTTP 响应。

```json
{
    "message": "The team name must be a string. (and 4 more errors)",
    "errors": {
        "team_name": [
            "The team name must be a string.",
            "The team name must be at least 1 characters."
        ],
        "authorization.role": [
            "The selected authorization.role is invalid."
        ],
        "users.0.email": [
            "The users.0.email field is required."
        ],
        "users.2.email": [
            "The users.2.email must be a valid email address."
        ]
    }
}
```

## 表单请求验证

### 创建表单请求

对于更复杂的验证方法，需要创建一个表单请求。表单请求时封装其自己的验证和授权逻辑的自定义请求类。

可以使用`make:request`命令进行创建：

```bash
php artisan make:request StorePostRequest
```

生成的表单请求列防止在目录`app/Http/Requests`中。包含两个方法：`authorize`和`rules`

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePostRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     */
    public function authorize(): bool
    {
        return false;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array<string, \Illuminate\Contracts\Validation\ValidationRule|array<mixed>|string>
     */
    public function rules(): array
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }
}

```

验证规则，只需对控制器方法中的请求进行类型提示即可：

```php
/**
 * Store a new blog post.
 */
public function store(StorePostRequest $request): RedirectResponse
{
    // The incoming request is valid...
 
    // Retrieve the validated input data...
    $validated = $request->validated();
 
    // Retrieve a portion of the validated input data...
    $validated = $request->safe()->only(['name', 'email']);
    $validated = $request->safe()->except(['name', 'email']);
 
    // Store the blog post...
 
    return redirect('/posts');
}
```

#### 执行其他验证

> 可以使用`after`方法在初始化验证完成后执行其他验证。
>
> 该 `after` 方法应返回一个可调用对象或闭包数组，这些数组将在验证完成后调用。
>
> 给定的可调用对象将收到一个 `Illuminate\Validation\Validator` 实例，允许引发其他错误消息。

```php
use Illuminate\Validation\Validator;
 
/**
 * Get the "after" validation callables for the request.
 */
public function after(): array
{
    return [
        function (Validator $validator) {
            if ($this->somethingElseIsInvalid()) {
                $validator->errors()->add(
                    'field',
                    'Something is wrong with this field!'
                );
            }
        }
    ];
}
```

> `after` 方法返回的数组也可能包含可调用的类。
>
> 这些类 `__invoke` 的方法将接收一个 `Illuminate\Validation\Validator` 实例。

```php
use App\Validation\ValidateShippingTime;
use App\Validation\ValidateUserStatus;
use Illuminate\Validation\Validator;
 
/**
 * Get the "after" validation callables for the request.
 */
public function after(): array
{
    return [
        new ValidateUserStatus,
        new ValidateShippingTime,
        function (Validator $validator) {
            //
        }
    ];
}
```

#### 在首次验证失败时停止

通过将 `stopOnFirstFailure` 属性添加到请求类中，可以通知验证程序，一旦发生单个验证失败，它就应该停止验证所有属性：

```php
/**
 * Indicates if the validator should stop on the first rule failure.
 *
 * @var bool
 */
protected $stopOnFirstFailure = true;
```

#### 自定义重定向位置

通过`$redirect`属性，可以自定义重定向位置：

```php
/**
 * The URI that users should be redirected to if validation fails.
 *
 * @var string
 */
protected $redirect = '/dashboard';
```

如果要将用户重定向到命名路由，可以改为定义`$redirectRoute`属性：

```php
/**
 * The route that users should be redirected to if validation fails.
 *
 * @var string
 */
protected $redirectRoute = 'dashboard';
```

### 授权表单请求

表单请求类还包含一个`authorize`方法。在此方法中，可以确定经过身份验证的用户是否有权限更新给定的资源。

```php
use App\Models\Comment;
 
/**
 * Determine if the user is authorized to make this request.
 */
public function authorize(): bool
{
    $comment = Comment::find($this->route('comment'));
 
    return $comment && $this->user()->can('update', $comment);
}
```

由于所有表单请求都扩展自基本的 Laravel 请求类，我们可以使用 `user` 方法来访问当前已认证的用户。

另外，请注意上面示例中对 `route` 方法的调用。这个方法允许你访问被调用路由上定义的 URI 参数，例如下面示例中的 `{comment}` 参数：

```php
Route::post('/comment/{comment}');
```

因此，如果应用程序利用了路由模型绑定，通过将解析后的模型作为请求的一个属性来访问，可以使代码更加简洁：

```php
return $this->user()->can('update', $this->comment);
```

如果 `authorize` 该方法返回 `false` ，则将自动返回具有 403 状态代码的 HTTP 响应，并且不会执行控制器方法。

如果计划在应用程序的另一部分处理请求的授权逻辑，则可以完全删除该 `authorize` 方法，或者直接返回 `true` ：

```php
/**
 * Determine if the user is authorized to make this request.
 */
public function authorize(): bool
{
    return true;
}
```

### 自定义错误消息

可以通过重写`message`方法来自定义表单请求使用的错误消息。

```php
/**
 * Get the error messages for the defined validation rules.
 *
 * @return array<string, string>
 */
public function messages(): array
{
    return [
        'title.required' => 'A title is required',
        'body.required' => 'A message is required',
    ];
}
```

#### 自定义验证属性

Laravel 的许多内置验证规则错误消息都包含占 `:attribute` 位符。

如果希望将验证消息的 `:attribute` 占位符替换为自定义属性名称，则可以通过重写 `attributes` 方法指定自定义名称。

此方法应返回属性/名称对的数组：

```php
/**
 * Get custom attributes for validator errors.
 *
 * @return array<string, string>
 */
public function attributes(): array
{
    return [
        'email' => 'email address',
    ];
}
```

### 准备用于验证的输入

如果需要在应用验证规则之前准备或清理请求中的任何数据，可以使用以下 `prepareForValidation` 方法：

```php
use Illuminate\Support\Str;
 
/**
 * Prepare the data for validation.
 */
protected function prepareForValidation(): void
{
    $this->merge([
        'slug' => Str::slug($this->slug),
    ]);
}
```

同样，如果需要在验证完成后规范化任何请求数据，可以使用以下 `passedValidation` 方法：

```php
/**
 * Handle a passed validation attempt.
 */
protected function passedValidation(): void
{
    $this->replace(['name' => 'Taylor']);
}
```

## 手动创建验证程序

如果不想再请求上使用`validate`方法，则可以使用`Validator` Facade 手动创建验证器实例。使用`make`方法生成一个新的验证器实例。

```php
<?php
 
namespace App\Http\Controllers;
 
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;
 
class PostController extends Controller
{
    /**
     * Store a new blog post.
     */
    public function store(Request $request): RedirectResponse
    {
        $validator = Validator::make($request->all(), [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);
 
        if ($validator->fails()) {
            return redirect('post/create')
                        ->withErrors($validator)
                        ->withInput();
        }
 
        // Retrieve the validated input...
        $validated = $validator->validated();
 
        // Retrieve a portion of the validated input...
        $validated = $validator->safe()->only(['name', 'email']);
        $validated = $validator->safe()->except(['name', 'email']);
 
        // Store the blog post...
 
        return redirect('/posts');
    }
}
```

传递给`make`方法的第一个参数是正在验证的数据。第二个参数是应该应用于数据的验证规则数组。

确定请求验证是否失败后，可以使用该`withErrors`方法将错误消息刷新到会话。

使用此方法时，`$errors`变量将在重定向后自动与视图共享，从而显示给用户。

`withErrors`方法接受验证器、`MessageBag`或者PHP `array`

#### 首次验证失败时停止

`stopOnFirstFailure` 方法将通知验证器，一旦发生单个验证失败，它就应该停止验证所有属性：

```php
if ($validator->stopOnFirstFailure()->fails()) {
    // ...
}
```

### 自动重定向

手动创建验证器实例，仍需要利用HTTP请求`validate`方法提供的自动重定向，则可以在现有验证器实例上调用该`validate`方法。

如果验证失败，用户将自动被重定向，如果是XHR请求，则将返回JSON响应：

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validate();
```

如果验证失败，可以使用该 `validateWithBag` 方法将错误消息存储在命名的错误包中：

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validateWithBag('post');
```

### 命名错误包

如果单个页面上有多个表单，则可能希望命名 `MessageBag` 包含验证错误的名称，以便检索特定表单的错误消息。

为此，请将名称作为第二个参数传递给 `withErrors` ：

```php
return redirect('register')->withErrors($validator, 'login');
```

然后，可以从 `$errors` 变量访问命名 `MessageBag` 实例：

```php+HTML
{{ $errors->login->first('email') }}
```

### 自定义错误消息

可以将自定义消息作为第三个参数传递给`Validator::make`方法，用来自定义错误消息

```php
$validator = Validator::make($input, $rules, $messages = [
    'required' => 'The :attribute field is required.',
]);
```

在此示例中， `:attribute` 占位符将替换为正在验证的字段的实际名称。

还可以在验证消息中使用其他占位符。例如：

```php
$messages = [
    'same' => 'The :attribute and :other must match.',
    'size' => 'The :attribute must be exactly :size.',
    'between' => 'The :attribute value :input is not between :min - :max.',
    'in' => 'The :attribute must be one of the following types: :values',
];
```

#### 为给定属性指定自定义消息

仅为特定属性指定自定义错误消息。可以使用`.`表示法：

```php
$messages = [
    'email.required' => 'We need to know your email address!',
];
```

#### 指定自定义属性值

Laravel 的许多内置错误消息都包含一个 `:attribute` 占位符，该占位符将替换为正在验证的字段或属性的名称。

若要自定义用于替换特定字段的这些占位符的值，可以将自定义属性数组作为第四个参数传递给 `Validator::make` 该方法：

```php
$validator = Validator::make($input, $rules, $messages, [
    'email' => 'email address',
]);
```

### 执行其他验证

在初始验证完成后执行其他验证。可以使用验证器`after`方法完成该操作。

该方法接受一个闭包或者一个可调用对象数组，这些可调用对象将在验证完成后调用。

给定的可调用对象将收到一个 `Illuminate\Validation\Validator` 实例，允许引发其他错误消息：

```php
use Illuminate\Support\Facades\Validator;
 
$validator = Validator::make(/* ... */);
 
$validator->after(function ($validator) {
    if ($this->somethingElseIsInvalid()) {
        $validator->errors()->add(
            'field', 'Something is wrong with this field!'
        );
    }
});
 
if ($validator->fails()) {
    // ...
}
```

该`after`方法还接受可调用对象数组，如果验证后逻辑封装在可调用类中，这将特别方便，这些类将通过其`__invoke`方法接收实例`Illuminate\Validation\Validator`：

```php
use App\Validation\ValidateShippingTime;
use App\Validation\ValidateUserStatus;
 
$validator->after([
    new ValidateUserStatus,
    new ValidateShippingTime,
    function ($validator) {
        // ...
    },
]);
```

## 使用经过验证的输入

在表单请求或验证程序实例上调用该 `validated` 方法。返回已验证的数据数组：

```php
$validated = $request->validated();
 
$validated = $validator->validated();
```

还可以在表单请求或验证程序实例上调用该 `safe` 方法。

此方法返回的 `Illuminate\Support\ValidatedInput` 实例。

此对象公开 `only` 、 `except` 和 `all` 方法，用于检索已验证数据的子集或已验证数据的整个数组：

```php
$validated = $request->safe()->only(['name', 'email']);
 
$validated = $request->safe()->except(['name', 'email']);
 
$validated = $request->safe()->all();
```

`Illuminate\Support\ValidatedInput` 实例可以像数组一样进行迭代和访问：

```php
// Validated data may be iterated...
foreach ($request->safe() as $key => $value) {
    // ...
}
 
// Validated data may be accessed as an array...
$validated = $request->safe();
 
$email = $validated['email'];
```

如果要向验证的数据添加其他字段，可以调用以下 `merge` 方法：

```php
$validated = $request->safe()->merge(['name' => 'Taylor Otwell']);
```

如果要将已验证的数据作为集合实例进行检索，可以调用以下 `collect` 方法：

```php
$collection = $request->safe()->collect();
```

## 使用错误消息

在实例`Validator`上调用该`errors`方法后，将收到一个`Illuminate\Support\MessageBag`实例，该实例具有多种处理错误消息的便携方法。

自动提供给所有视图的`$errors`变量也是该`MessageBag`类的实例。

#### 检索字段的首次错误消息

> `first`方法可以检索给定字段的第一条错误消息

```php
$errors = $validator->errors();
 
echo $errors->first('email');
```

#### 检索字段的所有错误消息

> `get`方法可以检索给定字段的所有消息的数组

```php
foreach ($errors->get('email') as $message) {
    // ...
}
```

如果要验证数组表单字段，则可以使用以下 `*` 字符检索每个数组元素的所有消息：

```php
foreach ($errors->get('attachments.*') as $message) {
    // ...
}
```

#### 检索所有字段的所有错误消息

> `all`方法可以检索所有字段的所有消息的数组

```php
foreach ($errors->all() as $message) {
    // ...
}
```

#### 确定字段是否存在消息

> `has`方法用于确定给定字段是否存在任何错误消息

```php
if ($errors->has('email')) {
    // ...
}
```

### 在语言文件中指定自定义消息

Laravel 的内置验证规则每个规则都有一个错误消息，该错误消息位于应用程序 `lang/en/validation.php` 的文件中。

如果应用程序没有 `lang` 目录，可以使用 `lang:publish` Artisan 命令创建它。

在`lang/en/validation.php` 文件中，可以根据应用程序的需要调整这些消息。

此外，也可以将此文件复制到其他语言目录，以翻译应用程序语言的消息。

#### 特定属性的自定义消息

可以在应用程序的验证语言文件中自定义用于指定属性和规则组合的错误消息。为此，请将消息自定义项添加到应用程序 `lang/xx/validation.php` 语言文件的 `custom` 数组中：

```php
'custom' => [
    'email' => [
        'required' => 'We need to know your email address!',
        'max' => 'Your email address is too long!'
    ],
],
```

### 在语言文件中指定属性

Laravel 的许多内置错误消息都包含一个 `:attribute` 占位符，该占位符将替换为正在验证的字段或属性的名称。

如果要将验证消息 `:attribute` 的部分替换为自定义值，可以在 `lang/xx/validation.php` 语言文件的 `attributes` 数组中指定自定义属性名称：

```php
'attributes' => [
    'email' => 'email address',
],
```

### 在语言文件中指定值

Laravel 一些内置的验证规则错误消息包含一个 `:value` 占位符，该占位符会被请求属性的当前值替换。

然而，有时可能需要将验证消息中的 `:value` 部分替换为自定义的值表示。

例如，考虑如下规则，该规则规定如果 `payment_type` 的值为 `cc`，则要求提供信用卡号码：

```php
Validator::make($request->all(), [
    'credit_card_number' => 'required_if:payment_type,cc'
]);
```

如果此验证规则失败，它将生成以下错误消息：

```tex
The credit card number field is required when payment type is cc.
```

可以在 `lang/xx/validation.php` 语言文件中定义一个 `values` 数组，以更用户友好的值表示方式，来替换显示 `cc` 作为支付类型值的情况：

```php
'values' => [
    'payment_type' => [
        'cc' => 'credit card'
    ],
],
```

定义此值后，验证规则将生成如下错误消息：

```php
The credit card number field is required when payment type is credit card.
```

## 可用的验证规则

详细见：[可用的验证规则](https://laravel.com/docs/11.x/validation#available-validation-rules)

## 有条件的添加规则

#### 当字段具有特定值时跳过验证

有时可能希望在另一个字段具有特定值的情况下不验证某个字段。

可以使用 `exclude_if` 验证规则来实现这一点。

在这个例子中，如果 `has_appointment` 字段的值为 `false`，则 `appointment_date` 和 `doctor_name` 字段将不会被验证：

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($data, [
    'has_appointment' => 'required|boolean',
    'appointment_date' => 'exclude_if:has_appointment,false|required|date',
    'doctor_name' => 'exclude_if:has_appointment,false|required|string',
]);
```

或者，可以使用 `exclude_unless` 规则，不验证某个字段，除非另一个字段具有特定值：

```php
$validator = Validator::make($data, [
    'has_appointment' => 'required|boolean',
    'appointment_date' => 'exclude_unless:has_appointment,true|required|date',
    'doctor_name' => 'exclude_unless:has_appointment,true|required|string',
]);
```

#### 存在时验证

在某些情况下，可能希望只有在某个字段存在于被验证的数据中时才对其进行验证检查。

要快速实现这一点，可以在规则列表中添加 `sometimes` 规则：

```php
$v = Validator::make($data, [
    'email' => 'sometimes|required|email', // 仅当 email 该字段存在于 $data 数组中时，才会对其进行验证。
]);
```

#### 复杂条件验证

例如：可能希望仅在另一个字段的值大于等于100时才需要给定字段。或者，仅当存在另一个字段时，才需要两个字段才具有给定值。

首先，使用静态规则创建一个`Validator`实例：

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'email' => 'required|email',
    'games' => 'required|numeric',
]);
```

然后，可以可以在`Validator`实例上使用`sometimes`方法

```php
use Illuminate\Support\Fluent;

$validator->sometimes('reason', 'required|max:500', function (Fluent $input) {
    return $input->games >= 100;
});
```

传递给`sometimes`方法的第一个参数是有条件验证的字段的名称。第二个参数是要添加的规则列表。如果第三个参数传递的闭包返回`true`，则将添加规则。

同样的一次也可以为多个字段添加条件验证：

```php
$validator->sometimes(['reason', 'cost'], 'required', function (Fluent $input) {
    return $input->games >= 100;
});
```

#### 复杂条件数据验证

有时可能希望根据同意嵌套数组中的另一个字段来验证一个字段，而不知道该字段的索引。这种情况下，可以允许闭包接收第二个参数，该参数将是正在验证的数据中的当前单个项：

```php
$input = [
    'channels' => [
        [
            'type' => 'email',
            'address' => 'abigail@example.com',
        ],
        [
            'type' => 'url',
            'address' => 'https://example.com',
        ],
    ],
];

$validator->sometimes('channels.*.address', 'email', function (Fluent $input, Fluent $item) {
    return $item->type === 'email';
});

$validator->sometimes('channels.*.address', 'url', function (Fluent $input, Fluent $item) {
    return $item->type !== 'email';
});
```

与传递给闭包的`$input`参数一样，当属性数据为数据时，`$item`参数是`Illuminate\Support\Fluent`的实例；否则，它是一个字符串。

## 验证数组

`array`规则接收允许的数组键列表。如果数组中存在任何其他键，则验证将失败：

```php
use Illuminate\Support\Facades\Validator;

$input = [
    'user' => [
        'name' => 'Taylor Otwell',
        'username' => 'taylorotwell',
        'admin' => true,
    ],
];

Validator::make($input, [
    'user' => 'array:name,username',
]);
```

通常情况下，应该始终允许存在于数组中的数组键。否则，验证器的`validate`和`validated`方法将返回所有已验证的数据，包括数组及其所有键，即使这些键未由其他嵌套数据验证规则验证。

### 验证嵌套数据输入

使用点表示法来验证数组中的属性：

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'photos.profile' => 'required|image',
]);
```

还可以验证数组的每个元素：

```php
$validator = Validator::make($request->all(), [
    'person.*.email' => 'email|unique:users',
    'person.*.first_name' => 'required_with:person.*.last_name',
]);
```

同样在自定义验证消息文件中，也可以使用`*`字符，从而为基于数组的字段使用单个验证消息变的轻而易举：

```php
'custom' => [
    'person.*.email' => [
        'unique' => 'Each person must have a unique email address',
    ]
],
```

### 访问嵌套数组数据

有时，在位属性分配验证规则时，可能需要访问给定嵌套数组元素的值，可以使用`Rule::forEach`方法完成此操作。

`forEach`方法接受一个闭包，该闭包将在验证中为`array`属性的每次迭代调用，并将接收该属性的值和显式的、完全扩展的属性名称。闭包应返回一个规则数组以分配给数组元素：

```php
use App\Rules\HasPermission;
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

$validator = Validator::make($request->all(), [
    'companies.*.id' => Rule::forEach(function (string|null $value, string $attribute) {
        return [
            Rule::exists(Company::class, 'id'),
            new HasPermission('manage-company', $value),
        ];
    }),
]);
```

### 错误消息索引和位置

验证数组时，显示错误消息中引用验证失败的特定项目的索引或位置，可以在自定义验证消息中包含`:index`（从`0`开始）和`:position`（从1开始）占位符：

```php
use Illuminate\Support\Facades\Validator;

$input = [
    'photos' => [
        [
            'name' => 'BeachVacation.jpg',
            'description' => 'A photo of my beach vacation!',
        ],
        [
            'name' => 'GrandCanyon.jpg',
            'description' => '',
        ],
    ],
];

Validator::validate($input, [
    'photos.*.description' => 'required',
], [
    'photos.*.description.required' => 'Please describe photo #:position.',
]);
```

如果有需要，可以通过`second-index`、`second-position`、`third-index`、`third-position`等方式引用嵌套更深的索引和位置：

```php
'photos.*.attributes.*.string' => 'Invalid attribute for photo #:second-position.',
```

## 验证文件

Laravel提供了一个`File`验证规则构建器：

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rules\File;

Validator::validate($input, [
    'attachment' => [
        'required',
        File::types(['mp3', 'wav'])
            ->min(1024)
            ->max(12 * 1024),
    ],
]);
```

也可以直接使用`File`规则的`image`构造函数方法来指定上传的文件应为图像

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;
use Illuminate\Validation\Rules\File;

Validator::validate($input, [
    'photo' => [
        'required',
        File::image()
            ->min(1024)
            ->max(12 * 1024)
            ->dimensions(Rule::dimensions()->maxWidth(1000)->maxHeight(500)),
    ],
]);
```

File文件大小支持指定为字符串，后缀表示文件大小的单位。仅支持`kb`、`mb`、`gb`和`tb`后缀：

```php
File::image()
    ->min('1kb')
    ->max('10mb')
```

## 验证密码

为了确保密码具有足够的复杂程度，可以使用`Password`规则对象：

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rules\Password;

$validator = Validator::make($request->all(), [
    'password' => ['required', 'confirmed', Password::min(8)],
]);
```

`Password`规则对象允许自定义应用程序的密码复杂性要求，例如指定密码至少需要一个字母、数字、符号或混合大小写的字符：

```php
// 需要至少 8 个字符...
Password::min(8)

// 需要至少一个字母...
Password::min(8)->letters()

// 需要至少一个大写字母和一个小写字母...
Password::min(8)->mixedCase()

// 需要至少一个数字...
Password::min(8)->numbers()

// 需要至少一个符号...
Password::min(8)->symbols()
```

还可以使用`uncompromised`方法确保密码未在公共密码数据泄露中泄露：

```php
Password::min(8)->uncompromised()
```

使用 uncompromised 方法的第一个参数自定义此阈值：

```php
// 确保密码在同一次数据泄露中出现的次数少于3次...
Password::min(8)->uncompromised(3);
```

支持链接的写法：

```php
Password::min(8)
    ->letters()
    ->mixedCase()
    ->numbers()
    ->symbols()
    ->uncompromised()
```

### 定义默认密码规则

使用`Password::defaults`方法轻松指定默认密码验证规则，改规则接受闭包。为`defaults`方法提供的闭包应返回Password规则的默认配置。通常，应在应用程序的某个服务提供商的`boot`方法中调用`defaults`规则：

```php
use Illuminate\Validation\Rules\Password;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Password::defaults(function () {
        $rule = Password::min(8);

        return $this->app->isProduction()
                    ? $rule->mixedCase()->uncompromised()
                    : $rule;
    });
}
```

如果将default规则应用于正在验证的特定密码时，可以调用不带参数的`defaults`方法：

```php
'password' => ['required', Password::defaults()],
```

也可以使用`rules`方法完成将其他验证规则附加到默认密码验证规则中：

```php
use App\Rules\ZxcvbnRule;

Password::defaults(function () {
    $rule = Password::min(8)->rules([new ZxcvbnRule]);

    // ...
});
```

## 自定义验证规则

### 使用规则对象

1、创建规则

```bash
php artisan make:rule Uppercase
```

2、定义其行为

规则对象包含一个方法：`validate`。此方法接收属性名称、其值和应在失败时调用的回调，并显示验证错误消息。

```php
<?php

namespace App\Rules;

use Closure;
use Illuminate\Contracts\Validation\ValidationRule;

class Uppercase implements ValidationRule
{
    /**
     * Run the validation rule.
     */
    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        if (strtoupper($value) !== $value) {
            $fail('The :attribute must be uppercase.');
        }
    }
}
```

3、通过将rule对象的实例与其他验证规则一起传递给验证器，将其附加到验证器：

```php
use App\Rules\Uppercase;

$request->validate([
    'name' => ['required', 'string', new Uppercase],
]);
```

#### 翻译验证消息

```php
if (strtoupper($value) !== $value) {
    $fail('validation.uppercase')->translate();
}
```

也可以提供占位符替换和首选语言作为`translate`方法的第一个和第二个参数：

```php
$fail('validation.location')->translate([
    'value' => $this->value,
], 'fr')
```

#### 访问其他数据

如果定义的自定义验证规则类需要访问所有其他正在验证的数据，则可以实现`Illuminate\Contracts\Validation\DataAwareRule`接口：

```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\DataAwareRule;
use Illuminate\Contracts\Validation\ValidationRule;

class Uppercase implements DataAwareRule, ValidationRule
{
    /**
     * All of the data under validation.
     *
     * @var array<string, mixed>
     */
    protected $data = [];

    // ...

    /**
     * Set the data under validation.
     *
     * @param  array<string, mixed>  $data
     */
    public function setData(array $data): static
    {
        $this->data = $data;

        return $this;
    }
}
```

如果验证规则需要访问执行验证者实例，则可以实现`ValidatorAwareRule`接口：

```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\ValidationRule;
use Illuminate\Contracts\Validation\ValidatorAwareRule;
use Illuminate\Validation\Validator;

class Uppercase implements ValidationRule, ValidatorAwareRule
{
    /**
     * The validator instance.
     *
     * @var \Illuminate\Validation\Validator
     */
    protected $validator;

    // ...

    /**
     * Set the current validator.
     */
    public function setValidator(Validator $validator): static
    {
        $this->validator = $validator;

        return $this;
    }
}
```

### 使用闭包

如果在整个应用程序中只需要一次自定义的功能，则可以使用闭包而不是rule对象。
闭包接收属性的名称、属性的值以及验证失败时应调用的`$fail`回调：

```php
use Illuminate\Support\Facades\Validator;
use Closure;

$validator = Validator::make($request->all(), [
    'title' => [
        'required',
        'max:255',
        function (string $attribute, mixed $value, Closure $fail) {
            if ($value === 'foo') {
                $fail("The {$attribute} is invalid.");
            }
        },
    ],
]);
```

### 隐式规则

默认情况下，当正在验证的属性不存在或包含空字符串时，不会运行常规验证规则（包括自定义规则）。

例如，`unique`规则不会正对空字符串运行：

```php
use Illuminate\Support\Facades\Validator;

$rules = ['name' => 'unique:users,name'];

$input = ['name' => ''];

Validator::make($input, $rules)->passes(); // true
```

要使自定义规则即使在属性为空时也运行，改规则必须暗示该属性是必需的。要快速生成新的隐式规则对象，可以使用带有`--implicit`选项的`make:rule` Artisan命令：

```php
php artisan make:rule Uppercase --implicit
```

