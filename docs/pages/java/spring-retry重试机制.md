### spring-retry 执行流程
![image](http://106.13.189.134:8888/group1/M00/00/00/wKgABF275OeAYqGMAAHXPKAXRTQ007.png)
> `RetryTemplate`：重试模板，是进入spring-retry框架的整体流程入口
> + `RetryCallback`：重试回调，用户包装业务流，第一次执行和产生重试执行都会调用这个callback代码
> + `RetryPolicy`：重试策略，不同策略有不同的重试方式
> + `BackOffPolicy`：两次重试之间的回避策略，一般采用超时时间机制
> + `RecoveryCallback`：当所有重试都失败后，回调该接口，提供给业务重试回复机制
> + `RetryContext`：每次重试都会将其作为参数传入RetryCallback中使用
> + `RetryListener`：监听重试行为，主要用于监控

### 重试策略
+ `NeverRetryPolicy`：只调用RetryCallback一次，不重试
+ `AlwaysRetryPolicy`：无限重试，最好不要用
+ `SimpleRetryPolicy`：重试n次，默认3，也是模板默认的策略。很常用
+ `TimeoutRetryPolicy`：在n毫秒内不断进行重试，超过这个时间后停止重试
+ `CircuitBreakerRetryPolicy`：熔断功能的重试
+ `ExceptionClassifierRetryPolicy`：可以根据不同的异常，执行不同的重试策略，很常用
+ `CompositeRetryPolicy`：将不同的策略组合起来，有悲观组合和乐观组合。悲观默认重试，有不重试的策略则不重试。乐观默认不重试，有需要重试的策略则重试

### 回避策略
+ `NoBackOffPolicy`：不回避
+ `FixedBackOffPolicy`：n毫秒退避后再进行重试
+ `UniformRandomBackOffPolicy`：随机选择一个[n,m]（如20ms，40ms)回避时间回避后，然后在重试
+ `ExponentialBackOffPolicy`：指数退避策略，休眠时间指数递增
+ `ExponentialRandomBackOffPolicy`：随机指数退避，指数中乘积会混入随机值

### 引入maven依赖
```xml
<!-- spring-retry -->
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
    <version>1.2.4.RELEASE</version>
</dependency>
```
### 添加`RetryConfig`配置类
```java
@EnableRetry //开启重试机制
@Configuration
public class RetryConfig {

    @Bean
    public RetryTemplate retryTemplate() {
        RetryTemplate template = new RetryTemplate();
        // 设置重试策略，主要设置重试次数
        SimpleRetryPolicy policy = new SimpleRetryPolicy(5, Collections.singletonMap(Exception.class, true));
        // 设置重试回退操作策略，主要设置重试间隔时间
        FixedBackOffPolicy fixedBackOffPolicy = new FixedBackOffPolicy();
        fixedBackOffPolicy.setBackOffPeriod(1000);
        template.setRetryPolicy(policy);
        template.setBackOffPolicy(fixedBackOffPolicy);
        return template;
    }
}
```
### 测试代码
```java
// 定义一个class，提供abc()方法并抛出异常
public class ABC {
    public String abc() throws NullPointerException {
        System.out.println("abc 正在执行...");
        String a = null;
        a.equals("abc");
        return "abc";
    }
}

// retryTemplate 具体使用
try {
    String result = retryTemplate.execute((RetryCallback<String, Exception>) ctx -> new ABC().abc(), ex -> {
        System.out.println("重试次数：" + ex.getRetryCount());
        return ex.getLastThrowable().toString();
    });
    System.out.println("程序执行结果为:" + result);
} catch (Throwable throwable) {
    throwable.printStackTrace();
}
```

