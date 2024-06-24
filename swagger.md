# swagger快速入门

阅读链接https://mp.weixin.qq.com/s/0-c0MAgtyOeKx6qzmdUG0w

1.配置依赖

springboot集成=>

  springfox-swagger2

  springfox-swagger-ui

2.配置类

访问测试http://localhost:8080/swagger-ui.html

```java
@Configuration //配置类
@EnableSwagger2// 开启Swagger2的自动配置
public class SwaggerConfig {

  // 设置要显示swagger的环境
  Profiles of = Profiles.of("dev", "test");
  // 判断当前是否处于该环境
  // 通过 enable() 接收此参数判断是否要显示
  boolean b = environment.acceptsProfiles(of);

  @Bean
  public Docket docket() {
   return new Docket(DocumentationType.SWAGGER_2)
      .groupName("hello") // 配置分组
      .apiInfo(apiInfo())
      .enable(b) //配置是否启用Swagger，如果是false，在浏览器将无法访问
      .select()// 通过.select()方法，去配置扫描接口,RequestHandlerSelectors配置如何扫描接口
      .apis(RequestHandlerSelectors.basePackage("com.kuang.swagger.controller"))
      .paths(PathSelectors.ant("/kuang/**"))// 配置如何通过path过滤,即这里只扫描请求以/kuang开头的接口
      .build();
  }

  //配置文档信息,html页面上的一些描述
  private ApiInfo apiInfo() {
    Contact contact = new Contact("联系人名字", "http://xxx.xxx.com/联系人访问链接", "联系人邮箱");
    return new ApiInfo(
            "Swagger学习", // 标题
            "学习演示如何配置Swagger", // 描述
            "v1.0", // 版本
            "http://terms.service.url/组织链接", // 组织链接
            contact, // 联系人信息
            "Apach 2.0 许可", // 许可
            "许可链接", // 许可连接
            new ArrayList<>()// 扩展
    );
  }
}

```
## RequestHandlerSelectors的方法
```
any() // 扫描所有，项目中的所有接口都会被扫描到

none() // 不扫描接口

// 通过方法上的注解扫描，如withMethodAnnotation(GetMapping.class)只扫描get请求
withMethodAnnotation(final Class<? extends Annotation> annotation)

// 通过类上的注解扫描，如.withClassAnnotation(Controller.class)只扫描有controller注解的类中的接口
withClassAnnotation(final Class<? extends Annotation> annotation)

basePackage(final String basePackage) // 根据包路径扫描接口
```
## paths()参数
```
any() // 任何请求都扫描

none() // 任何请求都不扫描

regex(final String pathRegex) // 通过正则表达式控制

ant(final String antPattern) // 通过ant()控制

```
## 多个分组
```
@Bean
public Docket docket1(){
   return new Docket(DocumentationType.SWAGGER_2).groupName("group1");
}
@Bean
public Docket docket2(){
   return new Docket(DocumentationType.SWAGGER_2).groupName("group2");
}
@Bean
public Docket docket3(){
   return new Docket(DocumentationType.SWAGGER_2).groupName("group3");
}
```
## 常用注解
| Swagger注解 | 简单说明 |
| ----------- | ------- |
| @Api(tags = "xxx模块说明") | 作用在模块类上 |
| @ApiOperation("xxx接口说明") | 作用在接口方法上 |
|@ApiModel("xxxPOJO说明")|作用在模型类上：如VO、PO|
|@ApiModelProperty(value = "xxx属性说明",hidden = true)|作用在类方法和属性上，hidden设置为true可以隐藏该属性|
|@ApiParam("xxx参数说明")|作用在参数、方法和字段上，类似@ApiModelProperty|
