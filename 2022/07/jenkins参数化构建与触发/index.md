# Jenkins参数化构建与触发


## Jenkins参数化构建

### 创建一个带参数的任务

> 勾选参数化构建

![](https://raw.githubusercontent.com/XD825/picgo/main/img/202207132311233.png)

> 添加选项参数

![](https://raw.githubusercontent.com/XD825/picgo/main/img/202207132316459.png)

> 字符参数

![image-20220713232227583](https://raw.githubusercontent.com/XD825/picgo/main/img/202207132322612.png)

> 文本参数

![image-20220713232150183](https://raw.githubusercontent.com/XD825/picgo/main/img/202207132321225.png)

> 文件参数

![image-20220713231731410](https://raw.githubusercontent.com/XD825/picgo/main/img/202207132317486.png)

### 获取参数信息

![image-20220713232310557](https://raw.githubusercontent.com/XD825/picgo/main/img/202207132323599.png)

### 构建带参数化的任务

![image-20220713232703214](https://raw.githubusercontent.com/XD825/picgo/main/img/202207132327250.png)

### 构建结果

![image-20220713232819695](https://raw.githubusercontent.com/XD825/picgo/main/img/202207132328753.png)

注：文件参数，则会上传到当前的工作空间

## Jenkins任务触发

在实际工作总结，经常会遇到，执行完任务1，然后再执行任务2。

像这种的连续触发任务，可以在任务1中添加构建后操作->构建其他工程

![image-20220713233646655](https://raw.githubusercontent.com/XD825/picgo/main/img/202207132336712.png)

### 构建其他工程配置

- 填写要构建的项目名称
- 选择什么时候触发

![image-20220713233734322](https://raw.githubusercontent.com/XD825/picgo/main/img/202207132337387.png)

### 任务详情展示效果

![image-20220713234158397](https://raw.githubusercontent.com/XD825/picgo/main/img/202207132341445.png)

![image-20220713234227307](https://raw.githubusercontent.com/XD825/picgo/main/img/202207132342363.png)

### 构建效果

- 构建任务1
- 任务1构建完成后，如果构建稳定则自动构建任务2

在任务1的日志中，可以看到成功的触发新构建

![image-20220713234355040](https://raw.githubusercontent.com/XD825/picgo/main/img/202207132343090.png)

在任务2的日志中，第一行可以看到任务2是由上游项目第2次构建启动的

![image-20220713234525064](https://raw.githubusercontent.com/XD825/picgo/main/img/202207132345113.png)

## Jenkins参数化任务触发

当任务1中有构建后需要传递给任务2的参数时（或者任务2需要传递参数时），要想实现带参数构建，需要借助一个Jenkins插件：[`Parameterized Trigger`](https://plugins.jenkins.io/parameterized-trigger/)

1. 在任务1中，将执行完毕的参数保存在profile.txt文件中

    因为终端shell执行完毕后变量都会回收，所以不能够将变量直接传递给任务2，需要将其写入到文件中然后以文件的形式传递，在构建的shell脚本中，添加一行写入文件

    ```shell
    echo "env=$env" > profile.txt
    ```

2. 在任务1中进行更改构建其他工程的方式，选择`Trigger parameterized build on other projects`

    ![image-20220714000358877](https://raw.githubusercontent.com/XD825/picgo/main/img/202207140003930.png)

3. 进行配置参数

    ![image-20220714001145570](https://raw.githubusercontent.com/XD825/picgo/main/img/202207140011615.png)

4. 在任务2中接收参数

    ![image-20220714001321270](https://raw.githubusercontent.com/XD825/picgo/main/img/202207140013299.png)

5. 运行任务1

    ![image-20220714003054189](https://raw.githubusercontent.com/XD825/picgo/main/img/202207140030236.png)

6. 任务2自动触发，获取到参数

    ![image-20220714003018417](https://raw.githubusercontent.com/XD825/picgo/main/img/202207140030461.png)

