# Laravel 11.x 控制台


----

[原文](https://laravel.com/docs/11.x/artisan)

----

## 介绍

Artisan 是 Laravel 附带的命令行界面。

Artisan 作为 `artisan` 脚本存在于应用程序的根目录中，提供了许多有用的命令，可以在构建应用程序时使用。

要查看所有可用 Artisan 命令的列表，可以使用以下 `list` 命令：

```bash
php artisan list
```

### Tinker（PEPL）

Laravel Tinker 是 Laravel 框架的强大 REPL，由 PsySH 软件包提供支持。

#### 安装

默认情况下，所有的Laravel应用程序都包含Tinker。也可以手动进行安装，使用Composer进行安装：

```bash
composer require laravel/tinker
```

#### 用法

Tinker 允许在命令行上于整个Laravel应用程序进行交互，包括Eloquent模型、作业、事件等。

```bash
# 进入tinker环境
php artisan tinker
```

使用以下 `vendor:publish` 命令发布 Tinker 的配置文件：

```bash
php artisan vendor:publish --provider="Laravel\Tinker\TinkerServiceProvider"
```

#### 命令允许列表

Tinker使用允许列表来确定允许在其shell中运行那些Artisan命令。

默认情况下，可以运行`clear-compiled`, `down`, `env`, `inspire`, `migrate`, `optimize`和 `up`命令。

如果要允许更多命令，可以将它们添加到`tinker.php`配置文件中的`commands`数组中：

```php
'commands' => [
    // App\Console\Commands\ExampleCommand::class,
],
```

#### 禁止使用别名的类

通常情况下Tinker交互时，会自动为类添加别名。但是可以设置禁止某些类不添加别名。

可以通过列出`tinker.php`配置文件`dont_alias`数组中的类来实现此目的：

```php
'dont_alias' => [
    App\Models\User::class,
],
```

## 编写命令

除了 Artisan 提供的命令外，还可以构建自己的自定义命令。

命令通常存储在 `app/Console/Commands` 目录中;

但是，也可以自由选择自己的存储位置，只要命令可以由 Composer 加载即可。

### 生成命令

> `make:command`命令用于创建新命令。
>
> 命令将在 `app/Console/Commands` 目录中创建一个新的命令类

```bash
php artisan make:command SendEmails
```

### 命令结构

生成命令后，应为类的 `signature` 和 `description` 属性定义适当的值。

在 `list` 屏幕上显示命令时，将使用这些属性。

该 `signature` 属性还允许定义命令的输入期望值。

执行命令时将调用该 `handle` 方法，可以将命令逻辑放在此方法中。

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class SendEmails extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'mail:send {user}';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Send a marketing email to a user';

    /**
     * Execute the console command.
     */
    public function handle()
    {
        $this->info('Sending email to ' . $this->argument('user'));
    }
}

```

#### 退出代码

如果 `handle` 该方法未返回任何内容，并且命令执行成功，则该命令将退出并显示 `0` 退出代码，指示成功。

但是，该 `handle` 方法可以选择返回一个整数来手动指定命令的退出代码：

```php
$this->error('Something went wrong.');
 
return 1;
```

如果想从命令中的任何方法“失败”命令，可以使用该 `fail` 方法。

该 `fail` 方法将立即终止命令的执行，并返回以下退出 `1` 代码：

```php
$this->fail('Something went wrong.');
```

### 闭包命令

基于闭包的命令提供了将控制台命令定义为类的替代方法。

就像路由闭包是控制器的替代方法一样，将命令闭包视为命令类的替代方法。

尽管该 `routes/console.php` 文件未定义 HTTP 路由，但它定义了应用程序中基于控制台的入口点（路由）。

在此文件中，可以使用该 `Artisan::command` 方法定义所有基于闭包的控制台命令。

该 `command` 方法接受两个参数：命令签名和接收命令参数和选项的闭包

```php
Artisan::command('mail:send {user}', function (string $user) {
    $this->info("Sending email to: {$user}!");
});
```

#### 类型提示依赖项

除了接收命令的参数和选项外，命令闭包还可能类型提示要从服务容器中解析的其他依赖项：

```php
use App\Models\User;
use App\Support\DripEmailer;
 
Artisan::command('mail:send {user}', function (DripEmailer $drip, string $user) {
    $drip->send(User::find($user));
});
```

#### 闭包命令说明

定义基于闭包的命令时，可以使用该 `purpose` 方法向命令添加说明。

当运行 `php artisan list` or `php artisan help` 命令时，将显示此说明：

```php
Artisan::command('mail:send {user}', function (string $user) {
    // ...
})->purpose('Send a marketing email to a user');
```

### 可分离的命令

若要使用此功能，应用程序必须使用 `memcached` 、 `redis`  、 `dynamodb`、 `database` 、 `file` 或 `array` 缓存驱动程序作为应用程序的默认缓存驱动程序。此外，所有服务器必须与同一中央缓存服务器通信。

有时，可能希望确保一次只能运行一个命令实例。

为此，可以在命令类上实现该 `Illuminate\Contracts\Console\Isolatable` 接口：

```php
<?php
 
namespace App\Console\Commands;
 
use Illuminate\Console\Command;
use Illuminate\Contracts\Console\Isolatable;
 
class SendEmails extends Command implements Isolatable
{
    // ...
}
```

当命令标记为 `Isolatable` 时，Laravel 将自动向该命令添加一个 `--isolated` 选项。

当使用该选项调用命令时，Laravel 将确保该命令的其他实例尚未运行。

Laravel 通过尝试使用应用程序的默认缓存驱动程序获取原子锁来实现此目的。

如果该命令的其他实例正在运行，则不会执行该命令;但是，该命令仍将退出，并显示成功退出状态代码：

```bash
php artisan mail:send 1 --isolated
```

如果要指定命令在无法执行时应返回的退出状态代码，可以通过以下 `isolated` 选项提供所需的状态代码：

```bash
php artisan mail:send 1 --isolated=12
```

#### Lock ID

默认情况下，Laravel 将使用命令的名称生成字符串键，该字符串键用于获取应用程序缓存中的原子锁。

但是，也可以通过在 Artisan 命令类上定义 `isolatableId` 方法来自定义此键，从而允许将命令的参数或选项集成到键中：

```php
/**
 * Get the isolatable ID for the command.
 */
public function isolatableId(): string
{
    return $this->argument('user');
}
```

#### Lock 过期时间

默认情况下，隔离锁在命令完成后过期。或者，如果命令中断且无法完成，则锁将在一小时后过期。

但是，可以通过在命令上定义 `isolationLockExpiresAt` 方法来调整锁定过期时间：

```php
use DateTimeInterface;
use DateInterval;
 
/**
 * Determine when an isolation lock expires for the command.
 */
public function isolationLockExpiresAt(): DateTimeInterface|DateInterval
{
    return now()->addMinutes(5);
}
```

## 定义输入期望

编写控制台命令时，通常通过参数或选项从用户那里收集输入。

Laravel 可以非常方便地使用命令上的 `signature` 属性来定义用户期望的输入。

该 `signature` 属性允许以单个富有表现力的类似路由的语法定义命令的名称、参数和选项。

### 参数

用户提供的所有参数和选项都用大括号括起来。

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'mail:send {user}';
```

还可以将参数设置为可选参数或定义参数的默认值：

```php
// Optional argument...
'mail:send {user?}'
 
// Optional argument with default value...
'mail:send {user=foo}'
```

### 选项

当通过命令行提供选项时，选项以两个连字符 （ `--` ） 为前缀。

有两种类型的选项：接收值的选项和不接收值的选项。

未收到值的选项充当布尔“开关”。

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'mail:send {user} {--queue}';
```

在此示例中，可以在调用 Artisan 命令时指定 `--queue` 开关。

如果传递了 `--queue` 开关，则该选项的值将为 `true` 。否则，该值将为 `false` ：

```bash
php artisan mail:send 1 --queue
```

#### 带值的选项

在选项名称后加上符号 `=` 定义带值的选项：

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'mail:send {user} {--queue=}';
```

在此示例中，用户可以像这样传递选项的值。

如果在调用命令时未指定该选项，则其值将为 `null` ：

```bash
php artisan mail:send 1 --queue=default
```

可以通过在选项名称后指定默认值来为选项分配默认值。

如果用户未传递任何选项值，则将使用默认值：

```php
'mail:send {user} {--queue=default}'
```

#### 选项快捷方式

要在定义选项时指定快捷方式，可以在选项名称之前指定快捷方式，并使用字符 `|` 作为分隔符将快捷方式与完整选项名称分开：

```php
'mail:send {user} {--Q|queue}'
```

在终端上调用命令时，选项快捷方式应以单个连字符为前缀，并且在指定选项值时不应包含 `=` 任何字符：

```bash
php artisan mail:send 1 -Qdefault
```

### 输入数组

如果要定义参数或选项以需要多个输入值，则可以使用字符 `*`

```php
'mail:send {user*}'
```

调用此方法时，可以将 `user` 参数按顺序传递到命令行。

例如，以下命令将 的 `user` 值设置为具有 `1` 和 `2` 作为其值的数组：

```bash
php artisan mail:send 1 2
```

#### 选项数组

定义需要多个输入值的选项时，传递给命令的每个选项值都应以选项名称为前缀：

```php
'mail:send {--id=*}'
```

可以通过传递多个 `--id` 参数来调用此类命令：

```bash
php artisan mail:send --id=1 --id=2
```

### 输入说明

可以通过使用冒号将参数名称与描述分开来为输入参数和选项分配说明。

如果需要一点额外的空间来定义命令，请随意将定义分散到多行：

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'mail:send
                        {user : The ID of the user}
                        {--queue : Whether the job should be queued}';
```



### 提示输入缺失

如果命令包含必需的参数，则用户将在未提供参数时收到错误消息。

或者，也可以通过实现 `PromptsForMissingInput` 接口将命令配置为在缺少所需参数时自动提示用户：

```php
<?php
 
namespace App\Console\Commands;
 
use Illuminate\Console\Command;
use Illuminate\Contracts\Console\PromptsForMissingInput;
 
class SendEmails extends Command implements PromptsForMissingInput
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'mail:send {user}';
 
    // ...
}
```

如果 Laravel 需要从用户那里收集所需的参数，它将通过使用参数名称或描述智能地表述问题来自动向用户询问参数。

如果希望自定义用于收集所需参数的问题，可以实现该 `promptForMissingArgumentsUsing` 方法，返回由参数名称键控的问题数组：

```php
/**
 * Prompt for missing input arguments using the returned questions.
 *
 * @return array<string, string>
 */
protected function promptForMissingArgumentsUsing(): array
{
    return [
        'user' => 'Which user ID should receive the mail?',
    ];
}
```

还可以使用包含问题和占位符的元组来提供占位符文本：

```php
return [
    'user' => ['Which user ID should receive the mail?', 'E.g. 123'],
];
```

如果要完全控制提示，可以提供一个闭包，该闭包应提示用户并返回其答案：

```php
use App\Models\User;
use function Laravel\Prompts\search;
 
// ...
 
return [
    'user' => fn () => search(
        label: 'Search for a user:',
        placeholder: 'E.g. Taylor Otwell',
        options: fn ($value) => strlen($value) > 0
            ? User::where('name', 'like', "%{$value}%")->pluck('name', 'id')->all()
            : []
    ),
];
```

如果希望提示用户选择或输入选项，可以在命令的 `handle` 方法中包含提示。

然而，如果只希望在用户因为缺少参数而被自动提示时才进行提示，那么可以实现 `afterPromptingForMissingArguments` 方法：

```php
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use function Laravel\Prompts\confirm;
 
// ...
 
/**
 * Perform actions after the user was prompted for missing arguments.
 */
protected function afterPromptingForMissingArguments(InputInterface $input, OutputInterface $output): void
{
    $input->setOption('queue', confirm(
        label: 'Would you like to queue the mail?',
        default: $this->option('queue')
    ));
}
```

## 命令I/O

### 检索输入

在执行命令时，可能需要访问命令接受的参数和选项的值。

为此，可以使用 `argument` 和 `option` 方法。

如果参数或选项不存在， `null` 则返回：

```php
/**
 * Execute the console command.
 */
public function handle(): void
{
    $userId = $this->argument('user');
}
```

如果需要将所有参数检索为 `array` ，请调用该 `arguments` 方法：

```php
$arguments = $this->arguments();
```

使用该 `option` 方法可以像检索参数一样轻松地检索选项。

若要将所有选项检索为数组，请调用以下 `options` 方法：

```php
// Retrieve a specific option...
$queueName = $this->option('queue');
 
// Retrieve all options as an array...
$options = $this->options();
```

### 输入提示

除了显示输出之外，还可以要求用户在执行命令期间提供输入。

该 `ask` 方法将提示用户给定的问题，接受他们的输入，然后将用户的输入返回到您的命令：

```php
/**
 * Execute the console command.
 */
public function handle(): void
{
    $name = $this->ask('What is your name?');
 
    // ...
}
```

该 `ask` 方法还接受可选的第二个参数，该参数指定在未提供用户输入时应返回的默认值：

```php
$name = $this->ask('What is your name?', 'Taylor');
```

`secret` 方法与 `ask` 方法类似，但用户在控制台输入时输入内容不会显示给他们。

此方法在询问诸如密码等敏感信息时非常有用：

```php
$password = $this->secret('What is the password?');
```

#### 要求确认

如果需要向用户询问一个简单的“是或否”确认，可以使用 `confirm` 方法。

默认情况下，此方法将返回 `false`。

然而，如果用户在提示中输入 `y` 或 `yes`，该方法将返回 `true`。

```php
if ($this->confirm('Do you wish to continue?')) {
    // ...
}
```

如果需要，可以通过将 `true` 作为第二个参数传递给 `confirm` 方法，从而指定确认提示默认应返回 `true`：

```php
if ($this->confirm('Do you wish to continue?', true)) {
    // ...
}
```

#### 自动完成

`anticipate` 方法可用于为可能的选项提供自动完成。

用户仍然可以提供任何答案，而不管自动完成提示如何：

```php
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);
```

或者，也可以将一个闭包作为第二个参数传递给 `anticipate` 方法。

每次用户输入一个字符时，都会调用该闭包。

闭包应接受一个包含用户当前输入的字符串参数，并返回一个用于自动完成的选项数组：

```php
$name = $this->anticipate('What is your address?', function (string $input) {
    // Return auto-completion options...
});
```

#### 多项选择

如果需要在询问问题时向用户提供一组预定义的选项，可以使用 `choice` 方法。

如果未选择任何选项，可以通过将索引作为第三个参数传递给该方法来设置默认返回值的数组索引：

```php
$name = $this->choice(
    'What is your name?',
    ['Taylor', 'Dayle'],
    $defaultIndex
);
```

此外，`choice` 方法还接受第四个和第五个可选参数，用于确定选择有效响应的最大尝试次数以及是否允许多重选择：

```php
$name = $this->choice(
    'What is your name?',
    ['Taylor', 'Dayle'],
    $defaultIndex,
    $maxAttempts = null,
    $allowMultipleSelections = false
);
```

### 写入输出

若要将输出发送到控制台，可以使用`line`, `info`, `comment`, `question`, `warn`和 `error`方法。

这些方法中的每一种都将使用适当的 ANSI 颜色来实现其目的。

`info` 方法将在控制台中显示为绿色文本：

```php
/**
 * Execute the console command.
 */
public function handle(): void
{
    // ...
 
    $this->info('The command was successful!');
}
```

`error` 方法将错误消息文本显示为红色：

```php
$this->error('Something went wrong!');
```

`line`方法显示纯文本、无色文本：

```php
$this->line('Display this on the screen');
```

`newLine`方法显示空行：

```php
// Write a single blank line...
$this->newLine();
 
// Write three blank lines...
$this->newLine(3);
```

#### Tables

`table` 方法使正确格式化多行/多列数据变得简单。

只需要提供列名和表格数据，Laravel 会自动为您计算表格的适当宽度和高度：

```php
use App\Models\User;
 
$this->table(
    ['Name', 'Email'],
    User::all(['name', 'email'])->toArray()
);
```

#### 进度条

对于长时间运行的任务，显示一个进度条来告知用户任务的完成情况是很有帮助的。

通过使用 `withProgressBar` 方法，Laravel 将显示一个进度条，并在遍历给定的可迭代值的每次迭代时推进其进度：

```php
use App\Models\User;
 
$users = $this->withProgressBar(User::all(), function (User $user) {
    $this->performTask($user);
});
```

有时，可能需要对进度条的推进进行更手动的控制。

首先，定义该过程将迭代的总步数。

然后，在处理每个项目后推进进度条：

```php
$users = App\Models\User::all();
 
$bar = $this->output->createProgressBar(count($users));
 
$bar->start();
 
foreach ($users as $user) {
    $this->performTask($user);
 
    $bar->advance();
}
 
$bar->finish();
```



## 注册命令

默认情况下，Laravel 会自动注册 `app/Console/Commands` 目录中的所有命令。

但是，可以指示 Laravel 使用应用程序 `bootstrap/app.php` 文件中的方法 `withCommands` 扫描其他目录中的 Artisan 命令：

```php
->withCommands([
    __DIR__.'/../app/Domain/Orders/Commands',
])
```

还可以通过向 `withCommands` 方法提供命令的类名来手动注册命令：

```php
use App\Domain\Orders\Commands\SendEmails;
 
->withCommands([
    SendEmails::class,
])
```

当 Artisan 启动时，应用程序中的所有命令都将由服务容器解析并向 Artisan 注册。

## 以编程方式执行命令

可以使用 `Artisan` facade 上的 `call` 方法来完成从路由或控制器中执行一个Artisan命令。

`call` 方法接受命令的签名名称或类名称作为第一个参数，以及一个命令参数数组作为第二个参数。返回值将是命令的退出码：

```php
use Illuminate\Support\Facades\Artisan;
 
Route::post('/user/{user}/mail', function (string $user) {
    $exitCode = Artisan::call('mail:send', [
        'user' => $user, '--queue' => 'default'
    ]);
 
    // ...
});
```

也可以将整个 Artisan 命令作为字符串传递 `call` 给该方法：

```php
Artisan::call('mail:send 1 --queue=default');
```

#### 传递数组值

如果命令定义了一个接受数组的选项，则可以将值数组传递给该选项：

```php
use Illuminate\Support\Facades\Artisan;
 
Route::post('/mail', function () {
    $exitCode = Artisan::call('mail:send', [
        '--id' => [5, 13]
    ]);
});
```

#### 传递布尔值

如果需要指定不接受字符串值的选项的值（如命令上 `--force` 的标志），则应传递 `true` or `false` 作为选项的 `migrate:refresh` 值：

```php
$exitCode = Artisan::call('migrate:refresh', [
    '--force' => true,
]);
```

#### 排队Artisan命令

通过在 `Artisan` facade 上使用 `queue` 方法，甚至可以将 Artisan 命令加入队列，以便它们在后台由队列工作者处理。

在使用此方法之前，请确保您已配置好队列并正在运行队列监听器：

```php
use Illuminate\Support\Facades\Artisan;
 
Route::post('/user/{user}/mail', function (string $user) {
    Artisan::queue('mail:send', [
        'user' => $user, '--queue' => 'default'
    ]);
 
    // ...
});
```

通过使用 `onConnection` 和 `onQueue` 方法，可以指定 Artisan 命令应被派发到的连接或队列：

```php
Artisan::queue('mail:send', [
    'user' => 1, '--queue' => 'default'
])->onConnection('redis')->onQueue('commands');
```



### 从其他命令调用命令

有时，可能希望从现有的 Artisan 命令中调用其他命令。

可以使用 `call` 方法来实现。

这种 `call` 方法接受命令名称和一个包含命令参数/选项的数组：

```php
/**
 * Execute the console command.
 */
public function handle(): void
{
    $this->call('mail:send', [
        'user' => 1, '--queue' => 'default'
    ]);
 
    // ...
}
```

如果希望调用另一个控制台命令并抑制其所有输出，可以使用 `callSilently` 方法。

`callSilently` 方法的签名与 `call` 方法相同：

```php
$this->callSilently('mail:send', [
    'user' => 1, '--queue' => 'default'
]);
```



## 信号处理

操作系统允许将信号发送到正在运行的进程。

例如，`SIGTERM` 信号是操作系统请求程序终止的方式。

如果需要在 Artisan 控制台命令中监听信号，并在信号发生时执行代码，可以使用 `trap` 方法：

```php
/**
 * Execute the console command.
 */
public function handle(): void
{
    $this->trap(SIGTERM, fn () => $this->shouldKeepRunning = false);
 
    while ($this->shouldKeepRunning) {
        // ...
    }
}
```

若要同时侦听多个信号，可以向 `trap` 该方法提供信号数组：

```php
$this->trap([SIGTERM, SIGQUIT], function (int $signal) {
    $this->shouldKeepRunning = false;
 
    dump($signal); // SIGTERM / SIGQUIT
});
```



## 存根定制

Artisan 控制台的 `make` 命令用于创建各种类，例如控制器、任务、迁移和测试。

这些类是使用 "stub" 文件生成的，这些文件根据您的输入填充值。

然而，可能希望对 Artisan 生成的文件进行一些小改动。

为此，可以使用 `stub:publish` 命令将最常用的 stubs 发布到应用程序中，以便可以对它们进行自定义：

```bash
php artisan stub:publish
```



## 事件

Artisan 在运行命令时会触发三个事件：`Illuminate\Console\Events\ArtisanStarting`、`Illuminate\Console\Events\CommandStarting` 和 `Illuminate\Console\Events\CommandFinished`。

`ArtisanStarting` 事件在 Artisan 开始运行时立即触发。

接下来，`CommandStarting` 事件在命令运行之前立即触发。

最后，`CommandFinished` 事件在命令执行完成后触发。

