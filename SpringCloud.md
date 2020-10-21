# SpringCloud使用



# 1.服务中心



## 1.1使用eureka服务端

> ## eureka服务治理体系

1. 引入pom依赖

   ```xml
<dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
   </dependency>
   ```
   
2. 配置yum文件

   

   ```yml
   server:
     port: 7001  #端口号
   
   spring: 
     application: 
       name: eureka-server7001
   
   eureka: 
     # 每个EurekaServer 都包含一个EurekaClient，用于请求其他节同步 
     client: 
        registerWithEureka: true #fales表示不向注册中心注册自己，（单机方式）
        fetchRegistry: false #false表示自己就是注册中心，我的职责就是维护服务实例，并不需要去检查服务
        service-url: #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
          defaultZone: http://localhost:7002/eureka/
     instance:
   #    hostname: localhost7002  #Eureka服务端的实例名称 
       prefer-ip-address: true #是否将自己IP地址注册到Eureka
       ip-address: 127.0.0.1
       # 指定实例名称，默认：${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}}  
       instance-id: ${spring.application.name}:${server.port}
     server:
       enable-self-preservation: true #启用自我保护模式
       eviction-interval-timer-in-ms: 6000  #如果客户端不发送心跳多久剔除，单位为毫秒
       
   ```

3. 启动类添加注解@EnableEurekaServer

   

   ```java
   @SpringBootApplication
   @EnableEurekaServer 		//表示该类为eureka服务端
   public class Springcloud01EurekaServer7001Application {
   	public static void main(String[] args) {
   		SpringApplication.run(Springcloud01EurekaServer7001Application.class, args);
   	}
   
   }
   ```

4. 访问 http://localhost:7001会进入管理eureka服务后台



## 1.2 使用eureka客户端

1. 导入pom依赖

2. ```xml
   <!-- 引入eureka客户端-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   <!-- 包含了sleuth + zipkin -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-zipkin</artifactId>
   </dependency>
   <!-- 引入lombok 可以使用@Data 注解 -->
   <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
       <scope>provided</scope>
   </dependency>
   <!-- 引入druid数据源 -->
   <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>druid</artifactId>
       <version>1.1.22</version>
   </dependency>
   <!-- druid使用log4j记录日志，切换为slf4j -->
   <dependency>
       <groupId>org.slf4j</groupId>
       <artifactId>log4j-over-slf4j</artifactId>
   </dependency>
   <!-- 启用热部署 -->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-devtools</artifactId>
       <scope>runtime</scope>
       <optional>true</optional>
   </dependency>
   <!--引入mysql -->
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <scope>runtime</scope>
   </dependency>
   ```

3. yml文件设置

   > 将自己注册进defaultZone：指定的服务中心

4. ```yml
   eureka: 
     # 每个EurekaServer 都包含一个EurekaClient，用于请求其他节同步 
     client: 
     #表示是否将自己注册EurekaServer
        registerWithEureka: true 
        #是否从EurekaServer抓取已有的注册信息，默认为true，单节点我所谓，集群必须设置为true才能配合ribbn使用负载均衡
        fetchRegistry: true 
        service-url: 
        #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址，如果服务器是集群就用    ， 号分割地址
          defaultZone: http://localhost:7001/eureka/,http://localhost:7002/eureka/
        eureka-server-connect-timeout-seconds: 60
        eureka-server-read-timeout-seconds: 60
        
     instance: 
       prefer-ip-address: true #是否将自己IP地址注册到Eureka
       ip-address: 127.0.0.1
       # 指定实例名称，默认：${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}}  
       instance-id: ${spring.application.name}:${server.port}
       lease-expiration-duration-in-seconds: 30 #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒，默认90秒，超时将剔除服务
       lease-renewal-interval-in-seconds: 10 #Eureka客户端向服务端发送心跳的时间间隔单位为秒默认30秒
   #    leaserenewalintervalinseconds: 10 #心跳时间
   ```

5. 启动类添加注解

6. ```java
   @EnableEurekaClient
   @EnableDiscoveryClient
   ```

7. 实体类 Payment + CommonResult(此类是json格式的封装体)

8. ```java
   @Data	//自动生成get，set，equals，toString		//必须导入lombok依赖
   @AllArgsConstructor	//自动生成所有有参构造
   @NoArgsConstructor
   public class Payment implements Serializable {
   	private Integer id;
   	
   	private String serial;
   
   }
   
   ```

   ```java
   /**
    * 项目如果前后端分离开发，传给前端一个json
    * 那么此类是json格式的封装体
    */
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class CommonResult<T>{
   	private Integer code;
   	private String message;
   	private T data;//传到前端的实体类数据
   	
   	public CommonResult(Integer code,String message) {
   		this.code=code;
   		this.message=message;
   	}
   }
   
   ```

9. dao层写入接口

   ```java
   @Mapper
   public interface PaymentMapper {
   	@Select(value = "select * from payment")
   	public List<Payment> findAll();
   
   	@Select(value = "select * from payment where id =#{id}")
   	public Payment findPaymentById(Integer id);
   
   }
   ```

   

10. controller层

    ```java
    @RestController
    @Slf4j
    public class PaymentController {
    	
    	@Autowired
    	PaymentMapper paymentmapper;
    	
    	
    	@GetMapping("/findPaymentAll")
    	public CommonResult<List<Payment>> findPaymentAll(){
    		 List<Payment> findAll = paymentmapper.findAll();
    		 if(findAll != null) {
    			 log.warn("查询所有成功");
    			 return new CommonResult<List<Payment>>(200,"查询所有",findAll);
    		 }else {
    			 log.error("查询所有失败");
    			 return new CommonResult<List<Payment>>(404,"查询失败",null);
    		 }
    	}
    	
    	@GetMapping("/findPaymentById/{id}")
    	public CommonResult<Payment> findPaymentById(@PathVariable("id") Integer id){
    		Payment payment = paymentmapper.findPaymentById(id);
    		if(payment != null) {
    			log.warn("根据id查询成功");
    			 return new CommonResult<Payment>(200,"根据id查询",payment);
    		 }else {
    			 log.error("根据id查询失败");
    			 return new CommonResult<Payment>(404,"查询失败",null);
    		 }
    	}
    }
    ```



## 1.3ribbon负载均衡模式

实现IRule接口的类，实现修改模式（该接口的jar，在eureka下）

1. com.netfix.loadbalancer.RoundRobinRule轮询
2. com.netfix.loadbalancer.RandomRule随机
3. com.netfix.loadbalancer.RetryRule 根据轮询，若失败进行重试，获取可用服务
4. WeigdthResponseTimeRule 轮询的扩展，响应速度越快权重越高,越容易被选择
5. BestAvailableRule  过滤掉有过故障记录的，然后访问并发量小的服务
6. AvailabliltyFilteringRule  过滤掉现有故障的，在选择并发量小的服务
7. ZoneAvoidanceRule 符合判断server所在区域的性能及其可用性



> 使用

注意：创建的配置类位置不能被启动类扫描到，否则所有客户端均使用该配置

```Java
package com.rule;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
@Configuration
public class RandomConfig {
	@Bean
	public IRule randomRule() {//定义随机负载均衡
		return new RandomRule();
	}
}
```

启动类指定配置类，以及服务名（大写的） Ribbon+RestTemplate 

```java
@RibbonClient(name = "CLOUD-PANMENT-SERVICE", configuration = RandomConfig.class)
```



## 1.4 zookeeper的安装

​	https://zookeeper.apache.org/releases.html#download下载链接

1. > ### zookeeper依赖于jdk，需要安装jdk

   ```kotlin
   rpm -ivh jdk
   /usr/java/jdk1.8.0_261-amd64  #jdk默认安装路径
   ```

   配置环境变量

   ```
   vim /etc/profile
   ```

   文件尾部添加信息

   ```
   export JAVA_HOME=/usr/java/jdk1.8.0_261-amd64
   export CLASSPATH=$:CLASSPATH:$JAVA_HOME/lib/
   export PATH=$PATH:$JAVA_HOME/bin
   ```

   刷新环境配置

   ```
   source /etc/profile
   ```

   

2. > ### 安装zookeeper

   ```
   tar -zxvf zookeeper-3.4.9			#解压文件
   ```

   zookeeper的conf目录下zoo_sample.cfg文件复制为zoo.cfg

   ```
   cp zoo_sample.cfg zoo.cfg
   ```

   ​	

   zoo.cfg文件发现

   ​		zookeeper默认端口

   ​				clientProt=2181

   ​		zookeeper数据位置

   ​				dataDir=/tmp/zookeeper

3. > ## 启动zookeeper

   ```
   #启动zookeeper
   ./zkServer.sh start			
   
   #停止zookeeper
   ./zkServer.sh stop
   
   #查看启动状态
   ./zkServer.sh status
   
   
   #进入客户端可查看服务
   ./zkCli.sh 
   
   #查看服务
   ls /
   
   #获得服务
   get /server/master
   ```



## 4.搭建dubbo项目

1. 导入pom依赖

   dubbo、zookeeper、zkclient(zookeeper客户端)





## 1.5 创建zookeeper项目

1. > ### pom引入zookeeper

   

   ```xml
   		<!-- SpringBoot整合Zookeeper客户端 -->
   		<dependency>
   			<groupId>org.springframework.cloud</groupId>
   			<artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
   			<version>2.2.3.RELEASE</version>
   			<!--屏蔽默认自带的版本，否则会报错，然后导入自己的zookeeper3.4.9 版本 -->
   			<exclusions>
   				<exclusion>
   					<groupId>org.apache.zookeeper</groupId>
   					<artifactId>zookeeper</artifactId>
   				</exclusion>
   			</exclusions>
   		</dependency>
   <!-- 引入zookeeper -->
   		<dependency>
   		     <groupId>org.apache.zookeeper</groupId>
   			  <artifactId>zookeeper</artifactId>
   			  <version>3.4.9</version>
   		</dependency>
   ```

2. > ### yml配置类指定服务中心

   

   ```yml
   server:
     port: 80
   
   #注册zookeeper到服务中心（192.168.0.201）
   spring:
     application:
       name: consumer-srver
     cloud:
       zookeeper:
         connect-string: 192.168.0.201
   ```

   

3. > ### 启动类添加注解

   

   ```java

   @EnableDiscoveryClient
   public class ZookeeperServerApplication {
   	public static void main(String[] args) {
   		SpringApplication.run(ZookeeperServerApplication.class, args);
   	}
   }
   ```

4. > ### 创建获得Http连接的配置类

   

   ```java
   @Configuration
   public class ZookeeperConfig {
   	@Bean//生成一个可以访问HTTP地址的对象
   	@LoadBalanced //实现负载均衡
   	public RestTemplate restTemplate() {
   		return new RestTemplate();
   	}
   }
   ```



## 1.6 consul使用

> ### 下载完成,双击运行，在安装目录下进入cmd；

```
查看版本号
consul --version

进入开发者模式
consul agent -dev

浏览器访问consul
http://localhost:8500
```

​	

### 1.6.1 consul+ResTemple实现

1. > ### pom

   ```xml
   <!-- Consul依赖 -->	
   		<dependency>
   			<groupId>org.springframework.cloud</groupId>
   			<artifactId>spring-cloud-starter-consul-discovery</artifactId>
   		</dependency>
   <!-- 健康监控 -->
   		<dependency>
   			<groupId>org.springframework.boot</groupId>
   			<artifactId>spring-boot-starter-actuator</artifactId>
   		</dependency>
   ```

   

2. > ### yum文件

   ```yml
   server:
     port: 80
   
   #注册consul到服务中心（localhost）
   spring:
     application:
       name: consumer-client
     cloud:
       consul:
         host: localhost
         port: 8500 
         discovery:
           service-name: ${spring.application.name}
   ```

3. > ### 创建Http远程访问类

   ```java
   @Configuration
   public class ConsulConfig {
   	@Bean//生成一个可以访问HTTP地址的对象
   	@LoadBalanced
   	public RestTemplate restTemplate() {
   		return new RestTemplate();
   	}
   }
   ```

4. > ### 启动类开启注册中心@EnableDiscoveryClient

5. > ### controller类注入远程访问类对象

   ```java
   @RestController
   public class ConsulController {
   	static final String Server_Url="http://producer-server";
   	
   	@Autowired
   	RestTemplate restTemplate;
   	
   	@RequestMapping("/consumer/conslu")
   	public String consluTest() {
   		return restTemplate.getForObject(Server_Url+"/conslu", String.class);
   	}
   
   }
   ```



### 1.6.2 conslu+feign的实现

1. > #### 生产者于消费者引入一样pom依赖

   ```xml
   <!--consul注册中心-->
   <dependency>
   	<groupId>org.springframework.cloud</groupId>
   	<artifactId>spring-cloud-starter-consul-discovery</artifactId>
   </dependency>
   <!--健康管理-->
   <dependency>
   	<groupId>org.springframework.boot</groupId>
   	<artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   <!--feign依赖-->
   <dependency>
   	<groupId>org.springframework.cloud</groupId>
   	<artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   
   ```

2. > #### yml文件的配置

   生产者于消费者改为端口不同，服务名不同即可

   ```yml
   server:
     port: 8888
     
   spring:
     application:
       name: consumer-client #注册服务名
     cloud:
       consul:
         port: 8500    #注册中心端口
         host: 127.0.0.1 #注册中心ip
         discovery:
           service-name: ${spring.application.name}
   ```

3. > ### 生产者创建controller

   ```java
   package com.xcw.controller;
   
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   public class ConsulController {
   	
   	@Value("${server.port}")
   	String port;
   	
   	@RequestMapping("/consul")
   	public String consulTest() {
   		return "conslu服务中心" + port;
   	}
   
   }
   
   ```

4. > ### 生产者启动类添加注解@EnableDiscoveryClient，将自己注册到注册中心

5. > ### 消费者创建service接口，添加@FeignClient注解，调用指定服务名

   ```java
   package com.xcw.service;
   
   import org.springframework.cloud.openfeign.FeignClient;
   import org.springframework.web.bind.annotation.RequestMapping;
   
   @FeignClient(name = "producer-server")//调用的服务名
   public interface ConsulService {
   	
   	@RequestMapping("/consul")
   	public String consulTest();
   }
   ```

6. > ### 消费者创建controller，调用service接口；

   ```java
   package com.xcw.controller;
   
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   import com.xcw.service.ConsulService;
   
   @RestController
   public class ConsulController {
   	@Autowired//根据类型注入
   	ConsulService consulService;
   
   	@RequestMapping("/consul")
   	public String consulTest() {
   		return consulService.consulTest();
   	};
   
   }
   
   ```

7. > ### 消费者启动类添加注解

   ```java
   @EnableDiscoveryClient					//让服务中心发现，该微服务
   @EnableFeignClients						//告诉框架扫描所有使用注解@FeignClient定义的feign客户端
   ```

   





# 2.服务调用

## 2.1 open Fegin使用

### 2.1.1 创建Fegin 项目

> #### open fegin 代替了RestTemplate 远程访问类



1. > ### pom

   ```xml
   <!-- openfeign依赖 -->	
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   <!-- eureka依赖 -->	
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   <!-- lombok依赖,注解实现get，set -->	
   <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
       <optional>true</optional>
   </dependency>
   ```

2. > ### yml

   ```yml
   server:
     port: 80
     
   
   spring: 
     application:
       name: cloud-openfeign-order-service #服务名称
   
   eureka: 
     # 每个EurekaServer 都包含一个EurekaClient，用于请求其他节同步 
     client: 
     #表示是否将自己注册EurekaServer
        registerWithEureka: true 
        #是否从EurekaServer抓取已有的注册信息，默认为true，单节点无所谓，集群必须设置为true才能配合ribbn使用负载均衡
        fetchRegistry: true 
        service-url: 
        #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址，如果服务器是集群就用    ， 号分割地址
          defaultZone: http://localhost:7001/eureka/,http://localhost:7002/eureka/
        eureka-server-connect-timeout-seconds: 60
        eureka-server-read-timeout-seconds: 60
        
     instance: 
       prefer-ip-address: true #是否将自己IP地址注册到Eureka
       ip-address: 127.0.0.1
       # 指定实例名称，默认：${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}}  
       instance-id: ${spring.application.name}:${server.port}
       lease-expiration-duration-in-seconds: 30 #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒，默认90秒，超时将剔除服务
       lease-renewal-interval-in-seconds: 10 #Eureka客户端向服务端发送心跳的时间间隔单位为秒默认30秒
   #    leaserenewalintervalinseconds: 10 #心跳时间
   
   
   #设置feign超时时间，openFeign默认支持ribbon
   ribbon:
   #指的是建立连接所用的时间，适用于网络正常的情况下，两端连接所用的时间
     ReadTimeout: 5000
     #指的是建立连接后从服务器读取到可用资源所用的时间
     ConnectTimeout: 5000
   ```

3. > ### 创建Service类，远程调用指定服务的方发

   ```java
   package com.xcw.service;
   
   import org.springframework.cloud.openfeign.FeignClient;
   import org.springframework.stereotype.Component;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   
   import com.xcw.entity.CommonResult;
   import com.xcw.entity.Payment;
   
   @Component
   @FeignClient(value = "cloud-panment-service") // 服务名
   public interface FeginService {
       
   	//查询所有
   	@GetMapping("/findPaymentAll")
   	public CommonResult findPaymentAll();
   	//根据id查询
   	@GetMapping("/findPaymentById/{id}")
   	public CommonResult<Payment> findPaymentById(@PathVariable("id") Integer id);
   
   }
   
   ```

4. > ### controller调用service的方法

   ```java
   package com.xcw.controller;
   
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RestController;
   
   import com.xcw.entity.CommonResult;
   import com.xcw.entity.Payment;
   import com.xcw.service.FeginService;
   
   @RestController
   public class PaymentController {
   	@Autowired
   	FeginService feginService;
   	
       //查询所有
   	@GetMapping("/consumer/findPaymentAll")
   	public CommonResult findPaymentAll() {
   		return feginService.findPaymentAll();
   	}
   	
       //根据id查询
   	@GetMapping("/consumer/findPaymentById/{id}")
   	public CommonResult<Payment> findPaymentById(@PathVariable("id") Integer id) {
   		return feginService.findPaymentById(id);
   	}
   
   }
   
   ```



### 2.1.2 Fegin 的延迟等待

Fegin的默认等待时间为1s，我们设置为5s；

在yml中添加：

```yml
#设置feign超时时间，openFeign默认支持ribbon
ribbon:
#指的是建立连接所用的时间，适用于网络正常的情况下，两端连接所用的时间
  ReadTimeout: 5000
  #指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000
```





### 2.1.3 Fegin 的远程日志级别

> 1. none：默认，不显示任何信息	
> 2. BASIC：仅记录请求方法，URL，响应状态码及时间
> 3. HEADERS：除了BASIC外，还有请求和响应的头信息
> 4. Full：除了HEADERS外，还有请求的正文和元数据



> ### 如何使用：只需要设置日志级别，创建配置类.

​	

1. > #### yml文件可以设置指定包下日志输出级别

   ```yml
   logging:
     level: 
       #设置所在日志级别
       com.xcw.service: debug 
   ```

2. > #### 创建配置类，设置远程日志级别;      注意Logger类，为Feign下的类；

   ```java
   package com.xcw.config;
   
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   
   import feign.Logger;
   
   @Configuration
   public class LoggingConfig {
   	@Bean//设置日志级别，返回响应头，元数据
   	public Logger.Level feignLoggerLevel() {
   		return Logger.Level.FULL;
   	}
   }
   ```

   



# 3.服务降级

## 3.1 测压工具JMeter

1. https://jmeter.apache.org/download_jmeter.cgi 下载安装zip，解压就可用

2. 配置JMETER_HOME环境变量：安装的目录

3. path添加：

   ```java
   %JMETER_HOME%\lib\ext\ApacheJMeter_core.jar
   
   %JMETER_HOME%\lib\jorphan.jar;
   ```

4. 点击安装目录下bin/jmeter.bat启动测压工具





## 3.2 Hystrix使用降级



### 生产者provider

1. > #### 生产者provider的pom引入依赖：eureka客户端,feign 调用，hystrix 和 hystrix-dashboard

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
   </dependency>
   ```

2. > #### 生产者yml文件

   ```yml
   server:
     port: 8005
     
   spring:
     application:
       name: coloud-hystrix-service #注册服务名
       
   eureka:
     client:
       fetch-registry: true #是否从EurekaServer中抓取信息，默认true，集群下必须为true配合ribbon
       register-with-eureka: true  #将自己注册进服务中心
       service-url:
         defaultZone: http://localhost:7002/eureka/,http://localhost:7001/eureka/
         
       
   ```

3. > #### 生产者业务类service

   ```java
   package com.xcw.service;
   
   import org.springframework.stereotype.Service;
   
   import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
   import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
   
   @Service
   public class PaymentService {
   
   	// 正常访问方法
   	public String paymentInfo() {
   		return Thread.currentThread().getName() + "--hystrix";
   	}
   
   	// 延迟三秒
   	@HystrixCommand(fallbackMethod = "paymentTimeout2", commandProperties = {
   			@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "3000")
   	})
   	public String payment_timout() {
   		try {
   			Thread.sleep(2000);
   			System.out.println("延迟了二秒");
   		} catch (InterruptedException e) {
   			e.printStackTrace();
   		}
   		return Thread.currentThread().getName() + "--hystrix--timout";
   	}
   	
   	public String paymentTimeout2() {
   		return "方法超时,或者异常！";
   	}
   }
   
   ```

4. > #### 生产者控制层controller

   ```java
   package com.xcw.controller;
   
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   import com.xcw.service.PaymentService;
   
   @RestController
   public class PaymentController {
   	
   	@Autowired
   	PaymentService paymentService;
   	
   	@RequestMapping("/hystrix")//正常访问
   	public String payment() {
   		return paymentService.paymentInfo();
   	}
   	
   	@RequestMapping("/hystrix/timeout")//超出时间
   	public String paymentTimeout() {
   		return paymentService.payment_timout();
   	}
   }
   
   ```

5. > ### 启动类开启降级服务@EnableCircuitBreaker

   ```java
   @SpringBootApplication
   @EnableEurekaClient
   @EnableCircuitBreaker//开启服务降级
   public class Springcloud11HystrixProvider8005Application {
   
   	public static void main(String[] args) {
   		SpringApplication.run(Springcloud11HystrixProvider8005Application.class, args);
   	}
   }
   ```

   





### 消费者consumer

1. > ### pom引入eureka客户端，hystrix；

   ```xml
   <dependency>
   	<groupId>org.springframework.cloud</groupId>
   	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   <dependency>
   	<groupId>org.springframework.cloud</groupId>
   	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
   </dependency>
   <dependency>
   	<groupId>org.springframework.cloud</groupId>
   	<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
   </dependency>
   <dependency>
   	<groupId>org.springframework.cloud</groupId>
   	<artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```

2. ### yml设置ribbon的连接，并注册到注册中心

   ```yml
   server:
     port: 80
     
   spring:
     application:
       name: cloud-consumer-hystrix
       
   eureka:
     client:
       fetch-registry: true  #发现其他服务，负载均衡下必须true
       register-with-eureka: true
       service-url:
         defaultZone: http://localhost:7001/eureka/,http://localhost:7002/eureka/
         
   ribbon:
     #指的是建立连接所用的时间，适用于网络正常的情况下，两端连接所用的时间
     ReadTimeout: 5000
     #指的是建立连接后从服务器读取到可用资源所用的时间
     ConnectTimeout: 5000
   ```

3. > ### 业务类service（添加注解调用对应服务）

4. ```java
   package com.xcw.service;
   
   import org.springframework.cloud.openfeign.FeignClient;
   import org.springframework.stereotype.Component;
   import org.springframework.web.bind.annotation.RequestMapping;
   
   @FeignClient(name = "COLOUD-HYSTRIX-SERVICE")
   public interface HystrixService {
   	
   	@RequestMapping("/hystrix")//正常访问
   	public String payment();
   	
   	@RequestMapping("/hystrix/timeout")//超出时间
   	public String paymentTimeout();
   }
   
   ```

5. > ### 控制层controller（调用service接口方法）

   ```java
   package com.xcw.controller;
   
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   import com.xcw.service.HystrixService;
   
   @RestController
   public class HystrixController {
   	@Value("${server.port}")
   	String port;
   	@Autowired
   	HystrixService hystrixService;
   
   	@RequestMapping("/consumer/hystrix") // 正常访问
   	public String payment() {
   		return hystrixService.payment() + "::" + port;
   	}
   	
   	@RequestMapping("/consumer/hystrix/timeout") // 超出时间
   	public String paymentTimeout() {
   		return hystrixService.paymentTimeout() + "::" + port;
   	}
   }
   
   ```

6. > ### 启动类添加扫描注解

   ```java
   @SpringBootApplication
   @EnableFeignClients 		//扫描带有feginclent注解的接口，放入IOC容器中
   public class Springcloud12HystrixConsumer80FeginApplication {
   	public static void main(String[] args) {
   		SpringApplication.run(Springcloud12HystrixConsumer80FeginApplication.class, args);
   	}
   
   }
   ```



## 3.3 Hystrix全局降级方法一



> 在提供者的service内添加注解@DefaultProperties，指定默认处理方法

```java
@DefaultProperties(defaultFallback = "defaultMethod")
```



​	

## 3.4 Hystrix全局降级方法二



1. > #### 在服务消费者的服务调用接口中添加@FeignClient（注意属性正确）

   ```java
   package com.xcw.service;
   
   import org.springframework.cloud.openfeign.FeignClient;
   import org.springframework.stereotype.Component;
   import org.springframework.web.bind.annotation.RequestMapping;
   
   @FeignClient(name = "COLOUD-HYSTRIX-SERVICE",fallbackFactory  = HystrixFallBack.class)
   public interface HystrixService {
   	
   	@RequestMapping("/hystrix")//正常访问
   	public String payment();
   	
   	@RequestMapping("/hystrix/timeout")//超出时间
   	public String paymentTimeout();
   }
   
   ```

   

2. > #### 实现服务调用接口

   ```java
   package com.xcw.service;
   
   import org.springframework.stereotype.Component;
   
   import feign.hystrix.FallbackFactory;
   
   @Component//该实现类，实现HystrixService的降级处理
   public class HystrixFallBack implements FallbackFactory<HystrixService> {
   
   	@Override
   	public HystrixService create(Throwable cause) {
   		return new HystrixService() {
   			
   			@Override
   			public String paymentTimeout() {
   				return "fallback";
   			}
   			
   			@Override
   			public String payment() {
   				return "fallback timeout";
   			}
   		};
   	}
   
   }
   
   ```

3. > #### controler调用service

   ```java
   package com.xcw.controller;
   
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   import com.xcw.service.HystrixService;
   
   @RestController
   public class HystrixController {
   	@Value("${server.port}")
   	String port;
   	@Autowired
   	HystrixService hystrixService;
   
   	@RequestMapping("/consumer/hystrix") // 正常访问
   	public String payment() {
   		return hystrixService.payment() + "::" + port;
   	}
   
   	@RequestMapping("/consumer/hystrix/timeout") // 超出时间
   	public String paymentTimeout() {
   		return hystrixService.paymentTimeout() + "::" + port;
   	}
   }
   
   ```

4. > #### 启动类添加注解

   ```java
   package com.xcw;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
   import org.springframework.cloud.netflix.hystrix.EnableHystrix;
   import org.springframework.cloud.openfeign.EnableFeignClients;
   
   @SpringBootApplication
   @EnableFeignClients			//扫描feign
   @EnableCircuitBreaker		//开启熔断降级
   public class Springcloud12HystrixConsumer80FeginApplication {
   	public static void main(String[] args) {
   		SpringApplication.run(Springcloud12HystrixConsumer80FeginApplication.class, args);
   	}
   
   }
   
   ```



> ### 注意：
>
> 1. 实现FallbackFactory接口的泛型类型
> 2. Service接口@FeignClient注解中的属性fallbackFactory是否正确
> 3. 启动类注解：@EnableFeignClients、@EnableCircuitBreaker 扫描注册feign客户端，开启熔断降级服务



# 4.服务熔断

> #### 降级熔断处理，设置请求时间为十秒内，请求次数十次，失败率超过60%，会进行熔断处理;



1. 服务生产者的service

   ```java
   @HystrixCommand(fallbackMethod = "fallbackFucu", commandProperties = {
   	@HystrixProperty(name = "circuitBreaker.enabled", value = "true"), // 是否开启熔断器
   	@HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"), // 请求次数
   	@HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"), //时间窗口期
   	@HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60")// 失败率多少跳闸
   	})
   public String serviceFuce(@PathVariable Integer id) {// 服务熔断器
   		if (id < 0) {
   			throw new RuntimeException("id不能为负数");
   		}
   		String Number = IdUtil.simpleUUID();
   		return "流水号为：" + Number;
   }
   
   // 熔断后，降级的方法
   public String fallbackFucu(@PathVariable Integer id) {
   		return "id 不能为负数" + id;
   }
   ```

2. 服务生产者Controller

   ```java
   	@RequestMapping("/hystrix/serviceFuce/{id}")//服务熔断
   	public String serviceFuce(@PathVariable Integer id) {
   		return paymentService.serviceFuce(id);
   	}
   ```

3. 服务消费者service

   ```java
   	@RequestMapping("/hystrix/serviceFuce/{id}")//服务熔断
   	public String serviceFuce(@PathVariable Integer id); 
   ```

4. 服务消费者controller

   ```java
   @RequestMapping("/consumer/hystrix/serviceFuce/{id}") // 服务降级熔断
   	public String serviceFuce(@PathVariable Integer id) {
   		return hystrixService.serviceFuce(id);
   }
   ```

5. 入口：http://localhost/consumer/hystrix/serviceFuce/1





```xml
	<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<!--hystrix -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
			<version>2.2.5.RELEASE</version>
		</dependency>
		<!--dashboard -->
		<!-- <dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
			<version>2.2.5.RELEASE</version>
		</dependency> -->
		<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-hystrix-dashboard -->
		<dependency>
		    <groupId>org.springframework.cloud</groupId>
		    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
		    <version>2.2.0.RELEASE</version> 
		</dependency>

```



# 5.服务网关

> ##### gateway 代替了zuul，SpringCloud Gateway 使用的Webflux中的reactor-netty响应式编程组件，底层使用Netty通讯框架



## gateway能干什么

##### 反向代理
鉴权
流量监控
熔断
日志监控



## 三大特点

#### 路由、断言、过滤器



## 测试：

> ##### 测试启动前，启动springcloud01-eureka-server7001，springcloud02-eureka-server7002，springcloud03-eureka-payment8001;

> ##### 编写，springcloud14-gateway9527



1. pom文件引入，eureka客户端，gateway网关服务,微服务依赖；

   注意：需要将spring-boot-starter-web依赖注释，否则启动报错；

   ```xml
   		<!-- 
   		<dependency>
   			<groupId>org.springframework.boot</groupId>
   			<artifactId>spring-boot-starter-web</artifactId>
   		</dependency>
    		-->
   		<!-- 微服务 -->
   		<dependency>
   			<groupId>org.springframework.cloud</groupId>
   			<artifactId>spring-cloud-dependencies</artifactId>
   			<version>Hoxton.SR8</version>
   			<type>pom</type>
   			<scope>import</scope>
   		</dependency>
   
   		<dependency>
   			<groupId>org.springframework.cloud</groupId>
   			<artifactId>spring-cloud-starter-gateway</artifactId>
   		</dependency>
   		<dependency>
   			<groupId>org.springframework.cloud</groupId>
   			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   		</dependency>
   ```

2. yml添加gateway配置，将自己注册进，注册中心

   ```yml
   server:
     port: 9527
     
   spring: 
     application:
       name: cloud-gateway #服务名称
     cloud:
       gateway:
         discovery:
           locator:
             enabled: true # 开启从注册中心动态创建路由的功能，利用微服务名进行路由
         routes:
           - id: payment_routh #payment_routh #路由的ID，没有固定规则，但要求唯一，建议配合服务名
             uri: http://localhost:8001  #匹配后提供服务的路由地址 没有进行负载均衡
   #          uri: lb://cloud-panment-service #匹配后提供服务的路由地址
             predicates:
               - Path=/findPaymentById/** #断言，路径相匹配的进行路由
   
           - id: payment_routh2 #payment_routh #路由的ID，没有固定规则，但要求唯一，建议配合服务名
             uri: http://localhost:8001  #匹配后提供服务的路由地址
   #          uri: lb://cloud-panment-service #匹配后提供服务的路由地址
             predicates:
               - Path=/findPaymentAll/** #断言，路径相匹配的进行路由
   #            - After=2020-09-09T17:25:38.551+08:00[Asia/Shanghai] #在这个时间之后才能访问lb
   #            - Cookie=username,lsp #cookie中带有username值为lsp
   #            - Header=X-Request-Id,\d+  #请求头要有X-Request-Id属性并且值为整数的正则表达式
   #            - Host=**.lsp.com #接收一组参数，一组匹配的域名列表，它通过参数中的主机地址作为匹配规则
   #            - Method=GET  #只能是GET请求才可以
   #            - Query=username,\d+ #要求参数名username并且值还要是整数才能路由
   eureka: 
     instance:
       hostname: cloud-gateway-service
     # 每个EurekaServer 都包含一个EurekaClient，用于请求其他节同步 
     client: 
     #表示是否将自己注册EurekaServer
        registerWithEureka: true 
        #是否从EurekaServer抓取已有的注册信息，默认为true，单节点我所谓，集群必须设置为true才能配合ribbn使用负载均衡
        fetchRegistry: true 
        service-url: 
        #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址，如果服务器是集群就用    ， 号分割地址
          defaultZone: http://localhost:7001/eureka/,http://localhost:7002/eureka/
   
   ```

3. 启动类：开启注册中心发现自己

   ```Java
   package com.xcw;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
   
   @SpringBootApplication
   @EnableEurekaClient
   public class Springcloud14Gateway9527Application {
   
   	public static void main(String[] args) {
   		SpringApplication.run(Springcloud14Gateway9527Application.class, args);
   	}
   }
   
   ```



> ## 访问入口：

​	访问网关设置的断言入口: http://localhost:9527/findPaymentById/1



## 配置类访问



```Java
package com.xcw.config;

import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration//网关配置类
public class GateWayConfig {
	//置了一个id为route-name的路由规则
	//当访问地址为http://localhost:9527/guonei时会自动转发到https://news.baidu.com/guonei
	@Bean
	public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
		RouteLocatorBuilder.Builder routes = builder.routes();
		routes.route("path_xcw", r -> r.path("/guonei").uri("https://news.baidu.com/guonei")).build();
		return routes.build();
	}
}
```





## 服务名设置网关

> 通过服务名，设置网关可实现负载均衡，固定写法： uri: lb://服务名

```yml
spring: 
  application:
    name: cloud-gateway #服务名称
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_routh #路由的ID，没有固定规则，但要求唯一，建议配合服务名
#          uri: http://localhost:8001  #匹配后提供服务的路由地址 没有进行负载均衡
          uri: lb://cloud-panment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/** #断言，路径相匹配的进行路由
```



## 网关的断言配置

```YML
spring: 
  application:
    name: cloud-gateway #服务名称
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh2 #payment_routh #路由的ID，没有固定规则，但要求唯一，建议配合服务名
          uri: lb://cloud-panment-service #匹配后提供服务的路由地址
          predicates:
          	- Path=/payment/lb/** #断言，路径相匹配的进行路由
          	- After=2020-10-20T16:18:22.095+08:00[Asia/Shanghai] #在此时间后有效，
          	- Before=2020-10-20T16:18:22.095+08:00[Asia/Shanghai] #在此时间前有效
          	- Between=2020-10-20T16:18:22.095+08:00[Asia/Shanghai],2020-10-				  20T16:18:22.095+08:00[Asia/Shanghai] #在此时间之间有效
          	- Cookie=username,xcw #cookie中带有username值为xcw
            - Header=X-Request-Id,\d+  #请求头要有X-Request-Id属性并且值为整数的正则表达式
            - Host=**.xcw.com #接收一组参数，一组匹配的域名列表，它通过参数中的主机地址作为匹配规则
            - Method=GET  #只能是GET请求才可以
            - Query=username,\d+ #要求参数名username并且值还要是整数才能路由
          	
```



> #### 获取全格式时间

```java
//获取当前时间工具类
public class ZonedDateTimeTest {
	public static void main(String[] args) {
		ZonedDateTime now = ZonedDateTime.now();
		System.out.println(now);
		//2020-10-20T16:18:22.095+08:00[Asia/Shanghai]
	}
}
```





> #### 有效期

​	-  After=2020-10-20T16:18:22.095+08:00[Asia/Shanghai] 								#在此时间后有效

​	-  Before=2020-10-20T16:18:22.095+08:00[Asia/Shanghai] 							#在此时间前有效

​	-  Between=时间一,时间二 																					 #在此时间之间有效



> ##### 携带cookie：

​	- Cookie=username,xcw																						#cookie中带有username值为xcw

​	cmd访问：		curl http://localhost:9527/payment/lb --cookie "username=xcw"



> ##### 请求头信息设置：

​	- Header=X-Request-Id,\d+																				#请求头要有X-Request-Id属性并且值为整数的正则表达式

​	cmd访问：		curl http://localhost:9527/payment/lb -H "X-Request-Id:123"



> #### Host配置：

​	- Host=**.xcw.com 												#接收一组参数，一组匹配的域名列表，它通过参数中的主机地址作为匹配规则

​	cmd访问：		curl http://localhost:9527/payment/lb	 -H  "HOST:www.xcw.com"



> #### 请求方式

​	- Method=GET  														#只能是GET请求才可以

​	

> #### 请求参数

​	- Query=username,\d+ 										#要求参数名username并且值还要是整数才能路由

​	curl  http://localhost:9527/payment/lb?username=123





## 网关的过滤器配置

> 编写过滤器，实现GlobalFilter，Ordered接口；

```java
package com.xcw.filter;

import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.HttpStatus;
import org.springframework.web.server.ServerWebExchange;

import reactor.core.publisher.Mono;
@Component
public class GeteWayFilter implements GlobalFilter, Ordered {

	@Override // 过滤内容
	public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
		// 获取请求参数为name的
		String name = exchange.getRequest().getQueryParams().getFirst("name");
		if (name == null) {
			System.out.println("用户为null，非法用户----------");
			//设置响应状态
			exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
			return exchange.getResponse().setComplete();
		}
		//跳到下一个过滤器
		return chain.filter(exchange);
	}

	@Override // 多个过滤器顺序
	public int getOrder() {
		return 0;
	}

}

```

