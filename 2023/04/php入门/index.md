# PHP入门


# 一、初识PHP？

## 1.1 `PHP`是什么

> PHP(Hypertext Preprocessor 超文本预处理器)的简称，是一种被广泛应用的开源通用的服务器端脚本语言，适用于web开发并可嵌入HTML中

- 通用：指跨平台，如：Windows、Linux、MacOS
- 开源：意味着你可以轻松获取全部源代码，并进行定制或扩展
- 免费：意味着你不必为PHP花一分钱，哪怕用在商业项目中
- 服务器端：意味着你必须将它安装在服务器环境下才可以使用
- 脚本语言：解释型语言，按编写顺序执行。是指不需要编译,直接由解释器/虚拟机执行的编程语言

## 1.2 `PHP`能做什么

- 可以快速动态的生成HTML页面
- 可以返回前端需要的各种类型的数据
- 可以高效安全的处理表单数据
- 可以安全的操作服务器上的文件
- 可以控制与客户端的会话( Cookie/Session )
- 可以对用户的行为进行授权控制
- 可以高效安全的操作各种类型的数据库
- 通过扩展，可以实现加密，压缩等其他功能
- 可以提供接口数据，包括：小程序、APP、等其他语言

## 1.3 `PHP`程序执行流程

![image-20230428125420741](https://raw.githubusercontent.com/XD825/picgo/main/img/202304281254809.png)

# 二、`PHP`基础

## 2.1 `php`程序

- PHP文件的默认扩展名是`.php`
- PHP文件中可以包含`html`、`CSS`、`JavaScript`代码

### 2.1.1 `PHP`标记

> 开始标记`<?php` 和结束标记`?>`中间写`php`代码
>
> 当解析一个文件时，PHP会寻找起始和结束标记，也就是告诉PHP开始和停止解析二者之间的代码。
>
> 此种解析方式使得PHP可以被嵌入到各种不同的文档中去，而任何起始和结束标记之外的部分都会被PHP解析器忽略

```php
<?php
  ...  
?>
```

### 2.1.2 `PHP`代码

| 指令    | 描述                                 |
| ------- | ------------------------------------ |
| `echo`  | 可以输出一个或多个字符串，用逗号隔开 |
| `print` | 只允许输出一个字符串                 |

### 2.1.3 语句结束符`;`

一行代码结束应该以`;`结束

```php
<?php
echo 111, 222;
print 11;

```

### 2.1.4 注释

```php
<?php
    // 这是单行注释
    /*
        这是多行注释
        注释后，在浏览器和网页源码中，是看不到的。
    */
    
```

## 2.3 `PHP`变量

### 2.3.1 声明变量

> 声明变量使用`$`
>
> `=` 属于赋值运算符

```php
<?php
$a = 2;
$b = 3;
echo $a + $b;

```

### 2.3.2 变量命名规则

- 不能以数字开头
- 中间不能有空格

```php
<?php
# 下划线命名法
$new_title = 'php是世界上最好的语言';
echo $new_title;
echo '<hr>';
# 小驼峰命名法
$newTitle  = 'php是世界上最好的语言';
echo $newTitle;
echo '<hr>';
# 大驼峰命名法
$NewTitle  = 'php是世界上最好的语言';
echo $NewTitle;

```

## 2.4 `php`标量数据类型

| 类型             | 描述                 |
| ---------------- | -------------------- |
| 布尔型 `Boolean` | `true` 和 `false`    |
| 整型 `Integer`   | 0~无限大             |
| 浮点型 `Float`   | 带小数的数字         |
| 字符串 `String`  | 汉字、英文、复合、等 |

> `echo` 输出数据值
>
> `var_dump` 可以打印数据类型和值

### 2.4.1 布尔型

- 布尔型通常用于条件判断
- 当`echo`遇上`true`的时候返回1，遇上`false`则返回空字符串

```php
<?php
$a = true;
echo $a;
var_dump($a);

$b = false;
echo $b;
var_dump($b);

```

### 2.4.2 整型

- 整数不能包含逗号或空格
- 整数是没有小数点的
- 整数可以是整数或负数
- 整数可以用三种格式来指定：十进制、十六进制、八进制

```php
<?php
$number = 0;
var_dump($number); //int(0)

$number = 67;
var_dump($number); //int(67)

$number = -322;
var_dump($number);//int(-322)

```

### 2.4.3 浮点型

```php
<?php
$number = 10.03;
var_dump($number); //float(10.03)

$number = -88.23;
var_dump($number); //float(-88.23)

```

### 2.4.4 字符串

```php
<?php
$str = '你好';
var_dump($str); //string(6) "你好" 一个中文字符占3个字节

$str = 'Hello PHP!';
var_dump($str); //string(10) "Hello PHP!"
```

## 2.5 `php`复合数据类型

| 类型     | 描述   |
| -------- | ------ |
| array    | 数组   |
| object   | 对象   |
| callable | 可调用 |
| iterable | 可迭代 |

### 2.5.1 数组

| 类型         | 描述                           |
| ------------ | ------------------------------ |
| 数组 `Array` | 数组可以在一个变量中存储多个值 |

1. 创建空数组

    ```php
    <?php
    $arr = array();
    var_dump($arr); //array(0) {}
    
    $arrs = [];
    var_dump($arrs); //array(0) {}
    ```

2. 创建索引数组

    ```php
    <?php
    $arr = array(
        '足球',
        '篮球',
        '排球'
    );
    var_dump($arr);
    /*
    array(3) {
      [0]=>
      string(6) "足球"
      [1]=>
      string(6) "篮球"
      [2]=>
      string(6) "排球"
    }
     */
    
    ```

3. 创建关联数组

    ```php
    <?php
    $arr = [
      'Football' => '足球',
      'Basketball' => '篮球',
      'Volleyball' => '排球'
    ];
    var_dump($arr);
    /*
    array(3) {
      ["Football"]=>
      string(6) "足球"
      ["Basketball"]=>
      string(6) "篮球"
      ["Volleyball"]=>
      string(6) "排球"
    }
     */
    ```

4. 输出数组值

    ```php
    <?php
    $arr = array(
      '足球',
      '篮球',
      '排球'
    );
    echo $arr[0]; //足球
    echo $arr[1]; //篮球
    echo $arr[2]; //排球
    
    $arr = [
      'Football' => '足球',
      'Basketball' => '篮球',
      'Volleyball' => '排球'
    ];
    echo $arr['Football']; //足球
    echo $arr['Basketball']; //篮球
    echo $arr['Volleyball']; //排球
    ```

5. 打印数组`print_r`

    ```php
    <?php
    $arr = array(
      '足球',
      '篮球',
      '排球'
    );
    print_r($arr);
    /*
    Array
    (
        [0] => 足球
        [1] => 篮球
        [2] => 排球
    )
    ```

6. `php`连接符

    ```php
    <?php
    $var1 = 'Hello';
    $var2 = 'World';
    var_dump($var1 . $var2); //string(10) "HelloWorld"
    var_dump($var1 . ' ' . $var2); //string(11) "Hello World"
    ```

### 2.5.2 `php`多维数组

1. 二维数组

    ```php
    <?php
    $arr = array(
      array(
        'name' => 'Joker',
        'age' => 18
      ),
      array(
        'name' => 'desire',
        'age' => 19
      ),
      array(
        'name' => 'tommy',
        'age' => 20
      )
    );
    
    var_dump($arr);
    /*
    array(3) {
      [0]=>
      array(2) {
        ["name"]=>
        string(5) "Joker"
        ["age"]=>
        int(18)
      }
      [1]=>
      array(2) {
        ["name"]=>
        string(6) "desire"
        ["age"]=>
        int(19)
      }
      [2]=>
      array(2) {
        ["name"]=>
        string(5) "tommy"
        ["age"]=>
        int(20)
      }
    }
    */
    
    print_r($arr);
    /*
    Array
    (
        [0] => Array
            (
                [name] => Joker
                [age] => 18
            )
    
        [1] => Array
            (
                [name] => desire
                [age] => 19
            )
    
        [2] => Array
            (
                [name] => tommy
                [age] => 20
            )
    
    )
    */
    
    ```

2. 三维数组

    ```php
    <?php
    $arr = [
      [
        'name' => 'Joker',
        'age' => 18,
        'hobby' => [
          '爬山',
          '读书',
          '徒步'
        ]
      ],
      [
        'name' => 'desire',
        'age' => 19,
        'hobby' => [
          '篮球',
          '攀岩',
          '游泳'
        ]
      ],
      [
        'name' => 'tommy',
        'age' => 20,
        'hobby' => [
          '足球',
          '潜水',
          '摄影'
        ]
      ]
    ];
    
    var_dump($arr);
    /*
    array(3) {
      [0]=>
      array(3) {
        ["name"]=>
        string(5) "Joker"
        ["age"]=>
        int(18)
        ["hobby"]=>
        array(3) {
          [0]=>
          string(6) "爬山"
          [1]=>
          string(6) "读书"
          [2]=>
          string(6) "徒步"
        }
      }
      [1]=>
      array(3) {
        ["name"]=>
        string(6) "desire"
        ["age"]=>
        int(19)
        ["hobby"]=>
        array(3) {
          [0]=>
          string(6) "篮球"
          [1]=>
          string(6) "攀岩"
          [2]=>
          string(6) "游泳"
        }
      }
      [2]=>
      array(3) {
        ["name"]=>
        string(5) "tommy"
        ["age"]=>
        int(20)
        ["hobby"]=>
        array(3) {
          [0]=>
          string(6) "足球"
          [1]=>
          string(6) "潜水"
          [2]=>
          string(6) "摄影"
        }
      }
    }
    */
    
    print_r($arr);
    /*
    Array
    (
        [0] => Array
            (
                [name] => Joker
                [age] => 18
                [hobby] => Array
                    (
                        [0] => 爬山
                        [1] => 读书
                        [2] => 徒步
                    )
    
            )
    
        [1] => Array
            (
                [name] => desire
                [age] => 19
                [hobby] => Array
                    (
                        [0] => 篮球
                        [1] => 攀岩
                        [2] => 游泳
                    )
    
            )
    
        [2] => Array
            (
                [name] => tommy
                [age] => 20
                [hobby] => Array
                    (
                        [0] => 足球
                        [1] => 潜水
                        [2] => 摄影
                    )
    
            )
    
    )
    */
    ```

3. 多维数组访问

    ```php
    <?php
    $arr = [
      [
        'name' => 'Joker',
        'age' => 18,
        'hobby' => [
          '爬山',
          '读书',
          '徒步'
        ]
      ],
      [
        'name' => 'desire',
        'age' => 19,
        'hobby' => [
          '篮球',
          '攀岩',
          '游泳'
        ]
      ],
      [
        'name' => 'tommy',
        'age' => 20,
        'hobby' => [
          '足球',
          '潜水',
          '摄影'
        ]
      ]
    ];
    
    echo $arr[0]['name'] . '----'; //Joker----
    echo $arr[0]['age'] . '----'; //18----
    echo $arr[0]['hobby'][0] . '----'; //爬山----
    echo $arr[0]['hobby'][1] . '----'; //读书----
    echo $arr[0]['hobby'][2] . '----'; //徒步----
    ```

### 2.5.3 `php`数组循环

1. `foreach`

    ```php
    <?php
    $arr = [
      'Football' => '足球',
      'Basketball' => '篮球',
      'Volleyball' => '排球'
    ];
    
    foreach ($arr as $v) {
      echo $v . "\n";
    }
    /*
    足球
    篮球
    排球
    */
    ```

2. `key`和`value`

    ```php
    <?php
    $arr = [
      'Football' => '足球',
      'Basketball' => '篮球',
      'Volleyball' => '排球'
    ];
    
    foreach ($arr as $k => $v) {
      echo $k . "=>" . $v . "\n";
    }
    /*
    Football=>足球
    Basketball=>篮球
    Volleyball=>排球
    */
    
    $arr = array(
      '足球',
      '篮球',
      '排球'
    );
    foreach ($arr as $k => $v) {
      echo $k . "=>" . $v . "\n";
    }
    /*
    0=>足球
    1=>篮球
    2=>排球
    */
    ```



## 2.6 `php`特殊数据类型

| 类型        | 描述           |
| ----------- | -------------- |
| 控制 `NULL` | 表示变量没有值 |
| resource    | 资源           |

### 2.6.1 NULL

```php
<?php
$null;
var_dump($null); // NULL 会有警告:Warning: Undefined variable $null in /var/www/html/php8/DataType/Data.php on line 25

$null = "";
var_dump($null); //string(0) ""

$null = null;
var_dump($null); //NULL

```

## 2.7 条件判断

### 2.7.1 三元运算符` ? : `

```php
<?php

$name = 'Joker';
$res = $name ? "我是$name" : '我是谁？';
var_dump($res); // string(11) "我是Joker"

$name = '';
$res = $name ? "我是$name" : '我是谁？';
var_dump($res); // string(12) "我是谁？"

```

### 2.7.2 `if`

```php
<?php

$name = 'Joker';
if ($name) {
    echo "我是$name"; //我是Joker
}

```



### 2.7.3 `if else`

```php
<?php
$name = 'Joker';
if ($name) {
    echo "我是$name"; //我是Joker
} else {
    echo "我是谁？";
}

$name = '';
if ($name) {
    echo "我是$name";
} else {
    echo "我是谁？"; //我是谁？
}
```

### 2.7.4 `if elseif else`

```php
<?php
$score = 80;

if ($score >= 80) {
    echo "良好"; // 良好
} elseif ($score >= 60) {
    echo "及格";
} else {
    echo "差";
}
```

### 2.7.5 `switch case default`

```php
<?php
$userID = 1;
switch ($userID) {
    case 1:
        echo 'Joker';
    case 2:
        echo 'Desire';
    case 3:
        echo 'Tommy';
    default:
        echo '未知';
}
// 结果：JokerDesireTommy未知 
// 出现的问题，会把所有的情况数据都打印出来
// 解决：使用break匹配到后自动结束掉
switch ($userID) {
    case 1:
        echo 'Joker'; // Joker
        break;
    case 2:
        echo 'Desire';
        break;
    case 3:
        echo 'Tommy';
        break;
    default:
        echo '未知';
        break;
}

```

### 2.7.6 `PHP8`新特性`match`

```php
<?php
$userID = 1003;
echo match ($userID) {
    1001 => 'Joker',
    1002 => 'Desire',
    1003 => 'Tommy'
}; // Tommy

$userID = 1004;
echo match ($userID) {
    1001 => 'Joker',
    1002 => 'Desire',
    1003 => 'Tommy',
    default => '未知'
}; // 未知
```

### 2.7.7 `switch`和`match`对比



## 运算符

### `php`运算符

| 运算符             | 描述             |
| ------------------ | ---------------- |
| `+`、`-`、`*`、`/` | 加减乘除         |
| `%`                | 取余             |
| `++`、`--`         | 加加、减减       |
| `.`                | 连接，用在字符串 |

```php
<?php
var_dump(120 + 3); //int(123)
var_dump(120 - 3); //int(117)
var_dump(120 - 130); //int(-10)
var_dump(120 * 3); //int(360)
var_dump(120 / 7); //float(17.142857142857142)
var_dump(120 % 7); //int(1)
$num = 10;
$num++;
var_dump($num); //int(11)
$num = 10;
$num--;
var_dump($num); //int(9)
$num = 10;
++$num;
var_dump($num); //int(11)
$num = 10;
--$num;
var_dump($num); //int(9)

// 
$a1 = 'Hello';
$a2 = 'World';
$num = 10;
var_dump($a1 . $a2); //string(10) "HelloWorld"
var_dump($a1 . $num);//string(7) "Hello10"
```

### `php`赋值运算符

| 运算符 | 描述         |
| ------ | ------------ |
| `=`    | 赋值运算符   |
| `+=`   | 先加再赋值   |
| `-=`   | 先减后赋值   |
| `*=`   | 先乘后赋值   |
| `/=`   | 先除后赋值   |
| `%=`   | 先取余后赋值 |
| `.=`   | 先连接后赋值 |

```php
<?php
$count = 100;
var_dump($count); //int(100)

$count = 100;
$count += 1;
var_dump($count); //int(101)

$count = 100;
$count -= 1;
var_dump($count); //int(99)

$count = 100;
$count *= 2;
var_dump($count); //int(200)

$count = 100;
$count /= 3;
var_dump($count); //float(33.333333333333336)

$count = 100;
$count %= 3;
var_dump($count); //int(1)

$count = 100;
$count .= 3;
var_dump($count); //string(4) "1003"
```

### `php`比较运算符

| 运算符 | 描述     |
| ------ | -------- |
| `>`    | 大于     |
| `>=`   | 大于等于 |
| `<`    | 小于     |
| `<=`   | 小于等于 |
| `==`   | 等于     |
| `!=`   | 不等于   |
| `===`  | 恒等于   |
| `!==`  | 恒不等   |

```php
<?php
// 大于
var_dump(100 > 100); //bool(false)
var_dump(100 > 90); //bool(true)
var_dump(100 > 110); //bool(false)
var_dump(true > false); //bool(true)
var_dump('php' > 'php'); //bool(false)

// 大于等于
var_dump(100 >= 100); //bool(true)
var_dump(100 >= 90); //bool(true)
var_dump(100 >= 110); //bool(false)
var_dump(true >= false); //bool(true)
var_dump('php' >= 'php'); //bool(true)

// 小于
var_dump(100 < 100); //bool(false)
var_dump(100 < 90); //bool(false)
var_dump(100 < 110); //bool(true)
var_dump(true < false); //bool(false)
var_dump('php' < 'php'); //bool(false)

// 小于等于
var_dump(100 <= 100); //bool(true)
var_dump(100 <= 90); //bool(false)
var_dump(100 <= 110); //bool(true)
var_dump(true <= false); //bool(false)
var_dump('php' <= 'php'); //bool(true)

// 等于
var_dump(100 == 100); //bool(true)
var_dump(true == true); //bool(true)
var_dump('php' == 'php'); //bool(true)

// 不等于
var_dump(100 != 100); //bool(false)
var_dump(true != true); //bool(false)
var_dump('php' != 'php'); //bool(false)

// 恒等于
var_dump(100 === 100); //bool(true)
var_dump(true === 'true'); //bool(false)

// 恒不等
var_dump(100 !== 100); //bool(false)
var_dump(true !== 'true'); //bool(true)

// php8新特性：字符串与数字的比较
var_dump('中国' > 100); //bool(true)
var_dump('中国' == 100); //bool(false)

```

### `php`逻辑运算符

| 运算符        | 描述 |
| ------------- | ---- |
| `and` 和 `&&` | 与   |
| `or` 和 `||`  | 或   |
| `xor`         | 异或 |
| `!`           | 非   |

```php
<?php
// and 和 &&
// 两个真返回真，有一个假则返回假
var_dump(100 && 30); //bool(true)
var_dump(true && true); //bool(true)
var_dump(true && false); //bool(false)
var_dump(false && false); //bool(false)

// or 和 ||
// 一个真返回真，两个假返回假
var_dump(100 || 30); //bool(true)
var_dump(true || true); //bool(true)
var_dump(true || false); //bool(true)
var_dump(false || false); //bool(false)

// xor
//一个真、返回真。两个真，返回假。两个假，也返回假。
var_dump(0 xor 1); //bool(true)
var_dump(true xor true); //bool(false)
var_dump(true xor false); //bool(true)
var_dump(false xor false); //bool(false)

// !
// 取反，如果是真，就是假。如果是假，就是真。
var_dump(!0); //bool(true)
var_dump(!true); //bool(false)
var_dump(!false); //bool(true)
var_dump(!1); //bool(false)
```



## 循环

### `while`

```php
<?php
// while
$int = 1;
while ($int < 10) {
    var_dump($int);
    $int++;
}
/*
int(1)
int(2)
int(3)
int(4)
int(5)
int(6)
int(7)
int(8)
int(9)
*/
```

### `do while`

```php
<?php
// do while
$int = 1;
do {
    var_dump($int);
} while ($int < 1);
/*
int(1)
*/
```

### `for`

```php
<?php
// for
for ($int = 1; $int < 10; $int++) {
    var_dump($int);
}
/*
int(1)
int(2)
int(3)
int(4)
int(5)
int(6)
int(7)
int(8)
int(9)
*/
```

### `continue`

- 结束当前循环，进入下次循环
- 在循环语句中使用 `while` `for`

```php
<?php
// continue
for ($int = 1; $int < 10; $int++) {
    if ($int % 2 == 0) {
        continue;
    }
    var_dump($int);
}
/*
int(1)
int(3)
int(5)
int(7)
int(9)
*/
```

### `break`

- 结束循环
- 在循环语句中使用 `while` `for` `switch`
- 可以跳出多层循环

```php
<?php
// break
for ($int = 1; $int < 10; $int++) {
    if ($int % 2 == 0) {
        var_dump($int);
        break;
    }
}
/*
int(2)
*/

$score = 80;

switch ($score) {
    case $score == 100:
        echo '相当优秀';
        break;
    case $score >= 80 && $score < 100:
        echo '优秀';
        break;
    case $score >= 60 && $score < 80:
        echo '良好';
        break;
    case $score < 60:
        echo '不及格';
        break;
    default:
        echo '得分有误';
        break;
}
```





# 三、函数

## 3.1 函数介绍

### 3.1.1 函数判断

```php
<?php

// 直接判断不存在的变量，会有警告（不会报错）  Warning: Undefined variable $name
var_dump($name); //NULL
if ($name) {
    echo "我是$name";
} else {
    echo "我是谁？";
}

// 使用isset函数判断
var_dump(isset($name)); //bool(false)
if (isset($name)) {
    echo "我是$name";
}

// 使用empty函数判断,Warning: Undefined variable $name
var_dump(empty($name)); //bool(true) 
if (empty($name)) {
    echo "我是$name";
}

```

### 3.1.2 函数的分类

- 根据函数的提供者分为两类：
    - 系统函数：编写语言开发者事先写好提供给开发者直接是用的（内置函数）
    - 自定义函数：用户根据自身需求，对系统功能进行扩展

## 3.2 系统函数

PHP拥有超过1000个内置函数

| 函数集合名   | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| `String`     | 字符串处理函数                                               |
| `Array`      | 数组函数允许访问和操作数组                                   |
| `MySQLi`     | 允许访问MySQL数据库服务器                                    |
| `Date`       | 服务器上获取日期和时间                                       |
| `Filesystem` | 允许访问和操作文件系统                                       |
| `Mail`       | 数学函数能处理integer和float范围内的值                       |
| `HTTP`       | 允许在其他输出被发送之前，对由web服务器发送到浏览器的信息进行操作 |
| `Calendar`   | 日历扩展包括了简化不同日历格式间转换的函数                   |
| `Directory`  | 允许获得关于目录及其内容的信息                               |
| `Error`      | 允许对错误进行处理和记录                                     |
| `Filter`     | 进行验证和过滤                                               |
| `FTP`        | 通过文件传输协议(FTP)提供对文件服务器的客户端访问            |
| `MySQL`      | 允许访问MySQL数据库服务器                                    |
| `SimpleXML`  | 允许把XML转换为对象                                          |
| `XML`        | 允许解析XML文档，但无法对其进行验证                          |
| `Zip`        | 压缩文件函数允许读取压缩文件                                 |

### 3.2.1 `String`字符串函数

| 函数            | 描述                         |
| --------------- | ---------------------------- |
| `strtolower()`  | 将字符串转化为小写           |
| `strtoupper()`  | 将字符串转化为大写           |
| `strlen()`      | 获取字符串长度               |
| `trim()`        | 去除字符串首尾处的空白字符   |
| `ltrim()`       | 去除字符串开头处的空白字符   |
| `rtrim()`       | 去除字符串结尾处的空白字符   |
| `str_replace()` | 字符串替换                   |
| `strpbrk()`     | 字符串中查找一组字符是否存在 |
| `explode()`     | 将字符串分割为数组           |
| `md5()`         | 将字符串进行md5加密          |

```php
<?php
$msg = "Hello World！";

// 将字符串转化为小写
$msgLower = strtolower($msg);
var_dump($msgLower); // string(14) "hello world！"

// 将字符串转为大写
$msgUpper = strtoupper($msg);
var_dump($msgUpper); //string(14) "HELLO WORLD！"s

// 获取字符串长度
$msgLen = strlen($msg);
var_dump($msgLen); //int(14)

$msg = " Hello World ";
var_dump($msg); //string(13) " Hello World "
// 去除字符串首尾处的空白字符
$msgTrim = trim($msg);
var_dump($msgTrim); //string(11) "Hello World"

$msgTrim = trim("Hello World!", "!");
var_dump($msgTrim); //string(11) "Hello World"

// 去除字符串开头的空白字符
$msgLtrim = ltrim($msg);
var_dump($msgLtrim); //string(12) "Hello World "

// 去除字符串尾部的空白字符
$msgRtrim = rtrim($msg);
var_dump($msgRtrim); //string(12) " Hello World"

// 字符串替换
$msg2 = str_replace("World", "PHP", $msg);
var_dump($msg2); //string(11) " Hello PHP "

// 字符串中查找一组字符是否存在
$res = strpbrk("w", $msg);
var_dump($res); //bool(false)
$res = strpbrk('ord', $msg);
var_dump($res); //string(3) "ord"

// 将字符串分割成数组
$msgArr = explode(",", "1,2,3,4");
var_dump($msgArr);
/*
array(4) {
  [0]=>
  string(1) "1"
  [1]=>
  string(1) "2"
  [2]=>
  string(1) "3"
  [3]=>
  string(1) "4"
}
*/

$msgMD5 = md5("我爱我家");
var_dump($msgMD5); //string(32) "b7e613a1cb13b29e4e63b9c596c1bb14"

```

### 3.2.2 `Array`数组函数

| 函数             | 描述                           |
| ---------------- | ------------------------------ |
| `count()`        | 数组中元素的数量               |
| `implode()`      | 把数组元素组合为字符串         |
| `array_merge()`  | 两个数组合并为一个数组         |
| `in_array()`     | 数组中是否存在指定的值         |
| `sort()`         | 对数值数组进行升序排序         |
| `rsort()`        | 对数值数组进行降序排序         |
| `array_unique()` | 移除数组中的重复的值           |
| `array_push()`   | 将一个或多个元素插入数组的末尾 |
| `array_pop()`    | 删除数组中的最后一个元素       |

```php
<?php

$arr = [
    'Joker',
    'Desire',
    'Tommy',
    'Ann'
];

// 数组中元素的数量
$count = count($arr);
var_dump($count); //int(4)

// 把数组元素组合为字符串
$implode = implode($arr);
var_dump($implode); //string(19) "JokerDesireTommyAnn"

$implode = implode(', ', $arr);
var_dump($implode); //string(25) "Joker, Desire, Tommy, Ann"

// 两个数组合并为一个数组
$arr2 = [
    'Ming',
    'Ning'
];
$arr3 = array_merge($arr, $arr2);
var_dump($arr3);

// 数组中是否存在指定的值
$res = in_array("Desire", $arr);
var_dump($res); //bool(true)

$res = in_array("Desire3", $arr);
var_dump($res); //bool(false)

$arr = [1, 3, 4, 2, 23, 56, 14, 8, 9, 74, 3];
// 对数值数组进行升序排序
sort($arr);
print_r($arr);
/*
Array
(
    [0] => 1
    [1] => 2
    [2] => 3
    [3] => 3
    [4] => 4
    [5] => 8
    [6] => 9
    [7] => 14
    [8] => 23
    [9] => 56
    [10] => 74
)
*/
rsort($arr);
print_r($arr);
/*
Array
(
    [0] => 74
    [1] => 56
    [2] => 23
    [3] => 14
    [4] => 9
    [5] => 8
    [6] => 4
    [7] => 3
    [8] => 3
    [9] => 2
    [10] => 1
)
*/
$arr = [1, 2, 2, 3, 2, 6, 5, 6, 45, 34, 1];
$arr1 = array_unique($arr);
print_r($arr1);
/*
Array
(
    [0] => 1
    [1] => 2
    [3] => 3
    [5] => 6
    [6] => 5
    [8] => 45
    [9] => 34
)
*/

$arr = ['Joker'];
array_push($arr, 'desire', 'tommy');
print_r($arr);
/*
Array
(
    [0] => Joker
    [1] => desire
    [2] => tommy
)
*/

// 删除数组中的最后一个元素
$arr = ['Joker', 'desire', 'tommy', 'Ann'];
array_pop($arr);
print_r($arr);
/*
Array
(
    [0] => Joker
    [1] => desire
    [2] => tommy
)
*/
```

## 3.3 自定义函数

### 3.3.1 函数的基本语法

- 必须使用关键字: `function`声明
- 函数名称不区分大小写

```php
function fun_name(参数列表)
{
    // 函数体，可以为空
}
```

### 3.3.2 调用函数

```php
function func_name()
{
    return 'a Function';
}

// 调用函数
$res = func_name();
var_dump($res); //"a Function"
```

### 3.3.3 函数参数

- 方法参数可以有默认值

```php
function func_name2($name)
{
    return "我是$name";
}

var_dump(func_name2("Joker")); // string(11) "我是Joker"

function func_name3($name, $age = 18)
{
    return "我是{$name}, 今年{$age}岁!";
}
var_dump(func_name3("Joker")); // string(25) "我是Joker, 今年18岁!"
var_dump(func_name3("Joker", 20)); // string(25) "我是Joker, 今年20岁!"
```

### 3.3.4 作用域

- php中，只有函数作用域和全局作用域
- 所有函数作用域的变量，外部不可见
- 全局作用域声明变量，在函数中是可见的

```php
$num = 1;

function func()
{
    global $num;
    $num = $num + 1;
    return $num;
}
var_dump(func());//int(2)
```

### 3.3.5 `PHP8`新特性：命名参数

```php
function func2($a, $b = 0, $c = 0, $d = 0)
{
    return $a . '--' . $b . '--' . $c . '--' . $d;
}

// php7
var_dump(func2(10, 20, 30, 40)); //string(14) "10--20--30--40"

// php8
var_dump(func2(10, 20, d: 30, c: 40));//string(14) "10--20--40--30"

```



# 四、类与对象

> 类是具有相同属性和操作的一组对象的集合

## 创建类

```php
// 创建类
class People
{
}
```

## 创建对象

```php
// 调用类（实例化类）
$p1 = new People();
$p2 = new People();
$p3 = new People();
```

## 复合数据类型

| 类型   | 描述 |
| ------ | ---- |
| object | 对象 |

```php
// 复合数据类型（object 对象）
var_dump($p1); //object(People)#1 (0) {}
var_dump($p2); //object(People)#2 (0) {}
var_dump($p3); //object(People)#3 (0) {}

var_dump($p1 == $p2); //bool(true)
var_dump($p1 == $p3); //bool(true)
var_dump($p2 == $p3); //bool(true)

var_dump($p1 === $p2); //bool(false)
var_dump($p1 === $p3); //bool(false)
var_dump($p2 === $p3); //bool(false)
```

## 成员变量

> 成员变量：类属性

```php
<?php
// 创建类
class People
{
    public $name = 'Joker';
    public $job = 'IT';
}

// 调用类（实例化类）
$p1 = new People();
// 获取类属性
var_dump($p1->name); //string(5) "Joker"
var_dump($p1->job); //string(2) "IT"

```

## 成员方法

```php
<?php
// 创建类
class People
{
    public $name = 'Joker';
    public $job = 'IT';

    public function fun1()
    {
        echo '我是People的没有返回值的fun1方法！';
    }

    public function fun2()
    {
        return '我是People的有返回值的fun2方法！';
    }
    public function fun3()
    {
        // 在类中使用伪变量:“$this”引用当前类的成员变量
        return 'fun3方法获取类属性，name=>' . $this->name . ' job=>' . $this->job;
    }
    public function fun4()
    {

        // 在类中使用伪变量:“$this”引用当前类的成员方法
        return $this->fun3();
    }
}

// 调用类（实例化类）
$p1 = new People();
$p1->fun1(); //我是People的没有返回值的fun1方法！
var_dump($p1->fun2()); //string(43) "我是People的有返回值的fun2方法！"
var_dump($p1->fun3()); //string(47) "fun3方法获取类属性，name=>Joker job=>IT"
var_dump($p1->fun4()); //string(47) "fun3方法获取类属性，name=>Joker job=>IT"
```

## 魔术方法

| 方法           | 描述                                           |
| -------------- | ---------------------------------------------- |
| `__construct`  | 构造方法                                       |
| `__destruct`   | 析构方法                                       |
| `__call`       | 在对象中调用一个不可访问方法时调用             |
| `__callStatic` | 在静态方式中调用一个不可访问方法时调用         |
| `__get`        | 获得一个类的成员变量时调用                     |
| `__set`        | 设置一个类的成员变量时调用                     |
| `__isset`      | 当对不可访问属性调用`isset()`或`empty()`时调用 |
| `__unset`      | 当对不可访问属性调用`unset()`时被调用          |
| `__sleep`      | 执行`serialize()`时，先会调用这个函数          |
| `__wakeup`     | 执行`unserialize()`时，先会调用这个函数        |
| `__toString`   | 类被当成字符串时的回应方法                     |
| `__invoke`     | 调用函数的方式调用一个对象时的回应方法         |
| `__set_state`  | 调用`var_export()`导出类时，此静态方法会被调用 |
| `__clone`      | 当对象复制完成时调用                           |
| `__autoload`   | 尝试加载未定义的类                             |
| `debugInfo`    | 打印所需调试信息                               |

```php
<?php
// 创建类
class People
{
    public $name = 'Joker';
    public $job = 'IT';

    /**
     * 构造方法
     */
    public function __construct($name, $job)
    {
        $this->name = $name;
        $this->job = $job;
    }

    /**
     * 析构方法
     * 当类调用执行完毕后被执行
     */
    public function __destruct()
    {
        echo '类执行完毕，要关闭了';
    }

    public function fun()
    {
        return "姓名：{$this->name}, 工作：{$this->job}";
    }
}

// 调用类（实例化类）
$obj = new People('Desire', 'teacher');
var_dump($obj->fun()); //string(33) "姓名：Desire, 工作：teacher"

```

## 类的三大特性

> - 继承：可以让某个类型的对象获得另一个类型的对象的属性和方法
> - 封装：指将客观事物抽象成类，每个类对自身的数据和方法实行保护
> - 多态：指同一个实体同时具有多种形式，它主要体现在类的继承体系中

### 继承

> `extends`关键词继承父类

```php
<?php
// 创建类
class People
{
    public $name = 'Joker';
    public $job = 'IT';

    /**
     * 构造方法
     */
    public function __construct($name, $job)
    {
        $this->name = $name;
        $this->job = $job;
    }

    /**
     * 析构方法
     * 当类调用执行完毕后被执行
     */
    public function __destruct()
    {
        echo '类执行完毕，要关闭了';
    }

    public function fun()
    {
        return "姓名：{$this->name}, 工作：{$this->job}";
    }
}

// 继承People类
class Teacher extends People
{
}

// 调用类（实例化类）
$obj = new Teacher('Desire', 'teacher');
var_dump($obj->fun()); //string(33) "姓名：Desire, 工作：teacher"

```

### 封装

> `public` 默认，关键词定义，类内、类外、子类都可见
>
> `protected` 关键词定义，类内、子类可见，类外不可见
>
> `pricate` 关键词定义，类内可见，子类、类外不可见

```php
<?php
// 创建类
class People
{
    public $name;
    protected $job; //类内、子类可见，类外不可见
    private $age; // 类内可见，子类、类外不可见

    /**
     * 构造方法
     */
    public function __construct($name, $job, $age)
    {
        $this->name = $name;
        $this->job = $job;
        $this->age = $age;
    }

    public function fun()
    {
        return "姓名：{$this->name}, 工作：{$this->job}, 年龄：{$this->age}";
    }

    /**
     * `protected` 关键词定义，类内、子类可见，类外不可见
     */
    protected function fun2()
    {
        return '类内、子类可见，类外不可见';
    }

    /**
     * `pricate` 关键词定义，类内可见，子类、类外不可见
     */
    private function fun3()
    {
        return '类内可见，子类、类外不可见';
    }
}

// 继承People类
class Teacher extends People
{
    /**
     * 在子类调用被protected修饰的方法，不可调用×
     */
    public function tecFunc()
    {
        return $this->func2();
    }

    /**
     * 在子类调用被private修饰的方法，不可调用×
     */
    public function tecFunc2()
    {
        return $this->fun3();
    }

    public function getName()
    {
        return $this->name;
    }

    /**
     * 在子类中调用被protected修饰的类属性，可调用√
     */
    public function getJob()
    {
        return $this->job;
    }

    /**
     * 在子类中调用被private修饰的类属性，不可调用×
     */
    public function getAge()
    {
        return $this->age;
    }
}



// 调用类（实例化类）
$obj = new People('Desire', 'teacher', 19);

var_dump($obj->fun()); //string(33) "姓名：Desire, 工作：teacher"

// 访问被private定义的类属性，不可访问
// Fatal error: Uncaught Error: Cannot access private property People::$age
// var_dump($obj->age); 

// 访问被protected定义的类属性，不可访问
// Fatal error: Uncaught Error: Cannot access protected property People::$job
// var_dump($obj->job);


// 访问被protected定义的类方法，不可访问
// Fatal error: Uncaught Error: Call to protected method People::fun2() from global scope in
// var_dump($obj->fun2());

// 访问被private定义的类方法，不可访问
//Fatal error: Uncaught Error: Call to private method People::fun3() from global scope in
// var_dump($obj->fun3());

// 调用子类
$p1 = new Teacher('Joker', 'Teacher', 19);

var_dump($p1->tecFunc());

// var_dump($p1->tecFunc2());

// var_dump($p1->getName());

var_dump($p1->getJob()); // string(7) "Teacher"

// var_dump($p1->getAge());

```

### 多态

> 实现多态的前提是要先继承再重写父类方法

```php
<?php
// 创建类
class People
{
    public $name;
    protected $job; //类内、子类可见，类外不可见
    private $age; // 类内可见，子类、类外不可见

    /**
     * 构造方法
     */
    public function __construct($name, $job, $age)
    {
        $this->name = $name;
        $this->job = $job;
        $this->age = $age;
    }

    public function fun()
    {
        return "姓名：{$this->name}, 工作：{$this->job}, 年龄：{$this->age}";
    }

    /**
     * `protected` 关键词定义，类内、子类可见，类外不可见
     */
    protected function fun2()
    {
        return '类内、子类可见，类外不可见';
    }

    /**
     * `pricate` 关键词定义，类内可见，子类、类外不可见
     */
    private function fun3()
    {
        return '类内可见，子类、类外不可见';
    }
}

class Student extends People
{
    /**
     * 通过重写父类方法实现多态
     */
    public function fun()
    {
        return '重写了父类的方法';
    }
}

$stu = new Student('Tommy', 'student', 16);
var_dump($stu->fun()); //string(24) "重写了父类的方法"

```

## 静态成员

> `static`  关键词定义静态成员

```php
<?php

class Teacher
{
    public static $name;
    public static $school;
    public static $gongfu = 'PHP';

    public function fun()
    {
        echo '普通成员方法' . PHP_EOL;
    }

    /**
     * 定义静态方法
     * 获取类静态属性，应该使用 :: 进行获取，静态方法只能获取静态属性
     */
    public static function fun2()
    {
        return '姓名：' . Teacher::$name . ', 学校：' . Teacher::$school . PHP_EOL;
    }
}

// 获取静态属性
echo Teacher::$gongfu . PHP_EOL; //PHP
// 获取静态方法
echo Teacher::fun2(); //姓名：, 学校：

// 可以修改静态属性值
Teacher::$name = 'Joker';
Teacher::$school = '家里蹲大学';

echo Teacher::fun2(); //姓名：Joker, 学校：家里蹲大学

```

## 抽象类

> `abstract` 关键词定义抽象方法/抽象类

```php
<?php

abstract class People
{

    public function fun()
    {
        return "普通方法" . PHP_EOL;
    }
    abstract public function fun2();
}

// 抽象类不可被实例化，只能被继承
// new People(); 

class Student extends People
{

    public function fun2()
    {
        return "继承抽象类，并实现抽象方法" . PHP_EOL;
    }
}

$stu = new Student();
// 调用抽象类的普通方法
echo $stu->fun(); //普通方法

// 调用子类实现的抽象类的抽象方法
echo $stu->fun2(); //继承抽象类，并实现抽象方法

```

## 接口

> `interface` 关键词创建接口。要求类必须实现的方法，但不需要定义方法的具体实现过程。
>
> `implements` 关键词使用接口。

```php
<?php

interface file
{
    public function fly();
    public function run();
}

class Cat implements file
{

    /**
     * 实现接口方法
     */
    function fly()
    {
        echo '飞起来了!' . PHP_EOL;
    }

    /**
     * 实现接口方法
     */
    function run()
    {
        echo '跑起来了！' . PHP_EOL;
    }
}
$cat = new Cat();
$cat->fly(); //飞起来了!
$cat->run(); //跑起来了！

```

## 接口常量

> `const`创建常量

```php
interface config
{
    // 使用const定义常量
    const HOST = 'localhost';
    const PORT = 3306;
    const USER = 'root';
    const PWD = 'root';
}

class DBConn implements config
{
    private $host = config::HOST;
    private $port = config::PORT;
    private $user = config::USER;
    private $pwd = config::PWD;

    function database()
    {
        return "mysql -P {$this->port} -h {$this->host} -u {$this->user} -p {$this->pwd} " . PHP_EOL;
    }
}
$db = new DBConn();

echo $db->database(); //mysql -P 3306 -h localhost -u root -p root

```

## 常量

```php
<?php
// 创建常量
// 方式一 define()
define('HOST', '127.0.0.1');
echo HOST; // 127.0.0.1

// 方式二 const
const NAME = 'PHP';
echo NAME; // PHP
```

## 关键词

| 关键词       | 类外声明 | 声明类 | 声明属性 | 声明方法 | 解释                           |
| ------------ | -------- | ------ | -------- | -------- | ------------------------------ |
| `const`      | √        |        | √        |          | 定义类常量                     |
| `extends`    |          | √      |          |          | 扩展类，用一个类去扩展它的父类 |
| `public`     |          |        | √        | √        | 公用属性或方法                 |
| `protected`  |          |        | √        | √        | 私有属性或方法                 |
| `pricate`    |          |        | √        | √        | 受保护的属性或方法             |
| `static`     |          | √      | √        | √        | 静态成员                       |
| `abstract`   |          | √      | √        |          | 抽象类或方法                   |
| `interface`  |          | √      |          |          | 创建接口                       |
| `implements` |          | √      |          |          | 实现接口                       |
| `final`      |          | √      |          | √        | 类不能被继承                   |
| `parent::`   |          |        |          |          | 访问父类                       |
| `$this->`    |          |        |          |          | 访问本类                       |
| `self::`     |          |        |          |          | 访问本类静态                   |
| `namespace`  | √        |        |          |          | 创建命名空间                   |

### `final`类不能被继承

```php
<?php

final class People
{
	public string $name;
	public int $age;

	// 构造方法
	public function __construct($name, $age)
	{
		$this->name = $name;
		$this->age = $age;
	}

	/**
	 * @return string
	 */
	public function func(): string
	{
		return "姓名：{$this->name}, 年龄：{$this->age}";
	}
}

// 子类继承被final定义的父类
class Student extends People
{

}

$student = new Student('Joker', 19);
echo $student->func();
// 执行报错：Fatal error: Class Student cannot extend final class

```

### `parent::` 访问父类

```php
<?php

class People
{
	public string $name;
	public int $age;

	// 构造方法
	public function __construct($name, $age)
	{
		$this->name = $name;
		$this->age = $age;
	}

	/**
	 * @return string
	 */
	public function func(): string
	{
		return "姓名：{$this->name}, 年龄：{$this->age}";
	}
}

class Student extends People
{
	/**
	 * 重写父类的方法，并且调用父类的此方法
	 * @return string
	 */
	public function func(): string
	{
		return parent::func();
	}

}

$student = new Student('Joker', 19);
echo $student->func(); // 姓名：Joker, 年龄：19

```

## 命名空间

> - 命名空间：解决全局成员的命名冲突问题，借鉴了文件目录的思想
> - 空间：同一空间内不允许成员重名，但不同空间内，允许同名成员存在

### 创建命名空间

> 命名空间使用 `namespace` 关键字声明

```php
// ---- 方式一

<?php
namespace one {
	function php()
	{

	}
}

// ---- 方式二
<?php
namespace space;
class demo
{

}
```

### 子命名空间

```php
<?php

namespace one {
	const NAME = 'one 命名空间';
	function php(): string
	{
		return '命名空间one定义的函数';
	}
}

namespace one\two {
	function php(): string
	{
		return 'one子命名空间two定义的函数';
	}
}
```

### 访问命名空间

> `use` 关键词引用命名空间
>
> `AS` 解决类名过长、重名问题

```php
<?php

namespace one {
	const NAME = 'one 命名空间';
}

namespace one\two {
	const NAME = 'one\two 命名空间';
}

namespace two\one {
	const NAME = 'two\one 命名空间';
}

namespace three\one {
	const NAME = 'three\one 命名空间';

}

namespace main {

	use one;
	use one\two;
	use two\one as to;
	use three\one as tho;

	echo one\NAME . PHP_EOL; // one 命名空间
	echo two\NAME . PHP_EOL; // one\two 命名空间
	echo to\NAME . PHP_EOL; // two\one 命名空间
	echo tho\NAME . PHP_EOL; // three\one 命名空间
}

```


