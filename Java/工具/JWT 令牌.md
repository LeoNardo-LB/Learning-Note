# JWT 令牌

## JWT简介

JWT是JSON Web Token的缩写，即JSON Web令牌，是一种自包含令牌。 

JWT的使用场景：

-   一种情况是webapi，类似之前的阿里云播放凭证的功能
-   另一种情况是多web服务器下实现无状态分布式身份验证

JWT 认证过程原理：

![0.921535905298511](_images/0.921535905298511.png)

步骤说明：

1.  浏览器发送登录请求，携带账号密码到Server
2.  服务器通过用户的账户密码等信息创建JWT令牌，并加密
3.  服务器将创建的JWT令牌返回给浏览器，浏览器保存起来
4.  浏览器再次访问服务器时，通过将JWT设置在 `Authorization Header` 中，来验证身份
5.  服务器检查JWT签名，如果签名符合，则从JWT令牌中获取用户信息
6.  服务器返回认证成功的信息给客户端。

JWT的作用：JWT 最重要的作用就是对 token信息的防伪作用

JWT的原理：

-   一个JWT由三个部分组成：JWT头、有效载荷、签名哈希
-   最后由这三者组合进行base64编码得到JWT



## JWT 结构

一个JWT对象：

```bash
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

![image-20201230190657409](_images/image-20201230190657409.png)

每一个子串表示了一个功能块，总共有以下三个部分：JWT头、有效载荷和签名

-   JWT头：描述JWT元数据的JSON对象，通常如下所示。

    ```json
    {
        "alg": "HS256",
        "typ": "JWT"
    }
    ```

    alg属性表示签名使用的算法，默认为HMAC SHA256（写为HS256）；typ属性表示令牌的类型，JWT令牌统一写为JWT。最后，使用Base64 URL算法将上述JSON对象转换为字符串保存。

-   有效载荷：有效载荷部分，是JWT的主体内容部分，也是一个JSON对象，包含需要传递的数据。 JWT内置七个默认字段供选择。

    ```bash
    iss: jwt签发者
    sub: 主题
    aud: 接收jwt的一方
    exp: jwt的过期时间，这个过期时间必须要大于签发时间
    nbf: 定义在什么时间之前，该jwt都是不可用的.
    iat: jwt的签发时间
    jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。
    ```

    可以自定义字段（私有字段）：

    ```json
    {
        "name": "Helen",
        "admin": true,
        "avatar": "helen.jpg"
    }
    ```

    默认情况下JWT是未加密的，任何人都可以解读其内容，因此不要构建隐私信息字段，存放保密信息，以防止信息泄露。

-   签名哈希

    签名哈希部分是对上面两部分数据签名，通过指定的算法生成哈希，以确保数据不会被篡改。

    首先，需要指定一个密码（secret）。该密码仅仅为保存在服务器中，并且不能向用户公开。然后，使用标头中指定的签名算法（默认情况下为HMAC SHA256）根据以下公式生成签名。

    ```bash
    HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(claims), secret)
    ```



## JWT 的用法

客户端接收服务器返回的JWT，将其存储在Cookie或localStorage中。

此后，客户端将在与服务器交互中都会带JWT。如果将它存储在Cookie中，就可以自动发送，但是不会跨域，因此一般是将它放入HTTP请求的Header Authorization字段中。

当跨域时，也可以将JWT放置于POST请求的数据主体中。

### Java 使用JWT

1、引入JWT依赖

```xml
<!-- JWT -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.7.0</version>
</dependency>
```

2、对JWT进行相关操作

```java
public class JwtTest {

    // 设置过期时间
    private static final long tokenExpiration = 1000 * 60 * 60 * 24;

    // 设置密钥
    private static final String secretKey = "123456";

    // 构建Jwt，使用 Jwts.builder() 方法
    @Test
    public void testBuiltJwt() {
        String compact = Jwts.builder()
            .setHeaderParam("alg", "HS256") //设置加密算法
            .setHeaderParam("typ", "jwt")   //设置类型（固定jwt）
            .setExpiration(new Date(System.currentTimeMillis() + tokenExpiration))
            .claim("nikename", "laonardo")	//设置jwtbody
            .claim("avatar", "1.jpg")	//设置jwtbody
            .signWith(SignatureAlgorithm.HS256, secretKey)	//设置加密算法
            .compact();	// 构建jwt，转为字符串
        System.out.println("compact = " + compact);
    }

    // 解析Jwt，使用 Jwts.parser().setSigningKey(<密钥>).parse(<jwt串>) 方法
    @Test
    public void testParseJwt() {
        // eyJhbGciOiJIUzI1NiIsInR5cCI6Imp3dCJ9.eyJleHAiOjE2MDk0MTQ2MTgsIm5pa2VuYW1lIjoibGFvbmFyZG8iLCJhdmF0YXIiOiIxLmpwZyJ9.Nx01kj_rugrfOY5PfSdbSRdu-PFdc8c7aELH7XwbbWk
        String jwt = "eyJhbGciOiJIUzI1NiIsInR5cCI6Imp3dCJ9.eyJleHAiOjE2MDk0MTQ2MTgsIm5pa2VuYW1lIjoibGFvbmFyZG8iLCJhdmF0YXIiOiIxLmpwZyJ9.Nx01kj_rugrfOY5PfSdbSRdu-PFdc8c7aELH7XwbbWk";
        String jwt2 = "eyJhbGciOiJIUzI1NiIsInR5cGUiOiJqd3QifQ.eyJleHAiOjE2MDk0MTQ3NDksIm5pa2VuYW1lIjoibGFvbmFyZG8iLCJhdmF0YXIiOiIxLmpwZyJ9.wb8ioudqiN0hJ7BXGSuN_zFT4GEWACA4dacgoAjbAe4";
        Jwt parse = Jwts.parser().setSigningKey(secretKey).parse(jwt2);

        System.out.println("parse = " + parse); //header={alg=HS256, typ=jwt},body={exp=1609414749, nikename=laonardo, avatar=1.jpg},signature=wb8ioudqiN0hJ7BXGSuN_zFT4GEWACA4dacgoAjbAe4

        Object body = parse.getBody();
        Header header = parse.getHeader();
        // 查看类型
        System.out.println("body.getClass() = " + body.getClass());
        System.out.println("header.getClass() = " + header.getClass());
        // 查看输出内容
        System.out.println("body = " + body);
        System.out.println("header = " + header);

        // 输出查看jwt的body内容
        Claims cBody = (Claims) body;
        Date exp = cBody.get("exp", Date.class);
        // 使用get获取指定内容
        String nikename = cBody.get("nikename", String.class);
        String avatar = cBody.get("avatar", String.class);
        System.out.println("exp = " + exp);
        System.out.println("nikename = " + nikename);
        System.out.println("avatar = " + avatar);

        // 输出查看jwt的header内容
        Object alg = header.get("alg");
        String algorithm = header.getCompressionAlgorithm();
        System.out.println("alg = " + alg); // HS256
        System.out.println("algorithm = " + algorithm); // null
    }

}
```