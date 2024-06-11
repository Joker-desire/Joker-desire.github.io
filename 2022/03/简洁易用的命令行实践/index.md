# 简洁易用的命令行实践

## 之前的命令行编写方式

### getopt实现方式（不讨论具体代码实现）👇🏻

```python
class ALSRun(object):

    def __init__(self, in_file, out_file=None):
        self.csv = in_file
        self.out_file = out_file

    def recommend(self):
        ......
        if not self.out_file:
            self.out_file = f'{time.strftime("%Y%m%d%H%M%S")}.csv'
        nrecommendations.coalesce(1).write.csv(self.out_file)
        print("🗣执行完毕🤝🤝🤝")


if __name__ == '__main__':
    # 获取文件名
    file_name = sys.argv[0]
    usage = f'用法：{file_name} -i <inputfile> -o <outputfile>\n参数解释 \n\t-i|--ifile: 执行（必填）\n\t<inputfile>: 表示要执行的csv文件\n\t-o|--ofile:输出（非必填） \n\t<outputfile>: 表示要输出的csv文件,默认以当前时间为文件名'
    try:
        opts, args = getopt.getopt(sys.argv[1:], "hi:o:", ["ifile=", "ofile="])
    except getopt.GetoptError:
        print(f'{file_name} -i <inputfile> -o <outputfile>')
        sys.exit(2)
    inputfile = None
    outputfile = None
    if len(opts) == 0:
        print(usage)
        sys.exit()
    for opt, arg in opts:
        if opt == '-h':
            print(usage)
            sys.exit()
        elif opt in ("-i", "--ifile"):
            inputfile = arg
        elif opt in ("-o", "--ofile"):
            outputfile = arg

    if inputfile.endswith(".csv") and (outputfile is None or outputfile.endswith(".csv")):
        print("==========开始执行ALS==========")
        als = ALSRun(in_file=inputfile, out_file=outputfile)
        als.recommend()

    else:
        print("错误的传参，请看如下用法↓")
        print(usage)
```

### 不足之处

1. 太繁琐，不简洁（仅仅就是为了实现一个 `python3 100als.py -i article.csv -o t.csv`命令就要多写20+行）
2. 不易懂（啥玩意儿，过段时间再看就费劲了）

**如何解决命令行的问题呢？**
![](https://raw.githubusercontent.com/XD825/picgo/main/img/202206151720615.png)

## 使用Click处理命令行

[![](https://raw.githubusercontent.com/XD825/picgo/main/img/202206151720765.png)](https://click.palletsprojects.com/en/8.0.x/)

### 什么是Click

Click是一个Python包，用于以可组合的方式使用尽可能少的代码创建漂亮的命令行界面，是 `命令行界面创建工具`、是高度可配置的，但带有开箱即用的合理默认值。
它旨在使编写命令行工具的过程变得快速而有趣，同时还防止因无法实现预期的 CLI API 而导致的任何挫败感。

### Click的优点

- 命令的任意嵌套
- 自动帮助页面生成
- 支持在运行时延迟加载子命令

### Click安装

```bash
pip install click
```

### 简单的例子

```python
import click

@click.command()
@click.option('--count', default=1, help='Number of greetings.')
@click.option('--name', prompt='Your name', help='The person to greet.')
def hello(count, name):
    """Simple program that greets NAME for a total of COUNT times."""
    for x in range(count):
        click.echo(f"Hello {name}!")

if __name__ == '__main__':
    hello()
```

![](https://raw.githubusercontent.com/XD825/picgo/main/img/202206151721294.png)
从上面的例子可以看到，就仅仅的使用了三个装饰器，就把命令行实现了（并且自带--help）👍🏻

### 简单的使用说明

- `@click.command()`：用来声明被装饰的类/方法是一个命令行
- `@click.option()`：给命令添加选项
  - `@click.option('--count', default=1, help='Number of greetings.')`
  - 以上面为例
    - 参数1：命令行选项名
    - default参数：默认值
    - help：帮助信息
    - prompt：提示信息（在没有传递选项的时候，会在控制台提示输入）
- 函数接收的值具体规则如下：
  - 选择名称的顺序：
    1. 如果名称没有前缀，则将其用作Python参数名称，而不将其视为命令行上的选项名称

```python
@click.command()
@click.option("-f", "--filename", "dest") # 名称为 dest
def echo(dest):
    print(f"{dest=}")
```

    2. 如果至少有一个名称以两个破折号为前缀，则将给定的第一个名称用作名称

```python
@click.command()
@click.option("--f", "--foo-bar") # 名称为f
def echo(f):
    print(f"{f=}")
```

    3. 否则使用以一个破折号为前缀的名字

```python
@click.command()
@click.option("-f", "-fb") # 名称为 f
def echo(f):
    print(f"{f=}")
```

## 解决问题

### 解决步骤

1. 清除掉之前使用getopt编写命令行方式的代码
2. 定义一个函数 `def run(infile, outfile):`，用来处理命令行（也可以直接在原有类上使用，但是不建议使用，会破坏原有逻辑）
3. 在 `__main__`中调用函数 `run()`
4. 在控制台使用命令行执行程序 `python3 als.py -i article.csv`

### 代码展示

```python
@click.command()
@click.option("-i", "--in", "infile", prompt="输入文件", help="输入文件（以.csv为后缀）")
@click.option("-o", "--out", "outfile", default=None, help="输出文件（默认为当前时间戳，以.csv为后缀）")
def run(infile, outfile):
    if infile.endswith(".csv") and (outfile is None or outfile.endswith(".csv")):
        ALSRun(infile=infile, outfile=outfile)
    else:
        print("参数格式有误！！！")


class ALSRun(object):

    def __init__(self, infile, outfile):
        ...


if __name__ == '__main__':
    run()
```

### 结果展示

```bash
$ python3 100als.py --help  

Usage: als.py [OPTIONS]

Options:
  -i, --in TEXT   输入文件（以.csv为后缀）
  -o, --out TEXT  输出文件（默认为当前时间戳，以.csv为后缀）
  --help          Show this message and exit.
```

```bash
$ python3 100als.py -i article.csv

输入文件：article.csv
输出文件：20220308175307.csv
==========开始执行ALS==========
...
```

### 优点说明

1. 简洁、易用、易读
2. 不污染原逻辑
3. 功能强大、拓展性强

### 总结

![](https://raw.githubusercontent.com/XD825/picgo/main/img/202206151722977.png)

## 附件

具体Click用法请参考官方文档「[链接](https://click.palletsprojects.com/en/8.0.x/)」

