# 一、编码前准备

## 1.版本控制

![image-20200428192729155](.\images\版本控制.png)

## 2.学习内容

![image-20200428193012210](.\images\学习内容.png)

------

# 1 微服务架构编码构建

## 1.规矩

约定>配置>编码

------

## 2.IDEA新建Project工作空间

总父工程POM：先建立project，然后再建立module。

微服务Cloud整体聚合父工程Project建立步骤：

1. New Project	![image-20200428200053148](.\images\new pproject.png)
2. 聚合总父工程名字
3. Maven选版本![maven3-5-2 ](.\images\maven3-5-2 .png)
4. 工程名字
5. 字符编码![image-20200428224127986](.\images\字符编码.png)
6. 注解生效激活![image-20200428224350516](.\images\注解生效激活.png)
7. java编译版本选8![image-20200428224552963](.\images\java8.png)
8. File Type过滤![image-20200428224956223](.\images\File Type过滤.png)

------

## 3.配置DepencyManagement

1.DepencyManagement应用场景

在我们项目顶层的POM文件中，我们会看到dependencyManagement元素。通过它元素来==管理jar包的版本==，让子项目中引用一个依赖而不用显示的列出版本号。Maven会沿着==父子层次==向上走，直到找到一个拥有dependencyManagement元素的项目，然后它就会使用在这个dependencyManagement元素中指定的版本号。

2.实现：

1. 指定这是一个POM工程![image-20200429082554595](.\images\指定是一个pom工程.png)
2. 使用properties标签，可以在properties下声明相应的版本信息，然后在dependency下引用的时候用${spring-version}就可以引入该版本jar包了![image-20200429082622518](.\images\properties统一管理jar包版本.png)
3. 引入各种jar包需要使用到的版本号![image-20200429082659404](.\images\dependencyManagement.png)

## 4.引入Build标签的时候会报错

当前解决方式：增加版本号

![image-20200429093045810](.\images\spring-boot-maven-plugin爆红.png)

------

# 2 Rest微服务工程构建

## 构建步骤

==微服务模块构建步骤：==

1. 建module
2. 改pom
3. 写yml
4. 主启动
5. 业务类

------

### 1.cloud-provider-payment8001微服务提供者支付Module模块

![image-20200429093732866](.\images\支付Module模块.png)

#### 1.建module

第一步：

![image-20200429094950613](.\images\建立支付module第一步.png)

第二步：

![image-20200429095024195](.\images\建立支付模块第二步.png)

第三步：

![image-20200429095054772](.\images\建立支付module第三步.png)

建立完成：

![image-20200429095119772](.\images\支付Module建立完成.png)

------

#### 2.改pom

![image-20200429111848721](.\images\支付module改pom.png)

#### 3.写yml

```java
server:
  port: 8001


spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: org.gjt.mm.mysql.Driver
    url: jdbc:mysql://localhost:3306/db2019?useUnicode=true&characterEnconding=utf-8&useSSL=false
    username: root
    password: 980219


mybatis:
  mapper-locations: classpath:mappper/*.xml
  type-aliases-package: com.atguigu.springcloud.entities
```

#### 4.主启动

```java
package com.atguigu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @author 郭泽鹏
 * @create 2020/04/29 11:33
 */
@SpringBootApplication
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class);
    }
}
```

#### 5.业务类

1.建表SQL

![image-20200430104114279](.\images\Payment Table.png)

2.entities

（1）主实体Payment

```java
package com.atguigu.springcloud.entities;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;

@Data
@AllArgsConstructor
@NoArgsConstructor
//序列化
public class Payment implements Serializable {
    private Long id;
    private String serial;
}
```

（2）Json封装体CommonResult

```java
package com.atguigu.springcloud.entities;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class CommonResult<T> {
    private Integer code;
    private String message;
    private T data;

    public CommonResult(Integer code, String message) {
        this(code, message, null);
    }
}
```

3.dao

Mapper接口：

```java
package com.atguigu.springcloud.dao;

import com.atguigu.springcloud.entities.Payment;
import org.apache.ibatis.annotations.Param;
import org.springframework.stereotype.Repository;

@Repository
public interface PaymentDao {
    //增
    public int create(Payment payment);

    //查
    public Payment getPaymentById(@Param("id") Long id);
}
```

mapper配置文件：

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.springcloud.dao.PaymentDao">
    <insert id="create" parameterType="Payment" useGeneratedKeys="true" keyProperty="id">
        insert into payment(serial) values(#{serial})
    </insert>
    <select id="getPaymentById" parameterType="Long" resultType="Payment">
        select * from payment
    </select>
</mapper>
```

4.service

service接口

```java
package com.atguigu.springcloud.service;

import com.atguigu.springcloud.entities.Payment;
import org.apache.ibatis.annotations.Param;

public interface PaymentService {
    //增
    public int create(Payment payment);

    //查
    public Payment getPaymentById(Long id);
}
```

service实现类

```java
package com.atguigu.springcloud.service.impl;

import com.atguigu.springcloud.dao.PaymentDao;
import com.atguigu.springcloud.entities.Payment;
import com.atguigu.springcloud.service.PaymentService;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

@Service
public class PaymentServiceImpl implements PaymentService {
    @Resource
    PaymentDao paymentDao;

    @Override
    public int create(Payment payment) {
        return paymentDao.create(payment);
    }

    @Override
    public Payment getPaymentById(Long id) {
        return paymentDao.getPaymentById(id);
    }
}
```

5.controller

```java
package com.atguigu.springcloud.controller;

import com.atguigu.springcloud.entities.CommonResult;
import com.atguigu.springcloud.entities.Payment;
import com.atguigu.springcloud.service.PaymentService;
import lombok.extern.slf4j.Slf4j;
import org.apache.ibatis.annotations.Param;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
//@Sl4j注解,可以使用log打印日志;
@Slf4j
public class PaymentController {

    @Resource
    PaymentService paymentService;
    //增
    @PostMapping(value = "/payment/create")
    public CommonResult create(Payment payment){
        int result = paymentService.create(payment);
        log.info("****插入结果：" + result);
        if(result > 0){
            return new CommonResult(200, "插入数据库成功", result);
        }else{
            return new CommonResult(444, "插入数据库失败", null);
        }
    }

    //查
    @GetMapping(value = "/payment/get/{id}")
    public CommonResult getPaymentById(@PathVariable("id") Long id){
        Payment payment = paymentService.getPaymentById(id);
        log.info("****查询结果：" + payment);
        if(payment != null){
            return new CommonResult(200, "查询成功", payment);
        }else{
            return new CommonResult(444, "没有对应记录，查询ID:" + id, null);
        }
    }
}
```

------

### *开启热部署

第一步：

![image-20200430132026491](.\images\开启热部署1.png)

第二步：

![image-20200430132107698](.\images\开启热部署2.png)

第三步：

![image-20200430132122756](.\images\开启热部署3.png)

第四步：

![image-20200430132138856](.\images\开启热部署4.png)

------

### 2.cloud-consumer-order80  消费者订单模块

#### 1）为什么使用80端口？

因为80是浏览器的默认端口

![image-20200430133742104](.\images\port80介绍.png)

#### 2）怎么调用支付模块？

一般情况下，消费者通过httpClient来调用微服务提供者提供的服务，这里restTemplate封装了httpTemplate，因此使用它即可。

![image-20200430170212055](.\images\调用支付Module流程.png)

------

#### 3）RestTemplate

RestTemplate是什么？

![image-20200430171005925](.\images\RestTemplate1.png)

[RestTemplate官网地址](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html)

使用：

![image-20200430171705186](.\images\RestTemplate2.png)

#### 4）具体操作：

第一步 配置config类：

```java
package com.atguigu.springcloud.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContextConfig {
    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

第二步：编写Controller类

```java
package com.atguigu.springcloud.controller;

import com.atguigu.springcloud.entities.CommonResult;
import com.atguigu.springcloud.entities.Payment;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderController {
    private static final String PAYMENT_URL = "http://localhost:8001";
    @Resource
    private RestTemplate restTemplate;

    @PostMapping("/consumer/payment/create")
    public CommonResult create(Payment payment){
        return restTemplate.postForObject(PAYMENT_URL + "/payment/create", payment, CommonResult.class);
    }

    @GetMapping("/consumer/payment/get/{id}")
    public CommonResult getPayment(@PathVariable("id") Long id){
        return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
    }
}
```

#### 5）*注意：

==消费者模块使用create方法将Payment对象以json数据形式传给支付模块，支付模块必须在Controller层中的对应方法上使用@RequestBody才能进行接收。否则接收不到数据，数据库没有插入数据。==

![image-20200430182702036](.\images\customer插入注意1.png)

![image-20200430182755767](.\images\customer插入注意2.png)

插入成功：

![image-20200430182902115](.\images\customer插入注意3.png)

### 3.工程重构

（1）观察问题：

![image-20200430193054869](.\images\工程重构1.png)

（2）新建

新建一个 cloud-api-commons Module

（3）POM

![image-20200430194311142](.\images\工程重构2.png)

（4）entities

复制entities包到新建的Module的对应位置中

（5）maven clean install命令

![image-20200430194956114](.\images\工程重构3.png)

（6）订单80和支付8001分别改造

![image-20200430195103541](.\images\工程重构4.png)

引入相关依赖

```java
<dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
    <groupId>com.atguigu.springcloud</groupId>
    <artifactId>cloud-api-commons</artifactId>
    <version>${project.version}</version>
</dependency>
```

------

# 3.Eureka服务注册与发现

## *加入Eureka之后的效果

![image-20200430221033109](.\images\加入Eureka效果.jpg)

------

## （1）Eureka基础知识

### 1）什么是服务治理？

![image-20200430215718801](D:\notes\SpringCloud\images\服务治理1.png)

### 2）什么是服务注册？

![image-20200430220525659](D:\notes\SpringCloud\images\服务注册.png)

3）Eureka的两个组件

![image-20200430220616796](D:\notes\SpringCloud\images\Eureka的两个组件.png)

------

## （2）单机Eureka构建步骤：

![image-20200430221422214](.\images\单机Eureka构建步骤.png)

### 1.建Module

### 2.改POM

*1.X 和 2.X的对比说明

![image-20200430222901902](.\images\Eureka版本比较.png)

------

![image-20200430230244048](D:\notes\SpringCloud\images\Eureka依赖.png)

------

### 3.写yml

```java
server:
  port: 7001

eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己。
    register-with-eureka: false
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
    #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

### 4.主启动

```java
package com.atguigu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
//代表这个模块是Eureka的服务注册中心
@EnableEurekaServer
public class EurekaMain7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaMain7001.class);
    }
}
```

### 5.启动

访问7001端口-----》 [Eureka](http://localhost:7001/)

------

## （3）EurekaClient端cloud-provider-payment8001将注册进EurekarServer成为服务提供者provider，类似尚硅谷学校对外提供授课服务

==步骤：==

![image-20200502165553164](D:\notes\SpringCloud\images\注册8001-1.png)

### 1.改POM

![image-20200502170213896](.\images\EurekaClient版本比较.png)

```java
<!--eureka-client-->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### 2.写yml

```java
eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
```

### 3.主启动

```java
//主启动上添加注解
@EnableEurekaClient
```

### 4.测试

（1）先要启动Eureka Server

（2）访问7001端口-----》 [Eureka](http://localhost:7001/)

![image-20200502172102790](.\images\注册8001-2.png)

（3）微服务注册名配置说明

![image-20200502172453553](D:\notes\SpringCloud\images\注册8001-3.png)

## （4）EurekaClient端cloud-consumer-order80将注册进EurekaServer成为服务消费者consumer，类似于来尚硅谷上课的各位同学

![image-20200502172843303](.\images\注册80-1.png)

### 1.改POM

```java
<!--eureka-client-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### 2.写yml

```java
spring:
  application:
    name: cloud-order-service

eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: false
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
    defaultZone: http://localhost:7001/eureka
```

### 3.主启动

```java
//主启动上添加注解
@EnableEurekaClient
```

### 4.测试

![image-20200502174158395](.\images\注册80-2.png)

------

## (3)集群Eureka构建步骤

### *原理

![image-20200502174737853](D:\notes\SpringCloud\images\Eureka集群原理说明1.png)

![image-20200502175042564](D:\notes\SpringCloud\images\Eureka集群原理说明2.png)

![image-20200502175404532](D:\notes\SpringCloud\images\Eureka集群原理说明3.png)

### 1.参考cloud-eureka-server7001新建module cloud-eureka-server7002

### 2.改POM（参考7001）

### 3.修改映射配置文件

（作用：使两个服务注册中心的地址不会都是localhost）

![image-20200502182425038](D:\notes\SpringCloud\images\Eureka集群1.png)

~~~
C:\Windows\System32\drivers\etc打开hosts文件尾部添加代码：
127.0.0.1 eureka7001.com
127.0.0.2 eureka7002.com
*没有权限问题：右击hosts文件-》属性-》安全-》Users（当前用户）-》编辑-》Users-》勾选允许完全控制-》应用-》确定
~~~

### 4.写yml(同时修改7001和7002配置文件)

7001：

```java
server:
  port: 7001


eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      #集群指向其它eureka，形成相互注册相互守望
      defaultZone: http://eureka7002.com:7002/eureka/
      #单机就是7001自己
      #defaultZone: http://eureka7001.com:7001/eureka/
```

7002：

```
server:
  port: 7002


eureka:
  instance:
    hostname: eureka7002.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      #集群指向其它eureka，形成相互注册相互守望
      defaultZone: http://eureka7001.com:7001/eureka/
      #单机就是7001自己
      #defaultZone: http://eureka7002.com:7002/eureka/
```

### 5.测试

![image-20200502191741203](D:\notes\SpringCloud\images\Eureka集群.png)

## （4）将支付服务8001和消费者订单80微服务发布到上面2台Eureka集群配置中

### 1.改yml

```java
service-url:
  #单机版
  #defaultZone: http://localhost:7001/eureka
  #集群版
  defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

### 2.测试：

![image-20200502192730205](D:\notes\SpringCloud\images\Eureka集群测试.png)

## （5）支付服务提供者8001集群环境构建

### 1.参考8001新建8001Module



### 2.改POM



### 3.写yml

```java
server:
  port: 8002


spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: org.gjt.mm.mysql.Driver
    url: jdbc:mysql://localhost:3306/db2019?useUnicode=true&characterEnconding=utf-8&useSSL=false
    username: root
    password: 980219

eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      #单机版
      #defaultZone: http://localhost:7001/eureka
      #集群版
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka

mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.atguigu.springcloud.entities
```

### 4.启动类

```java
package com.atguigu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
@MapperScan("com.atguigu.springcloud.dao")
public class Payment8002 {
    public static void main(String[] args) {
        SpringApplication.run(Payment8002.class);
    }
}
```

### 5.业务类



### 6.修改8001/8002的Controller

==（多加一个获取端口号的功能）==

```java
package com.atguigu.springcloud.controller;

import com.atguigu.springcloud.entities.CommonResult;
import com.atguigu.springcloud.entities.Payment;
import com.atguigu.springcloud.service.PaymentService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;

@RestController
//@Sl4j注解,可以使用log打印日志;
@Slf4j
public class PaymentController {

    @Resource
    PaymentService paymentService;
    //获取端口号
    @Value("${server.port}")
    private String serverPort;

    //增
    @PostMapping(value = "/payment/create")
    public CommonResult create(@RequestBody Payment payment){
        int result = paymentService.create(payment);
        log.info("****插入结果：" + result);
        if(result > 0){
            return new CommonResult(200, "插入数据库成功" + serverPort, result);
        }else{
            return new CommonResult(444, "插入数据库失败" + serverPort, null);
        }
    }

    //查
    @GetMapping(value = "/payment/get/{id}")
    public CommonResult getPaymentById(@PathVariable("id") Long id){
        Payment payment = paymentService.getPaymentById(id);
        log.info("****查询结果：" + payment + "哈哈哈");
        if(payment != null){
            return new CommonResult(200, "查询成功", payment);
        }else{
            return new CommonResult(444, "没有对应记录，查询ID:" + id, null);
        }
    }
}
```

### 6.测试

![image-20200503104749208](D:\notes\SpringCloud\images\Eureka8001实现集群.png)

## （6）负载均衡

### ==1.bug==

订单访问地址不能写死，否则每次访问只会调用8001端口提供服务

![image-20200503105520981](D:\notes\SpringCloud\images\Eureka负载均衡1.png)

![image-20200503105615717](D:\notes\SpringCloud\images\Eureka负载均衡2.png)

==*解决方案：==

1.将"http://localhost:8001"改为注册进服务中心的名字"http://CLOUD-PAYMENT-SERVICE"

2.使用@LoadBalanced注解赋予RestTemplate负载均衡的能力

```java
@Configuration
public class ApplicationContextConfig {
    @Bean
    //使用@LoadBalanced注解赋予RestTemplate负载均衡的能力
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

==成功实现负载均衡==

## （7）actuator微服务信息完善

### 1.主机名称：服务名称修改

#### ==1）当前问题：（含有主机名称）==

![image-20200503121705431](D:\notes\SpringCloud\images\actuator微服务信息完善1.png)

#### 2）解决：

修改cloud-provider-payment8001/8002 yml

```
eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      #单机版
      #defaultZone: http://localhost:7001/eureka
      #集群版
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    instance-id: payment8001	#不显示主机名称
```

效果：

![image-20200503122823154](D:\notes\SpringCloud\images\actuator微服务信息完善3.png)

### 2.访问信息有IP信息提示

当前问题：（没有IP地址提示）

![image-20200503122101186](D:\notes\SpringCloud\images\actuator微服务信息完善2.png)

#### 2）解决：

修改cloud-provider-payment8001/8002 yml

```java
eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      #单机版
      #defaultZone: http://localhost:7001/eureka
      #集群版
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    instance-id: payment8002  #不显示主机名称
    prefer-ip-address: true   #访问路径可以显示ip地址
```

效果：

![image-20200503123214486](D:\notes\SpringCloud\images\actuator微服务信息完善4.png)

## （8）服务发现 Discovery

==*效果==

对于注册进eureka里面的微服务，可以通过服务发现来获得该服务的信息

### 1.修改cloud-provider-payment8001的Controller

~~~java
	//自动注入DiscoveryClient属性：服务发现
	@Resource
    private DiscoveryClient discoveryClient;
	//
	@GetMapping("/payment/discovery")
    public Object discovery(){
        //获取注入到服务中心的服务列表信息
        List<String> services = discoveryClient.getServices();
        for (String element:
             services) {
            log.info("*******element" + element);
        }
        //获取其中的一个服务名的具体信息
        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        for (ServiceInstance instance:
             instances) {
            log.info(instance.getServiceId() + "\t" + instance.getHost() + "\t"
            + instance.getPort() + "\t" + instance.getUri());
        }
        return this.discoveryClient;
    }
~~~

### 2.8001主启动类

```java
//添加开启服务发现注解
@EnableDiscoveryClient
```

### 3.自测

访问-》》》》    [8001-Discovery](http://localhost:8001/payment/discovery)

![image-20200504095930986](D:\notes\SpringCloud\images\Discovery1.png)

![image-20200504095634924](D:\notes\SpringCloud\images\Discovery2.png)

------

## （9）Eureka自我保护

### 1.故障现象：

![image-20200504100540365](D:\notes\SpringCloud\images\Eureka自我保护1.png)

### 2.导致原因：

![image-20200504100621281](D:\notes\SpringCloud\images\Eureka自我保护2.png)

### 3.怎么禁止自我保护：

==这里设置为eurekau为单机版==

#### （1）注册端EurekaServer端7001

##### 1）出厂默认，自我保护机制使开启的server.enable-self-preservation=true

##### 2）使用server.enable-self-preservation=false关闭自我保护机制

##### 3）在EurekaServer端7001处设置关闭自我保护机制

```java
eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      #集群版：集群指向其它eureka，形成相互注册相互守望
      #defaultZone: http://eureka7002.com:7002/eureka/
      #单机版：单机就是7001自己
      defaultZone: http://eureka7001.com:7001/eureka/
  server:
    #关闭自我保护机制，保证不可用服务被机制剔除
    enable-self-preservation: false
    #设置多久没收到心跳则关闭服务:2s
    eviction-interval-timer-in-ms: 2000
```

##### 4）关闭效果

![image-20200504103257734](D:\notes\SpringCloud\images\禁止自我保护1.png)

##### 

#### （2）生产客户端eurekaClient端8001

##### 1）设置客户端发送心跳的时间和服务端等待心跳时间上限

```java
eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      #单机版
      defaultZone: http://localhost:7001/eureka
      #集群版
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    instance-id: payment8001  #不显示主机名称
    prefer-ip-address: true   #访问路径可以显示ip地址
    #Eureka客户端向服务端发送心跳的时间间隔，单位为秒（默认使30秒）
    lease-renewal-interval-in-seconds: 1
    #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒（默认使90秒），超时将剔除服务
    lease-expiration-duration-in-seconds: 2
```

##### 2）效果

8001服务一关闭后，eureka则剔除8001服务

![image-20200504104349504](D:\notes\SpringCloud\images\Eureka自我保护3.png)

------

# 4.ZooKeeper的注册与调用

## ==（1）Eureka已停止更新==

## （2）SpringCloud整合Zookeeper代替Eureka

![image-20200504105135864](D:\notes\SpringCloud\images\zookeeper实现效果.png)

### 1）注册中心Zookeeper

- Zookeeper是一个分布式协调工具，可以实现注册中心功能；
- 关闭Linux服务器防火墙之后启动zookeeper服务器
- zookerper服务器代替Eureka服务器，zk作为服务注册中心

==使用docker下载zookeeper，必须保证虚拟机ping通主机==

### 2）服务提供者

#### 1.新建cloud-provider-payment8004



#### 2.改POM

~~~java
	<dependencies>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
            <groupId>com.atguigu.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!-- SpringBoot整合zookeeper客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <!--先排除自带的zookeeper3.5.3-->
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!--添加zookeeper3.4.9版本-->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.9</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
~~~

#### 3.写yml

```java
#8004表示注册到zookeeper服务器的支付服务提供者端口号
server:
  port: 8004


#服务别名----注册zookeeper到注册中心名称
spring:
  application:
    name: cloud-provider-payment
  cloud:
    zookeeper:
      connect-string: 192.168.0.109:2181
```

#### 4.主启动

```java
package com.atguigu.cloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient  //该注解用于向使用consul或者zookeeper作为注册中心时注册服务
public class PaymentMain8004 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8004.class, args);
    }
}
```

#### 5.Controller

```java
package com.atguigu.cloud.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

@RestController
@Slf4j
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    @RequestMapping(value = "/payment/zk")
    public String paymentzk()
    {
        return "springcloud with zookeeper: "+serverPort+"\t"+ UUID.randomUUID().toString();
    }
}
```

#### 6.启动8004注册进Zookeeper

#### （1）linux下启动zookerper

~~~
1.进入容器内部：
docker exec -it containerID bash
2.开启客户端：
zkCli.sh
4.获取节点：
ls /
显示：[zookeeper]
~~~

#### （2）开启8004主启动

==（注意Zookerper引入的jar包和linux里的Zookeeper版本要一致，否则可能报错）==

1）启动没有报错后，在容器内部输入  ls /  出现如下界面，表示注册进Zookeeper成功。

![image-20200505101817207](D:\notes\docker\images\zookeeper2.png)

​																			【启动成功】

访问：---》》》	[PaymentController](http://localhost:8004/payment/zk)

![image-20200505102644756](D:\notes\docker\images\zookeeper3.png)

------

#### 7.测试验证

在linux中查看节点信息（表示微服务8004在Zookeeper下的基本信息）

![image-20200505103942827](D:\notes\docker\images\zookeeper4.png)

使用：---》》》[json工具](https://tool.lu/json/) 	转化后：

~~~java
{
  "name": "cloud-provider-payment",
  "id": "98b8b838-7529-49ba-b505-fd8c7b0d1914",			//自动生成services id的流水号
  "address": "localhost",
  "port": 8004,
  "sslPort": null,
  "payload": {
    "@class": "org.springframework.cloud.zookeeper.discovery.ZookeeperInstance",
    "id": "application-1",
    "name": "cloud-provider-payment",
    "metadata": {}
  },
  "registrationTimeUTC": 1588645876105,
  "serviceType": "DYNAMIC",
  "uriSpec": {
    "parts": [
      {
        "value": "scheme",
        "variable": true
      },
      {
        "value": "://",
        "variable": false
      },
      {
        "value": "address",
        "variable": true
      },
      {
        "value": ":",
        "variable": false
      },
      {
        "value": "port",
        "variable": true
      }
    ]
  }
}
~~~

#### 8.思考

服务节点是临时结点还是持久结点？		临时节点（一段时间发心跳没有回复则直接剔除）

------

## 3）服务消费者

### 1.新建cloud-consumerzk-order80



### 2.改POM

```java
<dependencies>
    <!-- SpringBoot整合Web组件 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- SpringBoot整合zookeeper客户端 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
        <!--先排除自带的zookeeper-->
        <exclusions>
            <exclusion>
                <groupId>org.apache.zookeeper</groupId>
                <artifactId>zookeeper</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <!--添加zookeeper3.4.9版本-->
    <dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
        <version>3.4.9</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 3.写yml

```java
server:
  port: 80

spring:
  application:
    name: cloud-consumer-order
  cloud:
    #注册到zookeeper地址
    zookeeper:
      connect-string: 192.168.0.109:2181
```

### 4.主启动

```java
package com.atguigu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class OrderZKMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderZKMain80.class, args);
    }
}
```

### 5.业务类

配置类

```java
package com.atguigu.springcloud.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestTemplateConfig {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

Controller类

```java
package com.atguigu.springcloud.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderZKController {
    private static final String INVOKE_URL = "http://cloud-provider-payment";
    @Resource
    private RestTemplate restTemplate;

    public String paymentInfo(){
        String result = restTemplate.getForObject(INVOKE_URL + "payment/zk", String.class);
        return result;
    }
}
```

### 6.测试验证

![image-20200505113528012](D:\notes\docker\images\zookeeper5.png)

### 7.访问测试地址

访问：---》》》	[orderZK80](http://localhost/consumer/payment/zk)

![image-20200505113626730](D:\notes\docker\images\zookeeper6.png)

------

# 5.Consul服务注册与发现

## （1）Consul简介

[Consul官网](https://www.consul.io/intro/index.html)

[Consul中文官网](https://www.springcloud.cc/spring-cloud-consul.html)

![image-20200505122617997](D:\notes\SpringCloud\images\consul简介1.png)

![image-20200505122658566](D:\notes\SpringCloud\images\consul简介2.png)



## （2）安装与运行Consul

下载：---》》》	[官网下载](https://www.consul.io/downloads.html)

运行：cmd定位到安装目录下

查看版本：consul --version

运行：consul agent -dev

访问：---》》》		[Consul8500](http://localhost:8500/ui/dc1/services)

![image-20200506094159290](D:\notes\SpringCloud\images\consul安装与运行1.png)

## （3）服务提供者

### 1.新建Module支付服务cloud-providerconsul-payment8006



### 2.改POM

```java
<dependencies>
    <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
    <dependency>
        <groupId>com.atguigu.springcloud</groupId>
        <artifactId>cloud-api-commons</artifactId>
        <version>${project.version}</version>
    </dependency>
    <!--SpringCloud consul-server -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-consul-discovery</artifactId>
    </dependency>
    <!-- SpringBoot整合Web组件 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--日常通用jar包配置-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>RELEASE</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>RELEASE</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 3.写yml

```java
###consul服务端口号
server:
  port: 8006

spring:
  application:
    name: consul-provider-payment
  ####consul注册中心地址
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        #hostname: 127.0.0.1
        service-name: ${spring.application.name}
```

### 4.主启动

```java
package com.atguigu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain8006 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8006.class, args);
    }
}
```

### 5.业务类Controller

```
package com.atguigu.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

@RestController
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    @RequestMapping(value = "/payment/consul")
    public String paymentConsul(){
        return "springCloud with Consul\t" + serverPort + "\t" + UUID.randomUUID().toString();
    }
}
```

### 6.验证测试

![image-20200506103630739](D:\notes\SpringCloud\images\Consul服务提供者1.png)

访问：---》》》	[ConsulController8006](http://localhost:8006/payment/consul)

![image-20200506103808234](D:\notes\SpringCloud\images\Consul服务提供者2.png)

## （4）服务消费者

### 1.新建Module消费服务cloud-consumerconsul-order80



### 2.改POM

~~~java
<dependencies>
        <!--SpringCloud consul-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--日常通用jar包配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
~~~



### 3.写yml

~~~java
###consul服务端口号
server:
  port: 80

spring:
  application:
    name: cloud-consumer-order
####consul注册中心地址
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        #hostname: 127.0.0.1
        service-name: ${spring.application.name}


~~~



### 4.主启动

```
package com.atguigu.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class OrderConsulMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderConsulMain80.class, args);
    }
}
```

### 5.业务类

配置类

```java
package com.atguigu.springcloud.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestTemplateConfig {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

Controller类

```java
package com.atguigu.springcloud.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
public class OrderController {
    private static final String INVOKE_URL = "http://consul-provider-payment";
    @Resource
    private RestTemplate restTemplate;
    @GetMapping("/consumer/payment/consul")
    public String paymentInfo(){
        String result = restTemplate.getForObject(INVOKE_URL + "/payment/consul", String.class);
        return result;
    }

}
```

### 6.验证测试

![image-20200506110401877](D:\notes\SpringCloud\images\Consul服务消费者1.png)

访问：---》》》		[Consul消费者](http://localhost/consumer/payment/consul)

![image-20200506110537951](D:\notes\SpringCloud\images\Consul服务消费者2.png)

## （5）三个注册中心异同点

1.

![image-20200506111704156](D:\notes\SpringCloud\images\三个注册中心异同点1.png)

2.![image-20200506111739249](D:\notes\SpringCloud\images\三个注册中心异同点2.png)

3.

![image-20200506111815585](D:\notes\SpringCloud\images\三个注册中心异同点3.png)



4.

![image-20200506111920977](D:\notes\SpringCloud\images\三个注册中心异同点5.png)

5.

![image-20200506111855764](D:\notes\SpringCloud\images\三个注册中心异同点4.png)

![image-20200506111949401](D:\notes\SpringCloud\images\三个注册中心异同点6.png)

6.

![image-20200506112015059](D:\notes\SpringCloud\images\三个注册中心异同点7.png)

![image-20200506112031377](D:\notes\SpringCloud\images\三个注册中心异同点8.png)

# 6.Ribbon负载均衡服务调用

## （1）概述

#### 1.Ribbon是什么？

![image-20200506121601482](D:\notes\SpringCloud\images\Ribbon概述1.png)

#### 2.Ribbon下载

目前正在维护中...

[Ribbon官网](https://github.com/Netflix/ribbon)

#### 3.能做什么？

==负载均衡+RestTemplate调用==

1）LB（负载均衡）

![image-20200506124617891](D:\notes\SpringCloud\images\Ribbon概述2.png)

（1）集中式LB

![image-20200506124847553](D:\notes\SpringCloud\images\Ribbon概述3.png)

（2）进程内LB

![image-20200506124911282](D:\notes\SpringCloud\images\Ribbon概述4.png)



## （2）Ribbon负载均衡演示

![image-20200506125145409](D:\notes\SpringCloud\images\Ribbon负载均衡演示1.png)

------

### 1.架构图

![image-20200506125339842](D:\notes\SpringCloud\images\Ribbon负载均衡演示2.png)

![image-20200506125515828](D:\notes\SpringCloud\images\Ribbon负载均衡演示3.png)

### ==2.Eureka依赖中已经引入了Ribbon依赖==

![image-20200506133347530](D:\notes\SpringCloud\images\Ribbon负载均衡演示4.png)

### 3.二说RestTemplate的使用

#### 1）getForObject方法/getForEntity方法

```java
//返回对象是响应体中数据转化成的对象，基本上可以理解为Json
@GetMapping("/consumer/payment/get/{id}")
public CommonResult getPayment(@PathVariable("id") Long id){
    return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
}
//返回对象为ResponseEntity对象，包含了响应中的一些重要信息，比如响应头，响应状态码，响应体等
@GetMapping("/consumer/payment/getForEntity/{id}")
public CommonResult getPayment2(@PathVariable("id") Long id){
    ResponseEntity<CommonResult> entity = restTemplate.getForEntity(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);

    if (entity.getStatusCode().is2xxSuccessful()){
        log.info("********************" + String.valueOf(entity.getHeaders()) + "\t" +  entity.getStatusCode());
        return entity.getBody();
    }else{
        return new CommonResult(400, "操作失败");
    }
}
```

#### 2）postForObject方法/postForEntity方法

```java
@PostMapping("/consumer/payment/create")
public CommonResult create(Payment payment){
    return restTemplate.postForObject(PAYMENT_URL + "/payment/create", payment, CommonResult.class);
}
@PostMapping("/consumer/payment/createEntity")
public CommonResult create1(Payment payment){
    ResponseEntity<CommonResult> entity = restTemplate.postForEntity(PAYMENT_URL + "/payment/create", payment, CommonResult.class);
    if (entity.getStatusCode().is2xxSuccessful()){
        return entity.getBody();
    }else{
        return new CommonResult<>(400, "插入失败");
    }
}
```

## （3）Ribbon核心组件IRule

### 1IRule：根据特定算法从服务列表中选取一个要访问的服务

![image-20200507000040852](D:\notes\SpringCloud\images\Ribbon核心组件IRule1.png)

## （4）Ribbon负载均衡算法

































# 三、错误处理

## 1.导入jar包错误问题

1.默认的配置文件中使用的仓库是maven的中央仓库，有些老版本的spring包在最新的maven仓库中找不到，那么碰到这种情况我们应该怎么解决呢？

解决方案：在pom.xml文件中引入spring的插件仓库，在最后面添加：

```java
    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
    	</repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>repository.spring.release</id>
            <name>Spring GA Repository</name>
            <url>https://repo.spring.io/plugins-release/</url>
		</pluginRepository>
    </pluginRepositories>
</project>
```

------

2.导入mybatis爆红

![image-20200429110916988](.\images\导入mybatis爆红.png)

==解决方案：==

![image-20200429111044036](.\images\解决mybatis爆红.png)

## 2.新建Module时maven界面对应的module为灰色

原因：父工程没有对其进行管理

![image-20200430164113050](.\images\Module灰色错误1.png)

第一步：

![image-20200430164211158](.\images\Module灰色错误2.png)

第二步：

![image-20200430164230408](.\images\Module灰色错误3.png)

完成：

![image-20200430165054292](.\images\Module灰色错误4.png)

------

## 3.没有自动出现Run Dashboard

（Spring2019.3 的Run Dashboard改Services）

做法：

![image-20200430184249070](.\images\没有出现Run Dashboard4.png)

第一步：

![image-20200430184124698](.\images\没有出现Run Dashboard1.png)

第二步：

![image-20200430184159559](.\images\没有出现Run Dashboard2.png)

第三步：

![image-20200430184231826](.\images\没有出现Run Dashboard3.png)

插入代码内容：

~~~java
<option name="configurationTypes">
	<set>
		<option value="SpringBootApplicationConfigurationType" />
	</set>
</option>
~~~

