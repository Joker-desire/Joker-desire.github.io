# Jenkins构建后执行脚本


## 构建后执行脚本-PostBuildScript

在执行完Jenkins构建失败时，想发起报警，通过shell脚本进行编写，然而Jenkins默认只能在构建前执行shell。

当有该需求时，Jenkins的[PostBuildScript](https://plugins.jenkins.io/postbuildscript/)插件就能够帮助我们实现。

## 使用步骤

### 1、安装[PostBuildScript](https://plugins.jenkins.io/postbuildscript/)插件

![image-20220805141708166](https://raw.githubusercontent.com/XD825/picgo/main/img/202208051417271.png)

### 2、在Job的构建后操作就可以看到Execute scripts

![image-20220805141825160](https://raw.githubusercontent.com/XD825/picgo/main/img/202208051418203.png)

### 3、支持多种方式的脚本执行

![image-20220805142114358](https://raw.githubusercontent.com/XD825/picgo/main/img/202208111422974.png)

#### Add generic script file：添加通用的脚本文件

![image-20220805142541945](https://raw.githubusercontent.com/XD825/picgo/main/img/202208051425997.png)

#### Add Groovy script file: 添加 Groovy 脚本文件

![image-20220805142828528](https://raw.githubusercontent.com/XD825/picgo/main/img/202208051428570.png)

#### Add Groovy script: 添加Groovy脚本

![image-20220805143017280](https://raw.githubusercontent.com/XD825/picgo/main/img/202208051430332.png)

#### Add post build step：添加后期构建步骤

![image-20220805143156055](https://raw.githubusercontent.com/XD825/picgo/main/img/202208051431111.png)

#### 4、指定构建结果执行脚本

![image-20220805143603746](https://raw.githubusercontent.com/XD825/picgo/main/img/202208051436800.png)

1. SUCCESS: 构建成功
2. UNSTABLE：构建不稳定
3. FAILURE：构建失败
4. NOT_BUILT：未构建
5. ABORTED：构建被终止

只有达到选定的构建结果之一时，才会执行脚本/步骤。使用键盘上的控制键选择多个结果。

## 实践一下

在Jenkins执行构建失败时，通过shell脚本发送报警

### 构建操作

![image-20220805144246742](https://raw.githubusercontent.com/XD825/picgo/main/img/202208051442815.png)

### 构建后操作

![image-20220805144407397](https://raw.githubusercontent.com/XD825/picgo/main/img/202208051444449.png)

### 执行结果

![image-20220805144635490](https://raw.githubusercontent.com/XD825/picgo/main/img/202208051446527.png)

### 构建成功的执行结果

![image-20220805144910384](https://raw.githubusercontent.com/XD825/picgo/main/img/202208051449448.png)

