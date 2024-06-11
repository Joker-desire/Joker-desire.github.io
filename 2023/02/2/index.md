# Selenium设置无头模式，页面自动切换H5模式问题解决


## 问题描述

在使用Selenium进行某一网站数据抓取的时候，出现一种现象

在页面中需要定位一个元素信息：`pages = driver.find_elements(By.XPATH, "//button[contains(@class,'page-button')]/span")`

本地调试的时候，一切正常，一放到线上就找不到此元素，日志信息`2023-02-08 12:02:05 [scrapy.utils.log] ERROR: pages = []`

线上开启了无头模式，具体配置如下：

```python
options = webdriver.ChromeOptions()
# 无界面模式
options.add_argument('--headless')
options.add_argument('--no-sandbox')
options.add_argument("--disable-blink-features=AutomationControlled")
options.add_experimental_option('excludeSwitches', ['enable-automation'])
options.add_experimental_option('useAutomationExtension', False)
driver = webdriver.Chrome(options=options)
```

## 问题排查

使用`driver.page_source`将页面源码打印出来，拷贝到文本中，转成html进行查看后发现，页面竟然变成了H5的页面😱

那么到底是怎么回事呢，在原网站上我又重新试了后发现拖拽下网页窗口大小，网站的布局会随之而变化😳

所以要想解决就需要先事先控制窗口的大小。

## 问题解决

为了窗口大小来回变动，就使用最大化窗口进行处理：`driver.maximize_window()`，然而放到线上去执行的时候，依旧会出问题😅

我就在想我都最大化窗口了，怎么还会变成H5呢，让我百思不得其解。后来就想了下，最大化窗口不行，那要不试下自定义窗口大小呢，然后就改成了：`driver.set_window_size(1280, 800)`，上线->执行，竟然成功了✌️

最后我的程序代码：

```python
options = webdriver.ChromeOptions()
# 无界面模式
options.add_argument('--headless')
options.add_argument('--no-sandbox')
options.add_argument("--disable-blink-features=AutomationControlled")
options.add_experimental_option('excludeSwitches', ['enable-automation'])
options.add_experimental_option('useAutomationExtension', False)
driver = webdriver.Chrome(options=options)
driver.set_window_size(1280, 800) # 自定义窗口大小
```



## 总结

由于是无头模式，最大化窗口（`driver.maximize_window()`）并不能确定窗口的大小（有可能最大化后还不能满足我们的需求，比如我这个需求）；

所以为了让我们程序更加准确，可以直接修改窗口大小（`driver.set_window_size(1280, 800)`），设置成我们需要的大小，这样更加精准无误。

