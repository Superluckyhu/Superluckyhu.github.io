# JUC

## 线程池关闭

> https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/ExecutorService.html 

```java
public static void finalShutdownAndAwaitTermination(ExecutorService threadPool)
    {
        if (threadPool != null && !threadPool.isShutdown())
        {
            threadPool.shutdown();
            try
            {
                if (!threadPool.awaitTermination(120, TimeUnit.SECONDS))
                {
                    threadPool.shutdownNow();

                    if (!threadPool.awaitTermination(120, TimeUnit.SECONDS))
                    {
                        System.out.println("Pool did not terminate");
                    }
                }
            } catch (InterruptedException ie) {
                threadPool.shutdownNow();
                Thread.currentThread().interrupt();
            }
        }
    }
```

<span style = "color:blue;font-weight:bold">note:</span>

**`shutdown()`:**

- 不会再接受新任务的提交
- 在shutdown()调用之前的任务会被执行下去，待执行的任务和正在执行的任务<span style = "color:red;font-weight:bold">都不会取消，将继续执行</span>
- 如果已经shutdown()，再调用不会有其他影响

**`shutdownNow()`:**

- 不会再接收新任务的提交
- 尝试停止所有正在执行的任务（仅仅只是做尝试，成功与否取决于是否响应interruptedException，以及对其做出的反应）
- 待执行的任务会取消并返回等待任务的列表
- 方法返回时，这些等待的任务将从队列中清空
- shutdownNow()将线程池状态置为STOP，试图让线程池立刻停止，但不一定能保证立即停止，要等所有正在执行的任务（被能被interrupt中断的任务）执行完才能停止

**`awaitTermination()`:**

- 阻塞当前线程，等已提交和已执行的任务都执行完，解除阻塞
- 当等待超过设置的时间，检查线程池是否停止，如果执行完了返回true；如果执行完之前超时了，返回false并解除阻塞





















