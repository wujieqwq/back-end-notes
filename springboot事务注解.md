EnableTransactionManagement 分析
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({TransactionManagementConfigurationSelector.class})
public @interface EnableTransactionManagement {
    boolean proxyTargetClass() default false;

    AdviceMode mode() default AdviceMode.PROXY;

    int order() default Integer.MAX_VALUE;
}

```
## proxyTargetClass 属性
proxyTargetClass属性表示是否强制采用 CGLIB 的方式来创建代理对象。在 Spring 中，事务管理是通过 AOP 特性来实现的，而 AOP 是通过对 Bean 的代理来实现的，包括 JDK 动态代理和 CGLIB 代理，默认情况下 Spring 会优先采用 JDK 动态代理的方式，在无法采用 JDK 动态代理的时候，则采用 CGLIB 的方式。
默认情况下，proxyTargetClass的值为false，如果将其配置为true，则所有的代理对象都会通过 CGLIB 的方式来创建。
值得注意的是，如果这里将proxyTargetClass配置为true，那么，不仅被标记了 @Transactional 注解的类以及其标记的方法所在的类会被强制采用 CGLIB 创建代理对象，Spring 容器中所有需要创建代理的 Bean 都会被强制采用 CGLIB 的方式来创建代理。

## mode 属性
mode的值是一个 AdviceMode 类型的枚举，默认值是AdviceMode.PROXY。它表示只能通过代理模式执行拦截的逻辑，这是通常情况下会采用的值。

## order 属性
order属性表示事务相关的增强的顺序，默认是Ordered.LOWEST_PRECEDENCE，也就是最小值。

