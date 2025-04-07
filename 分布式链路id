1. 生成一个uuid作为traceId
在网关服务生成traceId并添加到request的header中

2. 通过过滤器拦截请求获取traceId并设置到MDC中，并在日志打印中使用%X{traceId}
通过aop或者其他过滤器、拦截器获取request header中的traceId
设置到MDC中

```
public class TraceIdFilter extends OncePerRequestFilter {

    public static final String TRACE_ID = "traceId";

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        try {
            String traceId = request.getHeader(TRACE_ID);
            if (StringUtils.isBlank(traceId)) {
                traceId = UUID.randomUUID().toString().replace("-", "");
            }
            MDC.put(TRACE_ID, traceId);
            filterChain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}
```

3. 多线程中使用和线程池中使用
```
/**
 * 线程池配置
 */
@Configuration
@EnableAsync
@Slf4j
public class ThreadPoolConfig {

    @Bean(value = "threadPoolExecutor")
    public ThreadPoolTaskExecutor threadPoolExecutor() {
        log.info("start threadPoolExecutor");
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor() {
            /**
             * 所有线程都会委托给这个execute方法，在这个方法中我们把父线程的MDC内容赋值给子线程
             * https://logback.qos.ch/manual/mdc.html#managedThreads
             *
             * @param runnable runnable
             */
            @Override
            public void execute(Runnable runnable) {
                // 获取父线程MDC中的内容，必须在run方法之前，否则等异步线程执行的时候有可能MDC里面的值已经被清空了，这个时候就会返回null
                Map<String, String> context = MDC.getCopyOfContextMap();
                super.execute(() -> {
                    // 将父线程的MDC内容传给子线程
                    if (context != null) {
                        MDC.setContextMap(context);
                    }
                    try {
                        // 执行异步操作
                        runnable.run();
                    } finally {
                        // 清空MDC内容
                        MDC.clear();
                    }
                });
            }

            @Override
            public <T> Future<T> submit(Callable<T> task) {
                // 获取父线程MDC中的内容，必须在run方法之前，否则等异步线程执行的时候有可能MDC里面的值已经被清空了，这个时候就会返回null
                Map<String, String> context = MDC.getCopyOfContextMap();
                return super.submit(() -> {
                    // 将父线程的MDC内容传给子线程
                    if (context != null) {
                        MDC.setContextMap(context);
                    }
                    try {
                        // 执行异步操作
                        return task.call();
                    } finally {
                        // 清空MDC内容
                        MDC.clear();
                    }
                });
            }
        };
        executor.setCorePoolSize(15);
        // 配置最大线程数
        executor.setMaxPoolSize(100);
        // 空线程回收时间15s
        executor.setKeepAliveSeconds(15);
        Executors.defaultThreadFactory();
        // 配置队列大小
        executor.setQueueCapacity(3000);
        // 配置线程池中的线程的名称前缀
        executor.setThreadNamePrefix("async-order-service-");
        // rejection-policy：当pool已经达到max size的时候，如何处理新任务
        // CALLER_RUNS：不在新线程中执行任务，而是由调用者所在的线程来执行
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        // 执行初始化
        executor.initialize();
        return executor;
    }
}
```

