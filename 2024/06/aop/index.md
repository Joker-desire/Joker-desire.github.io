# Hyperf AOP


AOP：面向切面编程

目的：将横切关注点与业务代码分离，提供模块化程度

## 介绍

### 什么是Hyperf AOP

Hyperf AOP 是Hyperf框架提供的一种面向切面编程的技术。它通过将横切关注点（例如日志记录、事务管理、安全性检查等）从主业务逻辑代码中分离出来，以模块化的方式实现对这些关注点的管理和重用。

在 Hyperf AOP中，切面（Aspect）是一个模块化的关注点，它可以跨越多个对象，例如日志记录、事务管理等。

切面通过定义切点（Pointcut）和增强（Advice）来介入目标对象的方法执行过程：

- 切点是一个表达式，用于匹配目标对象的一组方法，在这些方法执行时切面会被触发。

- 增强则定义了切面在目标对象方法执行前、执行后或抛出异常时所要执行的逻辑。

Hyperf AOP 在项目启动前，每个被介入的目标类最终都会生成一个代理类（ProxyClass），来达到执行切面方法的目的。



### 为什么要用Hyperf AOP

使用Hyperf AOP的主要原因是它可以帮助我们更好地管理各种横切关注点，例如日志记录、事务管理、安全性检查等。以下是一些使用Hyperf AOP的优点：

1. **模块化：** Hyperf AOP 将横切关注点从主业务逻辑代码中分离出来，以模块化的方式实现对这些关注点的管理和重用。这样可以更容易地维护代码，并且可以将同一个关注点的逻辑应用到多个方法或类中。
2. **非侵入式：** 使用 Hyperf AOP 是，我们不需要修改原始业务逻辑代码，只需要在切点和增强中定义我们所需要的逻辑即可。这样，我们就能保持原始代码的简洁性和可读性。
3. **可重用性：** 可以将同一切面应用于多个目标对象进行横切处理。这样可以提高代码的重用性，并且可以更加方便地维护和更新切面逻辑。
4. **松耦合：** AOP 可以减少各个业务模块之间的耦合度，这是因为我们可以将某些通用的逻辑作为切面来实现，而不是直接在各个业务模块中实现。这样可以使得各个业务模块之间更加独立，从而提高代码的可维护性。

总之，使用 Hyperf AOP 可以帮助我们更好地管理和重用横切关注点逻辑，使得代码更易于维护和理解，并且可以提高代码的可重用性和灵活性。

### 核心概念

1. **切面（Aspect）：** 切面是一个模块化的横切关注点实现，它包括了连接点和增强。可以通过配置文件、注解方式定义切面。
2. **连接点（Joinpoint）：** 程序中能够被切面插入的点，典型的连接点包括方法调用、方法执行过程中的某个时点等等。
3. **增强（Advice）：** 在连接点处执行的代码。分为各种类型，如前置增强、后置增强、环绕增强等（在Hyperf AOP中对此简化了该功能的使用不做过细的划分，仅保留环绕一种形式）。
4. **切点（Pointcut）：**  用于定义那些连接点上应该应用增强。切点通过表达式进行定义，如匹配所有public方法或匹配某个包下的所有方法等
5. **织入（Weaving）：** 指将切面应用到目标对象并创建新的代理对象的过程。Hyperf AOP 在项目启动前，每个被介入的目标类最终都会生成一个代理类，来达到执行切面方法的目的。

### AOP 设计原则

**单一职责原则：** 切面只关注一个横切关注点

**开闭原则：** 切面代码不应影响业务代码

**依赖倒置原则：** 业务代码不应该依赖切面代码

**接口隔离原则：** 切面接口应尽可能细化

**最小知识原则：** 切面代码应知道最少的信息

### AOP 和 OOP 的区别

AOP (Aspect-Oriented Programming) 和 OOP (Object-Oriented Programming) 都是面向对象编程的范式：

1. **技术视角：** OOP 是一种代码组织技术，它通过将数据和操作封装在对象中来实现模块化和复用。AOP 是一种编程范式，它允许开发者在执行过程中（而非编译期）动态地添加、删除或修改代码的功能。
2. **关注点：** OOP 关注对象的内部结构和行为，其目标是更好地描述和设计一个系统的真实世界概念。AOP 关注横切关注点（Cross-cutting Concerns），即存在于应用程序各个层面的相同问题，如日志、事务、安全等。
3. **实现方式：** OOP 是通过类和接口来实现封装、继承和多态等特性，使得类能够更好地表达问题领域内的概念。AOP 是通过将通用功能抽取为切面（Aspect），并与核心业务逻辑混合使用，来实现这些横切关注点。
4. **关注点分离：** OOP 面对复杂系统可能导致代码重复或紧密耦合的问题，而 AOP 采用横切关注点的方式来解决这些问题，使得系统功能更加组合和重用。

总结就是：AOP 是对 OOP 的补充，用于解决 OOP难以解决的问题；AOP 与 OOP 相辅相成，共同构建模块化、可维护的代码。

## 基本使用

### 定义切面（Aspect）

每个切面（Aspect）必须实现`Hyperf\Di\Aop\AroundInterface`接口，并提供 `public` 的 `$classes` 和 `$annotations` 属性，为了方便使用，我们可以通过继承 `Hyperf\Di\Aop\AbstractAspect` 来简化定义过程。

每个 `切面(Aspect)` 必须定义 `#[Aspect]` 注解或在 `config/autoload/aspects.php` 内配置均可发挥作用。

```php
<?php
namespace App\Aspect;

use App\Service\SomeClass;
use App\Annotation\SomeAnnotation;
use Hyperf\Di\Annotation\Aspect;
use Hyperf\Di\Aop\AbstractAspect;
use Hyperf\Di\Aop\ProceedingJoinPoint;

#[Aspect]
class FooAspect extends AbstractAspect
{
    // 要切入的类或 Trait，可以多个，亦可通过 :: 标识到具体的某个方法，通过 * 可以模糊匹配
    public array $classes = [
        SomeClass::class,
        'App\Service\SomeClass::someMethod',
        'App\Service\SomeClass::*Method',
    ];

    // 要切入的注解，具体切入的还是使用了这些注解的类，仅可切入类注解和类方法注解
    public array $annotations = [
        SomeAnnotation::class,
    ];

    public function process(ProceedingJoinPoint $proceedingJoinPoint)
    {
        ...
    }
}

```

切点（Pointcut）也可以通过`#[Aspect]`注解本身的属性进行配置：

```php
#[
    Aspect(
        classes: [
            SomeClass::class,
            "App\Service\SomeClass::someMethod",
            "App\Service\SomeClass::*Method"
        ],
        annotations: [
            SomeAnnotation::class
        ]
    )
]
class FooAspect extends AbstractAspect
{
        ...
}
```

### 增强（Advice）

在Hyperf AOP中对此简化了该功能的使用不做过细的划分，仅保留`环绕（Around）`一种形式

```php
	public function process(ProceedingJoinPoint $proceedingJoinPoint)
    {
        // 切面切入后，执行对应的方法会由此来负责
        // $proceedingJoinPoint 为连接点，通过该类的 process() 方法调用原方法并获得结果
        // 在调用前进行某些处理
        $result = $proceedingJoinPoint->process();
        // 在调用后进行某些处理
        return $result;
    }
```

还可以通过获取原实例、方法反射、提交参数、获取注解等方式实现业务需求

```php
	public function process(ProceedingJoinPoint $proceedingJoinPoint)
    {
        // 获取当前方法反射原型
        /** @var \ReflectionMethod **/
        $reflect = $proceedingJoinPoint->getReflectMethod();

        // 获取调用方法时提交的参数
        $arguments = $proceedingJoinPoint->getArguments(); // array

        // 获取原类的实例并调用原类的其他方法
        $originalInstance = $proceedingJoinPoint->getInstance();
        $originalInstance->yourFunction();

        // 获取注解元数据
        /** @var \Hyperf\Di\Aop\AnnotationMetadata **/
        $metadata = $proceedingJoinPoint->getAnnotationMetadata();

        // 调用不受代理类影响的原方法
        $proceedingJoinPoint->processOriginalMethod();

        // 不执行原方法，做其他操作
        $result = date('YmdHis', time() - 86400);
        return $result;
    }
```

注意：`getInstance`获取到的类为代理类，里面的方法仍会被其他切面影响，相互嵌套调用会死循环耗尽内存。

## 应用场景

### 日志记录

```php
<?php

namespace App\Aspect;

use Hyperf\Di\Annotation\Aspect;
use Hyperf\Di\Annotation\Inject;
use Hyperf\Di\Aop\AbstractAspect;
use Hyperf\Di\Aop\ProceedingJoinPoint;
use Hyperf\HttpServer\Annotation\AutoController;
use Hyperf\HttpServer\Annotation\Controller;
use Psr\Log\LoggerInterface;

#[
	Aspect(
		classes: [
		],
		annotations: [
			Controller::class,
			AutoController::class
		]
	)
]
class LogAspect extends AbstractAspect
{
	#[Inject]
	private LoggerInterface $logger;

	public function process(ProceedingJoinPoint $proceedingJoinPoint)
	{
		$this->logger->info('start execute method: ' . $proceedingJoinPoint->className . '::' . $proceedingJoinPoint->methodName);
		$result = $proceedingJoinPoint->process();
		$this->logger->info('end execute method: ' . $proceedingJoinPoint->className . '::' . $proceedingJoinPoint->methodName);
		return $result;
	}
}
```

### 性能监控

```php
<?php

namespace App\Aspect;

use Hyperf\Di\Annotation\Aspect;
use Hyperf\Di\Annotation\Inject;
use Hyperf\Di\Aop\AbstractAspect;
use Hyperf\Di\Aop\ProceedingJoinPoint;
use Hyperf\HttpServer\Annotation\DeleteMapping;
use Hyperf\HttpServer\Annotation\GetMapping;
use Hyperf\HttpServer\Annotation\PostMapping;
use Hyperf\HttpServer\Annotation\PutMapping;
use Hyperf\HttpServer\Annotation\RequestMapping;
use Psr\Log\LoggerInterface;

#[
	Aspect(
		classes: [
		],
		annotations: [
			RequestMapping::class,
			GetMapping::class,
			PostMapping::class,
			PutMapping::class,
			DeleteMapping::class,
		]
	)
]
class PerformanceAspect extends AbstractAspect
{
	#[Inject]
	private LoggerInterface $logger;

	/**
	 * @inheritDoc
	 */
	public function process(ProceedingJoinPoint $proceedingJoinPoint)
	{
		$className = $proceedingJoinPoint->className;
		$methodName = $proceedingJoinPoint->methodName;
		$t = time();
		$result = $proceedingJoinPoint->process();
		$t2 = time();
		$this->logger->Info('【PerformanceAspect】' . $className . ' ' . $methodName . ' ' . ' 执行耗时 ' . ($t2 - $t));
		return $result;
	}
}
```

### 参数校验

#### 1、定义参数校验注解

```php
#[Attribute(Attribute::TARGET_PROPERTY)]
class Max
{
	public function __construct(public int $value)
	{

	}

}

-- Min.php

#[Attribute(Attribute::TARGET_PROPERTY)]
class Min
{
	public function __construct(public int $value)
	{

	}

}

-- NotNull.php

#[Attribute(Attribute::TARGET_PROPERTY)]
class NotNull
{
	public function __construct()
	{

	}

}
```

#### 2、定义参数校验AOP切面

此demo应用于所有Service类方法上

```php
<?php

namespace App\Aspect;

use App\Annotation\NotNull;
use App\Annotation\Max;
use App\Annotation\Min;
use Hyperf\Di\Aop\AbstractAspect;
use Hyperf\Di\Annotation\Aspect;
use Hyperf\Di\Aop\ProceedingJoinPoint;
use ReflectionClass;
use ReflectionProperty;

#[
	Aspect(
		classes: [
			'App\Service\*Service::*'
		],
		annotations: [
			Max::class,
			Min::class,
			NotNull::class,
		]
	)
]
class ValidationAspect extends AbstractAspect
{

	public function process(ProceedingJoinPoint $proceedingJoinPoint)
	{
		$params = $proceedingJoinPoint->arguments['keys'];
		foreach ($params as $param) {
			$reflection = new ReflectionClass($param);
			$properties = $reflection->getProperties();
			foreach ($properties as $property) {
				try {
					$this->validateProperty($param, $property);
				} catch (\InvalidArgumentException $e) {
					return array('code' => 500, 'msg' => $e->getMessage());
				}
			}
		}
		return $proceedingJoinPoint->process();
	}

	private function validateProperty($object, ReflectionProperty $property)
	{
		$attributes = $property->getAttributes();
		$property->setAccessible(true);
		$value = $property->getValue($object);

		foreach ($attributes as $attribute) {
			$instance = $attribute->newInstance();
			if ($instance instanceof NotNull && $value === null) {
				throw new \InvalidArgumentException("The property {$property->getName()} cannot be null.");
			}
			if ($instance instanceof Max && $value > $instance->value) {
				throw new \InvalidArgumentException("The property {$property->getName()} cannot be greater than {$instance->value}.");
			}
			if ($instance instanceof Min && $value < $instance->value) {
				throw new \InvalidArgumentException("The property {$property->getName()} cannot be less than {$instance->value}.");
			}
		}
	}
}

```

#### 3、使用

##### 3.1 Model中使用自定义参数校验注解 

```php
<?php

namespace App\Model;

use App\Annotation\Max;
use App\Annotation\Min;
use App\Annotation\NotNull;

class User
{
	#[NotNull]
	public $name;
	#[NotNull]
	#[Min(1)]
	#[Max(120)]
	public int $age;

}
```
##### 3.2 在Service层将 Model 作为参数继续传递
```php
class UserService extends AbstractService
{

	public function getUserList(User $user)
	{
		return [
			$user
		];
	}
}
```
##### 3.3 在 Controller 层调用service方法，并初始化 Model 进行传递
```php
<?php

declare(strict_types=1);

namespace App\Controller;

use App\Model\User;
use App\Service\UserService;
use Hyperf\Di\Annotation\Inject;
use Hyperf\HttpServer\Annotation\AutoController;

#[AutoController]
class UserController extends AbstractController
{

	#[Inject]
	private UserService $userService;


	public function list()
	{
		$user = new User();
		$user->name = null; // 修改此处进行验证切面
		$user->age = 130; // 修改此处进行验证切面
		$users = $this->userService->getUserList($user);
		return $users;
	}
}
```


