# SSM 整合

学习本章之前，我们应该对`SpringMVC`, `Spring`, `Mybatis`框架有一个完整的了解，并熟练使用单独框架进行简单的代码开发，如果不能做到，请返回到相关课件页面进行学习，再进入当前页学习本章内容，谢谢。

本章框架集成所使用的软件版本要求如下：

| 软件       | 版本          |
| :--------- | :------------ |
| SpringMVC  | 4.0.0.RELEASE |
| Spring     | 4.0.0.RELEASE |
| Mybatis    | 3.2.8         |
| C3P0连接池 | 0.9.2         |
| MYSQL驱动  | 5.1.37        |



## 创建Maven Web项目

在`Spring Tool Suite`中创建`Maven`项目，生成`web.xml`文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" id="WebApp_ID" version="2.5">

</web-app>
```

#### `web.xml`是整个项目的核心配置文件，也可以理解为Web程序访问的入口，非常重要

在`servlet3.0`及后续版本中，此配置文件可省略，采用注解方式替代，本课程暂不涉及。

在项目的`pom.xml`文件中增加依赖关系：点击详细



## 集成Spring框架

`Spring`框架是整个系统架构的核心，将前端请求数据的处理以及数据库的数据操作整合在一起，非常重要。

#### WEB配置文件

在`web.xml`文件中增加配置信息集成`Spring`框架

```xml
<web-app>
    ...
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:spring/spring-*.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    ...
</web-app>
```

Spring环境构建时需要读取web应用的初始化参数`contextConfigLocation`, 从`classpath`中读取配置文件`spring/spring-*.xml`

#### Spring配置文件

在项目`src/main/resources`目录中增加`spring`文件夹，并在其中增加`spring-context.xml`配置文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">
</beans>
```

`Spring`框架的核心是构建对象，整合对象之间的关系（`IOC`）及扩展对象功能（`AOP`），所以需要在`spring-context.xml`配置文件中增加业务对象扫描的相关配置。扫描后由`Spring`框架进行管理和组合。

```xml
<beans>
    ...
    <context:component-scan base-package="com.atguigu.*" >
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    ...
</beans>
```

>   **扫描配置中为什么要排除`Controller`注解**
>
>   `Controller`注解的的作用是声明控制器（处理器）类。
>
>   从数据流转的角度，这个类应该是由`SpringMVC`框架进行管理和组织的，所以不需要由`Spring`框架扫描。



## 集成SpringMVC框架

`SpringMVC`框架用于处理系统中数据的流转及控制操作。（从哪里来，到哪里去。多么有哲理的一句话。）

#### WEB配置文件

集成`SpringMVC`框架，需要在`web.xml`文件中增加配置信息

```xml
<web-app>
...
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/springmvc-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
...
</web-app>
```

SpringMVC环境构建时需要读取servlet初始化参数 `init-param` , 从 classpath 中读取配置文件 `spring/springmvc-context.xml`

#### SpringMVC配置文件

在项目 `src/main/resources/spring` 目录中，增加 `springmvc-context.xml` 配置文件。

```html
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

</beans>
```

`SpringMVC`框架的核心是处理数据的流转，所以需要在`springmvc-context.xml`配置文件中增加控制器对象（`Controller`）扫描的相关配置。扫描后由`SpringMVC`框架进行管理和组合。

```xml
<beans>
    ...
    <context:component-scan base-package="com.atguigu.*" use-default-filters="false" >
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    ...
</beans>
```

#### 静态资源如何不被`SpringMVC`框架进行拦截

在配置文件中增加`<mvc:default-servlet-handler/>`, `<mvc:annotation-driven />`即可

在实际的项目中静态资源不会和动态资源放在一起，也就意味着不会放置在服务器中，所以这些配置可以省略。

如果`SpringMVC`框架数据处理为页面跳转，那么需要配置相应的视图解析器`ViewResolver`。

```xml
<beans>
    ...
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" >
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    ...
</beans>
```

#### 如果有多个视图解析器怎么办？

`SpringMVC`框架中允许存在多个视图解析器，框架会按照配置声明顺序，依次进行解析。

`SpringMVC`框架中配置多个视图解析器时，如果将`InternalResourceViewResolver`解析器配置在前，那么即使找不到视图，框架也不会继续解析，直接发生`404`错误，所以必须将`InternalResourceViewResolver`解析器放置在最后。

如果`SpringMVC`框架数据处理为响应`JSON`字符串，那么为了浏览器方便对响应的字符串进行处理，需要明确字符串的类型及编码方式。

**如果增加了`<mvc:annotation-driven />`标签，下面的配置可省略。**

```xml
<beans>
    ...
    <bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter" >
        <property name="messageConverters" >
            <list>
                <bean class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter" >
                    <property name="supportedMediaTypes" >
                        <list>
                            <value>application/json;charset=UTF-8</value>
                        </list>
                    </property>
                </bean>
            </list>
        </property>
    </bean>
    ...
</beans>
```

#### 上传解析器

如果项目中含有文件上传业务，还需要增加文件上传解析器`MultipartResolver`

```xml
<beans>
    ...
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver" p:defaultEncoding="UTF-8" >
        <property name="maxUploadSize" value="2097152"/>
        <property name="resolveLazily" value="true"/>
    </bean>
    ...
</beans>
```



## 集成Mybatis框架

#### Spring中配置MyBatis

`Mybatis`框架主要处理业务和数据库之间的数据交互，所以创建对象和管理对象生命周期的职责可以委托`Spring`框架完成。如：创建`Mybatis`核心对象。

```xml
<beans>
    ...
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean" >
        <property name="configLocation" value="classpath:mybatis/config.xml" />
        <property name="dataSource" ref="dataSource" />
        <property name="mapperLocations" >
            <list>
                <value>classpath*:mybatis/mapper-*.xml</value>
            </list>
        </property>
    </bean>
    ...
    <bean id="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer" >
        <property name="basePackage" value="com.atguigu.atcrowdfunding.**.dao" />
    </bean>
    ...
</beans>
```

既然需要和数据库进行关联，那么`Mybatis`核心对象就需要依赖于数据库连接池（`C3P0`）,所以在`Spring`配置文件中增加相应的配置。

```xml
<beans>
    ...
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" >
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/atcrowdfunding?rewriteBatchedStatements=true&amp;useUnicode=true&amp;characterEncoding=utf8"/>
        <property name="user" value="root"/>
        <property name="password" value="root"/>
    </bean>
    ...
</beans>
```

#### MyBatis 核心配置

集成`Mybatis`框架时同时还需要增加核心配置文件`mybatis/config.xml`。

```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        ...
    </typeAliases>
</configuration>
```

#### Mapper 映射文件

及`SQL`映射文件`mybatis/mapper-*.xml`。

```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="xxx.XXDao" >
...
</mapper>
```

#### MyBatis 事务配置

为了保证数据操作的一致性，必须在程序中增加事务处理。`Spring`框架采用声明式事务，通过`AOP`的方式将事务增加到业务中。所以需要在`Spring`配置文件中增加相关配置

```xml
<beans>
    ...
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager" >
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <tx:advice id="transactionAdvice" transaction-manager="transactionManager" >
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED" isolation="DEFAULT" rollback-for="java.lang.Exception" />
            <tx:method name="query*" read-only="true" />
        </tx:attributes>
    </tx:advice>    
    <aop:config>
        <aop:advisor advice-ref="transactionAdvice" pointcut="execution(* com.atguigu..*Service.*(..))"/>
    </aop:config>
    ...
</beans>
```

#### 测试前，需要在数据库中增加`atcrowdfunding`库及`t_user`表。

```
CREATE DATABASE `atcrowdfunding`;
...
CREATE TABLE `t_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8;
```



## 框架集成测试

框架集成完毕后，需要增加代码程序进行简单的测试。

增加代码前请参考**《阿里巴巴Java开发手册》**

在`src/main/java/com/atguigu/atcrowdfunding/controller`目录中增加`UserController`

```java
package com.atguigu.atcrowdfunding.controller;

import java.util.HashMap;
import java.util.Map;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/index")
    public String index() {
        return "user/index";
    }
    
    @ResponseBody
    @RequestMapping("/json")
    public Object json() {
        Map map = new HashMap();
        map.put("username", "张三");
        return map;
    }
}
```

将`web`项目发布到服务器中，启动服务器，浏览器中分别输入路径`http://127.0.0.1:8080/应用路径名称/user/index`，`http://127.0.0.1:8080/应用路径名称/user/json`进行测试。

#### 如果访问成功，说明项目中`SpringMVC`框架集成成功。

在`src/main/java/com/atguigu/atcrowdfunding/service`目录中增加`UserService`接口。

```java
package com.atguigu.atcrowdfunding.service;

public interface UserService {

}
```

在`src/main/java/com/atguigu/atcrowdfunding/service/impl`目录中增加`UserServiceImpl`实现类。

```java
package com.atguigu.atcrowdfunding.service.impl;

import com.atguigu.atcrowdfunding.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserDao userDao;
}
```

在`src/main/java/com/atguigu/atcrowdfunding/dao`目录中增加`UserDao`接口。

```java
package com.atguigu.atcrowdfunding.dao;

public interface UserDao {

}
```

修改`UserController`类，增加对`UserService`接口的引用。

```java
package com.atguigu.atcrowdfunding.controller;

import java.util.HashMap;
import java.util.Map;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;
    
    @RequestMapping("/index")
    public String index() {
        return "user/index";
    }
    
    @ResponseBody
    @RequestMapping("/json")
    public Object json() {
        Map map = new HashMap();
        map.put("username", "张三");
        return map;
    }
}
```

重启服务器，重新通过浏览器中访问路径`http://127.0.0.1:8080/应用路径名称/user/index`进行测试。

#### 如果访问成功，说明项目中`Spring`框架（`IOC`功能）集成成功。

在`src/main/java/com/atguigu/corwdfunding/bean`目录中增加`User`实体类。

```java
package com.atguigu.atcrowdfunding.bean;

public class User {

}
```

在`UserService`接口中增加方法声明`queryById`，并在`UserServiceImpl`类中默认实现

```java
package com.atguigu.atcrowdfunding.service;

import com.atguigu.atcrowdfunding.bean.User;
public interface UserService {
    public User queryById();
}
package com.atguigu.atcrowdfunding.service.impl;

import com.atguigu.atcrowdfunding.bean.User;
import com.atguigu.atcrowdfunding.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserDao userDao;
    
    public User queryById() {
        return userDao.queryById();
    }
}
```

#### 如果访问成功，说明项目中`Spring`框架（`AOP`功能）集成成功。

在`UserDao`中增加查询语句，实现数据库查询

```java
package com.atguigu.atcrowdfunding.dao;

import com.atguigu.atcrowdfunding.bean.User;
import org.apache.ibatis.annotations.Select;

public interface UserDao {
    
	@Select("select * from t_user where id = 1")
	public User queryById();
}
```

#### 如果访问成功，说明项目中`Mybatis`框架集成成功。



## 模拟生产环境

Web软件开发中，由于开发阶段不同，系统环境主要分为：开发环境，测试环境，生产环境。

将系统部署到生产环境后，经常会听开发人员这么说：这个`Bug`不应该呀，我们在测试时没问题呀，怎么到了生产环境就不行了呢!!!

其实之所以会出现这种情况，很大程度上是因为我们在项目的不同阶段时，对应用系统的访问方式不一样所造成的。

一般在开发，测试阶段时，我们都会以本地服务器地址`http://localhost:8080/project`访问系统，但是在生产环境中，我们的访问方式发生了变化：`http://com.xxxxx/`，由于环境的不同，导致访问方式的变化，那么就会产生很多之前没有的问题。

**如果能将开发，测试，生产环境的访问方式保持一致的话，那么就可以提前发现客户在使用时所发生的问题，将这些问题提前解决，还是非常不错的。**

-   将`Tomcat`服务器的默认`HTTP`监听端口号：`8080`修改为`80`
-   将项目中`.settings`目录下的配置文件`org.eclipse.wst.common.component`中的`context-root`属性修改为`/(斜杠)`
-   修改系统主机文件`c:\Windows\System32\drivers\etc\hosts`增加`IP`地址和域名的解析关系：`127.0.0.1 www.atcrowdfunding.com`