##注册服务

在上一节中，我们运行了我们第一个agent,查询了集群内的成员，并且查询请求了一个节点。在这一节中，我们将注册我们第一个服务，并对它就行查询操作。

#定义一个服务

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
