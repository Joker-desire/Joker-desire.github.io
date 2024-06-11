# Casbin


# Casbin概述

**Casbin**是一个强大和高效的开放源码访问控制库，它支持各种访问控制模型以强制全面执行授权。

## 是什么

Casbin是一个授权库，在我们希望特定用户访问特定的**对象**或实体的流程中可以使用**主题**访问类型，例如**动作**可以是读取、写入、删除等。这是Casbin最广泛的使用，它叫做"标准" 或经典 `{ subject, object, action }` 流程。

## 能做什么

1. 按照经典的 `{ subject, object, action }` 格式或您定义的自定义格式执行策略。支持允许和拒绝授权
2. 具有访问控制模型model和策略policy两个核心概念。
3. 支持RBAC中的多层角色继承，不止主体可以有角色，资源也可以具有角色。
4. 支持内置超级用户，如 `root` 或 `administrator`。 超级用户可以在没有明确权限的情况下做任何事情。
5. 支持多个规则匹配运算符。 For example, `keyMatch` can map a resource key `/foo/bar` to the pattern `/foo*`.

## 不能做什么

1. 身份认证 authentication（即验证用户的用户名和密码），Casbin 只负责访问控制。应该有其他专门的组件负责身份认证，然后由 Casbin 进行访问控制，二者是相互配合的关系。
2. 管理用户列表或角色列表。

# 支持的模型

1. [**ACL (Access Control List, 访问控制列表)**](https://en.wikipedia.org/wiki/Access_control_list)
2. **具有 [超级用户](https://en.wikipedia.org/wiki/Superuser) 的 ACL**
3. **没有用户的 ACL**: 对于没有身份验证或用户登录的系统尤其有用。
4. **没有资源的 ACL**: 某些场景可能只针对资源的类型, 而不是单个资源, 诸如 `write-article`, `read-log`等权限。 它不控制对特定文章或日志的访问。
5. **[RBAC (基于角色的访问控制)](https://en.wikipedia.org/wiki/Role-based_access_control)**
6. **支持资源角色的RBAC**: 用户和资源可以同时具有角色 (或组)。
7. **支持域/租户的RBAC**: 用户可以为不同的域/租户设置不同的角色集。
8. **[ABAC (基于属性的访问控制)](https://en.wikipedia.org/wiki/Attribute-Based_Access_Control)**: 支持利用`resource.Owner`这种语法糖获取元素的属性。
9. **[RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer)**: 支持路径, 如 `/res/*`, `/res/: id` 和 HTTP 方法, 如 `GET`, `POST`, `PUT`, `DELETE`。
10. **拒绝优先**: 支持允许和拒绝授权, 拒绝优先于允许。
11. **优先级**: 策略规则按照先后次序确定优先级，类似于防火墙规则。

# Model语法

- Model CONF 至少应包含四个部分: `[request_definition], [policy_definition], [policy_effect], [matchers]`。
- 如果 model 使用 RBAC, 还需要添加`[role_definition]`部分。

- 模型CONF可以包含注释。 注释开头是 `#`, `#` 将注释该行的其余部分。

## Request定义

> `[request_definition]`是访问请求的定义
>
> 它定义了`e.Enforce(...)`函数中的参数

```conf
[request_definition]
r = sub, obj, act
```

- `sub`：访问实体（Subject）
- `obj`：访问资源（Object）
- `act`：访问方法（Action）
- `eft`：策略结果，一般为空，默认指定`allow`，还可以定义为`deny`

## Policy定义

> `[policy_definition]`是策略的定义
>
> 它界定了该策略的含义

```conf
[policy_definition]
p = sub, obj, act
p2 = sub, act
```

## Policy effect定义

> `[policy_effect]`是对Policy生效范围的定义

```conf
[policy_effect]
e = some(where (p.eft == allow))
```

该Effect原语表示如果存在任意一个决策结果为`allow`的匹配规则，则最终决策结果为`allow`，即allow-override。 其中`p.eft` 表示策略规则的决策结果，可以为`allow` 或者`deny`，当不指定规则的决策结果时,取默认值`allow` 。

```conf
[policy_effect]
e = !some(where (p.eft == deny))
```

该Effect原语表示不存在任何策略结果为`deny`的匹配规则，则最终决策结果为`allow`，即deny-ouverride.

`some`量词判断是否存在一条策略规则满足匹配器。

policy effect还可以利用逻辑运算符进行连接：

```conf
[policy_effect]
e = some(where (p.eft == allow)) && !some(where (p.eft == deny))
```

该Effect原语表示当至少存在一个决策结果为`allow`的匹配规则，且不存在决策结果为`deny`的匹配规则时，则最终决策结果为`allow`。 这时`allow`授权和`deny`授权同时存在，但是`deny`优先。

| Policy effect定义                                            | 意义             | 示例                                                         |
| ------------------------------------------------------------ | ---------------- | ------------------------------------------------------------ |
| some(where (p.eft == allow))                                 | allow-override   | [ACL, RBAC, etc.](https://casbin.org/zh/docs/supported-models#examples) |
| !some(where (p.eft == deny))                                 | deny-override    | [拒绝改写](https://casbin.org/zh/docs/supported-models#examples) |
| some(where (p.eft == allow)) && !some(where (p.eft == deny)) | allow-and-deny   | [同意与拒绝](https://casbin.org/zh/docs/supported-models#examples) |
| priority(p.eft) \|\| deny                                    | priority         | [优先级](https://casbin.org/zh/docs/supported-models#examples) |
| subjectPriority(p.eft)                                       | 基于角色的优先级 | [主题优先级](https://casbin.org/zh/docs/supported-models#examples) |

## Matchers 匹配器

> `[matchers]`是策略匹配器的定义
>
> 匹配器是表达式，它确定了如何根据请求评估策略规则

```conf
[matchers]
m = r.sub == p.sub && r.obj == p.obj && r.act == p.act
```

上述匹配器是最简单的，表示请求的三元组：主题、对象、行为都应该匹配策略规则中的表达式。

在匹配器中，可以使用算数运算符，也可以使用逻辑运算符（`&&` `||` `!`）

## Role定义

> `[role_definition]`原语定义了RBAC中的角色继承关系。 Casbin支持RBAC系统的多个实例，例如，用户可以有角色和继承关系，而资源也可以有角色和继承关系。 这两个RBAC系统不会干扰

此部分是可选的。 如果在模型中不使用 RBAC 角色, 则省略此部分。

```conf
[role_definition]
g = _, _ # 表示以角色为基础
g2 = _, _, _ # 表示以域为基础（多商户模式）
```

## 完整Model示例

### ACLModel

```conf
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = r.sub == p.sub && r.obj == p.obj && r.act == p.act
```

### RBAC Model

```conf
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[role_definition]
g = _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = g(r.sub, p.sub) && r.obj == p.obj && r.act == p.act
```

### RBAC Model 多商户

```conf
[request_definition]
r = sub, dom, obj, act

[policy_definition]
p = sub, dom, obj, act

[role_definition]
g = _, _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = g(r.sub, p.sub, r.dom) && r.dom == p.dom && r.obj == p.obj && r.act == p.act
```

# 存储

## model存储

model只能加载，不能保存。

### 从`.conf`文件中加载model

**`.conf`文件的内容：**

```conf
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[role_definition]
g = _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = g(r.sub, p.sub) && r.obj == p.obj && r.act == p.act
```

**加载模型文件：**

```go
e := casbin.NewEnforcer("examples/rbac_model.conf", "examples/rbac_policy.csv")
```

### 从代码加载model

**模型可以从代码中动态初始化：**

```go
import (
	"github.com/casbin/casbin/v2"
	"github.com/casbin/casbin/v2/model"
	fileadapter "github.com/casbin/casbin/v2/persist/file-adapter"
)

// 从代码中初始化模型
m := model.NewModel()
m.AddDef("r", "r", "sub, obj, act")
m.AddDef("p", "p", "sub, obj, act")
m.AddDef("g", "g", "_, _")
m.AddDef("e", "e", "some(where (p.eft == allow))")
m.AddDef("m", "m", "g(r.sub, p.sub) && r.obj == p.obj && r.act == p.act")

// 从csv文件adapter中加载策略规则
a := fileadapter.NewAdapter("./RBACPolicy.csv")
// 创建enforcer
e, _ := casbin.NewEnforcer(m, a)
```

### 从字符串加载model

**模型也可以从多行字符串中加载整个模型文本，这种方式的使用可以不需要维护模型文件：**

```go
import (
	"github.com/casbin/casbin/v2"
	"github.com/casbin/casbin/v2/model"
	fileadapter "github.com/casbin/casbin/v2/persist/file-adapter"
)

modelText := `
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[role_definition]
g = _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = g(r.sub, p.sub) && r.obj == p.obj && r.act == p.act
`
m, _ := model.NewModelFromString(modelText)

// 从csv文件adapter中加载策略规则
a := fileadapter.NewAdapter("./RBACPolicy.csv")
// 创建enforcer
e, _ := casbin.NewEnforcer(m, a)
```



## Policy存储

### 从csv文件载入策略

csv文件示例：RBAC策略

```
p, alice, data1, read
p, bob, data2, write
p, data2_admin, data2, read
p, data2_admin, data2, write

g, alice, data2_admin
```

### 数据库存储格式

**表结构：**

```sql
CREATE TABLE `casbin_rule` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `ptype` varchar(100) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `v0` varchar(100) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `v1` varchar(100) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `v2` varchar(100) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `v3` varchar(100) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `v4` varchar(100) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `v5` varchar(100) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_casbin_rule` (`ptype`,`v0`,`v1`,`v2`,`v3`,`v4`,`v5`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

| id   | ptype | v0          | v1          | v2    | v3   | v4   | v5   |
| ---- | ----- | ----------- | ----------- | ----- | ---- | ---- | ---- |
| 1    | p     | alice       | data1       | read  |      |      |      |
| 2    | p     | bob         | data2       | write |      |      |      |
| 3    | p     | data2_admin | data2       | read  |      |      |      |
| 4    | p     | data2_admin | data2       | write |      |      |      |
| 5    | g     | alice       | data2_admin |       |      |      |      |

- `id`：数据库主键，生成方式取决于特定的适配器
- `ptype`：对应`p`，`g`，`g2`等
- `v0-v5`：列名称没有特定的意义, 并对应 `policy csv` 从左到右的值。 列数取决于您自己定义的数量。 理论上，可以有无限的列数。

## 策略子集加载

> 要使用支持的adapter处理过滤后的策略，只需调用 `LoadFilteredPolicy` 方法。 过滤器参数的有效格式取决于所用的适配器。 为了防止意外数据丢失，当策略已经加载， `SavePolicy` 方法会被禁用。

```go
import (
	"fmt"
	"github.com/casbin/casbin/v2"
	fileadapter "github.com/casbin/casbin/v2/persist/file-adapter"
)

// 创建enforcer
enforcer, _ := casbin.NewEnforcer()

// 创建一个过滤适配器
adapter := fileadapter.NewFilteredAdapter("./RBACPolicy.csv")

_ = enforcer.InitWithAdapter("./RBACModel.conf", adapter)
// 添加过滤策略，只加载过滤器中的策略
filter := &fileadapter.Filter{
    P: []string{"bob", "data2"},
    G: []string{"", "data2_admin"},
}
_ = enforcer.LoadFilteredPolicy(filter)
```



# 简单使用

## 安装

`go get github.com/casbin/casbin/v2`

## 使用配置文件

定义两个配置文件：`model.conf` 和 `policy.csv`

> model.conf

```conf
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = r.sub == p.sub && r.obj == p.obj && r.act == p.act
```

> policy.csv

```
p, alice, data1, read
```


