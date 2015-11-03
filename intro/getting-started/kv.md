##KEY/VALUE DATA
除里提供服务发现和完整的健康检查，Consul提供了一个易于使用的键／值存储。这可以用来保存动态配置，辅助服务协调，建立领导人选举和任意一个开发者能想到的功能。


这一步假设你至少有一个Consul agent已经在运行。

###简单使用
为来证明它是多么的简单上手，我们将在KV存储中操作一些键。

查询本地的agent，我们可以先验证是否存在一些键在KV存储中。

```
$ curl -v http://localhost:8500/v1/kv/?recurse
* About to connect() to localhost port 8500 (#0)
*   Trying 127.0.0.1... connected
> GET /v1/kv/?recurse HTTP/1.1
> User-Agent: curl/7.22.0 (x86_64-pc-linux-gnu) libcurl/7.22.0 OpenSSL/1.0.1 zlib/1.2.3.4 libidn/1.23 librtmp/2.3
> Host: localhost:8500
> Accept: */*
>
< HTTP/1.1 404 Not Found
< X-Consul-Index: 1
< Date: Fri, 11 Apr 2014 02:10:28 GMT
< Content-Length: 0
< Content-Type: text/plain; charset=utf-8
<
* Connection #0 to host localhost left intact
* Closing connection #0

```


由于不存在KEY，我们得到404响应。现在，我们能`PUT`一些例子。


```
$ curl -X PUT -d 'test' http://localhost:8500/v1/kv/web/key1
true
$ curl -X PUT -d 'test' http://localhost:8500/v1/kv/web/key2?flags=42
true
$ curl -X PUT -d 'test'  http://localhost:8500/v1/kv/web/sub/key3
true
$ curl http://localhost:8500/v1/kv/?recurse
[{"CreateIndex":97,"ModifyIndex":97,"Key":"web/key1","Flags":0,"Value":"dGVzdA=="},
 {"CreateIndex":98,"ModifyIndex":98,"Key":"web/key2","Flags":42,"Value":"dGVzdA=="},
 {"CreateIndex":99,"ModifyIndex":99,"Key":"web/sub/key3","Flags":0,"Value":"dGVzdA=="}]

```

这里我们创建了三个键，每个键都为"test"。需要注意值返回的结果是base64编码的非utf-8字符。对于key"web/key2"，我们设置了一个值为42的flag。所有的key支持设置一个64位的整数flag。这并不是在内部使用的，它可以被客户端用于有意义的元数据增加到如何的KV。

设置这些值之后，我们在发出GET请求加上`?recurse`参数取回多个key。

你也可以很容易的获取一个key值。


```
$ curl http://localhost:8500/v1/kv/web/key1
[{"CreateIndex":97,"ModifyIndex":97,"Key":"web/key1","Flags":0,"Value":"dGVzdA=="}]

```

删除键也很简单，通过使用`DELETE`请求。我们能通过制定完整的路径删除单个key键，或者我们可以使用`?recurse`删除当前地址目录下的所有keys。


```
$ curl -X DELETE http://localhost:8500/v1/kv/web/sub?recurse
$ curl http://localhost:8500/v1/kv/web?recurse
[{"CreateIndex":97,"ModifyIndex":97,"Key":"web/key1","Flags":0,"Value":"dGVzdA=="},
 {"CreateIndex":98,"ModifyIndex":98,"Key":"web/key2","Flags":42,"Value":"dGVzdA=="}]
```

key能通过`PUT`请求修改相同的URI的值。此外，Consul提供了验证设置操作。激活原子更新。通过GET获取最后一个`ModifyIndex`值放入到`?cas=`参数中。举个例子，我们去更新"web/key1":

```
$ curl -X PUT -d 'newval' http://localhost:8500/v1/kv/web/key1?cas=97
true
$ curl -X PUT -d 'newval' http://localhost:8500/v1/kv/web/key1?cas=97
false

```

在这种情况下，第一个 CAS更新成功，因为`ModifyIndex`是97.但第二次操作失败了，因为`ModifyIndex`不再是97。


我们也能使用`ModifyIndex`去等待一个值的改变。比如，假设我们要等待key2值的改变：

```
$ curl "http://localhost:8500/v1/kv/web/key2?index=101&wait=5s"
[{"CreateIndex":98,"ModifyIndex":101,"Key":"web/key2","Flags":42,"Value":"dGVzdA=="}]


```

通过提供"?index=",我们要求等到`ModifIndex`的值大于101.然而参数"?wait=5s"限制查询到最多5秒，返回当前，不变的值。这样可以有效的等待键修改。另外，同样的方法可以被用来等待键的列表，等待直到所有的键有一个最新的修改时间。

















