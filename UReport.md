# springboot中UReport2

## 一、快速开始

1. 依赖配置

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>mysql</groupId>
   <artifactId>mysql-connector-java</artifactId>
   <scope>runtime</scope>
</dependency>
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
   <groupId>com.bstek.ureport</groupId>
   <artifactId>ureport2-console</artifactId>
   <version>2.2.9</version>
</dependency>
<!--连接池-->
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid-spring-boot-starter</artifactId>
   <version>1.2.8</version>
</dependency>
```

2. 在resources目录下创建context.properties文件

```properties
# 用于定义UReport2中提供的默认基于文件系统的报表存储目录
ureport.fileStoreDir=F:/ureportfiles
```

3. config类

根据实际情况拆开配置

```java
@Configuration
//导入ureport-console-context.xml文件
@ImportResource("classpath:ureport-console-context.xml")
@Slf4j
public class ReportConfig implements BuildinDatasource {
    //添加 report 的servlet
    @Bean
    public ServletRegistrationBean<Servlet> ureport2Servlet() {
        return new ServletRegistrationBean<>(new UReportServlet(), "/ureport/*");
    }
    //这一步省略了创建配置文件
    @Bean
    public UReportPropertyPlaceholderConfigurer UReportPropertyPlaceholderConfigurer(){
        UReportPropertyPlaceholderConfigurer propertyConfigurer=new UReportPropertyPlaceholderConfigurer();
        propertyConfigurer.setIgnoreUnresolvablePlaceholders(true);
        ClassPathResource pathResource=new ClassPathResource("context.properties");
        propertyConfigurer.setLocation(pathResource);
        return propertyConfigurer;
    }


    //创建数据源，应该单独在一个配置类中，这里就写在同一个配置类中
    @Primary
    @Bean
    public DataSource businessDataSource(){
        DruidDataSource dataSource=new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/demo?useSSL=false&useUnicode=true&characterEncoding=UTF-8");
        dataSource.setUsername("root");
        dataSource.setPassword("root");
        return dataSource;
    }

    /**
     * 数据源名称
     **/
    @Override
    public String name() {
        return "ReportSource";
    }

    /**
     * 获取连接
     **/
    @Override
    public Connection getConnection() {
        try {
            return businessDataSource().getConnection();
        } catch (SQLException e) {
            log.error("Ureport 数据源 获取连接失败！");
            e.printStackTrace();
        }
        return null;
    }
}
```

4. 访问 http://localhost:****/ureport/designer

后续官方教程https://www.w3cschool.cn/ureport/ureport-vpna2h8m.html