##注册服务

在上一节中，我们运行了我们第一个agent,查询了集群内的成员，并且查询请求了一个节点。在这一节中，我们将注册我们第一个服务，并对它就行查询操作。

###定义一个服务

一个服务能从两个地方被定义。一个是服务描述或者是通过相关HTTP API设置。


一个服务描述是最常见的注册服务的方法，所以我们将在这一步中讨论它。我们将构建一个agent配置文件用在上一节的命令中。

首相，创建一个Consul配置目录。Consul将加载目录内的所有配置文件，所以在Unix系统中通常把目录名称设置为 ｀／etc/consul.d｀。

```
$sudo mkdir /etc/consul.d
```

接下来，我们将写一个服务描述配置文件。假设我们有个服务叫“web”运行在80端口上。另外我们加上一个tag为了我们能通过它查询这个服务：

```
$ echo '{"service": {"name": "web", "tags": ["rails"], "port": 80}}' \
    >/etc/consul.d/web.json

```

现在，重启agent,加上之前的配置文件目录参数：

```
$ consul agent -server -bootstrap-expect 1 -data-dir /tmp/consul \
    -config-dir /etc/consul.d
==> Starting Consul agent...
...
    [INFO] agent: Synced service 'web'
...

```

你会注意到输出中写着 “synced” web 服务。这意味着已经加载了配置中的信息。

如果你想注册多个服务，你可以创建多个服务描述文件在Consul配置目录中。



###查询服务

再一次的agent启动了并且服务就行了链接同步，现在我们能通过DNS或者HTTP API来查询服务了。

####DNS API


####HTTP API

```
$ curl http://localhost:8500/v1/catalog/service/web
[{"Node":"agent-one","Address":"172.20.20.11","ServiceID":"web", \
    "ServiceName":"web","ServiceTags":["rails"],"ServicePort":80}]

```

###更新服务配置列表

服务的描述配置能通过修改配置文件和发送 SIGHUP 给agent即可从新得到加载。这就可以让我们在不重启的情况下进行服务描述更新。

或者，通过 HTTP API 能去增加，删除，修改服务描述。


###下一节
我们已经配置了一个独立的agent和注册了一个服务。这是一个好的开始，现在让我们探究Consul剩下的价值`设置我们第一个集群`










