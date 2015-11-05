###CONSUL WEB UI

Consul自带的漂亮，功能界面开箱即用。UIS上可以查看所有服务与节点，查看所有健康检查和它们当前状态，还可以读取和设置key/value值。UIS还自动的支持多数据中心。

有两种方法运行Consul UI：使用 Atlas by HashiCorp去连接你的数据中心或者自己搭建开源的UI。


##Atlas-hosted Dashboard




##Self-hosted Dashboard
要自己搭建自己的UI，下载web UI包，解压到，并且保证Consul已经安装。重启启动Consul，并且增加`-ui-dir`参数指向你减压的UI目录（包含index.html文件的目录）：

```
$ consul agent -ui-dir /path/to/ui
...
```

UI通过访问`/ui`地址和API一样的端口。默认地址是`http://localhost:8500/ui`。

你可以访问在线的UI例子。

虽然在线例子能够访问所有数据中心的数据，我们也已经设置演示终端的具体数据中心：AMS2（阿姆斯特丹），SFO1（旧金山），和NYC3（纽约）。


###下一步
我们结束了入门指南。看下面步骤页面，了解更多有关如何继续学习Consul。