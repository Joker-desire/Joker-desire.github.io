# Mixed Content The page at was loaded over HTTPS but requested an insecure resource This request has been blocked the content must be served over HTTPS


## 描述

在HTML文件中引用js文件`<script src="{{ url_for('static', path='/js/app.js') }}"></script>` ，在本地调试正常，上线后发现网页打不开，打开控制台看到了如下报错信息：

![](https://raw.githubusercontent.com/XD825/picgo/main/img/202303052104620.png)

## 原因

由于线上使用的协议是https协议，而生成的js文件链接是http的，HTML默认是不支持混合使用的，所以才出现上面的错误。

## 解决

在HTML文件中添加<meta> 标记允许混合内容即可正常使用。

```html
<meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">
```



## 附录

https://stackoverflow.com/questions/67765238/mixed-content-the-page-at-was-loaded-over-https-but-requested-an-insecure-resour

