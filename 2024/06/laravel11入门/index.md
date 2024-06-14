# Laravel 11.x 入门


# Laravel 11

Laravel 11 通过引入简化的应用程序结构、每秒速率限制、健康路由、优雅的加密密钥轮换、队列测试改进、重新发送邮件传输、提示验证器集成、新的 Artisan 命令等功能，继续改进了 Laravel 10.x 版本。此外，Laravel Reverb，一个官方推出的可扩展 WebSocket 服务器，为您的应用程序提供了强大的实时功能。

**Laravel 11.x 最低要求 PHP 版本为 8.2。**

# Installation

## 创建 Laravel 项目

**前提：需要安装 PHP 和 Composer**

```bash
# 方式一（通过 Composer create-project命令创建）：
composer create-project laravel/laravel example-app

# 方式二（通过Composer全局安装Laravel安装程序进行创建）：
composer global require laravel/installer
laravel new example-app
```

通过 Laravel Artisan serve 命令启动本地开发服务器：

```bash
cd example-app
php artisan serve
```

# Configuration

Laravel 框架的所有配置文件都存储在 `config` 目录中。

> about Artisan 命令可以进行查看应用程序的配置、驱动程序和环境的概述。
>
> 可以通过 `--only` 选项筛选指定部分

```bash
php artisan about
php artisan about --only=environment
```

> 需要浏览更详细配置文件的值，可以使用 `config:show` Artisan 命令

```bash
php artisan config:show database
```

## 环境配置

在全新 Laravel 安装中，应用程序的根目录将包含一个`.env.example` 定义许多常见环境变量的文件。在安装过程中，此文件将自动复制到`.env`。

目录中的`config` 配置文件使用 Laravel `env` 函数进行读取（e: `env('APP_DEBUG', false)` ）。



> 注意：`.env`文件中的任何变量都可以被外部环境变量（如服务器级或系统级环境变量）覆盖。



在加载应用程序的环境变量之前，Laravel 会确定是否已从外部提供 `APP_ENV`环境变量或者是否指定了 `--env` CLI 参数。如果有指定，Laravel 将尝试加载文件 `.env.[APP_ENV]`，如果不存在则将加载默认`.env`文件。

## 环境文件安全

`.env`文件不应该提交到应用程序的源代码管理，如果泄漏，任何敏感凭证都会暴露。

但是，可以使用 Laravel 的内置环境加密来加密文件。加密后的可以安全的放置在源代码管理中。

> `env:encrypt` 命令加密环境文件，加密后的内容放入`.env.encrypted`文件中。
>
> 解密密钥显示在命令的输出中，应存储在安全的密码管理器中。

```bash
php artisan env:encrypt

# 可以提供加密密钥，通过 --key 选项
php artisan env:encrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF

# 有多个环境文件（.env / .env.staging），可以通过 --env 来指定
php artisan env:encrypt --env=staging
```

>  `env:decrypt` 命令解密环境文件。
>
> 默认将从`LARAVEL_ENV_ENCRYPTION_KEY`  环境变量中检索该密钥
>
> 也可以通过 `--key` 选项直接向命令提供密钥
>
> 解密后的文件内容保存在`.env`文件中

```bash
php artisan env:decrypt

php artisan env:decrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF

# 有多个环境文件（.env / .env.staging），可以通过 --env 来指定
php artisan env:decrypt --env=staging

# 要覆盖现有环境文件，可以通过 --force 选项
php artisan env:decrypt --force
```

## 访问配置值

> 获取配置值：可以使用`Config` Facade  或全局 `config`函数。

```php
$value = Config::get('app.timezone');

$value = config('app.timezone');

// 如果不存在，则返回默认值
$value = config('app.timezone', 'Asia/Shanghai');
```

> 设置配置值：可以使用 `Config::set` 或将数组传递给`config`函数

```php
Config::set('app.timezone', 'Asia/Shanghai');
 
config(['app.timezone' => 'Asia/Shanghai']);
```

> 类型化配置检索，如果检索到的配置值与预期不匹配，则会引发异常

```php
Config::string('config-key');
Config::integer('config-key');
Config::float('config-key');
Config::boolean('config-key');
Config::array('config-key');
```

## 配置缓存

> `config:cache` Artisan命令将所有配置文件缓存到单个文件中。
>
> 这会将应用程序的所有配置选项合并到一个文件中，框架可以加速加载该文件。

```bash
php artisan config:cache
```

> `config:clear` 用于清除缓存的配置

```bash
php artisan config:clear
```

## 配置发布

Laravel的大多数配置文件已经发布在应用程序的 config 目录中，然而，像 `cors.php` 和 `view.php` 这样的一些配置文件默认情况下并不会发布，因为大多数应用程序从不需要修改它们。

不过，可以使用 `config:publish` Artisan 命令来发布那些默认未发布的配置文件：

```bash
php artisan config:publish
 
php artisan config:publish --all
```

# 目录结构

### 根目录

```lua
项目根目录
|-- app (用于存放应用程序核心代码)
|-- bootstrap
|   |-- cache (用于存放框架生成的文件以进行性能优化，比如路由和服务缓存文件)
|   |-- app.php (初始化框架)
|-- config (用于存放所有应用程序的配置文件)
|-- database (包含数据库迁移文件、模型工厂和种子文件。也可以存放SQLite数据库文件)
|-- public
|   |-- index.php (进入应用程序的所有请求的入口，并配置了自动加载)
|   |-- Other (存放资源文件，如：图片、JS和CSS)
|-- resources (存放视图以及原始的、未编译的资产，例如：CSS或JS)
|-- routes (包含应用程序的所有路由定义)
|   |-- web.php (包含Laravel放置在web中间件组中的路由，该中间件组提供会话状态、CSRF 保护和 Cookie 加密)
|   |-- console.php (包含所有基于闭包的控制台命令)
|-- storage 
|   |-- app (用于存储应用程序生成的任何文件)
|   |   |-- public (可用于存储用户生成的文件，例如配置文件头像，这些文件是可公开访问的)
|   |-- framework (用于存储框架生成的文件和缓存)
|   |-- logs (包含应用程序的日志文件)
|-- tests (用于自动化测试)
|-- vendor (包含Composer依赖项)
```
### App 目录文件生成命令

`app`目录中许多类都可以由Artisan通过命令生成

```bash
# 查看可用命令
php artisan list make
```

#### 命令汇总

| 命令                         | 描述                                   |
|:-----------------------------|----------------------------------------|
| `make:cache-table`           | [cache:table] 创建缓存数据库表的迁移文件 |
| `make:cast`                  | 创建一个新的自定义 Eloquent 类型转换类   |
| `make:channel`               | 创建一个新的通道类                     |
| `make:class`                 | 创建一个新的类                         |
| `make:command`               | 创建一个新的 Artisan 命令               |
| `make:component`             | 创建一个新的视图组件类                 |
| `make:controller`            | 创建一个新的控制器类                   |
| `make:enum`                  | 创建一个新的枚举                       |
| `make:event`                 | 创建一个新的事件类                     |
| `make:exception`             | 创建一个新的自定义异常类               |
| `make:factory`               | 创建一个新的模型工厂                   |
| `make:interface`             | 创建一个新的接口                       |
| `make:job`                   | 创建一个新的任务类                     |
| `make:listener`              | 创建一个新的事件监听器类               |
| `make:mail`                  | 创建一个新的邮件类                     |
| `make:middleware`            | 创建一个新的中间件类                   |
| `make:migration`             | 创建一个新的迁移文件                   |
| `make:model`                 | 创建一个新的 Eloquent 模型类           |
| `make:notification`          | 创建一个新的通知类                     |
| `make:notifications-table`   | [notifications:table] 创建通知表的迁移文件|
| `make:observer`              | 创建一个新的观察者类                   |
| `make:policy`                | 创建一个新的策略类                     |
| `make:provider`              | 创建一个新的服务提供者类               |
| `make:queue-batches-table`   | [queue:batches-table] 创建批处理数据库表的迁移文件  |
| `make:queue-failed-table`    | [queue:failed-table] 创建失败队列任务数据库表的迁移文件|
| `make:queue-table`           | [queue:table] 创建队列任务数据库表的迁移文件    |
| `make:request`               | 创建一个新的表单请求类                 |
| `make:resource`              | 创建一个新的资源                       |
| `make:rule`                  | 创建一个新的验证规则                   |
| `make:scope`                 | 创建一个新的作用域类                   |
| `make:seeder`                | 创建一个新的数据库填充类               |
| `make:session-table`         | [session:table] 创建会话数据库表的迁移文件|
| `make:test`                  | 创建一个新的测试类                     |
| `make:trait`                 | 创建一个新的特性                       |
| `make:view`                  | 创建一个新的视图                       |

# 部署

## 服务器要求

- PHP >= 8.2
- Ctype PHP Extension
- cURL PHP Extension
- DOM PHP Extension
- Fileinfo PHP Extension
- Filter PHP Extension
- Hash PHP Extension
- Mbstring PHP Extension
- OpenSSL PHP Extension
- PCRE PHP Extension
- PDO PHP Extension
- Session PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension

## 服务器配置

### Nginx

```yml
server {
    listen 80;
    listen [::]:80;
    server_name example.com;
    root /srv/example.com/public;
 
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
 
    index index.php;
 
    charset utf-8;
 
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
 
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }
 
    error_page 404 /index.php;
 
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
    }
 
    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

### FrankenPHP

[FrankenPHP](https://frankenphp.dev/) 是一个用 Go 编写的现代 PHP 应用程序服务器。为 Laravel PHP 应用程序提供服务，只需调用其`php-server`命令：

```bash
frankenphp php-server -r public/
```

## 优化

将应用程序部署到生产环境时，有多种文件应进行缓存，包括：配置、事件、路由和视图。

Laravel 提供了一个单一且方便的`optimize` Artisan命令，可以缓存所有这些文件。

通常应该作为应用程序部署过程的一部分来调用：

```bash
php artisan optimize
# 清除所有缓存
php artisan optimize:clear
```

## 缓存

### Config 缓存

> `config:cache`命令，将配置进行缓存。
>
> 这样会将所有配置文件合并到一个缓存文件中，这极大地减少了框架在加载配置值时访问文件系统的次数。

```bash
php artisan config:cache
```

**注意：使用此命令，应确保只在配置文件中调用了`env`函数。因为一旦配置被缓存，`.env`文件将不会被加载，所有对`.env`变量的`env`函数调用将返回`null`。**

### Event 缓存

> `event:cache`命令，在部署过程中缓存应用程序自动发现的事件到监听器的映射。

```bash
php artisan event:cache
```

### Routes 缓存

> `route:cache`命令，将所有的路由注册简化为在一个缓存文件中的单个方法调用，从而在注册数百个路由时提高路由注册的性能。

```bash
php artisan route:cache
```

### Views 缓存

> `view:cache`命令，该命令会预编译所有的 Blade 视图，因此它们不会在需要时即时编译，从而提高返回视图的每个请求的性能。

```bash
php artisan view:cache
```

## Debug 模式

`config/app.php` 配置文件中的 debug 选项决定了实际向用户显示多少关于错误的信息。

默认情况下，该选项设置为遵循存储在应用程序 `.env` 文件中的 `APP_DEBUG` 环境变量的值。

**注意：在生产环境中，此值应始终为false。**

## 健康路由

Laravel 包含一个内置的健康检查路由，可以用来监控应用程序状态。

在生产环境中，这个路由可以用来向正常运行时间监控器、负载均衡器或者像 Kubernetes 这样的编排系统报告您的应用程序状态。

默认情况下，健康检查路由提供于 `/up`，如果应用程序启动时没有异常，则会返回 200 HTTP 响应。否则，会返回 500 HTTP 响应。

可以在应用程序的 `bootstrap/app` 文件中配置此路由的 URI：

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    // health: '/up', 
    health: '/status', 
)
```

当向此路由发出 HTTP 请求时，Laravel 还会调度一个 `Illuminate\Foundation\Events\DiagnosingHealth` 事件，允许执行与应用程序相关的额外健康检查。

在此事件的监听器中，可以检查应用程序的数据库或缓存状态。如果检测到应用程序有问题，可以直接在监听器中抛出一个异常。
