# Python3.10新特性初体验

## 一、结构模式匹配（新增[PEP 635](https://www.python.org/dev/peps/pep-0635/)）

如果你熟悉别的语言，你肯定知道 `switch->case`语句，python3.10之前的版本是一直没有这种语法的，但是这一次的更新中3.10+版本（20211004）开始支持这种结构模式匹配的语法书写了--`match->case`

### 1、基本语法

```python
match expression: # 条件，可为任意表达式、变量
    case pattern_1: # 模式，待匹配的条件1
        ... # 需要执行的代码块
    case pattern_2: # 模式，待匹配的条件2
        ... # 需要执行的代码块
    case _: # 模式，表示default
        ... # 需要执行的代码块
```

### 2、模式

#### 1. as模式(AS)

```python
def simplify_expr(tokens):
    match tokens:
        case [('(' | '[') as l, *expr, (')' | ']') as r] if (l + r) in ('()', '[]'):
            # 第一个参数得为('(' | '['),取别名为l
            # 最后一个参数得为('(' | '['),取别名为r
            # 并且需判断(l + r) in ('()', '[]')为True才会进入此代码块执行
            print(expr)
        case [0, ('+' | '-') as op, right]:
            # 第一个参数必须为0
            # 第二个参数得为('+' | '-'),取别名为op
            # 第三个参数为任意数
            # 序列的长度必须为3个
            print(op, right)
        case [(int() | float()) as value]:
            # 序列的长度只能有1个
            # 并且，类型只能是int或者float
            print(value)
        case _:
            # 匹配不到则走这里
            print("未匹配到")


simplify_expr(['(', "aaa", "bbb", ")"])  # ['aaa', 'bbb']
simplify_expr(['[', "aaa", "bbb", "]"])  # ['aaa', 'bbb']
simplify_expr(['[', "aaa", "bbb", ")"])  # 未匹配到
simplify_expr(['[', "]"])  # []
simplify_expr(['bbb', '[', "]"])  # 未匹配到
simplify_expr([0, "+", 3])  # + 3
simplify_expr([0, "-", 3])  # - 3
simplify_expr([0, "-", 3, 34])  # 未匹配到
simplify_expr([1])  # 1
simplify_expr([1.0])  # 1.0
simplify_expr([1.0, 1])  # 未匹配到
simplify_expr(["aaaa"])  # 未匹配到
```

#### 2. or模式(|)

```python
def demo(code):
    match code:
        case 200:
            print("200成功了")
        case 400 | 404:
            print("出问题了")
        case (1002 | 1006, 200): # 把两个值组织起来（组织模式）
            print("接口请求成功")
        case _:
            print("匹配不到")


demo(200) # 200成功了
demo(400) # 出问题了
demo(404) # 出问题了
demo((1006, 200)) # 接口请求成功
demo(502) # 匹配不到
```

#### 3. 文字模式(Literal)

```python
def simplify(expr):
    match expr:
        case ('+', 0, x):
            return x
        case ('+' | '-', x, 0):
            return x
        case ('and', True, x):
            return x
        case ('and', False, x):
            return False
        case ('or', False, x):
            return x
        case ('or', True, x):
            return True
        case ('not', ('not', x)):
            return x
    return expr


print(simplify(('+', 0, 12)))  # 12
print(simplify(('+', 13, 0)))  # 13
print(simplify(('-', 13, 0)))  # 13
print(simplify(('and', True, 0)))  # 0
print(simplify(('and', False, 0)))  # False
print(simplify(('or', False, 1)))  # 1
print(simplify(('or', True, 1)))  # True
print(simplify(('not', ('not', 23))))  # 23
```

#### 4. 捕获模式(Capture)

```python
def average(*args):
    match args:
        case [x, y]:
            return (x + y) / 2
        case [x]:
            return x
        case []:
            return 0
        case [x, y, z]:
            return x + y + z
        case a:
            return sum(a) / len(a)


print(average(1, 2, 3, 4)) # 2.5 ==> a ==> sum(a) / len(a)==> (1+2+3+4)/4 = 2.5
print(average(1, 2))  # 1.5 ==> [x, y] ==> (x+y)/2 ==> (1+2)/2=1.5
print(average(1))  # 1 ==> [x] ==> x=1
print(average())  # 0 ==> [] ==> 0
# 如果传递的是三个元素，则会按照长度进行匹配，跟传字典类型不同
print(average(1, 2, 3))  # 6 ==> [x, y, z] ==> 1+2+3=6
```

#### 5. 通配符模式(Wildcard)

- _：在前面就表示任意字符
- case __：这里的__在末尾，表示default默认

```python
def is_closed(sequence):
    match sequence:
        case [_]:  # 只有一个元素的序列
            return True
        case [start, *_, end]:  # 至少含有两个元素的序列
            return start == end
        case _:  # 匹配任意类型
            return False


print(is_closed(1))  # False ==> _ ==> False
print(is_closed([1]))  # True ==> [_] ==> True
print(is_closed([1, 2, 3]))  # False ==> [start, *_, end] ==>>1 ==3 ==>False
```

#### 6. 值模式(value)

```python
class HttpStatus(object):
    OK = 200
    MOVED_PERMANENTLY = 301
    NOT_FOUND = 404


class MimeType(object):
    TEXT = "text"
    APPL_ZIP = "appl_zip"


def handle_reply(reply):
    match reply:
        case (HttpStatus.OK, MimeType.TEXT, body):
            print(f"请求成功，类型是TEXT,body==>{body}")
        case (HttpStatus.OK, MimeType.APPL_ZIP, body):
            print(f"请求成功，类型是APPL_ZIP,body==>{body}")
        case (HttpStatus.MOVED_PERMANENTLY, new_URI):
            print(f"301了，赶紧检查下URI==>{new_URI}")
        case (HttpStatus.NOT_FOUND):
            print("网站挂了。。。")


handle_reply([200, "text", "dddddd"])  # 请求成功，类型是TEXT,body==>dddddd
handle_reply([200, "appl_zip", "dddddd"])  # 请求成功，类型是APPL_ZIP,body==>dddddd
handle_reply([200, "appl_zip", "dddddd"])  # 请求成功，类型是APPL_ZIP,body==>dddddd
handle_reply([301, "sip:smith@zte.com.cn"]) # 301了，赶紧检查下URI==>sip:smith@zte.com.cn
handle_reply(404)  # 网站挂了。。。
```

#### 7. 组织模式(Group)

使用or模式实现

```python
def demo(code):
    match code:
        case 200:
            print("200成功了")
        case 400 | 404:
            print("出问题了")
        case (1002 | 1006, 200): # 把两个值组织起来（组织模式）
            print("接口请求成功")
        case _:
            print("匹配不到")

demo((1006, 200)) # 接口请求成功
```

#### 8. 序列模式(Sequence)

```python
def is_closed(sequence):
    match sequence:
        case [_]:  # 只有一个元素的序列
            return True
        case [start, *_, end]:  # 至少含有两个元素的序列
            return start == end
        case _:  # 匹配任意类型
            return False


print(is_closed(1))  # False ==> _ ==> False
print(is_closed([1]))  # True ==> [_] ==> True
print(is_closed([1, 2, 3]))  # False ==> [start, *_, end] ==>>1 ==3 ==>False
```

#### 9. 映射模式（Mapping）

```python
def change_red_to_blue(json_obj):
    match json_obj:
        case {'color': ('red' | '#FF0000')}:
            json_obj['color'] = 'blue'
        case {'children': children}:
            for child in children:
                change_red_to_blue(child)


color = {"color": "red"}
print(color)  # {'color': 'red'}
change_red_to_blue(color)
print(color)  # {'color': 'blue'}

# 注意：如果传参是字典，依次从下进行匹配，只要匹配到则不会再往下匹配了
# 如下，color包含color和children两个键，匹配时，匹配到了color就结束了，不会再进行下一步匹配children了
color = {"color": "red", "children": [{"color": "#FF0000"}]}
print(color)  # {'color': 'red', 'children': [{'color': '#FF0000'}]}
change_red_to_blue(color)
print(color)  # {'color': 'blue', 'children': [{'color': '#FF0000'}]} children中的color则不会被修改
```

#### 10. 类模式(Class)

```python
class MyClassA(object):
    def __init__(self, name) -> None:
        self.name = name


def demo(class_):

    match class_:
        case MyClassA(name="desire"):
            print("是desire啊")
        case _:
            print(class_.name)


demo(MyClassA("ronin")) # ronin ==> _ ==> MyClassA("ronin").name ==> class_.name=ronin
demo(MyClassA("desire")) # 是desire啊 ==> MyClassA(name="desire") ==> print("是desire啊")
```

### 3、匹配要注意的顺序

- 当 match 的对象是一个 list 或者 tuple 的时候，需要长度和元素值都能匹配，才能命中，在 `捕获模式`中体现的有。
- 当 match 的对象是一个 dict 的时候，规则却有所不同，只要 case 表达式中的 key 在所 match 的对象中有存在，即可命中，在 `映射模式`中体现的有。
- 而当 match 的对象是类对象时，匹配的规则是，跟 dict 有点类似，只要对象类型和对象的属性有满足 case 的条件，就能命中，在 `类模式`中体现的有。

## 二、union类型允许X | Y （[PEP 604](https://www.python.org/dev/peps/pep-0604/)）

新的语法union接受函数,变量和参数注释。

### 1、简单的语法

```python
# before
from typing import List, Union, Optional


def f(list: List[Union[int, str]], param: Optional[int]) -> Union[float, str]:
    pass

# now 更加简洁了
from typing import List


def f(list: List[int | str], param: int | None) -> float | str:
    pass
```

### 2、`typing.Union` 和 `|` 是等价的

```python
int | str == typing.Union[int, str] # True
```

### 3、顺序是没有要求的

```python
(int | str) == (str | int) # True
(int | str | float) == typing.Union[str, float, int] # True
```

### 4、isinstance和issubclass支持

```python
isinstance(5, int | str)  # True
isinstance("aaa", int | str)  # True
isinstance(1.22, int | str)  # False
issubclass(bool, int | float)  # True
isinstance(None, int | None)  # True
isinstance(42, None | int)  # True
```

## 三、带圆括号的上下文管理器

已支持使用外层圆括号来使多个上下文管理器可以连续多行地书写。 这允许将过长的上下文管理器集能够以与之前 import 语句类似的方式格式化为多行的形式。
一下四种格式都是支持的

```python
# 之前的格式不影响使用
with open("readme.md", 'r') as f:
    pass

with (open("readme.md", 'r') as f):
    pass

with (open("readme.md", 'r') as f1,
      open("readme.md", 'r') as f2):
    pass

with (open("readme.md", 'r'),
      open("readme.md", 'r') as f2):
    pass
with (open("readme.md", 'r'),
      open("readme.md", 'r'),
      ):
    pass
```

