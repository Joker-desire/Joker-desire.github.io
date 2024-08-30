# Laravel 11.x Events


----

[原文](https://laravel.com/docs/11.x/events)

----

## 介绍

Laravel 的事件提供了一种简单的观察者模式实现，允许你订阅和监听在应用程序中发生的各种事件。

事件的基本功能：

- 事件提供了一种简单的观察者模式实现。
- 允许订阅和监听应用程序中的各种事件。

事件和监听器的存储位置：

- 事件类通常存储在 app/Events 目录中。
- 监听器存储在 app/Listeners 目录中。
- 如果目录不存在，使用 Artisan 命令生成事件和监听器时会自动创建。

事件解耦：

- 事件使得应用程序的各个部分更松散耦合。
- 单个事件可以有多个相互独立的监听器。

示例：

- 订单发货时发送 Slack 通知。
- 触发 App\Events\OrderShipped 事件，而不是将订单处理代码与 Slack 通知代码耦合。
- 监听器接收事件并发送 Slack 通知。

## 生成事件和监听器

### 快速生成事件 `make:event`

```bash
php artisan make:event PodcastProcessed
```

### 快速生成监听器 `make:listener`

- `--event`：用于指定监听的事件名

```bash
php artisan make:listener SendPodcastNotification --event=PodcastProcessed
```

## 注册事件和监听器

### 事件发现

自动发现和注册：

- Laravel 会自动扫描 `Listeners` 目录。
- 自动找到并注册事件监听器。

方法匹配：

- 监听器类中以 `handle` 或 `__invoke` 开头的方法会被识别为事件监听器。
- 这些方法会注册为方法签名中类型提示的事件的监听器。

示例：

```php
use App\Events\PodcastProcessed;
 
class SendPodcastNotification
{
    /**
     * Handle the given event.
     */
    public function handle(PodcastProcessed $event): void
    {
        // ...
    }
}
```

#### 自定义监听器目录

- 如果监听器不在默认的 `Listeners` 目录中，可以指定其他目录。

使用 `withEvents` 方法：

- 在 `bootstrap/app.php` 文件中使用 `withEvents` 方法指定 Laravel 需要扫描的目录。

示例：

- 在 `bootstrap/app.php` 文件中添加代码：

     ```php
     $app->withEvents([__DIR__.'/../app/CustomListeners']);
     ```

- 也可以添加多个目录：

     ```php
     $app->withEvents([
         __DIR__.'/../app/Listeners',
         __DIR__.'/../app/CustomListeners',
     ]);
     ```

#### 列出所有监听器 `event:list`

```bash
php artisan event:list
```

#### 生成环境事件发现

缓存监听器清单：

- 使用 `optimize` 或 `event:cache` 命令缓存所有应用程序监听器的清单。
- 缓存清单将帮助框架加速事件注册过程。

部署过程：

- 这个命令通常应该作为应用程序部署过程的一部分运行。

使用示例：

- 缓存监听器清单：

     ```bash
     php artisan event:cache
     ```

- 或者：

     ```bash
     php artisan optimize
     ```

- 清除事件缓存：

     ```bash
     php artisan event:clear
     ```

### 手动注册事件

- 使用 Event facade 手动注册事件及其监听器。
- 在 AppServiceProvider 的 boot 方法中进行注册。

示例：

```php
use App\Domain\Orders\Events\PodcastProcessed;
use App\Domain\Orders\Listeners\SendPodcastNotification;
use Illuminate\Support\Facades\Event;
 
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(
        PodcastProcessed::class,
        SendPodcastNotification::class,
    );
}
```

### 闭包监听器

基于类的监听器：

- 通常，监听器是定义为类的，并在事件被触发时处理逻辑。

基于闭包的监听器：

- 可以在 AppServiceProvider 的 boot 方法中手动注册基于闭包的事件监听器。

```php
use App\Events\PodcastProcessed;
use Illuminate\Support\Facades\Event;
 
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(function (PodcastProcessed $event) {
        // ...
    });
}
```

#### 可排队的匿名事件监听器（Queueable Anonymous Event Listeners）

可排队的闭包事件监听器：

- 使用 queueable 函数将监听器闭包包装起来，以使其通过队列执行。

    ```php
    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;
    
    /**
    * Bootstrap any application services.
    */
    public function boot(): void
    {
        Event::listen(queueable(function (PodcastProcessed $event) {
            // ...
        }));
    }
    ```

自定义排队监听器:

- 可以使用 onConnection 来指定连接：

    ```php
    onConnection('database')
    ```

- 可以使用 onQueue 来指定队列：

    ```php
    onQueue('listeners')
    ```

- 可以使用 delay 来设置延迟时间：

    ```php
    delay(now()->addSeconds(10))
    ```

处理监听器失败：

- 可以使用 catch 方法来提供一个处理失败的闭包：

    ```php
    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;
    use Throwable;
    
    Event::listen(queueable(function (PodcastProcessed $event) {
        // ...
    })->catch(function (PodcastProcessed $event, Throwable $e) {
        // The queued listener failed...
    }));
    ```

#### 通配符事件监听器（Wildcard Event Listeners）

- 使用 `*` 字符作为通配符参数注册监听器。
- 可以通过一个监听器处理多个事件。

参数说明：

- 第一个参数：事件名称。
- 第二个参数：整个事件数据数组。
使用示例：

```php
Event::listen('event.*', function (string $eventName, array $data) {
    // ...
});
```

## 定义事件（Defining Events）

事件类的作用：

- 事件类本质上是数据容器。
- 保存与特定事件相关的信息

示例：

- `OrderShipped` 事件类接收一个 `Eloquent ORM` 对象：

    ```php
    <?php
 
    namespace App\Events;
    
    use App\Models\Order;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Foundation\Events\Dispatchable;
    use Illuminate\Queue\SerializesModels;
    
    class OrderShipped
    {
        use Dispatchable, InteractsWithSockets, SerializesModels;
    
        /**
        * Create a new event instance.
        */
        public function __construct(
            public Order $order,
        ) {}
    }
    ```

## 定义监听器（Defining Listeners）

- 监听器在其 `handle` 方法中接收事件实例。

使用 Artisan 命令生成监听器：

- 使用 `make:listener` 命令，并带上 `--event` 选项：

    ```bash
    php artisan make:listener SendShipmentNotification --event=OrderShipped
    ```

- `make:listener` 命令会自动导入正确的事件类并进行类型提示

示例：

```php
<?php
 
namespace App\Listeners;
 
use App\Events\OrderShipped;
 
class SendShipmentNotification
{
    /**
     * Create the event listener.
     */
    public function __construct()
    {
        // ...
    }
 
    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        // Access the order using $event->order...
    }
}
```

### 停止事件传播

- 可以在监听器的 `handle` 方法中返回 `false` 来实现。

示例：

```php
namespace App\Listeners;

use App\Events\OrderShipped;

class SendShipmentNotification
{
    /**
     * 处理事件。
     *
     * @param  \App\Events\OrderShipped  $event
     * @return bool
     */
    public function handle(OrderShipped $event)
    {
        // 执行处理逻辑

        // 决定是否停止事件传播
        if ($event->order->isPriority()) {
            return false; // 停止传播
        }

        \Log::info('Order shipped: ' . $event->order->id);
        return true; // 继续传播
    }
}
```

## 排队事件监听器

排队监听器的益处：

- 适用于执行较慢任务（如发送电子邮件或HTTP请求）的监听器。
- 可以提升应用程序的响应速度。

配置前提：

- 在使用排队监听器之前，需要配置队列并启动队列工作进程（Queue Worker）。

如何排队监听器：

- 在监听器类中实现 `ShouldQueue` 接口。
- 使用 `make:listener` 命令生成的监听器类已自动导入此接口。

示例：

- 示例监听器类实现 ShouldQueue 接口：

    ```php
    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        /**
        * Handle the event.
        *
        * @param  \App\Events\OrderShipped  $event
        * @return void
        */
        public function handle(OrderShipped $event)
        {
            // 发送电子邮件或其他处理逻辑
            \Log::info('Order shipped: ' . $event->order->id);
        }
    }
    ```

工作机制：

- 当事件被派发时，事件调度器会自动将监听器排队。
- 如果监听器在队列中执行时没有抛出异常，处理完成后排队的作业会自动删除。

### 自定义队列

#### 自定义静态属性

- 可以在监听器类上定义以下属性来自定义队列设置：
  - `$connection`：作业应发送到的连接名称。
  - `$queue`：作业应发送到的队列名称。
  - `$delay`：作业处理前的时间（秒）。

示例：

```php
namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    public $connection = 'sqs';
    public $queue = 'listeners';
    public $delay = 60;
}
```

#### 自定义运行时方法

- 可以在监听器类中定义以下方法来在运行时动态设置队列参数：
  - `viaConnection()`：获取队列连接名称。
  - `viaQueue()`：获取队列名称。
  - `withDelay()`：获取作业处理前的秒数。

示例：

```php
public function viaConnection(): string
{
    return 'sqs';
}

public function viaQueue(): string
{
    return 'listeners';
}

public function withDelay(OrderShipped $event): int
{
    return $event->highPriority ? 0 : 60;
}
```

### 有条件地排队监听器

- 根据运行时的数据决定监听器是否应排队。
- 使用 `shouldQueue` 方法进行判断。

实现 `ShouldQueue` 接口：

- 在监听器类中实现 `ShouldQueue` 接口。

添加 shouldQueue 方法：

- 在监听器类中添加 `shouldQueue` 方法返回`bool` 类型。
- `shouldQueue` 方法根据事件数据决定是否排队。

示例：

```php
<?php
 
namespace App\Listeners;
 
use App\Events\OrderCreated;
use Illuminate\Contracts\Queue\ShouldQueue;
 
class RewardGiftCard implements ShouldQueue
{
    /**
     * Reward a gift card to the customer.
     */
    public function handle(OrderCreated $event): void
    {
        // ...
    }
 
    /**
     * Determine whether the listener should be queued.
     */
    public function shouldQueue(OrderCreated $event): bool
    {
        return $event->order->subtotal >= 5000;
    }
}
```

### 手动与队列交互

- 可手动访问监听器底层队列作业的 `delete` 和 `release` 方法。

使用 `InteractsWithQueue` trait：

- 使用 `Illuminate\Queue\InteractsWithQueue` trait 来访问这些方法。
- 该 trait 在生成的监听器中默认被导入。

示例代码：

```php
namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Handle the event.
     * 处理事件。
     */
    public function handle(OrderShipped $event): void
    {
        if (true) {
            // 释放作业并延迟30秒重新处理
            $this->release(30);
        }
    }
}
```

解释：

- `InteractsWithQueue` trait 提供了 `delete` 和 `release` 方法。
- `delete` 方法用于删除队列作业。
- `release` 方法用于释放队列作业并指定延迟时间（秒）。

### 排队事件监听器和数据库事务

- 排队监听器可能在数据库事务提交之前被处理。
- 事务中的更新或创建的模型和记录可能尚未反映或存在于数据库中。

潜在问题：

- 如果监听器依赖于事务中的模型或记录，可能会发生意外错误。

解决方案：`ShouldQueueAfterCommit` 接口：

- 如果队列连接的 `after_commit` 配置选项设置为 `false`，可以在监听器类上实现 `ShouldQueueAfterCommit` 接口。
- 该接口指示监听器在所有开放的数据库事务提交之后被派发。

示例代码：

- 示例展示了如何实现 `ShouldQueueAfterCommit` 接口：

     ```php
     namespace App\Listeners;

     use Illuminate\Contracts\Queue\ShouldQueueAfterCommit;
     use Illuminate\Queue\InteractsWithQueue;

     class SendShipmentNotification implements ShouldQueueAfterCommit
     {
         use InteractsWithQueue;

         // 处理逻辑
     }
     ```

### 处理失败的Jobs

- 如果排队监听器超过最大尝试次数，将调用 `failed` 方法。
- `failed` 方法接收事件实例和导致失败的 `Throwable` 实例。

示例：

```php
namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Throwable;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
    * 处理事件。
    */
    public function handle(OrderShipped $event): void
    {
        // ...
    }

    /**
    * 处理作业失败。
    */
    public function failed(OrderShipped $event, Throwable $exception): void
    {
        // ...
    }
}
```

#### 指定最大尝试次数

- 使用 `$tries` 属性指定监听器可以被尝试的次数。

示例：

```php
namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
    * 队列监听器可以被尝试的次数。
    *
    * @var int
    */
    public $tries = 5;
}
```

#### 指定重试截止时间

- 使用 `retryUntil` 方法指定监听器的重试截止时间。

示例：

```php
use DateTime;

/**
* 确定监听器应超时的时间。
*/
public function retryUntil(): DateTime
{
    return now()->addMinutes(5);
}
```

## 调度事件（Dispatching Events）

- 使用静态`dispatch`方法调度
- 这个方法通过 `Illuminate\Foundation\Events\Dispatchable` trait 提供。

传递参数：

- 传递给`dispatch`的参数都将传递给事件的构造函数

示例：

```php
<?php
 
namespace App\Http\Controllers;
 
use App\Events\OrderShipped;
use App\Http\Controllers\Controller;
use App\Models\Order;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
 
class OrderShipmentController extends Controller
{
    /**
     * Ship the given order.
     */
    public function store(Request $request): RedirectResponse
    {
        $order = Order::findOrFail($request->order_id);
 
        // Order shipment logic...
 
        OrderShipped::dispatch($order);
 
        return redirect('/orders');
    }
}
```

### 有条件的调度事件

- 使用 `dispatchIf` 方法在条件为真时派发事件。
- 使用 `dispatchUnless` 方法在条件为假时派发事件。
- 传递额外的参数，都将传递给事件的构造函数

示例：

```php
namespace App\Events;

use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;
use App\Models\Order;

class OrderShipped
{
    use Dispatchable, SerializesModels;

    public $order;

    /**
     * 创建一个事件实例。
     *
     * @param  \App\Models\Order  $order
     * @return void
     */
    public function __construct(Order $order)
    {
        $this->order = $order;
    }
}

use App\Events\OrderShipped;
use App\Models\Order;

$order = Order::find(1);
$isPriorityOrder = $order->isPriority(); // 假设这是一个判断订单优先级的方法

// 条件为真时派发事件
OrderShipped::dispatchIf($isPriorityOrder, $order);

// 条件为假时派发事件
$isBackOrder = $order->isBackOrder(); // 假设这是一个判断是否为缺货订单的方法
OrderShipped::dispatchUnless($isBackOrder, $order);
```

### 数据库事务后调度事件

仅在活动数据库事务提交后调度事件，则可以在事件类上实现`ShouldDispatchAfterCommit`接口

- 在当前数据库事务提交之前不派发事件。
- 如果事务失败，事件将被丢弃。

条件：

- 如果在事件派发时没有进行中的数据库事务，该事件将立即被派发。

示例：

```php
<?php
 
namespace App\Events;
 
use App\Models\Order;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;
 
class OrderShipped implements ShouldDispatchAfterCommit
{
    use Dispatchable, InteractsWithSockets, SerializesModels;
 
    /**
     * Create a new event instance.
     */
    public function __construct(
        public Order $order,
    ) {}
}
```

## 事件订阅者

### 编写事件订阅者

**事件订阅者类**：

- 允许在单个类中订阅和处理多个事件。

**定义 `subscribe` 方法**：

- `subscribe` 方法接收一个事件调度器实例 (`Dispatcher`)。

**注册事件监听器**：

- 使用 `listen` 方法在事件调度器上注册事件监听器。

**直接返回事件和方法的关联数组**：

- 如果监听器方法在订阅者类本身中定义，可以返回一个包含事件和方法名称的数组。
- Laravel 会自动确定订阅者的类名。

**示例代码**：

   **使用 listen 方法注册监听器**：

   ```php
   namespace App\Listeners;

   use Illuminate\Auth\Events\Login;
   use Illuminate\Auth\Events\Logout;
   use Illuminate\Events\Dispatcher;

   class UserEventSubscriber
   {
       public function handleUserLogin(Login $event): void {}
       public function handleUserLogout(Logout $event): void {}

       public function subscribe(Dispatcher $events): void
       {
           $events->listen(
               Login::class,
               [UserEventSubscriber::class, 'handleUserLogin']
           );

           $events->listen(
               Logout::class,
               [UserEventSubscriber::class, 'handleUserLogout']
           );
       }
   }
   ```

**返回关联数组注册监听器**：

   ```php
   namespace App\Listeners;

   use Illuminate\Auth\Events\Login;
   use Illuminate\Auth\Events\Logout;
   use Illuminate\Events\Dispatcher;

   class UserEventSubscriber
   {
       public function handleUserLogin(Login $event): void {}
       public function handleUserLogout(Logout $event): void {}

       public function subscribe(Dispatcher $events): array
       {
           return [
               Login::class => 'handleUserLogin',
               Logout::class => 'handleUserLogout',
           ];
       }
   }
   ```

### 注册事件订阅者

- 使用 `Event` facade 的 `subscribe` 方法来注册事件订阅者。

**注册位置**：

- 通常在 `AppServiceProvider` 类的 `boot` 方法中注册。

**示例代码**：

```php
namespace App\Providers;

use App\Listeners\UserEventSubscriber;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
    * 启动任何应用程序服务。
    */
    public function boot(): void
    {
        Event::subscribe(UserEventSubscriber::class);
    }
}
```

## 测试

**防止监听器执行**：

- 使用 `Event::fake()` 方法防止实际执行监听器。
- 测试派发事件的代码，然后使用断言方法检测事件的派发情况。

**断言方法**：

- `assertDispatched`：断言事件被派发。
- `assertNotDispatched`：断言事件没有被派发。
- `assertNothingDispatched`：断言没有事件被派发。

**示例代码**：

- 使用 Pest 测试：

    ```php
    <?php
    
    use App\Events\OrderFailedToShip;
    use App\Events\OrderShipped;
    use Illuminate\Support\Facades\Event;
    
    test('orders can be shipped', function () {
        Event::fake();
    
        // Perform order shipping...
    
        // Assert that an event was dispatched...
        Event::assertDispatched(OrderShipped::class);
    
        // Assert an event was dispatched twice...
        Event::assertDispatched(OrderShipped::class, 2);
    
        // Assert an event was not dispatched...
        Event::assertNotDispatched(OrderFailedToShip::class);
    
        // Assert that no events were dispatched...
        Event::assertNothingDispatched();
    });
    ```

- 使用 PHPUnit 测试：

    ```php
    <?php
    
    namespace Tests\Feature;
    
    use App\Events\OrderFailedToShip;
    use App\Events\OrderShipped;
    use Illuminate\Support\Facades\Event;
    use Tests\TestCase;
    
    class ExampleTest extends TestCase
    {
        /**
        * Test order shipping.
        */
        public function test_orders_can_be_shipped(): void
        {
            Event::fake();
    
            // Perform order shipping...
    
            // Assert that an event was dispatched...
            Event::assertDispatched(OrderShipped::class);
    
            // Assert an event was dispatched twice...
            Event::assertDispatched(OrderShipped::class, 2);
    
            // Assert an event was not dispatched...
            Event::assertNotDispatched(OrderFailedToShip::class);
    
            // Assert that no events were dispatched...
            Event::assertNothingDispatched();
        }
    }
    ```

**使用断言闭包**：

- 传递闭包给 `assertDispatched` 或 `assertNotDispatched` 方法：

     ```php
     Event::assertDispatched(function (OrderShipped $event) use ($order) {
         return $event->order->id === $order->id;
     });
     ```

**断言监听器**：

- 使用 `assertListening` 方法断言监听器正在监听指定事件：

     ```php
     Event::assertListening(
         OrderShipped::class,
         SendShipmentNotification::class
     );
     ```

### 伪造事件子集（fake）

使用`fake`方法为一组特定事件伪造监听

**示例**：

- 使用Pest示例：

    ```php
    test('orders can be processed', function () {
        Event::fake([
            OrderCreated::class,
        ]);
    
        $order = Order::factory()->create();
    
        Event::assertDispatched(OrderCreated::class);
    
        // Other events are dispatched as normal...
        $order->update([...]);
    });
    ```

- 使用PHPUnit示例：

    ```php
    /**
     * Test order process.
     */
    public function test_orders_can_be_processed(): void
    {
        Event::fake([
            OrderCreated::class,
        ]);
    
        $order = Order::factory()->create();
    
        Event::assertDispatched(OrderCreated::class);
    
        // Other events are dispatched as normal...
        $order->update([...]);
    }
    ```

**使用except方法伪造除一组指定事件之外的所有事件**

```php
Event::fake()->except([
    OrderCreated::class,
]);
```

### 范围事件伪造（fakeFor）

使用`fakeFor`方法为测试的一部分伪造事件侦听器
**示例**：

- 使用Pest示例：

    ```php
    <?php
    
    use App\Events\OrderCreated;
    use App\Models\Order;
    use Illuminate\Support\Facades\Event;
    
    test('orders can be processed', function () {
        $order = Event::fakeFor(function () {
            $order = Order::factory()->create();
    
            Event::assertDispatched(OrderCreated::class);
    
            return $order;
        });
    
        // Events are dispatched as normal and observers will run ...
        $order->update([...]);
    });
    ```

- 使用PHPUnit示例：

    ```php
    <?php
    
    namespace Tests\Feature;
    
    use App\Events\OrderCreated;
    use App\Models\Order;
    use Illuminate\Support\Facades\Event;
    use Tests\TestCase;
    
    class ExampleTest extends TestCase
    {
        /**
         * Test order process.
         */
        public function test_orders_can_be_processed(): void
        {
            $order = Event::fakeFor(function () {
                $order = Order::factory()->create();
    
                Event::assertDispatched(OrderCreated::class);
    
                return $order;
            });
    
            // Events are dispatched as normal and observers will run ...
            $order->update([...]);
        }
    }
    ```

