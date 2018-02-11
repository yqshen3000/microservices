注册中心应该是整个微服务架构中最核心的组件，用于自动管理系统中的各个微服务实例。使用手动的方式管理所有的微服务实例通常是不现实的，因此spring cloud提供了eureka作为注册中心的一个解决方案。此外，还有诸如zookeeper、consul等方案，但是个人认为就作为注册中心而言，更应该注重的是ap而不是cp或者ca，因此eureka更加适合作为一个注册中心。

eureka的解决方案分为客户端和服务端两个部分，而使用spring cloud的解决方案也可以很方便的进行集成。以下是一个进行集成的简单例子，我使用的较为习惯的ide是sts，因为是免费的~~

----------
# 1.1 快速入门 #
# 1.1.1 eureka服务端 #
将eureka作为注册中心服务端，与spring cloud可以快速的进行接入。
## 开启服务 ##
开启服务需要引入如下的依赖包，完整的pom文件信息可以查看附录1
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```
加入引用包之后，就可以通过加入 *@EnableEurekaServer* 注解来开启注册中心的功能，具体的代码示例如下。通过这个特性，我们可以很方便的将他与其他的功能进行集成，例如spring cloud config服务中心。
```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServerApplication.class, args);
	}
}
```

为了快速演示服务端的例子，我们还配置了以下的最简配置信息
```
server:
  port: 8761

spring:
  application:
    name: fern-cloud-governance
    
eureka:
  client:
    service-url:
      defaultZone: http://localhost:${server.port}/eureka/
```
其中，各个配置含义如下

| 配置项 | 含义 | 
| - | :- |
|server.port| 服务端口号 |
|spring.application.name | 服务名称 | 
|eureka.client.service-url.defaultZone | 中心默认注册域的注册地址，如果只有一个数据中，可以按照这个方式进行配置，后续做详细说明 |

启动服务之后，eureka的服务端内置了一个简单的后台，按照上述的配置，后台管理地址为http://localhost:8761/ 。为了演示注册中心的功能，我们将eureka服务端注册到自身。此外为了实现高可用的方案，添加了一个eureka的客户端，将自身注册到eureka集群中。

# 1.1.2 eureka客户端 #
根据项目使用的技术不同，集成的方式也略显不同，建议对于初创的项目采用spring boot的方式。
## spring boot方式集成，服务提供者 ##
为了开启增加eureka客户端的相关能力，如果使用的是spring boot方式集成，可以通过如下的方式进行集成。首先，在pom文件中引入如下的依赖包
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```
引入框架包之后，可以通过在启动文件中增加 *@EnableDiscoveryClient* 或者 *@EnableEurekaClient* 注解来开启注册中心的客户端功能，具体代码如下
```java
@EnableDiscoveryClient
@SpringBootApplication
@RestController
public class EurekaClientProviderApplication {
	public static void main(String[] args) {
		SpringApplication.run(EurekaClientProviderApplication.class, args);
	}
	
	@GetMapping("/time/{type}")
	public String now(@PathVariable String type) {
	    if (StringUtils.hasText(type) && "UTC".equals(type.toUpperCase())) {
	        return ZonedDateTime.now().toString();
	    } else {
	        return LocalDateTime.now().toString();
	    }
	}
}
```
此外，修改配置文件信息如下
```
server:
  port: 9090

spring:
  application:
    name: eureka-client-provider

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/ #eureka server的注册地址
```
该客户端作为一个服务提供者，同时对外提供了一个获取当前时间的接口，该接口根据不同的参数提供不同的格式的日期格式。

## 普通spring mvc方式集成，服务消费者 ##

