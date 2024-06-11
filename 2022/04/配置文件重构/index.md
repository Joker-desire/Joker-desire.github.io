# 配置文件重构

# 一、为什么要重构

之前的方式，比较low

![](https://raw.githubusercontent.com/XD825/picgo/main/img/20220615171326.png)

# 二、Dynaconf是什么

[![](https://raw.githubusercontent.com/XD825/picgo/main/img/20220615171450.png)](https://www.dynaconf.com/)

- Dynaconf 是一个旨在成为 Python 中管理配置的最佳选择的库。
- 它可以从各种来源读取设置，包括环境变量、文件、配置服务器、保险库等。
- 它适用于任何类型的 Python 程序，包括 Flask 和 Django 扩展。
- 它是高度可定制且经过大量测试的。

# 三、怎么使用Dynaconf

## 3.1、安装Dynaconf

```bash
pip install dynaconf
```

## 3.2、Dynaconf初始化文档说明

![](https://raw.githubusercontent.com/XD825/picgo/main/img/image%20(4).png)

## 3.3、初始化一个配置

```bash
dynaconf init -f toml
```

**自动生成文件↓**
![](https://raw.githubusercontent.com/XD825/picgo/main/img/image%20(5).png)

## 3.4、文件说明

- `.gitignore`: git 要忽略的文件模式

```
# Ignore dynaconf secret files
.secrets.*
```

- `.secrets.toml`: 密码和令牌等敏感数据(可选)

```toml
TOKEN = xxxx
```

- `config.py`: 导入设置对象的地方(必需的)

```python

from dynaconf import Dynaconf

settings = Dynaconf(
    envvar_prefix="DYNACONF",
    settings_files=['settings.toml', '.secrets.toml'], # 在项目中此列表元素要填写文件基于项目的相对路径
    environments=True,
    ENV_FOR_DYNACONF="develop"  # 修改默认使用的环境，默认为：[development]
)

# `envvar_prefix` = export envvars with `export DYNACONF_FOO=bar`.
# `settings_files` = Load these files in the order.
```

- `settings.toml`: 应用setttings(可选)
  - 分为三部分：`[default]` `[develop]` `[master]`
  - 通过设置环境变量 `【ENV_FOR_DYNACONF】`进行控制 `[develop]` `[master]`
  - 如果没有设置环境变量，则根据 `config.py`文件中设置的默认使用环境进行使用

```toml
[default]
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 44640  # 一个月过期
SQLALCHEMY_DATABASE_URL_1 = "@format mysql+pymysql://{this.DB_USER}:{this.DB_PASSWORD}@{this.DB_HOST}:3306/xxx?charset=utf8"
SQLALCHEMY_DATABASE_URL_2 = "@format mysql+pymysql://{this.DB_USER}:{this.DB_PASSWORD}@{this.DB_HOST}:3306/xxx_ddd?charset=utf8"
SQLALCHEMY_DATABASE_URL_3 = "@format mysql+pymysql://{this.DB_USER}:{this.DB_PASSWORD}@{this.DB_HOST}:3306/xxx_lll?charset=utf8"


[develop]
BASE_URL = "https://fastapi.debug.com.hk/"
SECRET_KEY = "xxxxxx"
DB_HOST = "192.00.00.00"
DB_USER = "xxxx"
DB_PASSWORD = "xxxxxxxx"
REDIS_HOST = '192.22.22.22'
REDIS_PORT = 6380


[master]
BASE_URL = "https://fastapi.com.hk/"
SECRET_KEY = "zzzzzzzzz"
DB_HOST = "192.00.1.12"
DB_USER = "eeee"
DB_PASSWORD = "ddxrrr"
REDIS_HOST = '192.00.00.100'
REDIS_PORT = 6381
```

## 3.5、简单使用

### 3.5.1. 导包

```python
from config.config import settings
```

### 3.5.2. 获取配置项

```python
# default
print(f"{'*'*10}==>>default")
print(f"{settings.ALGORITHM=}")
print(f"{settings.ACCESS_TOKEN_EXPIRE_MINUTES=}")

# 环境区分
print(f"{'*'*10}==>>环境区分: {settings.ENV_FOR_DYNACONF}")
print(f"{settings.BASE_URL=}")
print(f"{settings.SECRET_KEY=}")
print(f"{settings.DB_HOST=}")
print(f"{settings.DB_USER=}")
print(f"{settings.DB_PASSWORD=}")
print(f"{settings.REDIS_HOST=}")
print(f"{settings.REDIS_PORT=}")

# 在default中使用通过环境区分的变量
print(f"{'*'*10}==>>在default中使用通过环境区分的变量")
print(f"{settings.SQLALCHEMY_DATABASE_URL_1=}")
print(f"{settings.SQLALCHEMY_DATABASE_URL_2=}")
print(f"{settings.SQLALCHEMY_DATABASE_URL_3=}")
```

#### 1）使用默认情况（或者 `ENV_FOR_DYNACONF=develop`）的配置项

![](https://raw.githubusercontent.com/XD825/picgo/main/img/image%20(6).png)

#### 2）使用 `ENV_FOR_DYNACONF=master`的自定义配置项

**① 设置环境变量**

```bash
export ENV_FOR_DYNACONF=master
```

**② 结果**
![](https://raw.githubusercontent.com/XD825/picgo/main/img/image%20(7).png)

## 3.6、使用系统环境变量值

### 3.6.1. 自定义系统环境变量前缀

设置的系统环境变量需要以指定的前缀命名

```python
settings = Dynaconf(
    envvar_prefix="DYNACONF", # 读取系统环境变量的前缀（可自定义）
    settings_files=['settings.toml', '.secrets.toml'],
    environments=True,
    ENV_FOR_DYNACONF="develop"  # 修改默认使用的环境，默认为：[development]
)
```

### 3.6.2. 设置环境变量

```bash
export DYNACONF_USER=Desire
export DYNACONF_PWD=123456
export DYNACONF_ACCESS_TOKEN_EXPIRE_MINUTES=60 # 这一个环境变量和配置文件中的名称【ACCESS_TOKEN_EXPIRE_MINUTES】一样
```

### 3.6.3. 读取系统环境变量

按照指定的前缀添加系统环境变量，读取的时候直接忽略前缀即可

- 如果读取不存在的系统环境变量则会抛出异常
- 如果设置的系统环境变量和配置文件中的同名，则会优先读取系统环境变量的值

```python
print(f"{settings.USER=}")
print(f"{settings.PWD=}")
print(f"{settings.ACCESS_TOKEN_EXPIRE_MINUTES=}")
```

![](https://raw.githubusercontent.com/XD825/picgo/main/img/image%20(8).png)

# 四、项目中应用

## 4.1、配置文件路径结构（`fast_api/app/core/config`）

![](https://raw.githubusercontent.com/XD825/picgo/main/img/202206151718066.png)

## 4.2、`config.py`配置项中修改 `settings_files`使用基于项目的相对路径

如果此处直接使用 `settings_files=['settings.toml','.secrets.toml']`在 `fast_api/app/core/config`文件夹之外的模块中使用则会报错找不到配置文件位置
![](https://raw.githubusercontent.com/XD825/picgo/main/img/202206151719146.png)

## 4.3、添加 `__init__.py`文件

```python

from app.core.config.config import settings

```

在 `__init__.py`文件中就一行导包语句，那么我为什么要多此一举呢，是为了在进行配置项导包的时候可以少导一层，这样导包看起来也比较整洁

### 添加之前使用配置文件

```python
# 导包
from app.core.config.config import settings
```

### 添加之后使用配置文件

```python
# 导包
from app.core.config import settings
```

## 4.4、新增配置项（有代码块的配置项）

可以放在 `__init__.py`文件中，也可以放在 `settings.py`文件中。

```python
# 项目绝对路径配置项
settings.BASE_APP_DIR = os.path.dirname(os.path.dirname(os.path.dirname(__file__)))
# or 也可以通过__setattr__()进行添加
# settings.__setattr__("BASE_APP_DIR", os.path.dirname(os.path.dirname(os.path.dirname(__file__))))
# or 通过字典添加键值对的方式进行添加
# settings["BASE_APP_DIR"] = os.path.dirname(os.path.dirname(os.path.dirname(__file__)))
```

# 附件

【[Dynaconf官方文档](https://www.dynaconf.com/)】

