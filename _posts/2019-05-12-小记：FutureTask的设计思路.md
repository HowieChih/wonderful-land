---
layout: post
title:  "小记：FutureTask的设计思想"
date:   2019-05-12 00:00:00 +0800
---

1. callable计算完的结果，需要future实例来存
2. 计算和存储结果的过程如果要移交给线程异步执行的话，必须包装成一个runnable

```
define func(callable c, future f) {
	var result = c.call()
	f.sotre(result)
} as runnable
```

futureTask就是这么个玩意，把计算过程和结果存储打包一起后变成runnable，交给异步线程处理

```java
FutureTask<String> feature = new FutureTask<>(() -> "bool");
new Thread(feature).start();
String result = feature.get();
```

再往后看，executorService.submit(callable task)，底层也是将 callable task 封装成了 futureTask，最后将futureTask当作runnable给了executorService.execute。

```java
RunnableFuture<T> ftask = newTaskFor(task);
	--> return new FutureTask<T>(callable);
execute(ftask);
return ftask;
```



