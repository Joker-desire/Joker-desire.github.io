# Laravel 11.x Collections


----

[原文](https://laravel.com/docs/11.x/collections)

----

## 介绍

`Illuminate\Support\Collection`类为处理数据数组提供了一个流畅、方便的包装。
例如，以下代码将使用`collect`辅助函数从数组创建一个新的集合实例，对每个元素运行`strtoupper`函数，然后移除所有空元素：

```php
$collection = collect(['taylor', 'abigail', null])->map(function (?string $name) {
    return strtoupper($name);
})->reject(function (string $name) {
    return empty($name);
});
```

如上所见，`Collection`类允许你链式调用其方法，以对底层数组进行流畅的映射和简化。一般来说，集合是不可变的，这意味着每个`Collection`方法都会返回一个全新的`Collection`实例。

### 创建集合

`collect`辅助函数为给定数组返回一个新的`Illuminate\Support\Collection`实例。

```php
$collection = collect([1, 2, 3]);
```

***ps: Eloquent 查询的结果始终作为 Collection 实例返回。***

### 扩展集合

集合是 '可宏化的'，这允许你在运行时为 Collection 类添加其他方法。Illuminate\Support\Collection 类的 `macro` 方法接受一个闭包，该闭包将在你的宏被调用时执行。宏闭包可以通过 `$this` 访问集合的其他方法，就像它是真正的集合类方法一样。例如，以下代码为 Collection 类添加了一个 `toUpper` 方法：

```php
use Illuminate\Support\Collection;
use Illuminate\Support\Str;

Collection::macro('toUpper', function () {
    return $this->map(function (string $value) {
        return Str::upper($value);
    });
});

$collection = collect(['first', 'second']);

$upper = $collection->toUpper();

// ['FIRST', 'SECOND']
```

通常，应该在服务提供商的 `boot` 方法中声明集合宏。

#### Macro 参数

可以定义接受其他参数的宏：

```php
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\Lang;

Collection::macro('toLocale', function (string $locale) {
    return $this->map(function (string $value) use ($locale) {
        return Lang::get($value, [], $locale);
    });
});

$collection = collect(['first', 'second']);

$translated = $collection->toLocale('es');
```

## 可用方法

<table>
<tr>
<th>方法</th>
<th>说明</th>
</tr>
<tr>
<td>after()</td>
<td>

返回给定项之后的项。如果给定项未找到或者它是最后一项，则返回 `null`

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->after(3);

// 4

$collection->after(5);

// null
```

此方法使用'宽松'比较来搜索给定项，这意味着包含整数值的字符串将被视为与相同值的整数相等。要使用'严格'比较，你可以向方法提供 `strict` 参数：

```php
collect([2, 4, 6, 8])->after('4', strict: true);

// null
```

或者，你可以提供自己的闭包来搜索通过给定真值测试的第一个项：

```php
collect([2, 4, 6, 8])->after(function (int $item, int $key) {
    return $item > 5;
});

// 8
```

</td>
</tr>
<tr>
<td>all()</td>
<td>

返回集合表示的基础数组

```php
collect([1, 2, 3])->all();

// [1, 2, 3]
```

</td>
</tr>
<tr>
<td>average()</td>
<td>

avg方法的别名

</tr>
<tr>
<td>avg()</td>
<td>

返回给定键的平均值

```php
$average = collect([
    ['foo' => 10],
    ['foo' => 10],
    ['foo' => 20],
    ['foo' => 40]
])->avg('foo');

// 20

$average = collect([1, 1, 2, 4])->avg();

// 2
```

</td>
</tr>
<tr>
<td>before()</td>
<td>

返回给定项目之前的项目。如果未找到给定的项货位第一项，则返回`null`

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->before(3);

// 2

$collection->before(1);

// null

collect([2, 4, 6, 8])->before('4', strict: true);

// null

collect([2, 4, 6, 8])->before(function (int $item, int $key) {
    return $item > 5;
});

// 4
```

</td>
</tr>
<tr>
<td>chunk()</td>
<td>

将集合分解为多个给定大小的较小集合

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7]);

$chunks = $collection->chunk(4);

$chunks->all();

// [[1, 2, 3, 4], [5, 6, 7]]
```

</td>
</tr>
<tr>
<td>chunkWhile()</td>
<td>

基于给定回调的评估将集合分解为多个较小的集合。传递给闭包的 `$chunk` 变量可以用于检查前一个元素

```php
$collection = collect(str_split('AABBCCCD'));

$chunks = $collection->chunkWhile(function (string $value, int $key, Collection $chunk) {
    return $value === $chunk->last();
});

$chunks->all();

// [['A', 'A'], ['B', 'B'], ['C', 'C', 'C'], ['D']]
```

</td>
</tr>
<tr>
<td>collapse()</td>
<td>

将数组的集合折叠成一个单一的、扁平的集合

```php
$collection = collect([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]);

$collapsed = $collection->collapse();

$collapsed->all();

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

</td>
</tr>
<tr>
<td>collect()</td>
<td>

返回一个新的 `Collection` 实例，其中包含当前在集合中的项目

```php
$collectionA = collect([1, 2, 3]);

$collectionB = $collectionA->collect();

$collectionB->all();

// [1, 2, 3]
```

`collect`方法主要用于将惰性集合转化为标准`Collection`实例：

```php
$lazyCollection = LazyCollection::make(function () {
    yield 1;
    yield 2;
    yield 3;
});

$collection = $lazyCollection->collect();

$collection::class;

// 'Illuminate\Support\Collection'

$collection->all();

// [1, 2, 3]
```

***ps：当你有一个 `Enumerable` 实例并且需要一个非惰性集合实例时，`collect` 方法特别有用。由于 `collect()` 是 `Enumerable` 合约的一部分，因此您可以安全地使用它来获取 `Collection` 实例。***
</td>
</tr>
<tr>
<td>combine()</td>
<td>

将集合的值作为键与另一个数组或集合值组合在一起

```php
$collection = collect(['name', 'age']);

$combined = $collection->combine(['George', 29]);

$combined->all();

// ['name' => 'George', 'age' => 29]
```

</td>
</tr>
<tr>
<td>concat()</td>
<td>

将给定数组或集合的值附加到另一个集合的末尾

```php
$collection = collect(['John Doe']);

$concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

$concatenated->all();

// ['John Doe', 'Jane Doe', 'Johnny Doe']
```

</td>
</tr>
<tr>
<td>contains()</td>
<td>

确定集合中是否包含给定的项目。可以将一个闭包传递给`contains`方法，以确定集合中是否存在与给定真值测试匹配的元素

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->contains(function (int $value, int $key) {
    return $value > 5;
});

// false
```

可以将字符串传递给 contains 方法，以确定集合是否包含给定的 item 值：

```php
$collection = collect(['name' => 'Desk', 'price' => 100]);

$collection->contains('Desk');

// true

$collection->contains('New York');

// false
```

也可以将一个键/值对传递给 contains 方法，该方法将确定给定的对是否存在于集合中：

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);

$collection->contains('product', 'Bookcase');

// false
```

`contains` 方法在检查项值时使用'宽松'比较，这意味着包含整数值的字符串将被视为与相同值的整数相等。使用 `containsStrict` 方法可以使用'严格'比较进行筛选。

</td>
</tr>
<tr>
<td>containsOneItem()</td>
<td>
确定集合是否包含单个项目

```php
collect([])->containsOneItem();

// false

collect(['1'])->containsOneItem();

// true

collect(['1', '2'])->containsOneItem();

// false
```

</td>
</tr>
<tr>
<td>containsStrict()</td>
<td>

此方法与 `contains` 方法具有相同的签名；然而，所有值都使用'严格'比较进行比较。

</td>
</tr>
<tr>
<td>count()</td>
<td>

返回集合中的项目总数

```php
$collection = collect([1, 2, 3, 4]);

$collection->count();

// 4
```

</td>
</tr>
<tr>
<td>countBy()</td>
<td>

计算集合中值的出现次数

```php
$collection = collect([1, 2, 2, 2, 3]);

$counted = $collection->countBy();

$counted->all();

// [1 => 1, 2 => 3, 3 => 1]

// 将一个闭包传递给countBy方法，以按自定义值对所有项目进行计数
$collection = collect(['alice@gmail.com', 'bob@yahoo.com', 'carlos@gmail.com']);

$counted = $collection->countBy(function (string $email) {
    return substr(strrchr($email, "@"), 1);
});

$counted->all();

// ['gmail.com' => 2, 'yahoo.com' => 1]
```

</td>
</tr>
<tr>
<td>crossJoin()</td>
<td>

在给定数组或集合之间交叉连接集合的值，返回具有所有可能排列的笛卡尔积<

```php
$collection = collect([1, 2]);
 
$matrix = $collection->crossJoin(['a', 'b']);
 
$matrix->all();
 
/*
    [
        [1, 'a'],
        [1, 'b'],
        [2, 'a'],
        [2, 'b'],
    ]
*/
 
$collection = collect([1, 2]);
 
$matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);
 
$matrix->all();
 
/*
    [
        [1, 'a', 'I'],
        [1, 'a', 'II'],
        [1, 'b', 'I'],
        [1, 'b', 'II'],
        [2, 'a', 'I'],
        [2, 'a', 'II'],
        [2, 'b', 'I'],
        [2, 'b', 'II'],
    ]
*/
```

</td>
</tr>
<tr>
<td>dd()</td>
<td>

输出集合的项并结束脚本的执行

```php
$collection = collect(['John Doe', 'Jane Doe']);
 
$collection->dd();
 
/*
    Collection {
        #items: array:2 [
            0 => "John Doe"
            1 => "Jane Doe"
        ]
    }
*/
```

</td>
</tr>
<tr>
<td>diff()</td>
<td>

根据集合的值将集合与另一个集合或普通的数组进行比较。此方法将返回原始集合中不存在于给定集合中的值

```php
$collection = collect([1, 2, 3, 4, 5]);
 
$diff = $collection->diff([2, 4, 6, 8]);
 
$diff->all();
 
// [1, 3, 5]
```

</td>
</tr>
<tr>
<td>diffAssoc()</td>
<td>

根据集合的键和值将集合与另一个集合或者数组进行比较。此方法将返回原始集合中不存在与给定集合中的键值对

```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6,
]);
 
$diff = $collection->diffAssoc([
    'color' => 'yellow',
    'type' => 'fruit',
    'remain' => 3,
    'used' => 6,
]);
 
$diff->all();
 
// ['color' => 'orange', 'remain' => 6]
```

</td>
</tr>
<tr>
<td>diffAssocUsing()</td>
<td>

与`diffAssoc`不同，`diffAssocUsing`接受用户提供的回调函数来进行索引比较。

```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6,
]);
 
$diff = $collection->diffAssocUsing([
    'Color' => 'yellow',
    'Type' => 'fruit',
    'Remain' => 3,
], 'strnatcasecmp');
 
$diff->all();
 
// ['color' => 'orange', 'remain' => 6]
```

回调必须是返回小于、等于或大于零的整数的比较函数。

</td>
</tr>
<tr>
<td>diffKeys()</td>
<td>

根据集合的键将集合与另一个集合或数组进行比较。此方法将返回原始集合中不存在于给定集合中的键值对：

```php
$collection = collect([
    'one' => 10,
    'two' => 20,
    'three' => 30,
    'four' => 40,
    'five' => 50,
]);
 
$diff = $collection->diffKeys([
    'two' => 2,
    'four' => 4,
    'six' => 6,
    'eight' => 8,
]);
 
$diff->all();
 
// ['one' => 10, 'three' => 30, 'five' => 50]
```

</td>
</tr>
<tr>
<td>doesntContain()</td>
<td>

确定集合是否不包含给定的项目。

可以将闭包传递给`doesntContain`方法，以确定集合中是否不存在与给定真值测试匹配的元素：

```php
$collection = collect([1, 2, 3, 4, 5]);
 
$collection->doesntContain(function (int $value, int $key) {
    return $value < 5;
});
 
// false
```

也可以将字符串传递给`doesntContain`方法来确定集合是否不包含给定的项目值：

```php
$collection = collect(['name' => 'Desk', 'price' => 100]);
 
$collection->doesntContain('Table');
 
// true
 
$collection->doesntContain('Desk');
 
// false
```

还可以将键值对传递给`doesntContain`方法，该方法将确定给定的对是否在集合中不存在：

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);
 
$collection->doesntContain('product', 'Bookcase');
 
// true
```

`doesntContain`方法在检查项目值时使用“松散”比较，这意味着具有整数值的字符串将被视为等于相同值的整数。
</td>
</tr>
<tr>
<td>dot()</td>
<td>

将多维集合展平为使用“点”表示法表示深度的单级集合：

```php
$collection = collect(['products' => ['desk' => ['price' => 100]]]);
 
$flattened = $collection->dot();
 
$flattened->all();
 
// ['products.desk.price' => 100]
```

</td>
</tr>
<tr>
<td>dump()</td>
<td>

输出集合的项：

```php
$collection = collect(['John Doe', 'Jane Doe']);
 
$collection->dump();
 
/*
    Collection {
        #items: array:2 [
            0 => "John Doe"
            1 => "Jane Doe"
        ]
    }
*/
```

</td>
</tr>
<tr>
<td>duplicates()</td>
<td>

从集合中检索并返回重复值：

```php
$collection = collect(['a', 'b', 'a', 'c', 'b']);
 
$collection->duplicates();
 
// [2 => 'a', 4 => 'b']
```

如果集合包含数组或对象，可以传递要检查重复值的属性的键：

```php
$employees = collect([
    ['email' => 'abigail@example.com', 'position' => 'Developer'],
    ['email' => 'james@example.com', 'position' => 'Designer'],
    ['email' => 'victoria@example.com', 'position' => 'Developer'],
]);
 
$employees->duplicates('position');
 
// [2 => 'Developer']
```

</td>
</tr>
<tr>
<td>duplicatesStrict()</td>
<td>

该方法与`duplicates`方法具有相同的签名；但是，所有值都是使用“严格”比较进行比较的。

</td>
</tr>
<tr>
<td>each()</td>
<td>

迭代集合中的项目并将每个项目传递给闭包：

```php
$collection = collect([1, 2, 3, 4]);
 
$collection->each(function (int $item, int $key) {
    // ...
});
```

如果要停止迭代，则可以返回`false`：

```php
$collection->each(function (int $item, int $key) {
    if (/* condition */) {
        return false;
    }
});
```

</td>
</tr>
<tr>
<td>eachSpread()</td>
<td>

迭代集合的项目，将每个嵌套项目值传递到给定的回调中：

```php
$collection = collect([['John Doe', 35], ['Jane Doe', 33]]);
 
$collection->eachSpread(function (string $name, int $age) {
    // ...
});
```

可以通过从回调返回`false`来停止迭代项目：

```php
$collection->eachSpread(function (string $name, int $age) {
    return false;
});
```

</td>
</tr>
<tr>
<td>ensure()</td>
<td>

用于验证集合的所有元素是否属于给定类型或类型列表。否则，将抛出`UnexpectedValueException`：

```php
return $collection->ensure(User::class);
 
return $collection->ensure([User::class, Customer::class]);
```

还可以指定基本类型，例如string 、 int 、 float 、 bool和array ：

```php
return $collection->ensure('int');
```

</td>
</tr>
<tr>
<td>every()</td>
<td>

用于验证集合的所有元素是否通过给定的真值测试：

```php
collect([1, 2, 3, 4])->every(function (int $value, int $key) {
    return $value > 2;
});
 
// false
```

如果集合为空，则every方法将返回 true：

```php
$collection = collect([]);
 
$collection->every(function (int $value, int $key) {
    return $value > 2;
});
 
// true
```

</td>
</tr>
<tr>
<td>except()</td>
<td>

返回集合中除具有指定键的项目之外的所有项目：

```php
$collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);
 
$filtered = $collection->except(['price', 'discount']);
 
$filtered->all();
 
// ['product_id' => 1]
```

</td>
</tr>
<tr>
<td>filter()</td>
<td>

使用给定的回调过滤集合，仅保留那些通过给定真值测试的项目：

```php
$collection = collect([1, 2, 3, 4]);
 
$filtered = $collection->filter(function (int $value, int $key) {
    return $value > 2;
});
 
$filtered->all();
 
// [3, 4]
```

如果未提供回调，则集合中所有等于false条目都将被删除：

```php
$collection = collect([1, 2, 3, null, false, '', 0, []]);
 
$collection->filter()->all();
 
// [1, 2, 3]
```

</td>
</tr>
<tr>
<td>first()</td>
<td>

返回集合中通过给定真值测试的第一个元素：

```php
collect([1, 2, 3, 4])->first(function (int $value, int $key) {
    return $value > 2;
});
 
// 3
```

还可以调用不带参数的first方法来获取集合中的第一个元素。如果集合为空，则返回null ：

```php
collect([1, 2, 3, 4])->first();
 
// 1
```

</td>
</tr>
<tr>
<td>firstOrFail()</td>
<td>

与first方法相同；然而，如果没有找到结果， Illuminate\Support\ItemNotFoundException 将抛出异常：

```php
collect([1, 2, 3, 4])->firstOrFail(function (int $value, int $key) {
    return $value > 5;
});
 
// Throws ItemNotFoundException...
```

还可以调用不带参数的`firstOrFail`方法来获取集合中的第一个元素。如果集合为空，则 `Illuminate\Support\ItemNotFoundException` 将抛出异常：

```php
collect([])->firstOrFail();
 
// Throws ItemNotFoundException...
```

</td>
</tr>
<tr>
<td>firstWhere()</td>
<td>

返回具有给定键/值对的集合中的第一个元素：

```php
$collection = collect([
    ['name' => 'Regena', 'age' => null],
    ['name' => 'Linda', 'age' => 14],
    ['name' => 'Diego', 'age' => 23],
    ['name' => 'Linda', 'age' => 84],
]);
 
$collection->firstWhere('name', 'Linda');
 
// ['name' => 'Linda', 'age' => 14]
```

还可以使用比较运算符调用firstWhere方法：

```php
$collection->firstWhere('age', '>=', 18);
 
// ['name' => 'Diego', 'age' => 23]
```

与`where`方法一样，可以将一个参数传递给`firstWhere`方法。在这种情况下， `firstWhere`方法将返回给定项键值为“truthy”的第一个项：

```php
$collection->firstWhere('age');
 
// ['name' => 'Linda', 'age' => 14]
```

</td>
</tr>
<tr>
<td>flatMap()</td>
<td>

迭代集合并将每个值传递给给定的闭包。闭包可以自由地修改项目并返回它，从而形成修改项目的新集合。然后，将数组展平一级：

```php
$collection = collect([
    ['name' => 'Sally'],
    ['school' => 'Arkansas'],
    ['age' => 28]
]);
 
$flattened = $collection->flatMap(function (array $values) {
    return array_map('strtoupper', $values);
});
 
$flattened->all();
 
// ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];
```

</td>
</tr>
<tr>
<td>flatten()</td>
<td>

将多维集合展平为单维：

```php
$collection = collect([
    'name' => 'taylor',
    'languages' => [
        'php', 'javascript'
    ]
]);
 
$flattened = $collection->flatten();
 
$flattened->all();
 
// ['taylor', 'php', 'javascript'];
```

可以向`flatten`方法传递一个“深度”参数：

```php
$collection = collect([
    'Apple' => [
        [
            'name' => 'iPhone 6S',
            'brand' => 'Apple'
        ],
    ],
    'Samsung' => [
        [
            'name' => 'Galaxy S7',
            'brand' => 'Samsung'
        ],
    ],
]);
 
$products = $collection->flatten(1);
 
$products->values()->all();
 
/*
    [
        ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
    ]
*/
```

</td>
</tr>
<tr>
<td>flip()</td>
<td>

将集合的键与其对应的值交换：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);
 
$flipped = $collection->flip();
 
$flipped->all();
 
// ['taylor' => 'name', 'laravel' => 'framework']
```

</td>
</tr>
<tr>
<td>forget()</td>
<td>

通过其键从集合中删除项目：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);
 
// Forget a single key...
$collection->forget('name');
 
// ['framework' => 'laravel']
 
// Forget multiple keys...
$collection->forget(['name', 'framework']);
 
// []
```

***ps: 与大多数其他集合方法不同， forget不会返回新的修改后的集合；它修改它所调用的集合。***

</td>
</tr>
<tr>
<td>forPage()</td>
<td>

返回一个新集合，其中包含将出现在给定页码上的项目。该方法接受页码作为其第一个参数，并接受每页显示的项目数作为其第二个参数：

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);
 
$chunk = $collection->forPage(2, 3);
 
$chunk->all();
 
// [4, 5, 6]
```

</td>
</tr>
<tr>
<td>get()</td>
<td>

返回给定键处的项目。如果键不存在，则返回null ：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);
 
$value = $collection->get('name');
 
// taylor
```

可以选择传递默认值作为第二个参数：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);
 
$value = $collection->get('age', 34);
 
// 34
```

甚至可以传递回调作为该方法的默认值。如果指定的key不存在，则返回回调结果：

```php
$collection->get('email', function () {
    return 'taylor@example.com';
});
 
// taylor@example.com
```

</td>
</tr>
<tr>
<td>groupBy()</td>
<td>

按给定键对集合的项目进行分组：

```php
$collection = collect([
    ['account_id' => 'account-x10', 'product' => 'Chair'],
    ['account_id' => 'account-x10', 'product' => 'Bookcase'],
    ['account_id' => 'account-x11', 'product' => 'Desk'],
]);
 
$grouped = $collection->groupBy('account_id');
 
$grouped->all();
 
/*
    [
        'account-x10' => [
            ['account_id' => 'account-x10', 'product' => 'Chair'],
            ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ],
        'account-x11' => [
            ['account_id' => 'account-x11', 'product' => 'Desk'],
        ],
    ]
*/
```

可以传递回调，而不是传递字符串key 。回调应返回您希望作为组键的值：

```php
$grouped = $collection->groupBy(function (array $item, int $key) {
    return substr($item['account_id'], -3);
});
 
$grouped->all();
 
/*
    [
        'x10' => [
            ['account_id' => 'account-x10', 'product' => 'Chair'],
            ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ],
        'x11' => [
            ['account_id' => 'account-x11', 'product' => 'Desk'],
        ],
    ]
*/
```

多个分组标准可以作为数组传递。每个数组元素将应用于多维数组中的相应级别：

```php
$data = new Collection([
    10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
    20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
    30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
    40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
]);
 
$result = $data->groupBy(['skill', function (array $item) {
    return $item['roles'];
}], preserveKeys: true);
 
/*
[
    1 => [
        'Role_1' => [
            10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
            20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        ],
        'Role_2' => [
            20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        ],
        'Role_3' => [
            10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
        ],
    ],
    2 => [
        'Role_1' => [
            30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
        ],
        'Role_2' => [
            40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
        ],
    ],
];
*/
```

</td>
</tr>
<tr>
<td>...</td>
<td>

[详细见](https://laravel.com/docs/11.x/collections#available-methods)

</td>
</tr>

</table>

## 高阶消息（higher order messages）

集合还支持'高级消息（higher order messages）'，这些是用于对集合执行常见操作的快捷方式。提供高级消息的集合方法有：average, avg, contains, each, every, filter, first, flatMap, groupBy, keyBy, map, max, min, partition, reject, skipUntil, skipWhile, some, sortBy, sortByDesc, sum, takeUntil, takeWhile, 和 unique。

每个高级消息都可以作为集合实例上的动态属性进行访问。例如，让我们使用 `each` 高级消息来对集合中的每个对象调用一个方法：

```php
use App\Models\User;
 
$users = User::where('votes', '>', 500)->get();
 
$users->each->markAsVip();
```

同样，我们可以使用高阶消息sum来收集用户集合的“投票”总数：

```php
$users = User::where('group', 'Development')->get();
 
return $users->sum->votes;
```

## Lazy集合

### 介绍

为了补充已经很强大的`Collection`类， `LazyCollection`类利用 PHP 的生成器来允许处理非常大的数据集，同时保持较低的内存使用率。

例如，假设您的应用程序需要处理一个多 GB 的日志文件，同时利用 Laravel 的收集方法来解析日志。惰性集合可以用于在给定时间仅将文件的一小部分保留在内存中，而不是立即将整个文件读入内存：

```php
use App\Models\LogEntry;
use Illuminate\Support\LazyCollection;
 
LazyCollection::make(function () {
    $handle = fopen('log.txt', 'r');
 
    while (($line = fgets($handle)) !== false) {
        yield $line;
    }
})->chunk(4)->map(function (array $lines) {
    return LogEntry::fromLines($lines);
})->each(function (LogEntry $logEntry) {
    // Process the log entry...
});
```

或者，假设您需要迭代 10,000 个 Eloquent 模型。当使用传统的 Laravel 集合时，所有 10,000 个 Eloquent 模型必须同时加载到内存中：

```php
use App\Models\User;
 
$users = User::all()->filter(function (User $user) {
    return $user->id > 500;
});
```

但是，查询构建器的`cursor`方法返回一个`LazyCollection`实例。这允许您仍然只对数据库运行一个查询，而且一次只将一个 Eloquent 模型加载到内存中。在此示例中，直到我们实际单独迭代每个用户时才会执行`filter`回调，从而大幅减少内存使用量：

```php
use App\Models\User;
 
$users = User::cursor()->filter(function (User $user) {
    return $user->id > 500;
});
 
foreach ($users as $user) {
    echo $user->id;
}
```

### 创建惰性集合

要创建惰性集合实例，应该将 PHP 生成器函数传递给集合的make方法：

```php
use Illuminate\Support\LazyCollection;
 
LazyCollection::make(function () {
    $handle = fopen('log.txt', 'r');
 
    while (($line = fgets($handle)) !== false) {
        yield $line;
    }
});
```

### 可枚举合约

几乎`Collection`类上可用的所有方法也可在`LazyCollection`类上使用。这两个类都实现了`Illuminate\Support\Enumerable`契约，它定义了以下方法：
[详细见链接🔗](https://laravel.com/docs/11.x/collections#the-enumerable-contract)

### 惰性集合方法

除了`Enumerable`合约中定义的方法之外， `LazyCollection`类还包含以下方法：

#### takeUntilTimeout()

返回一个新的惰性集合，它将枚举指定时间之前的值。之后，集合将停止枚举：

```php
$lazyCollection = LazyCollection::times(INF)
    ->takeUntilTimeout(now()->addMinute());
 
$lazyCollection->each(function (int $number) {
    dump($number);
 
    sleep(1);
});
 
// 1
// 2
// ...
// 58
// 59
```

#### tapEach()

虽然`each`方法立即为集合中的每个项目调用给定的回调，但`tapEach`方法仅在项目被逐一从列表中拉出时调用给定的回调：

```php
// Nothing has been dumped so far...
$lazyCollection = LazyCollection::times(INF)->tapEach(function (int $value) {
    dump($value);
});
 
// Three items are dumped...
$array = $lazyCollection->take(3)->all();
 
// 1
// 2
// 3
```

#### throttle()

将限制惰性集合，以便在指定的秒数后返回每个值。此方法对于可能与限制传入请求速率的外部 API 交互的情况特别有用：

```php
use App\Models\User;
 
User::where('vip', true)
    ->cursor()
    ->throttle(seconds: 1)
    ->each(function (User $user) {
        // Call external API...
    });
```

#### remember()

返回一个新的惰性集合，它将记住已枚举的任何值，并且不会在后续集合枚举中再次检索它们：

```php
// No query has been executed yet...
$users = User::cursor()->remember();
 
// The query is executed...
// The first 5 users are hydrated from the database...
$users->take(5)->all();
 
// First 5 users come from the collection's cache...
// The rest are hydrated from the database...
$users->take(20)->all();
```

