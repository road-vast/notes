# 一、使用devTools实现热部署

步骤：

第一步：导入依赖

![image-20200429140859097](.\images\实现热部署1.png)

第二步：

![image-20200429140931873](.\images\实现热部署23.png)

第三步：

![image-20200429140952724](.\images\实现热部署3.png)

第四步：

![image-20200429141018050](.\images\实现热部署4.png)

第五步：

![image-20200429141222688](.\images\实现热部署5.png)

第六步：

![image-20200429141244149](.\images\实现热部署6.png)

第七步：

==解决不能打开谷歌网上应用商店：==

1 在谷歌中下载谷歌访问助手；

2 打开-》更多工具-》扩展程序界面；

3 将谷歌访问助手文件拉到扩展程序界面，添加谷歌访问助手；

4 打开谷歌应用商店；

第八步：

下载liveReload，点击liveReload插件图标进行同步之后，

开启项目，访问项目对应的网页，然后点击liveReload插件图标，显示为实心表示实现了静态网页的热部署

![image-20200429153201045](D:\notes\SpringBoot\images\实现静态网页的热部署.png)

------

# 二、使用fileupload实现文件上传

第一步：导入依赖

![image-20200430001125459](.\images\实现文件上传1.png)

第二步：

![image-20200430001201843](.\images\实现文件上传2.png)

第三步：实现文件上传和删除

```jaav
@Override
public Integer updateAvatarUrl(Integer uid,  MultipartFile newPhoto) {
    //用户原来的头像名
    String oldPhotoName = userMapper.getAvatarUrlById(uid);
    //删除用户本地原来的头像信息
    //本地头像路径
    Integer deleteResult = UserUtil.deleteLocalPhoto(oldPhotoName);

    String filePath = "D://avatarUrl//";

    //删除成功
    if (deleteResult == 0){
        //插入成功
        String newPhotoName = UserUtil.insertLocalPhoto(newPhoto);
        //将新的图片名更新到数据库
        return userMapper.updateAvatarUrl(uid, newPhotoName);
    }
    return -1;
}
```



```java
package com.itcast.springboot.utils;

import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.io.IOException;
import java.util.UUID;

public class UserUtil {
    //本地头像存储路径
    private final static String filePath = "D://avatarUrl//";

    //删除本地头像
    public static Integer deleteLocalPhoto(String oldPhotoName){
        //删除用户本地原来的头像信息
        //本地头像路径
        String oldPhotoLocalPath = filePath.concat(oldPhotoName);
        File file = new File(oldPhotoLocalPath);
        //0表示成功，-1表示失败
        Integer deleteResult = 0;
        if (file.exists()) {
            if (file.delete()) {
                deleteResult = 0;
            } else {
                deleteResult =  -1;
            }
        } else {
            //不存在则默认是删除了
            deleteResult =  0;
        }
        return deleteResult;
    }

    //将用户头像添加到本地
    public static String insertLocalPhoto(MultipartFile avatarUrl){
        //获取原始图片名称
        String photoName = avatarUrl.getOriginalFilename();
        //新的图片名字
        String newPhotoName = UUID.randomUUID() + photoName;
        System.out.println("新的图片名称： " + newPhotoName);
        //图片想要上传的本地路径位置
        String photoLocalPath = filePath.concat(newPhotoName);
        //在内存中创建File文件映射对象
        File dest = new File(photoLocalPath);
        //父目录是否存在，不存在则创建
        if (!dest.getParentFile().exists()) {
            dest.getParentFile().mkdirs();
        }
        try {
            //创建文件
            avatarUrl.transferTo(dest);
            //上传成功
            return newPhotoName;
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

# 三、使用Lombok

第一步：

![image-20200430105230712](D:\notes\SpringBoot\images\使用Lombok1.png)

第二步：

![image-20200430105251924](D:\notes\SpringBoot\images\使用Lombok2.png)

第三步：引入Lombok依赖

```java
<!--引入lombok-->
<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.10</version>
</dependency>
```

------

# 四、集成Swagger

第一步：引入依赖

```java
<!--导入swagger-->
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
    <exclusions>
        <exclusion>
            <groupId>io.swagger</groupId>
            <artifactId>swagger-models</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>io.swagger</groupId>
    <artifactId>swagger-models</artifactId>
    <version>1.5.22</version>
</dependency>
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

第二步：配置Swagger

```java
package com.itcast.springboot.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.core.env.Environment;
import org.springframework.core.env.Profiles;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;
import java.util.Map;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Autowired
    Environment environment;
    //为API配置分组
    @Bean
    public Docket docketUser(){
        //在dev和test的环境下才能访问dev：开发环境；test：测试环境
        Profiles profiles = Profiles.of("dev", "test");
        boolean isEnable = environment.acceptsProfiles(profiles);
        return new Docket(DocumentationType.SWAGGER_2)
                //为分组起名字
//                .groupName("用户")
                .select()
                //扫描对应的包
                .apis(RequestHandlerSelectors.basePackage("com.itcast.springboot.controller")).build()
                //该分组只显示/user路径下的
//                .paths(PathSelectors.ant("/user")).build()
                //忽略掉map类型的参数
                .ignoredParameterTypes(Map.class)
                //是否能被访问
                .enable(isEnable);
    }
}
```

访问：---》》》[Swagger-ui](http://localhost:8080/swagger-ui.html#/)

------

