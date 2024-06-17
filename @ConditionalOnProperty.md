@ConditionalOnProperty 是Spring框架中的一个条件注解，用于控制配置类或配置类中的bean是否应该被注册到Spring容器中。这个注解基于环境属性值来决定是否应用某个配置。
例子：
```
@ConditionalOnProperty(name = {"local-flag"}, havingValue = "false")
```
这个注解的作用是：只有当应用程序的环境属性（Environment Property）中存在名为local-flag的属性，并且该属性的值为false时，被该注解标记的类或方法才会生效，即对应的bean会被创建并加入到Spring的上下文中。如果local-flag属性不存在或者其值不是false，那么相关的配置就不会被应用。

这个机制常用于实现条件化的配置管理，使得应用能够在不同的部署环境（如开发、测试、生产环境）下，根据外部配置灵活地启用或禁用某些功能模块，而无需更改代码逻辑。开发者可以利用这样的注解来实现更精细的环境隔离和配置管理。