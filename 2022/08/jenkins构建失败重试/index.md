# Jenkins构建失败重试


有一些构建任务可能会由于某些不可抗因素导致构建失败，但是重新构建一次就构建成功了，针对这种情况，可以使用Jenkins插件[Naginator](https://plugins.jenkins.io/naginator/)进行构建失败重试

## 使用Naginator

### 安装Naginator插件

![image-20220805154932236](https://raw.githubusercontent.com/XD825/picgo/main/img/202208051549272.png)

### 使用

#### 在构建后操作中选择：Retry build after failure

![image-20220805155359409](https://raw.githubusercontent.com/XD825/picgo/main/img/202208051553440.png)

#### 配置信息

![image-20220805155743658](https://raw.githubusercontent.com/XD825/picgo/main/img/202208051557687.png)

#### 运行结果

1. 构建历史，如果构建失败，则会重试三次

    ![image-20220805161148705](https://raw.githubusercontent.com/XD825/picgo/main/img/202208051611749.png)

2. 查看最后一次构建历史，可以看到是由Naginator启动的

    ![image-20220805160119245](https://raw.githubusercontent.com/XD825/picgo/main/img/202208051601274.png)

