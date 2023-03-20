# Redis-incr计数


## `Incr`

> Incr 命令将 key 中储存的数字值增一

1. 如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCR 操作。

2. 如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
3. 返回值为执行 INCR 命令之后 key 的值。

## 命令行使用

数值递增

> INCR KEY_NAME

获取数值

> GET KEY_NAME

## Python使用

```python
from contextlib import contextmanager

from redis import Redis

@contextmanager
def redis_connect() -> Redis:
    conn = Redis(host=settings.REDIS_HOST, port=settings.REDIS_PORT, db=0)
    yield conn
    conn.close()
def redis_incr_count(key):
    with redis_connect() as redis:
        res = redis.incr("incr:count")
    return res

if __name__ == '__main__':

    from concurrent.futures import ThreadPoolExecutor

    # 多线程测试
    with ThreadPoolExecutor(max_workers=200) as executor:
        for _ in range(200):  # 模拟多个任务
            future = executor.submit(redis_incr_count, "incr:count")
            print(future.result())
```

**执行结果：**

![image-20230320154226247](https://raw.githubusercontent.com/XD825/picgo/main/img/202303201542323.png)

