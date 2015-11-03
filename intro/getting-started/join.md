##ONSUL CLUSTER

我们已经开始了我们第一个agent并且用它注册和查询了服务。这表明着是多么的容易去使用Consul，但是这并没有说明任何可以扩展到一个可扩展到生产级的服务发现基础设施。在这一步中，我们将创建我们第一个真正的集群并且具有多个成员。


当开始一个Consul agent时，它不需要知道任何其他的节点细节：它是一个独立集群中的一个。想要去了解其他集群的成员，agent必须要加入一个现有的集群中。要加入现有的集群，只要知道一个现有的成员。它加入后，agent将 gossip与其它成员并且快速发现集群中的其他成员。一个Consul agent能加入任何其他的agent,不需要agents是server模式。



##开始Agents
为了模拟一个更真实的集群，我们用Vagrant执行两个节点的集群。The Vagrantfile we will be using can be found in the demo section of the Consul repo.

We first boot our two nodes:

```
$ vagrant up
```

Once the systems are available, we can ssh into them to begin configuration of our cluster. We start by logging in to the first node:

```
$ vagrant ssh n1
```

集群中的每个节点必须有一个唯一的名称。默认情况下，Consul使用机器的主机名，但我们将使用-node命令行选项手动覆盖它。


我们也将指定绑定的地址：这是Consul监听的地址，并且它必须能由集群中的所有其他节点进行访问。虽然绑定地址并不做严格要求（Consul会默认监听系统上的第一个私有IP），它总是尽力提供的。生产服务器通常有多个接口地址，所以指定绑定地址可确保你永远不会绑定Consul到错误的接口地址。


第一个节点将作为我们在这个集群中唯一的服务器，我们表明这一点使用server switch

总之这些设置产生一个 Consul agent 命令像：

```
vagrant@n1:~$ consul agent -server -bootstrap-expect 1 \
    -data-dir /tmp/consul -node=agent-one -bind=172.20.20.10
...
```


Now, in another terminal, we will connect to the second node:

```
$ vagrant ssh n2
```


这一次，我们设置绑定地址并且节点名称设置为agent-two。至此这个节点并不是一个Consul server.我们不要提供server开关。

总之，这些配置产生命令向这样：

```
vagrant@n2:~$ consul agent -data-dir /tmp/consul -node=agent-two \
    -bind=172.20.20.11
...
```

在这一点上，你必须运行两个Consul agents。一个server一个client。这两个Consul agents仍然不知道对方的存在，它们都是自己节点集群的一部分。你可以验证这一点，通过对每个agent 执行`consul members`命令发现只有一个成员节点显示。





##加入一个集群
现在，我们在新的终端运行命令，让第一个agent加入第二个agent：
```
$ vagrant ssh n1
...
vagrant@n1:~$ consul join 172.20.20.11
Successfully joined cluster by contacting 1 nodes.

```

你能看道每一个 agent日志道输出。如果你仔细看，你能看到它们收到加入的信息。如果你在每个agent运行`consul members`,你能看到两个彼此知道的agent。


```
vagrant@n2:~$ consul members
Node       Address            Status  Type    Build  Protocol
agent-two  172.20.20.11:8301  alive   client  0.5.0  2
agent-one  172.20.20.10:8301  alive   server  0.5.0  2
```

记住：去加入到一个集群中，一个Consul anget只需要了解到一个现有成员。加入集群后，agent gossip相互传播正式成员到信息。


##启动时自动加入集群
理想情况，每当一个新的节点背带入了你的数据中心，它将自动连接集群，无需人工干预。要做到这一点，你可以使用 Atlas by HashiCorp 和 ` -atlas-join` flag。 下面显示了一个例子：


```
$ consul agent -atlas-join \
  -atlas=ATLAS_USERNAME/infrastructure \
  -atlas-token="YOUR_ATLAS_TOKEN"
```

To get an Atlas username and token, create an account and replace the respective values in your Consul configuration with your credentials. Now, whenever a new node comes up with a Consul agent, it will automatically join your Consul cluster without any hardcoded configuration.

另外，你在启动时候可以使用`-join` flag或者`start_join`设置能连接到的Consul agents地址。


###查询节点
就像查询服务一样，Consul也有查询节点的API,你可以通过DNS或者HTTP API进行查询。


###退出集群
要离开集群，你可以优雅的退出agent（使用 Ctrl-C）或者强制杀死agent。优雅退出准许节点过度到离开状态；否则，其他节点将其检测为已失败。更详细地在这里。


###下一步
我们现在运行了多节点的集群，让我们使我们的服务更加强劲，给它们做健康检查。





