## 线程池中 ThreadLocal 的处理
线程池满后，默认拒绝策略会将任务交给当前线程执行，如果当前线程已将ThreadLocal清除，会出现获取不到的异常，解决方案是在拒绝策略中重新设置 ThreadLocal:
```java
threadPoolTaskExecutor.setRejectedExecutionHandler((r, executor) -> {
    if (!executor.isShutdown()) {
        // 获取 ThreadLocal 中的数据
        Object obj = ThreadLocalContext.get();
        try {
            r.run();
        } finally {
            ThreadLocalContext.set(obj);
        }
    }
});
```