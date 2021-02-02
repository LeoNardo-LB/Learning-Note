# Dubbo & Zookeeper

## Dubbo 简介

Dubbo官网地址：http://dubbo.apache.org

Apache Dubbo是一款高性能的Java RPC框架。其前身是阿里巴巴公司开源的一个高性能、轻量级的开源Java RPC框架，可以和Spring框架无缝集成。

![img](_images/dubbo-service-governance.jpg)

RPC：RPC全称为remote procedure call，即**远程过程调用**。比如两台服务器A和B，A服务器上部署一个应用，B服务器上部署一个应用，A服务器上的应用想调用B服务器上的应用提供的方法，由于两个应用不在一个内存空间，不能直接调用，所以需要通过网络来表达调用的语义和传达调用的数据。需要注意的是RPC并不是一个具体的技术，而是指整个网络远程调用过程。

### Dubbo的三大核心能力

1.  面向接口的远程方法调用
2.  智能容错和负载均衡
3.  以及服务自动注册和发现。

### Dubbo 的架构

![img](_images/dubbo-architecture.jpg)

**节点角色说明**

| 节点        | 角色说明                               |
| ----------- | -------------------------------------- |
| `Provider`  | 暴露服务的服务提供方                   |
| `Consumer`  | 调用远程服务的服务消费方               |
| `Registry`  | 服务注册与发现的注册中心               |
| `Monitor`   | 统计服务的调用次数和调用时间的监控中心 |
| `Container` | 服务运行容器                           |

##### 调用关系说明

1.  服务容器负责启动，加载，运行服务提供者。
2.  服务提供者在启动时，向注册中心注册自己提供的服务。
3.  服务消费者在启动时，向注册中心订阅自己所需的服务。
4.  注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
5.  服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
6.  服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

Dubbo 架构具有以下几个特点，分别是连通性、健壮性、伸缩性、以及向未来架构的升级性。



## Zookeeper 简介

下载地址：http://archive.apache.org/dist/zookeeper/

通过前面的Dubbo架构图可以看到，Registry（服务注册中心）在其中起着至关重要的作用。Dubbo官方推荐使用Zookeeper作为服务注册中心。

Zookeeper 是 Apache Hadoop 的子项目，是一个树型的目录服务，支持变更推送，适合作为 Dubbo 服务的注册中心，工业强度较高，可用于生产环境。

![zookeeper](_images/zookeeper.png)

流程说明：

-   服务提供者(Provider)启动时: 向 `/dubbo/com.foo.BarService/providers` 目录下写入自己的 URL 地址
-   服务消费者(Consumer)启动时: 订阅 `/dubbo/com.foo.BarService/providers` 目录下的提供者 URL 地址。并向 `/dubbo/com.foo.BarService/consumers` 目录下写入自己的 URL 地址
-   监控中心(Monitor)启动时: 订阅 `/dubbo/com.foo.BarService` 目录下的所有提供者和消费者 URL 地址



## 快速开始

Dubbo 配合Zookeeper，可以采用 Spring 配置方式，透明化接入应用，对应用没有任何 API 侵入，只需用 Spring 加载 Dubbo 的配置即可。

有三个主体步骤需要实现：

1.  打开Zookeeper
2.  引入依赖
3.  编写服务提供方与服务消费方
4.  启动服务提供方与服务消费方
5.  测试即可

### 1、打开zookeeper

将zookeeper部署在Linux服务器上，只需要 如下几个步骤：

1.  解压zookeeper

2.  配置环境变量

3.  将zookeeper目录下，`conf/zoo-sample.cfg` 文件 复制一份，重命名为 `zoo.cfg` 

4.  输入启动命令：

    ```bash
    zkServer.sh start 
    ```

    >   常用命令说明：
    >
    >   `start`：zookeeper后台启动
    >
    >   `start-foreground`：zookeeper前台启动
    >
    >   `stop`：关闭zookeeper
    >
    >   `status`：查看zookeeper状态

### 2、引入依赖

将Spring作为容器，只需要在Spring的基础上引入 Dubbo 与 Zookeeper的依赖即可：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo</artifactId>
    <version>2.6.0</version>
</dependency>
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.7</version>
</dependency>
<dependency>
    <groupId>com.github.sgroschupf</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.1</version>
</dependency>
<dependency>
    <groupId>javassist</groupId>
    <artifactId>javassist</artifactId>
    <version>3.12.1.GA</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.47</version>
</dependency>
```

### 3、编写服务提供方与服务消费方

#### 服务提供方

1.  定义服务接口

    该接口需单独打包，在服务提供方和消费方共享

    DemoService.java：

    ```java
    package org.apache.dubbo.demo;
    
    public interface DemoService {
        String sayHello(String name);
    }
    ```

2.  在服务提供方实现接口

    对服务消费方隐藏实现

    DemoServiceImpl.java：

    ```java
    package org.apache.dubbo.demo.provider;
     
    import org.apache.dubbo.demo.DemoService;
     
    public class DemoServiceImpl implements DemoService {
        public String sayHello(String name) {
            return "Hello " + name;
        }
    }
    ```

3.  用 Spring 配置声明暴露服务

    provider.xml：

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
        xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
     
        <!-- 提供方应用信息，用于计算依赖关系 -->
        <dubbo:application name="hello-world-app"  />
     
        <!-- 使用Zookeeper注册中心暴露服务地址 -->
        <dubbo:registry address="zookeeper://192.168.72.132:2181" />
     
        <!-- 用dubbo协议在20880端口暴露服务 -->
        <dubbo:protocol name="dubbo" port="20880" />
        
        <!-- 声明需要暴露的服务接口 -->
        <dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService" />
        <!-- 和本地bean一样实现服务 -->
        <bean id="demoService" class="org.apache.dubbo.demo.provider.DemoServiceImpl" />
    </beans>
    ```

    >后两个声明暴露接口的标签可以使用注解驱动的方式代替： `<dubbo:annotation package="<包路径>"/> `，然后再相应需要暴露的方法类上加入 Dubbo的 `@Service`注解即可。

4.  加载 Spring 配置

    Provider.java：

    ```java
    import org.springframework.context.support.ClassPathXmlApplicationContext;
     
    public class Provider {
        public static void main(String[] args) throws Exception {
            ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"META-INF/spring/dubbo-demo-provider.xml"});
            context.start();
            System.in.read(); // 按任意键退出
        }
    }
    ```

#### 服务消费者

1.  通过 Spring 配置引用远程服务

    consumer.xml：

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
        xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
     
        <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
        <dubbo:application name="consumer-of-helloworld-app"  />
     
        <!-- 使用zookeeper注册中心暴露发现服务地址 -->
        <dubbo:registry address="zookeeper://192.168.72.132" />
     
        <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
        <dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService" />
    </beans>
    ```

    >最后一个生成远程服务代理的标签可以由注解代替：`<dubbo:annotation package="<包路径>"/> ` ，然后使用 Dubbo的 `@Reference` 注解标注在需要调用的远程服务，调用远程服务自动注入即可。

2.  加载Spring配置，并调用远程服务

    也可以使用 IoC 注入

    Consumer.java：

    ```java
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    import org.apache.dubbo.demo.DemoService;
     
    public class Consumer {
        public static void main(String[] args) throws Exception {
           ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"META-INF/spring/dubbo-demo-consumer.xml"});
            context.start();
            DemoService demoService = (DemoService)context.getBean("demoService"); // 获取远程服务代理
            String hello = demoService.sayHello("world"); // 执行远程方法
            System.out.println( hello ); // 显示调用结果
        }
    }
    ```



## Dubbo相关配置说明

### 包扫描

```xml
<dubbo:annotation package="com.maweiqi.service" />
```

服务提供者和服务消费者都需要配置，表示包扫描，作用是扫描指定包(包括子包)下的类。

如果不使用包扫描，也可以通过如下配置的方式来发布服务：

```xml
<bean id="helloService" class="com.maweiqi.service.impl.HelloServiceImpl" />
<dubbo:service interface="com.maweiqi.api.HelloService" ref="helloService" />
```

作为服务消费者，可以通过如下配置来引用服务：

```xml
<!-- 生成远程服务代理，可以和本地bean一样使用helloService -->
<dubbo:reference id="helloService" interface="com.maweiqi.api.HelloService" />
```

上面这种方式发布和引用服务，一个配置项(<dubbo:service>、<dubbo:reference>)只能发布或者引用一个服务，如果有多个服务，这种方式就比较繁琐了。推荐使用包扫描方式。

### 协议

```xml
<dubbo:protocol name="dubbo" port="20880"/>
```

一般在服务提供者一方配置，可以指定使用的协议名称和端口号。

其中Dubbo支持的协议有：dubbo、rmi、hessian、http、webservice、rest、redis等。

**推荐使用dubbo协议。**

dubbo 协议采用单一长连接和 NIO 异步通讯，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况。不适合传送大数据量的服务，比如传文件，传视频等，除非请求量很低。

也可以在同一个工程中配置多个协议，不同服务可以使用不同的协议，例如：

```xml
<!-- 多协议配置 -->
<dubbo:protocol name="dubbo" port="20880" />
<dubbo:protocol name="rmi" port="1099" />
<!-- 使用dubbo协议暴露服务 -->
<dubbo:service interface="com.maweiqi.api.HelloService" ref="helloService" protocol="dubbo" />
<!-- 使用rmi协议暴露服务 -->
<dubbo:service interface="com.maweiqi.api.DemoService" ref="demoService" protocol="rmi" /> 
```

### 启动时检查

```xml
<dubbo:consumer check="false"/>
```

上面这个配置需要配置在服务消费者一方，如果不配置默认check值为true。Dubbo 缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止 Spring 初始化完成，以便上线时，能及早发现问题。可以通过将check值改为false来关闭检查。

建议在开发阶段将check值设置为false，在生产环境下改为true。

### 负载均衡

负载均衡（Load Balance）：其实就是将请求分摊到多个操作单元上进行执行，从而共同完成工作任务。

在集群负载均衡时，Dubbo 提供了多种均衡策略（包括随机、轮询、最少活跃调用数、一致性Hash），缺省为random随机调用。

配置负载均衡策略，既可以在服务提供者一方配置，也可以在服务消费者一方配置，如下：

```java
@Controller
@RequestMapping("/demo")
public class HelloController {
    //在服务消费者一方配置负载均衡策略
    @Reference(check = false,loadbalance = "random")
    private HelloService helloService;

    @RequestMapping("/hello")
    @ResponseBody
    public String getName(String name){
        //远程调用
        String result = helloService.sayHello(name);
        System.out.println(result);
        return result;
    }
}

//在服务提供者一方配置负载均衡
@Service(loadbalance = "random")
public class HelloServiceImpl implements HelloService {
    public String sayHello(String name) {
        return "hello " + name;
    }
}
```

可以通过启动多个服务提供者来观察Dubbo负载均衡效果。

>   注意：因为我们是在一台机器上启动多个服务提供者，所以需要修改tomcat的端口号和Dubbo服务的端口号来防止端口冲突。
>
>   （在实际生产环境中，多个服务提供者是分别部署在不同的机器上，所以不存在端口冲突问题。）



## 常见问题

### 1、事务代理Dubbo的Service

**问题描述：**

通过入门案例我们可以看到通过Dubbo提供的标签配置就可以进行包扫描，扫描到@Service注解的类就可以被发布为服务。

但是我们如果在服务提供者类上加入@Transactional事务控制注解后，服务就发布不成功了。原因是事务控制的底层原理是为服务提供者类创建代理对象，而默认情况下Spring是基于JDK动态代理方式创建代理对象，而此代理对象的完整类名为com.sun.proxy.$Proxy42（最后两位数字不是固定的），导致Dubbo在发布服务前进行包匹配时无法完成匹配，进而没有进行服务的发布。

**解决方案：**

1.  修改applicationContext-service.xml配置文件，开启事务控制注解支持时指定proxy-target-class属性，值为true。其作用是使用cglib代理方式为Service类创建代理对象,添加如下配置：

    ```xml
    <!--开启事务控制的注解支持-->
    <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>
    ```

2.  修改HelloServiceImpl类，在Service注解中加入interfaceClass属性，值为HelloService.class，作用是指定服务的接口类型

    ```java
    import com.alibaba.dubbo.config.annotation.Service;
    import com.maweiqi.service.HelloService;
    import org.springframework.transaction.annotation.Transactional;
    
    // 使接口类为被实现的类
    @Service(interfaceClass = HelloService.class)
    @Transactional
    public class HelloServiceImpl implements HelloService {
        @Override
        public String sayHello(String name) {
            return "8086 hello " + name;
        }
    }
    ```

    

