# SpringContextHolder 工具类
SpringContextHolder 通常是一个自定义的类，用来静态地访问Spring的ApplicationContext，从而获取Bean。这个类通常会有一个静态方法来返回Spring的上下文，然后通过上下文获取Bean。
```
@Component
public class SpringContextHolder implements ApplicationContextAware {
    private static ApplicationContext applicationContext;


    public static ApplicationContext getApplicationContext(){
        assertApplicationContext();
        return SpringContextHolder.applicationContext;
    }

    public static <T> T getBean(String beanName){
        assertApplicationContext();
        return (T)SpringContextHolder.applicationContext.getBean(beanName);
    }

    public static <T> T getBean(Class<T> typeName){
        assertApplicationContext();
        return applicationContext.getBean(typeName);
    }

    // 确保ApplicationContext已被注入
    private static void assertContextInjected() {
        if (context == null) {
            throw new IllegalStateException("applicaitonContext属性未注入，请在applicationContext.xml中定义SpringContextHolder");
        }
    }

    // 实现ApplicationContextAware接口方法，注入ApplicationContext
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        SpringContextHolder.applicationContext = applicationContext;
    }

}
```