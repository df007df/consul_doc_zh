##健康检查

我们已经看到它是如何简单的运行Consul,节点和服务，并且查询节点和服务。
在本节中，我们将继续通过增加健康检查，检查两个服务和节点。健康检查时服务发现的重要部分，防止使用到不健康的服务。

这一步创建在之前的集群中，此时，你应该有个包含两个节点的集群运行着。



###健康检查
类似于服务，健康检查可以通过提供一个检查描述或者调用合适的HTTP API。

我们将使用检查描述方式，因为就像服务，描述是最常见的健康检查方式。

在第二个节点的Consul配置目录中创建两个描述文件像：


```
vagrant@n2:~$ echo '{"check": {"name": "ping", \
  "script": "ping -c1 baidu.com >/dev/null", "interval": "30s"}}' \
  >/etc/consul.d/ping.json

vagrant@n2:~$ echo '{"service": {"name": "web", "tags": ["rails"], "port": 80,\
  "check": {"script": "curl localhost >/dev/null 2>&1", "interval": "10s"}}}' \
  >/etc/consul.d/web.json


```


第一个描述增加了叫“ping”的一个host等级的检查。每30秒跑一次`ping -c1 google.com`。如果脚本返回非0的退出码，这个节点将标记为不健康。这是一个基于脚本的健康检查。

第二个命令修改了名为`web`的服务，它增加了一个每10秒请求一次的通过curl去验证web服务是否可访问。跟host等级健康检查一样，如果脚本返回非0的退出码，这个节点将标记为不健康。

现在，重新启动第二个agent或者对它发送`SIGHUP`。你将看到以下日志输出：

```
==> Starting Consul agent...
...
    [INFO] agent: Synced service 'web'
    [INFO] agent: Synced check 'service:web'
    [INFO] agent: Synced check 'ping'
    [WARN] Check 'service:web' is now critical

```


这前几行表示了agent已经同步了最新配置描述。最后一行显示，我们增加的web服务检查时严重状态。这是因为我们并没有运行一个web服务，所以这个curl测试是失败的。



###检查健康状态
现在，我们增加了一些简单的检查。我们可以使用HTTP API去检查它们。首先，我们可以查看任何的失败节点（注意，命令可以在任意节点上运行）。

```
vagrant@n1:~$ curl http://localhost:8500/v1/health/state/critical
[{"Node":"agent-two","CheckID":"service:web","Name":"Service 'web' check","Status":"critical","Notes":"","ServiceID":"web","ServiceName":"web"}]

```
我们可以看到，只有一个检查，我们的web服务检查，在严重状态。

此外，我们可以尝试用DNS查询web服务.Consul将不返回任何结果，因为服务是不健康的：

```
dig @127.0.0.1 -p 8600 web.service.consul
...

;; QUESTION SECTION:
;web.service.consul.        IN  A
```


###下一步
在本节中，你学会了如何轻松的增加健康检查。检查描述配置可以通过向agent发送`SIGHUP`信号去更新改变配置。可替代地，用HTTP API可以动态的增加增加，移除和修改检查内容。API还允许一个“死人开关”，一个基于TTL的检查。TTL检查可用于应用程序与Consul进行更紧密的集成。使得业务逻辑被评价为评估检查状态的一部分。

接下来，我们将探讨Consul的 K/V 存储。













