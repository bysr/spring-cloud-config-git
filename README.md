#### 客户端配置
##### 1. 添加依赖

- 备注：spring boot依赖1.5版本，在2.0的基础上试过，出现过一系列问题，比如数据库配置无法热更新，git仓库服务刷新地址无法访问等等

```

    <!--设定spring boot版本-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Dalston.SR1</spring-cloud.version>
    </properties>
```

```
     <!--eureka 和 config依赖包-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
```
##### 2. 设置数据库热更新源文件，创建neo-config-datasource文件，内容如下
```
spring:
  datasource:
     # 配置数据源类型
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://10.10.152.69:3306/smart_cloud_liuliqiang?useUnicode=true&characterEncoding=utf-8
    username: root
    password: hipad123456
    # 初始化，最小，最大连接数

    initialSize: 3
    minidle: 3
    maxActive: 18
    # 获取数据库连接等待的超时时间
    maxWait: 60000
    # 配置多久进行一次检测，检测需要关闭的空闲连接 单位毫秒
    timeBetweenEvictionRunsMillis: 60000
    validationQuery: SELECT 1 FROM dual
    # 配置监控统计拦截的filters,去掉后，监控界面的sql无法统计
```
##### 3.配置rabbit远程文件
```
spring:
  cloud:
    bus:
      ack:
        enabled: true
  rabbitmq:
    host: 10.10.152.51
    port: 5672
    username: guest
    password: guest

```
##### 4. 配置数据库管理文件
```
package com.hipad.smartcloud.config;

import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
import org.springframework.boot.context.properties.ConfigurationProperties;

import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

/**
 * Created with IntelliJ IDEA.
 * User: wangyawen
 * Date: 2018/11/12 0012
 * Time: 下午 3:02
 * Description: No Description
 */
@RefreshScope
@Configuration// 配置数据源
public class DataSourceConfigure {

    @Bean
    @RefreshScope// 刷新配置文件
    @ConfigurationProperties(prefix = "spring.datasource") // 数据源的自动配置的前缀
    public DataSource dataSource() {
        System.out.println("======>配置数据源");
        return DataSourceBuilder.create().build();
    }


}

```
##### 5. 配置bootstrap.properties文件，改文件启动程序的时候优先执行，通过配置datasource等一些远程文件地址，可以再项目启动的时候直接加载
```
spring.cloud.config.name=neo-config
#数据库,rabbit配置
spring.cloud.config.profile=datasource,rabbit
spring.cloud.config.label=master
spring.cloud.config.discovery.enabled=true
#spring.cloud.config.discovery.serviceId=spring-cloud-config-server
spring.cloud.config.discovery.serviceId=config-server-git
eureka.client.serviceUrl.defaultZone=http://10.10.152.69:58000/eureka/,http://10.10.153.73:58000/eureka/,http://10.10.153.114:58000/eureka/
eureka.instance.lease-expiration-duration-in-seconds=30
eureka.instance.lease-renewal-interval-in-seconds=10
management.security.enabled=false
### 开启消息跟踪
spring.cloud.bus.trace.enabled=true
spring.cloud.bus.ack.enabled=true
```
