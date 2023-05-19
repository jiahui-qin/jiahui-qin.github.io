---
title: NetconfBaseService 到设备的调用链
date: 2021-12-29 11:22:44
tags:
- OpenDayLight
- java
categories:
- develop
---

今天起了兴趣想要看一下lightly包中的NetconfBaseService是如何调用到设备上的，其实之前同事和我讲过这个问题，不过当时没搞太明白。正好今天有空，就大概学习一下lightly是怎么做的：

---

最开始是先在最外层创建一个NetconfBaseService对象，要传的参数也很简单了,只要传入挂载设备的id就可以了

<!--more-->

```java
NetconfBaseService netconfBaseService = netconfSBPlugin.getNetconfBaseService(NodeId.getDefaultInstance("1")).orElseThrow(()->new NullPointerException("no such device in device tree!"));
DOMRpcResult domRpcResult= netconfBaseService.editConfig(NetconfMessageTransformUtil.NETCONF_RUNNING_QNAME,
                Optional.ofNullable(yangInstanceIdentifierNormalizedNodeEntry.getValue()),
                yangInstanceIdentifierNormalizedNodeEntry.getKey(),
                Optional.empty(),
                Optional.empty(),
                true
        ).get();
```

然后看一下editConfig方法，毕竟就是靠这个方法来沟通设备的，这个方法在io.lighty.modules.southbound.netconf.impl里边，还在lightly里边。NetconfBaseService其实是个接口，继承了DomService, 里边实现了很多对设备的操作方法。

editConfig的参数大致如下：
1. QName：要编辑的数据库的名字，比如candidate、running
2. NormalizedNode： 要向设备传入的内容(odl解析yang文件出来的类)
3. YangInstanceIdentifier 上边解析出来的类的类名

```java
public interface NetconfBaseService extends DOMService {
    ListenableFuture<? extends DOMRpcResult> get(Optional<YangInstanceIdentifier> var1);

    ListenableFuture<? extends DOMRpcResult> getConfig(QName var1, Optional<YangInstanceIdentifier> var2);

    ListenableFuture<? extends DOMRpcResult> editConfig(QName var1, Optional<NormalizedNode<?, ?>> var2, YangInstanceIdentifier var3, Optional<ModifyAction> var4, Optional<ModifyAction> var5, boolean var6);

    ListenableFuture<? extends DOMRpcResult> copyConfig(QName var1, QName var2);

    ......
}
```

然后接着看一下editConfig是怎么实现的：

```java
    public ListenableFuture<? extends DOMRpcResult> editConfig(QName targetDatastore, Optional<NormalizedNode<?, ?>> data, YangInstanceIdentifier dataPath, Optional<ModifyAction> dataModifyActionAttribute, Optional<ModifyAction> defaultModifyAction, boolean rollback) {
        Preconditions.checkNotNull(targetDatastore);
        DataContainerChild<?, ?> editStructure = NetconfUtils.createEditConfigStructure(this.schemaContext, data, dataModifyActionAttribute, dataPath);
        Preconditions.checkNotNull(editStructure);
        return this.domRpcService.invokeRpc(NetconfMessageTransformUtil.toPath(NetconfMessageTransformUtil.NETCONF_EDIT_CONFIG_QNAME), NetconfUtils.getEditConfigContent(targetDatastore, editStructure, defaultModifyAction, rollback));
    }
```

看一下代码就知道在editConfig里边主要实现功能的还是invokeRpc，前边的几行代码都是在做各种校验。传进invokeRpc的就两个参数，NetconfUtils.getEditConfigContent 这个方法负责将传入的参数组装为我们想要传输的xml，前边这个参数就只是一个编辑config的常量。


```java
package org.opendaylight.mdsal.dom.api;

public interface DOMRpcService extends DOMService {
    @NonNull
    ListenableFuture<? extends DOMRpcResult> invokeRpc(@NonNull SchemaPath type, @NonNull NormalizedNode<?, ?> input);

    @NonNull
    <T extends DOMRpcAvailabilityListener> ListenerRegistration<T> registerRpcListener(@NonNull T listener);
}
```

这里的DOMRpcService也是继承了DomService，当然这个影响好像不是很大，就是一个空接口。我们继续看看invokeRpc的实现：

这里有多个实现，我们用的是在package org.opendaylight.netconf.sal.connect.netconf.sal里边，(NetconfMessage)this.transformer.toRpcRequest(type, input) 这里就可以将输入转换为xml了。

```java
    public ListenableFuture<DOMRpcResult> invokeRpc(final SchemaPath type, final NormalizedNode<?, ?> input) {
        ListenableFuture<RpcResult<NetconfMessage>> delegateFuture = this.communicator.sendRequest((NetconfMessage)this.transformer.toRpcRequest(type, input), type.getLastComponent());
        final SettableFuture<DOMRpcResult> ret = SettableFuture.create();
        Futures.addCallback(delegateFuture, new FutureCallback<RpcResult<NetconfMessage>>() {
            public void onSuccess(final RpcResult<NetconfMessage> result) {
                try {
                    ret.set(result.isSuccessful() ? NetconfDeviceRpc.this.transformer.toRpcResult((NetconfMessage)result.getResult(), type) : new DefaultDOMRpcResult(result.getErrors()));
                } catch (Exception var3) {
                    ret.setException(new DefaultDOMRpcException("Unable to parse rpc reply. type: " + type + " input: " + input, var3));
                }

            }

            public void onFailure(final Throwable cause) {
                ret.setException(new DOMRpcImplementationNotAvailableException(cause, "Unable to invoke rpc %s", new Object[]{type}));
            }
        }, MoreExecutors.directExecutor());
        return ret;
    }
```

看到这里就知道了：第一行代码sendRequest就像设备发送了xml，然后就开始尝试捕捉这个结果··

这里的communicator是一个RemoteDeviceCommunicator\<NetconfMessage\>对象，进去看看这个方法的实现：

```java
 public ListenableFuture<RpcResult<NetconfMessage>> sendRequest(final NetconfMessage message, final QName rpc) {
        this.sessionLock.lock();

        ListenableFuture var3;
        try {
            if (this.semaphore != null && !this.semaphore.tryAcquire()) {
                LOG.warn("Limit of concurrent rpc messages was reached (limit: {}). Rpc reply message is needed. Discarding request of Netconf device with id: {}", this.concurentRpcMsgs, this.id.getName());
                int var10002 = this.concurentRpcMsgs;
                FluentFuture var7 = FluentFutures.immediateFailedFluentFuture(new NetconfDocumentedException("Limit of rpc messages was reached (Limit :" + var10002 + ") waiting for emptying the queue of Netconf device with id: " + this.id.getName()));
                return var7;
            }

            var3 = this.sendRequestWithLock(message, rpc);
        } finally {
            this.sessionLock.unlock();
        }

        return var3;
    }
```
在这里也就是对这个session锁住，然后发送信息。

```java
    private ListenableFuture<RpcResult<NetconfMessage>> sendRequestWithLock(final NetconfMessage message, final QName rpc) {
        if (LOG.isTraceEnabled()) {
            LOG.trace("{}: Sending message {}", this.id, msgToS(message));
        }

        if (this.currentSession == null) {
            LOG.warn("{}: Session is disconnected, failing RPC request {}", this.id, message);
            return FluentFutures.immediateFluentFuture(this.createSessionDownRpcResult());
        } else {
            NetconfDeviceCommunicator.Request req = new NetconfDeviceCommunicator.Request(new UncancellableFuture(true), message);
            this.requests.add(req);
            this.currentSession.sendMessage(req.request).addListener((future) -> {
                if (!future.isSuccess()) {
                    LOG.debug("{}: Failed to send request {}", new Object[]{this.id, XmlUtil.toString(req.request.getDocument()), future.cause()});
                    if (future.cause() != null) {
                        req.future.set(createErrorRpcResult(ErrorType.TRANSPORT, future.cause().getLocalizedMessage()));
                    } else {
                        req.future.set(this.createSessionDownRpcResult());
                    }

                    req.future.setException(future.cause());
                } else {
                    LOG.trace("Finished sending request {}", req.request);
                }

            });
            return req.future;
        }
    }
```
显然message就是要传给设备的信息。


更底层的可以参考[SDNLAB的这篇文章](https://www.sdnlab.com/22997.html)




