---
layout: post
title:  "Java多线程-关于InterruptedException的处理"
date:   2019-08-07 09:25:00 +0800
---
当线程的run方法执行方法体中最后一条语句后，并经由执行return语句返回时，或者在方法中出现了没有捕获的异常时，线程将终止。

每一个线程都有一个 boolean 标志来表示中断状态。每个线程都应该不时检测这个标志，以判断线程是否被中断。

当对一个线程调用 interrupt 方法时，线程的这个中断状态将由默认的false设置为true。但是，当在一个被阻塞的线程（调用sleep或者wait方法），
上调用 interrupt 方法时，就无法检测中断状态，阻塞调用就会被InterruptedException中断。这是产生 InterruptedException 异常的原因。

需要注意的是中断一个线程不过是为了它的注意，被中断的线程可以决定如何响应中断。

```
public class InterruptTest implements Runnable {
    @Override
    public void run() {
        while (true){
            System.out.println(Thread.currentThread().isInterrupted());
        }
    }


    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(new InterruptTest());
        t.start();
        Thread.sleep(5000);
        t.interrupt();
    }
}
```

上面这段程序前5s输出false，5s后输出true，虽然状态变了，但线程不会停。

检测interrupted状态的两种方法：

- Thread.currentThread().isInterrupted()

- Thread.interrupted(); 注意静态方法调用后中断标志会从true自动变为false。

**线程内抛出 InterruptedException 后标志也会从 true 自动变为 false**

为了防止这种中断状态自动变更的问题，InterruptedException 处理最佳实践有两种方法：

1、catch中重新调用interrupt方法使得状态变为true，方便栈上层调用者检测中断，做出响应。
```
catch(InterruptedException e){

	Thread.currentThread().interrupt();

}
```

2、throws InterruptedException