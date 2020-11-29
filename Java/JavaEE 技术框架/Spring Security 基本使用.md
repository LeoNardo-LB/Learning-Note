# Spring Security

## Spring Security简介

Spring Security，这是一种基于 Spring AOP 和 Servlet 过滤器的安全框架。它提供全面的安全性解决方案，同时在 Web 请求级和方法调用级处理身份确认和授权。

Spring Security 命名空间的引入可以简化我们的开发，它涵盖了大部分 Spring Security 常用的功能。它的设计是基于框架内大范围的依赖的，可以被划分为以下几块。

-   Web/Http 安全：这是最复杂的部分。通过建立 filter 和相关的 service bean 来实现框架的认证机制。当访问受保护的 URL 时会将用户引入登录界面或者是错误提示界面。
-   业务对象或者方法的安全：控制方法访问权限的。
-   AuthenticationManager：处理来自于框架其他部分的认证请求。
-   AccessDecisionManager：为 Web 或方法的安全提供访问决策。会注册一个默认的，但是我们也可以通过普通 bean 注册的方式使用自定义的 AccessDecisionManager。
-   AuthenticationProvider：AuthenticationManager 是通过它来认证用户的。
-   UserDetailsService：跟 AuthenticationProvider 关系密切，用来获取用户信息的。



## Spring Security 与 Spring框架整合

SpringSecurity常用于与Spring框架整合.

### Quick Start

此处只说明SpringSecurity需要配置的内容。

最基本的权限管理的实现步骤如下：

1.  引入SpringWeb与SpringSecurity的jar包，采用Maven形式

    ```xml
    <!--SpringSecurity的Jar包-->
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-web</artifactId>
        <version>5.0.5.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-config</artifactId>
        <version>5.0.5.RELEASE</version>
    </dependency>
    ```

2.  配置web.xml，添加名为springSecurityFilterChain的委托过滤器代理（固定写死，不可更改）

    ```xml
    <filter>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        <filter-name>springSecurityFilterChain</filter-name>
    </filter>
    <!-- 这里拦截了所有请求 -->
    <filter-mapping>
        <filter-name>springSecurityFilterChain</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ```

3.  在Spring配置文件中配置SpringSecurity，最基础的配置如下

    ```xml
    <!--http 标签用于定义相关权限控制
        可以设置none表示不被 SpringSecurity 接管
    -->
    <security:http security="none" pattern="/static/**"/>
    <security:http security="none" pattern="/login.html"/>
    
    <!--配置拦截路径：
        - auto-config：自动配置，使用SpringSecurity提供的登录、登出的url及页面
        - use-expressions：使用表达式来配置
    -->
    <security:http auto-config="true" use-expressions="true">
        <!--security:intercept-url：配置拦截地址
            - pattern：配置拦截的url路径
            - access：指定访问需要的的角色及权限，常用的如下
                - isAuthenticated()：需要已认证（登录）
                - hasRole('XXX')：需要拥有XXX角色
                - hasAuthority('XXX|ROLE_XXX')：是否有权限或角色，加了ROLE_前缀则表示要求角色，反之则为权限。
                - denyAll()：不允许访问，全部拒绝
        -->
        <!--注意：网页过滤查找权限的顺序为声明顺序，自上而下-->
        <security:intercept-url pattern="/index.html" access="isAuthenticated()"/>
        <security:intercept-url pattern="/a.html" access="hasRole('a')"/>
        <security:intercept-url pattern="/b.html" access="hasAuthority('intoB')"/>
        <security:intercept-url pattern="/c.html" access="denyAll()"/>
        <security:intercept-url pattern="/**" access="hasRole('ROLE_ADMIN')"></security:intercept-url>
    </security:http>
    
    <!--security:authentication-manager：认证管理器，通过该标签来管理认证，授权等信息-->
    <security:authentication-manager>
        <!--security:authentication-provider，认证提供者，可以配置多个。
            用于管理user-service，一般由外部自定义引入（实现UserDetails接口）。这里只是简单演示，因此直接内部配置即可。
    		- user-service-ref：用于引用实现了UserDetails接口的类（该类用于配置权限信息）
        -->
        <security:authentication-provider>
            <!--security:user-service：具体用户管理的标签，可在内部配置临时的用户名、密码及权限授权信息-->
            <security:user-service>
                <!-- {noop}用于取消加密措施，临时作用 -->
                <security:user name="admin" authorities="ROLE_ADMIN" password="{noop}admin"></security:user>
                <security:user name="xiaoming" authorities="ROLE_a,intoB" password="{noop}xiaoming"></security:user>
            </security:user-service>
        </security:authentication-provider>
    </security:authentication-manager>
    ```

### 退出登录

用户完成登录后Spring Security框架会记录当前用户认证状态为已认证状态，即表示用户登录成功了。如果需要退出登录（一般都需要），则可以在可以在配置文件中开启。

```xml
<security:logout logout-url="/logout.do" logout-success-url="/login.html" invalidate-session="true"/>
```

参数说明：

-   logout-url：执行退出登录的 url
-   logout-success-url：成功退出跳转的页面 url
-   invalidate-session：控制是否使session无效

### 自定义登录表单

可以在 `<security:http auto-config="true" use-expressions="true"> ... </security:http>` 内部使用 `<security:form-login` 标签来自定义登录表单。

| form-login 属性 | 说明 |
| --------------- | ---- |
| login-page |表示指定登录页面|
| username-parameter |使用登录名的名称，默认值是username|
| password-parameter |使用登录名的密码，默认值是password|
| login-processing-url |登录表单提交的url地址|
| default-target-url |登录成功后的url地址|
| always-use-default-target="true"|登录成功后，始终跳转到default-target-url指定的地址，即登录成功的默认地址|
| authentication-failure-url |认证失败后跳转的url地址，失败后指定/login.html|

**代码示例**

SpringSecurity配置文件的表单配置如下：

```xml
<security:http auto-config="true" use-expressions="true">
    <security:intercept-url pattern="/index.html" access="isAuthenticated()"/>

    <!--SpringSecurity 自定义表单-->
    <security:form-login login-page="/myLoginPage.html"
                         username-parameter="loginAccount"
                         password-parameter="loginPassword"
                         login-processing-url="/login"
                         default-target-url="/index.html"
                         authentication-failure-url="/loginFail.html"
                         always-use-default-target="true"/>

    <!--关闭盗链安全请求-->
    <security:csrf disabled="true" />

</security:http>
```

>   只允许由一个 `form-login` 标签

自定义的页面如下：

```html
<!-- 自定义的登录页面 -->
<form action="/login" method="post">
    请输入你的账号：<input type="text" name="loginAccount"> <br>
    请输入你的密码：<input type="password" name="loginPassword"> <br>
    <input type="submit" value="点击登录">
</form>
```

>   注意需要使用post提交

### 后端处理用户名密码

前面的xml配置用户名密码不常用，通常用户名密码会发送到Handler，并交由后端进行处理。

步骤如下：

1.  由后端定义一个类，实现 `UserDetails` 接口中的 `UserDetails loadUserByUsername(String username)` 方法。

    -   此方法传入的username参数即为xml中规定的，表达中与之对应的name值。可以理解为用户名（principal）
    -   返回值：返回一个SpringSecurity提供的User对象，该对象封装了Principal、credential与角色及授权信息。

    **代码示例**

    ```java
    @Component
    public class LoginService implements UserDetailsService {
    
        // 模拟从数据库中取数据，key为查询条件，value为返回的pojo对象
        static Map<String, Account> accountMap = new HashMap<>();
    
        static {
            // 下面三个密码是经过 BcryptPasswordEncoding 编码后的数据
            accountMap.put("leonardo", new Account("leonardo", "$2a$10$sCNayRNSbKqbYPYHVX0kFOGNrLAytlJaq3mHYHZW3yr2UioIbThw2"));
            accountMap.put("xxt", new Account("xxt", "$2a$10$0a49xT1ta2YSKS3ZPZoSmO3Wg5lU5e9p5/JRh3ve6U6u27YxDzUiG"));
            accountMap.put("xiaotongtong", new Account("xiaotongtong", "$2a$10$wnGzXNOq3EQn/8bENWh0re3ZiO3Y31QiXX8eHhC3Ytecoh2ATuWfC"));
        }
    
        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            Account account = accountMap.get(username);
    
            List<GrantedAuthority> list = new ArrayList<>();
    
            // 模拟从数据库查询出的权限
            if (account.getUsername().equalsIgnoreCase("leonardo")) {
                list.add(new SimpleGrantedAuthority("APage"));
                list.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
            }
    
            return new User(account.getUsername(), account.getPassword(), list);
        }
    ```

    >   注意：要将该类加入IOC容器中（配置包扫描及标注@Component注解，以便交由SpringContext管理

2.  在SpringSecurity配置文件中配置编码器、验证的类（上面那个）

    ```xml
    <!-- 配置编码器，一般用SpringSecurity提供的BCryptPasswordEncoder -->
    <bean id="passwordEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder"/>
    
    <!-- 在认证管理器的授权提供者引用 -->
    <security:authentication-manager>
    	<!-- 引用刚配好的UserDetails的实现类 -->
        <security:authentication-provider user-service-ref="loginService">
            <!-- 指定编码器 -->
            <security:password-encoder ref="passwordEncoder"/>
        </security:authentication-provider>
    </security:authentication-manager>
    ```

### 访问权限的细粒化控制

#### 1、SpringSecurity配置文件控制 uri 资源的访问

在 `<security:http auto-config="true" use-expressions="true">` 标签内使用 `<security:intercept-url` 可以对uri资源进行细粒化控制。一般用于控制对精确匹配或模糊匹配的**页面资源**进行访问控制，访问遍历顺序从上到下。

-   pattern：uri的匹配模式
-   access：访问控制，指定访问的角色或权限及其他访问控制

```xml
<security:http auto-config="true" use-expressions="true">
    <!--<security:intercept-url pattern="/**" access="hasRole('ROLE_ADMIN')"></security:intercept-url>-->
    <!--只要认证通过就可以访问-->
    <security:intercept-url pattern="/index.html"  access="isAuthenticated()" />
    <security:intercept-url pattern="/a.html"  access="isAuthenticated()" />

    <!--拥有add权限就可以访问b.html页面-->
    <security:intercept-url pattern="/b.html"  access="hasAuthority('add')" />

    <!--拥有ROLE_ABC角色就可以访问d.html页面-->
    <security:intercept-url pattern="/d.html"  access="hasRole('ABC')" />
    
    <!-- 拒绝全部访问 -->
    <security:intercept-url pattern="/c.html" access="denyAll()"/>
</security:http
```

#### 2、注解控制方法级别的访问

在Controller类中的Handler方法上标注 `@PreAuthorize` 注解可以对访问的角色及权限进行控制。

使用步骤：

1.  在SpringSecurity配置文件中开启注解控制

    ```xml
    <!--开启注解方式权限控制-->
    <security:global-method-security pre-post-annotations="enabled" />
    ```

2.  在Controller中的方法上标注 `PreAuthorize()` 注解

    ```java
    @RestController
    @RequestMapping("/hello")
    public class HelloController {
    
        @RequestMapping("/add")
        @PreAuthorize("hasAuthority('add')")//表示用户必须拥有add权限才能调用当前方法
        public String add(){
            System.out.println("add...");
            return "success";
        }
    
        @RequestMapping("/update")
        @PreAuthorize("hasRole('ROLE_ADMIN')")//表示用户必须拥有ROLE_ADMIN角色才能调用当前方法
        public String update(){
            System.out.println("update...");
            return "success";
        }
    
        @RequestMapping("/delete")
        @PreAuthorize("hasRole('ABC')")//表示用户必须拥有ABC角色才能调用当前方法
        public String delete(){
            System.out.println("delete...");
            return "success";
        }
    }
    ```

     

