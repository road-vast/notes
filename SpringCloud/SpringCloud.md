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

