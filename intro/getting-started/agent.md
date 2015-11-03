##运行Consul Agent

Consul安装之后，agent必须要运行起来。agent能在两种模式中运行：server, client。每一个数据中心最少要有一个server,然而一个集群中推荐最少3到5个server。在只有一个server的环境中，如果发生失败，既有可能会丢失数据。

所有其他的agents运行在client模式下。一个client是非常轻巧的进程去完成注册服务，健康检查，转发查询请求给server。agent必须运行在集群中的每一个节点内。


更多细节参考初始化一个数据中心，访问 `详情`。


###启动Agent

一个最简单的例子，我们将配置一个服务模式下的Consul agent：

```
$ consul agent -server -bootstrap-expect 1 -data-dir /tmp/consul
==> WARNING: BootstrapExpect Mode is specified as 1; this is the same as Bootstrap mode.
==> WARNING: Bootstrap mode enabled! Do not enable unless necessary
==> WARNING: It is highly recommended to set GOMAXPROCS higher than 1
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Consul agent running!
       Node name: 'Armons-MacBook-Air'
      Datacenter: 'dc1'
          Server: true (bootstrap: true)
     Client Addr: 127.0.0.1 (HTTP: 8500, DNS: 8600, RPC: 8400)
    Cluster Addr: 10.1.10.38 (LAN: 8301, WAN: 8302)

==> Log data will now stream in as it occurs:

[INFO] serf: EventMemberJoin: Armons-MacBook-Air.local 10.1.10.38
[INFO] raft: Node at 10.1.10.38:8300 [Follower] entering Follower state
[INFO] consul: adding server for datacenter: dc1, addr: 10.1.10.38:8300
[ERR] agent: failed to sync remote state: rpc error: No cluster leader
[WARN] raft: Heartbeat timeout reached, starting election
[INFO] raft: Node at 10.1.10.38:8300 [Candidate] entering Candidate state
[INFO] raft: Election won. Tally: 1
[INFO] raft: Node at 10.1.10.38:8300 [Leader] entering Leader state
[INFO] consul: cluster leadership acquired
[INFO] consul: New leader elected: Armons-MacBook-Air
[INFO] consul: member 'Armons-MacBook-Air' joined, marking health alive


```


你能看到，Consul agent 有一些日志输出。通过日志内容，你能看出我们的agent已经运行在server模式下并且要求在集群中升级为leader。另外本地的成员已经被标记为健康状态。



##集群成员
如果你在其他终端运行命令 `consul members`，你能看到集群中的所有成员。在下一章节我们会介绍如何加入集群的操作，但是现在，你已经能看到一个成员（之前启动的agent）


输出显示了我们正在运行的的节点，地址，健康状态，在集群中的角色和一些版本信息等。额外的元数据查看可以增加运行参数`-detailed`。


由成员节点输出的命令是在`gossip protocol`基础之上的，能保持最终一致。意味着，在任一时间点上可能本地agent看到的数据不一定和server端完全一致。为了保持强一致性，可以使用`HTTP API`它主动请求Consul server端：

```
$ curl localhost:8500/v1/catalog/nodes
[{"Node":"Armons-MacBook-Air","Address":"10.1.10.38"}]
```

###停止Agent

我们可以通过 `Ctrl-C`(中段信号)优雅的中断agent。在中断agent之后，你能看到它离开了集群并且关闭。


通过优雅推出，Consul能通知到其他的Cluster成员这个节点退出。如果你是强制退出agent进程，在集群中的其他成员也可以察觉到此节点已经失败。当一个成员节点离开时，它当服务和checks会从catalog中移除。当一个成员节点失败时，它当健康状态被简单的标记为critical,但是它并不会从catalog中移除。Consul将自动尝试重新连接失败的节点，等待成员节点恢复它的网络连接，最后离开的节点不在被长连接。

另外，如果一个运行模式为server的agent，一个优雅退出是非常必须的，可以避免发生意外的中断影响到consensus protoco。看相关`章节`学习如何安全的增加和移除服务。


##下一步
你已经运行了简单的Consul集群。现在让我们向里面增加点服务。













