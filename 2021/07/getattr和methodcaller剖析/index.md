# Getattr和methodcaller剖析

# 一、getattr解析

## 1.1 getattr是什么？

getattr() 函数获取某个类实例对象中指定属性的值

## 1.2 语法格式

```python
getattr(obj, name[, default])
```

- obj 表示指定的类实例对象
- name 表示指定的属性名
- default 是可选参数，用于设定该函数的默认返回值
  - 即当函数查找失败时，如果不指定 default 参数，则程序将直接报 AttributeError 错误
  - 反之该函数将返回 default 指定的值。

## 1.3 使用

### 1.3.1 构造一个类方法

```python
class KeyWord(object):
    driver = "chrome驱动"
    def openBrowser(self, browser):
        print(f"openBrowser is running {browser}")
```

### 1.3.2 调用存在的类属性

```python
kw = Keywork()
driver = getattr(kw, "driver")
print(driver)
# ----------------------
chrome驱动
```

### 1.3.3 调用不存在的类属性(会报错)

```python
name = getattr(kw, "name")
print(name)
# -------------------
AttributeError: 'Keywork' object has no attribute 'name'
```

### 1.3.4 调用不存在的类属性(设置默认值)

```python
name = getattr(kw, "name","Desire")
print(name)
# -------------------
Desire
```

### 1.3.5 调用类方法

```python
openBrowser = getattr(kw, "openBrowser") # 获取类方法
openBrowser(browser="Chrome") # 调用类方法
```

# 二、methodcaller解刨

## 2.1 methodcaller 是什么？

Python内部提供的一个可以通过字符串调用类方法的一个包，在operator库中。

## 2.2 使用

### 2.2.1 导包

```python
from operator import methodcaller
```

### 2.2.2 使用字符串"openBrowser"执行对应的openBrowser类方法

```python
method = methodcaller("openBrowser", browser="Chrome")
keyword = KeyWord()
method(keyword)
```

### 2.2.3 执行结果

```cmd
# --------------------------------
openBrowser is running Chrome
```

## 2.3 解刨

### 2.3.1 源码

```python
def methodcaller(name, /, *args, **kwargs):
    def caller(obj):
        return getattr(obj, name)(*args, **kwargs)
    return caller
```

### 2.3.2 解读

- 采用了闭包的形式
- 调用methodcaller函数时，返回的是methodcaller内部的函数

  - 也就是说调用后返回的还是一个函数
  - 调用methodcaller函数的返回值（method），其实就是caller(obj)
- method == caller，所以我们可以通过method(obj)执行methodcaller内部方法caller中的函数体了
- caller函数操作：通过getattr() 函数从指定的对象返回指定属性的值，从而实现了通过字符串调用类方法

# 三、比对总结

从上面的methodcaller源码和解读中可以看到，methodcaller底层还是使用的getattr，对于字符串调用类方法，这两种方式都是可行的，自行选择使用。

