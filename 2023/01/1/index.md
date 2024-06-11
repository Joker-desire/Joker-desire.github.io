# ImportError: cannot import name 'get_distribution' from 'pkg_resources'问题解决

## 问题

![image-20230131153151384](https://raw.githubusercontent.com/XD825/picgo/main/img/202301311531966.png)

## 原因

在使用Docker部署的时候，首先进行了更新pip，更新命令：`/usr/local/bin/python -m pip install --upgrade pip` 

在这里只确保了pip是最新的，从而忽略了`setuptools`和`wheel`，虽然仅 pip 就足以从预构建的二进制存档安装，但 setuptools 和 wheel 项目的最新副本有助于确保您也可以从源存档安装。[「Python官方文档说明」](https://packaging.python.org/en/latest/tutorials/installing-packages/#ensure-pip-setuptools-and-wheel-are-up-to-date)

![image-20230131154604424](https://raw.githubusercontent.com/XD825/picgo/main/img/202301311546523.png)

## 解决

将更新命令更改为：`/usr/local/bin/python -m pip install --upgrade pip setuptools wheel`

