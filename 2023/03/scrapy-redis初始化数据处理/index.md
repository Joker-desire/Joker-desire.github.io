# Scrapy-Redis初始化数据处理


## 一、思考

1. 在使用Scrapy-Redis进行数据初始化时，你是如何进行数据写入Redis队列的？
2. 数据写入的格式如果是JSON，你是如何进行处理的？



## 二、数据初始化

### 2.1 之前的写法

####  2.1.1 在Scrapy文件头部进行Redis初始化写入

```python
import redis

from scrapy.utils.project import get_project_settings

# 从配置文件中获取Redis配置
redis_host = get_project_settings().get('REDIS_HOST')
redis_port = get_project_settings().get('REDIS_PORT')

conn = redis.Redis(host=redis_host, port=redis_port, db=0)
conn.sadd('scrapy:start_urls', 'https://www.xxxx.com/page01', 'https://www.xxxx.com/page02', 'https://www.xxxx.com/page03')

```

#### 2.1.2 存在的问题

1. 会在爬虫部署时就开始执行Redis数据初始化

2. 如果此处的处理耗时比较长，则会导致部署超时失败

    ![](https://raw.githubusercontent.com/XD825/picgo/main/img/202303201133036.png)

注：部署使用的是`scrapydweb` 

### 2.2 现在的写法

#### 2.2.1 使用类的初始化方法进行Redis数据初始化

```python
class ScrapySpider(RedisSpider):

    def __init__(self, name=None, **kwargs):
        redis_host = get_project_settings().get('REDIS_HOST')
        redis_port = get_project_settings().get('REDIS_PORT')

        conn = redis.Redis(host=redis_host, port=redis_port, db=0)
        conn.sadd('scrapy:start_urls', 'https://www.xxxx.com/page01', 'https://www.xxxx.com/page02', 'https://www.xxxx.com/page03')
        super(ScrapySpider, self).__init__(name, **kwargs)
```

#### 2.2.2 执行出现的问题

在部署后进行执行，出现了报错信息，`__init __()` 得到了一个意外的关键字参数`_ job`

![image-20230320113754785](https://raw.githubusercontent.com/XD825/picgo/main/img/202303201137861.png)

#### 2.2.3 解决意外关键字参数`_job`

直接在初始化时删除即可，为了保证可靠性，先判断是否存在，然后再进行删除

```python
class ScrapySpider(RedisSpider):

    def __init__(self, name=None, **kwargs):
        redis_host = get_project_settings().get('REDIS_HOST')
        redis_port = get_project_settings().get('REDIS_PORT')

        conn = redis.Redis(host=redis_host, port=redis_port, db=0)
        conn.sadd('scrapy:start_urls', 'https://www.xxxx.com/page01', 'https://www.xxxx.com/page02', 'https://www.xxxx.com/page03')
        
        # 判断是否存在 _job关键字参数
        if kwargs.get('_job'):
            # 存在则删除
            kwargs.pop('_job')
        super(ScrapySpider, self).__init__(name, **kwargs)
```



## 三、数据JSON格式处理

在上面的初始化Redis数据时，传入的数据都是固定的一个链接，但是在实际的使用中，链接并不能满足需求，需要存储JSON格式的数据，那么再处理爬虫的时候，就要在Redis pop出数据后进行处理JSON字符串获得指定的链接进行处理。

### 3.1 查询源码，找到对应的处理逻辑

#### Scrapy-Redis >> `make_request_from_data` : 

```python
    def make_request_from_data(self, data):
        """Returns a Request instance from data coming from Redis.

        By default, ``data`` is an encoded URL. You can override this method to
        provide your own message decoding.

        Parameters
        ----------
        data : bytes
            Message from redis. 直接读取Redis的数据

        """
        # 将Redis pop出的数据转成字符串
        url = bytes_to_str(data, self.redis_encoding)
        # 调用Scrapy的方法进行发送请求
        return self.make_requests_from_url(url)
```

#### Scrapy >> `make_requests_from_url`

```python
    def make_requests_from_url(self, url):
        """ This method is deprecated. """
        warnings.warn(
            "Spider.make_requests_from_url method is deprecated: "
            "it will be removed and not be called by the default "
            "Spider.start_requests method in future Scrapy releases. "
            "Please override Spider.start_requests method instead."
        )
        # 发送请求，并返回scrapy.http.Request对象
        return Request(url, dont_filter=True)
```

### 3.2 进行重写

从上面的源码中，我们可以直接重写`make_request_from_data` 方法，在方法内部进行处理JSON数据，并返回`scrapy.http.Request`对象即可。

```python
import json

import redis
from scrapy import Request
from scrapy.utils.project import get_project_settings
from scrapy_redis.spiders import RedisSpider

class ScrapySpider(RedisSpider):

    def __init__(self, name=None, **kwargs):
        redis_host = get_project_settings().get('REDIS_HOST')
        redis_port = get_project_settings().get('REDIS_PORT')

        conn = redis.Redis(host=redis_host, port=redis_port, db=0)
        conn.sadd('scrapy:start_urls', 
                  '{"id": 9531, "source": 1, "mobile": "62196161", "url": "https://www.xxxx.com/detail001"}', 
                  '{"id": 8160, "source": 0, "mobile": "60376386", "url": "https://www.xxxx.com/detail002"',
                  '{"id": 7334, "source": 0, "mobile": "62631381", "url": "https://www.28hse.com/detail003"')
        
        # 判断是否存在 _job关键字参数
        if kwargs.get('_job'):
            # 存在则删除
            kwargs.pop('_job')
        super(ScrapySpider, self).__init__(name, **kwargs)
        
    def make_request_from_data(self, data):
        """
        :param data: 从reids pop到的数据
        :return:
        """
        # 将获取的数据进行转换
        req_data = json.loads(data)
        # 获取所需参数信息
        url = req_data['url']
        source = req_data['source']
        mobile = req_data.get('mobile', '')
        id = req_data['id']
        
		# 发送请求，并返回
        return Request(
            url,
            meta={"id": id, "source": source, "mobile": mobile} # 将参数作为元数据传递
        )
```


