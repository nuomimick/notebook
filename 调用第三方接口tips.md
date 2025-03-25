### 如果第三方接口是幂等的，则应该添加重试机制

### 重试的间隔可以是固定的，也可以是依次增长的

### 如果第三方接口不是幂等的，应该根据异常情况添加重试机制和回查机制

**重试机制**
1. 网络原因

**回查机制**
1. 当第三方接口很耗时，而调用方http client设置了超时时间，就可能会抛出请求超时异常。实际第三方服务已经接受到请求并执行了，因此我们需要一个回查机制判断业务的执行情况。

根据上述方案提供如下框架代码

**HttpClientStableTool**
```
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.springframework.util.function.ThrowingSupplier;

import java.util.concurrent.TimeUnit;

@Data
@Slf4j
public class HttpClientStableTool {

    private RetryConfig config = new RetryConfig();

    private HttpExceptionHandler exceptionHandler = new DefaultHttpExceptionHandler();

    private LocalQueryService queryService = e -> log.info("LocalQueryService deal nothing.");

    public HttpClientStableTool() {
    }

    public HttpClientStableTool(RetryConfig config, HttpExceptionHandler exceptionHandler,
        LocalQueryService queryService) {
        if (config != null) {
            this.config = config;
        }
        if (exceptionHandler != null) {
            this.exceptionHandler = exceptionHandler;
        }
        if (queryService != null) {
            this.queryService = queryService;
        }
    }

    @Data
    public static class RetryConfig {
        private int maxRetries = 3;

        private long initialDelay = 1000; // 初始延迟（毫秒）

        private long maxDelay = 5000; // 最大延迟（毫秒）

        private double backoffFactor = 2.0;
    }

    private long randomJitter(long maxJitter) {
        return (long) (Math.random() * maxJitter);
    }

    private long calculateDelay(int retryCount) {
        long delay = (long) (config.getInitialDelay() * Math.pow(config.getBackoffFactor(), retryCount));
        delay = Math.min(delay, config.getMaxDelay());
        delay += randomJitter(500); // 添加随机因子，防止雪崩
        return delay;
    }

    private void sleep(long delay) {
        try {
            TimeUnit.MILLISECONDS.sleep(delay);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("Interrupted while sleeping", e);
        }
    }

    public <T> T run(ThrowingSupplier<T> supplier) {
        int retryCount = 0;
        while (retryCount <= config.getMaxRetries()) {
            try {
                return supplier.get();
            } catch (Exception e) {
                if (exceptionHandler.shouldRetry(e)) {
                    if (retryCount >= config.getMaxRetries()) {
                        log.error("Max retries ({}) reached. Last error: {}", config.getMaxRetries(), e.getMessage());
                        throw e;
                    } else {
                        long delay = calculateDelay(retryCount);
                        log.info("Retrying {}/{} in {}ms due to error: {}", retryCount + 1, config.getMaxRetries(),
                            delay, e.getMessage());
                        sleep(delay);
                        retryCount++;
                    }
                } else if (exceptionHandler.shouldCheck(e)) {
                    queryService.deal(e);
                    log.error("Start query transaction due to error: {}", e.getMessage());
                    throw new RuntimeException("Start query transaction, please check status later");
                } else {
                    log.error("HttpClientStableTool run failed", e);
                    throw e;
                }
            }
        }
        // never run while loop
        return null;
    }
}

```

**DefaultHttpExceptionHandler**
```
public class DefaultHttpExceptionHandler implements HttpExceptionHandler{

    @Override
    public boolean shouldRetry(Exception e) {
        return true;
    }

    @Override
    public boolean shouldCheck(Exception e) {
        return false;
    }
}
```

**HttpExceptionHandler**
```
public interface HttpExceptionHandler {

    boolean shouldRetry(Exception e);

    boolean shouldCheck(Exception e);
}
```

**LocalQueryService**
```
public interface LocalQueryService {

    void deal(Exception e);
}
```
