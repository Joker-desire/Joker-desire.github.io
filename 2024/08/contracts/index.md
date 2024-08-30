# Laravel 11.x Contracts


----

[原文](https://laravel.com/docs/11.x/contracts)

----

## 介绍

Laravel 的'契约'（contracts）是一组接口，这些接口定义了框架提供的核心服务。例如，`Illuminate\Contracts\Queue\Queue` 契约定义了队列作业所需的方法，而 `Illuminate\Contracts\Mail\Mailer` 契约定义了发送电子邮件所需的方法。

每个契约都有框架提供的相应实现。例如，Laravel 提供了具有多种驱动的队列实现，并提供了由 Symfony Mailer 支持的邮件发送实现。

### Contracts vs. Facades

Laravel 的 Facades 和辅助函数提供了一种简单的方式来使用 Laravel 的服务，而无需通过服务容器进行类型提示和解析契约。在大多数情况下，每个 Facade 都有一个等效的契约。

与 Facades 不同，Facades 不需要你在类的构造函数中引入它们，而契约允许你为类定义显式依赖。一些开发者更喜欢以这种方式显式定义他们的依赖，因此倾向于使用契约，而其他开发者则享受 Facades 带来的便利。总体而言，大多数应用程序在开发过程中可以无障碍地使用 Facades。

## 何时使用Contracts

选择使用契约还是 Facades 取决于个人喜好和你的开发团队的偏好。契约和 Facades 都可以用来创建健壮的、经过良好测试的 Laravel 应用程序。契约和 Facades 并不互斥。你的应用程序的一些部分可以使用 Facades，而其他部分可以依赖于契约。只要你保持类的职责专注，你会发现使用契约和 Facades 之间的实际差异非常小。

总体而言，大多数应用程序在开发过程中可以无障碍地使用 Facades。如果你正在构建一个与多个 PHP 框架集成的包，你可能希望使用 `illuminate/contracts` 包来定义与 Laravel 服务的集成，而无需在你的包的 `composer.json` 文件中要求 Laravel 的具体实现。

## 如何使用Contracts

Laravel 中的许多类型的类都是通过服务容器解析的，包括控制器、事件监听器、中间件、排队作业，甚至路由闭包。因此，要获得合约的实现，只需在正在解析的类的构造函数中“类型提示”接口即可。

以下示例，当事件侦听器被解析时，服务容器将读取类的构造函数上的类型提示，并注入适当的值。

```php
<?php
 
namespace App\Listeners;
 
use App\Events\OrderWasPlaced;
use App\Models\User;
use Illuminate\Contracts\Redis\Factory;
 
class CacheOrderInformation
{
    /**
     * Create a new event handler instance.
     */
    public function __construct(
        protected Factory $redis,
    ) {}
 
    /**
     * Handle the event.
     */
    public function handle(OrderWasPlaced $event): void
    {
        // ...
    }
}
```

