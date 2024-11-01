# 接口防抖

## 1.自定义注解+redis+拦截器

```java
// 1. 创建自定义注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface NoRepeatSubmit {
    long value() default 3000; // 防重复时间（毫秒）
}

// 2. 实现拦截器
@Component
public class NoRepeatSubmitInterceptor implements HandlerInterceptor {
    private RedisTemplate redisTemplate;
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        if (handler instanceof HandlerMethod) {
            HandlerMethod handlerMethod = (HandlerMethod) handler;
            NoRepeatSubmit noRepeatSubmit = handlerMethod.getMethodAnnotation(NoRepeatSubmit.class);
            
            if (noRepeatSubmit != null) {
                String key = generateKey(request); // 生成唯一标识
                
                if (redisTemplate.hasKey(key)) {
                    throw new RuntimeException("请勿重复提交");
                }
                
                redisTemplate.opsForValue().set(key, "", noRepeatSubmit.value(), TimeUnit.MILLISECONDS);
            }
        }
        return true;
    }
    
    private String generateKey(HttpServletRequest request) {
        // 可以组合用户ID、请求URL、请求参数等生成唯一标识
        return "repeat_submit_key:" + request.getRequestURI() + "_" + request.getSession().getId();
    }
}

// 3. 注册拦截器
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    private NoRepeatSubmitInterceptor myInterceptor

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //创建InterceptorRegistration对象并将自定义拦截器传入;
        InterceptorRegistration interceptorRegistration = registry.addInterceptor(myInterceptor);
        //addPathPatterns方法（指定拦截路径，往往使用 "/**"）
        interceptorRegistration.addPathPatterns("/**");
        //Bean加载的时候有先后顺序 默认也是0 和@Order(0) 一个作用
        interceptorRegistration.order(0);
        //excludePathPatterns方法（指定排除拦截路径，用于登录等部分开放接口）
        interceptorRegistration.excludePathPatterns("/mhh/Interceptor/excludeInterceptorTest");
    }
}
```

## 2.自定义注解+redis+切面

### 1.幂等注解

```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Idempotent {

    /**
     * 幂等的超时时间，默认为 1 秒
     *
     * 注意，如果执行时间超过它，请求还是会进来
     */
    int timeout() default 1;

    /**
     * 时间单位，默认为 SECONDS 秒
     */
    TimeUnit timeUnit() default TimeUnit.SECONDS;

    /**
     * redis锁前缀
     * @return
     */
    String keyPrefix() default "idempotent";

    /**
     * key分隔符
     * @return
     */
    String delimiter() default "|";

    /**
     * 提示信息，正在执行中的提示
     */
    String message() default "重复请求，请稍后重试";
}
```

    @Idempotent 注解定义了几个基础的属性，redis锁时间、redis锁时间单位、redis锁前缀、key分隔符、提示信息。

    key分隔符是用来将多个参数合并在一起的，比如name是测试项目，contractNo是001，那么完整的key就是"测试项目|001"，最后再加上redis锁前缀，就组成了一个唯一key。

### 2.参数可选->创建参数注解

```java

/**
 * @description 加上这个注解可以将参数设置为key
 */
@Target({ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
public @interface RequestKeyParam {

}

```

### 3.key生成类

```java
public class RequestKeyGenerator {

    /**
     * 获取LockKey
     *
     * @param joinPoint 切入点
     * @return
     */
    public static String getLockKey(ProceedingJoinPoint joinPoint) {
        //获取连接点的方法签名对象
        MethodSignature methodSignature = (MethodSignature)joinPoint.getSignature();
        //Method对象
        Method method = methodSignature.getMethod();
        //获取Method对象上的注解对象
        Idempotent idempotent = method.getAnnotation(Idempotent.class);
        //获取方法参数
        final Object[] args = joinPoint.getArgs();
        //获取Method对象上所有的注解
        final Parameter[] parameters = method.getParameters();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < parameters.length; i++) {
            final RequestKeyParam keyParam = parameters[i].getAnnotation(RequestKeyParam.class);
            //如果属性不是RequestKeyParam注解，则不处理
            if (keyParam == null) {
                continue;
            }
            //如果属性是RequestKeyParam注解，则拼接 连接符 "& + RequestKeyParam"
            sb.append(idempotent.delimiter()).append(args[i]);
        }
        //如果方法上没有加RequestKeyParam注解
        if (StringUtils.isEmpty(sb.toString())) {
            //获取方法上的多个注解（为什么是两层数组：因为第二层数组是只有一个元素的数组）
            final Annotation[][] parameterAnnotations = method.getParameterAnnotations();
            //循环注解
            for (int i = 0; i < parameterAnnotations.length; i++) {
                final Object object = args[i];
                //获取注解类中所有的属性字段
                final Field[] fields = object.getClass().getDeclaredFields();
                for (Field field : fields) {
                    //判断字段上是否有RequestKeyParam注解
                    final RequestKeyParam annotation = field.getAnnotation(RequestKeyParam.class);
                    //如果没有，跳过
                    if (annotation == null) {
                        continue;
                    }
                    //如果有，设置Accessible为true（为true时可以使用反射访问私有变量，否则不能访问私有变量）
                    field.setAccessible(true);
                    //如果属性是RequestKeyParam注解，则拼接 连接符" & + RequestKeyParam"
                    sb.append(idempotent.delimiter()).append(ReflectionUtils.getField(field, object));
                }
            }
        }
        //返回指定前缀的key
        return idempotent.keyPrefix() + sb;
    }
}
```

### 4.切面类

```java
@Aspect
@Configuration
@Order(2)
@Slf4j
public class IdempotentAspect {

    private RedissonClient redissonClient;

    @Autowired
    public IdempotentAspect(RedissonClient redissonClient) {
        this.redissonClient = redissonClient;
    }

    @Around("execution(public * * (..)) && @annotation(Idempotent)")
    public Object interceptor(ProceedingJoinPoint joinPoint) {
        MethodSignature methodSignature = (MethodSignature)joinPoint.getSignature();
        Method method = methodSignature.getMethod();
        Idempotent idempotent = method.getAnnotation(Idempotent.class);
        if (StringUtils.isEmpty(idempotent.keyPrefix())) {
            throw new CommonExcept("重复提交前缀不能为空");
        }
        //获取自定义key
        final String lockKey = RequestKeyGenerator.getLockKey(joinPoint);
        // 使用Redisson分布式锁的方式判断是否重复提交
        RLock lock = redissonClient.getLock(lockKey);
        boolean isLocked = false;
        try {
            //尝试抢占锁
            isLocked = lock.tryLock();
            //没有拿到锁说明已经有了请求了
            if (!isLocked) {
                throw new CommonExcept(idempotent.message());
            }
            //拿到锁后设置过期时间
            lock.lock(idempotent.timeout(), idempotent.timeUnit());
            try {
                return joinPoint.proceed();
            } catch (Throwable throwable) {
                log.info("系统异常，", throwable);
                throw new CommonExcept("系统异常，" + throwable.getMessage());
            }
        } catch (Exception e) {
            throw new CommonExcept(e.getMessage());
        } finally {
            //释放锁
            if (isLocked && lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
}
```
