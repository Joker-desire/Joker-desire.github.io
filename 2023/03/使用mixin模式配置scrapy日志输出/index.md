# 使用Mixin模式配置Scrapy日志输出


## 背景

### 为什么要重新配置Scrapy日志输出

在使用kafka进行处理爬虫数据后出现了一种现象：日志打印会将整个item数据打印出，由于使用kafka处理数据，都是将整个页面数据进行存入item的，每天的日志都将占很大的内存空间。之前的做法是提高整个爬虫的日志等级为`LOG_LEVEL = "ERROR"` ，这样处理会带来一个问题，如果想要写日志进行排查问题时，就只能往项目中添加`logging.error()` 级别的日志了，这样会造成日志输出混乱，不易排查问题。所以就需要放弃这种形式，进行重新配置日志输出，将一些不需要的日志输出进行排除掉。



## Mixin模式

Mixin模式是一种面向对象编程的设计模式，它允许将一个类的方法和属性混合到另一个类中，从而实现代码的复用和灵活性。

在Mixin模式中，一个Mixin类通常只包含一组相关的方法和属性，而不是完整的类定义。这些Mixin类可以被多个类引用，从而实现代码的复用。Mixin类通常不会被单独实例化，而是被其他类继承或混合使用。

在一些编程语言中，如Python和Ruby，Mixin模式可以通过多重继承来实现。在这种情况下，一个类可以继承多个Mixin类，并从中继承方法和属性。在其他编程语言中，如Java和C#，Mixin模式可以通过接口和委托来实现。

Python中使用Mixin模式的示例代码:

```python
class SayHiMixin:
    
    def say(self):
        print(f"Hi, {self.name}")

class User(SayHiMixin):
    
    def __init__(self, name):
        self.name = name

user = User("Tommy")
user.say() # Hi, Tommy
```



## 使用LogConfigMixin实现Scrapy日志配置

```python
import logging

class LogConfigMixin:

    def __init__(self, *args, **kwargs):
        super(LogConfigMixin, self).__init__(*args, **kwargs)
        # 去掉不需要的log handler（元组格式）
        remove_handlers = (logging.StreamHandler,)
        for handler in logging.root.handlers:
            if isinstance(handler, remove_handlers):
                logging.root.handlers.remove(handler)

        # 添加自定义 handler
        handler = logging.StreamHandler()
        handler.setLevel(logging.DEBUG)
        logger_fmt = '{asctime}.{msecs:03.0f} {levelname} 【{filename}:{funcName}:{lineno}】: {message}'
        _format = logging.Formatter(fmt=logger_fmt, style='{', datefmt='%Y-%m-%d %H:%M:%S')
        handler.setFormatter(fmt=_format)
        logging.root.addHandler(handler)

        # 设置 scrapy 某些组件的日志级别
        # 此日志中包含了item的输出结果，由于是整个页面数据，故需要调整日志级别，防止日志过大
        logging.getLogger('scrapy.core.scraper').setLevel(logging.WARNING)
        logging.getLogger('scrapy.core.engine').setLevel(logging.DEBUG)
        logging.getLogger('scrapy.middleware').setLevel(logging.DEBUG)
        logging.getLogger('scrapy.utils.log').setLevel(logging.DEBUG)
        # kafka相关日志设置级别
        logging.getLogger("kafka.cluster").setLevel(logging.WARNING)
        logging.getLogger("kafka.producer.sender").setLevel(logging.WARNING)
        logging.getLogger("kafka.producer.record_accumulator").setLevel(logging.WARNING)
        logging.getLogger("kafka.protocol.parser").setLevel(logging.WARNING)
        logging.getLogger("kafka.metrics.metrics").setLevel(logging.WARNING)
        logging.getLogger("kafka.client").setLevel(logging.WARNING)
        logging.getLogger("kafka.conn").setLevel(logging.WARNING)
        logging.getLogger("kafka.send").setLevel(logging.WARNING)
        logging.getLogger("kafka.close").setLevel(logging.WARNING)
```

## 在爬虫中进行使用

### 自定义`Spider`和`RedisSpider`类

使用自定义的`Spider`和`RedisSpider`为官方的`Spider`和`RedisSpider`基础爬虫类增加日志配置功能

```python
from abc import ABC

from scrapy import Spider as _Spider
from scrapy_redis.spiders import RedisSpider as _RedisSpider

class Spider(LogConfigMixin, _Spider, ABC):
    pass


class RedisSpider(LogConfigMixin, _RedisSpider, ABC):
    pass
```

这样做的好处：就可以在不改变原有的爬虫程序，只用修改导包路径即可，修改成本极低✌️

### 例子

#### 原爬虫程序

```python
from scrapy import Spider
from estate.items import HuiLvItem

class DemoSpider(Spider):  
    name = "demo"

    custom_settings = {
        'ITEM_PIPELINES': {
            'estate.pipelines.DemoPipeline': 300
        },
    }

    start_urls = [
        'https://www.xxxx.cc/HKD_CNY/',
    ]

    def parse(self, response, **kwargs):
        rate = response.xpath('//div[@class="dollar_two"]/span/text()').extract_first()
        if rate:
            rate = rate
        else:
            rate = 0.85551
        item = HuiLvItem()
        item['rate'] = rate
        yield item
```

#### 增加日志配置功能后爬虫程序

```python
from estate.mixin import Spider
from estate.items import HuiLvItem

class DemoSpider(Spider):  
    name = "demo"

    custom_settings = {
        'ITEM_PIPELINES': {
            'estate.pipelines.DemoPipeline': 300
        },
    }

    start_urls = [
        'https://www.xxxx.cc/HKD_CNY/',
    ]

    def parse(self, response, **kwargs):
        rate = response.xpath('//div[@class="dollar_two"]/span/text()').extract_first()
        if rate:
            rate = rate
        else:
            rate = 0.85551
        item = HuiLvItem()
        item['rate'] = rate
        yield item
```

#### 修改成本极低

只需将官方导包`from scrapy import Spider` 更改为自定义导包路径 `from estate.mixin import Spider`


