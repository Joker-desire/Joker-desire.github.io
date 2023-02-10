# Jenkins任务中使用Pipenv


## 前言

📍pipenv是Python官方推荐的包管理工具。

📍在使用Python环境，基本上都是使用虚拟环境进行处理的。

📍在Jenkins任务shell中使用常用的方式却无法使用pipenv虚拟环境😤

🩸那么如何解决这个问题呢❓

## 探索在本地pipenv使用流程

✍️[pipenv使用指南](https://crazygit.wiseturtles.com/2018/01/08/pipenv-tour/)

1. 安装pipenv包

    ```bash
    pip3 install pipenv
    ```

    

2. 创建pipenv虚拟环境

    ```bash
    # 指定Python版本
    pipenv --python 3.10
    ```

    

3. 进入虚拟环境

    ```bash
    # 激活当前项目的虚拟环境
    pipenv shell
    ```

4. 虚拟环境存放路径

    ```bash
    /Users/`USER`/.local/share/virtualenvs/
    ```

5. 虚拟环境文件夹规则

    1. 前缀为创建虚拟环境时的文件夹名，后面部分为随机生成：`python_file-ZSRTT-ad`

    2. 查找此文件夹可使用

        ```shell
        # 通过当前任务名，查询虚拟环境文件夹名称
        ➜  ~ echo $(ls $HOME/.local/share/virtualenvs | grep "python_file-")
        python_file-ZSRTT-ad
        ```

6. 进入到虚拟环境后，查看当前系统环境变量：`env`

    1. `PATH`环境变量：`PATH=/Users/USER/.local/share/virtualenvs/python_file-ZSRTT-ad/bin:...`
    2. `VIRTUAL_ENV`变量（启动虚拟环境变量后新增的）：`VIRTUAL_ENV=/Users/USER/.local/share/virtualenvs/python_file-ZSRTT-ad`

7. 通过查看系统环境变量，可以得到结论

    1. 使用虚拟环境后，会自动在环境变量中增加一个变量`VIRTUAL_ENV`，值为虚拟环境变量路径

    2. 在`PATH`环境变量中，增加了虚拟环境的`bin`目录

        ```bash
        VIRTUAL_ENV="虚拟环境路径"
        PATH=$VIRTUAL_ENV:$PATH
        ```


## 在Jenkins中实践

1. 创建一个名为`py3.10`的任务

2. 在构建步骤中添加shell

    ```shell
    # 初始化虚拟环境
    pipenv --python 3.10
    # 获取当前任务的虚拟环境路径
    virName=$(ls $HOME/.local/share/virtualenvs | grep "$JOB_NAME-")
    # 设置虚拟环境临时环境变量
    export VIRTUAL_ENV=$HOME/.local/share/virtualenvs/$virName
    # 把bin加入到path环境变量中
    export PATH=${VIRTUAL_ENV}/bin:$PATH
    # 检查当前python版本
    python3 --version
    ```

3. 构建，查看打印的python版本是否为3.10

    ![image-20220718100102272](https://raw.githubusercontent.com/XD825/picgo/main/img/202207181001348.png)

4. 成功了✌️

![查看源图像](https://raw.githubusercontent.com/XD825/picgo/main/img/202207181010056.jpeg)

## 优化构建步骤中的shell

从上面的shell可以看到，每次构建一个新的项目，都需要写五行shell脚本，总是做重复性的工作，对于这一块进行了优化，如下：

1. 把重复性的shell单独存放在一个文件(`initPipenv.sh`)中，存放在`/tmp`文件夹下，通过传递参数执行

    ```shell
    #! /bin/bash
    # Author: Joker-desire
    # Date: 2022-07-15
    
    
    initPipfile()
    {
        echo "Init python virtual env"
        # 参数1：python版本
        if [ -f "Pipfile" ]; then
            echo "Python virtual env is existed"
        else
            echo "Start build python virtual env"
            pipenv --python $1
        fi
    }
    getVirtName()
    {
        # 参数1: 虚拟环境名称，实际就是Jenkins任务名
        # 通过当前任务名，查询虚拟环境文件夹名称
        echo $(ls $HOME/.local/share/virtualenvs | grep "$1-")
    }
    
    addEnv()
    {
        echo "Add temporary system env variables"
        # 设置虚拟环境临时环境变量
        export VIRTUAL_ENV=$HOME/.local/share/virtualenvs/$1
        # 把bin加入到path环境变量中
        export PATH=${VIRTUAL_ENV}/bin:$PATH
    }
    
    main()
    {
        echo "come into workspace"
        cd $3
        py=$1
        jobName=$2
        initPipfile $py
        virtName=`getVirtName $jobName`
        addEnv $virtName
        # 检查当前python版本
        python3 --version
    }
    
    # 参数1 python版本
    # 参数2 Jenkins任务名称 $JOB_NAME
    # 参数3 Jenkins Job文件夹路径
    main $1 $2 $3
    ```

2. 在构建shell中，直接执行此shell脚本即可

    ```shell
    # 参数1：python版本
    # 参数2：当前Jenkins任务名称
    # 参数3：当前的路径
    . /tmp/initPipenv.sh "3.10" $JOB_NAME `pwd`
    ```

3. 构建，输出结果和上面一样

    ![image-20220718101753841](https://raw.githubusercontent.com/XD825/picgo/main/img/202207181017885.png)

4. 每次如果有新的Jenkins py任务，就可以直接在构建shell中加一行执行`initPipenv.sh`即可使用虚拟环境

‼️**需要注意执行shell脚本的方式，需要使用`.`执行shell**


