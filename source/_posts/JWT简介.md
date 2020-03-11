---
title: JWT简介
tags:
  - JWT
categories:
  - Dev
  - Java
date: 2019-12-21 08:43:00
cover: true

---

![](http://q6pznk9ej.bkt.clouddn.com/img%20%2811%29.jpeg)
<!-- more -->


## 前言
JSON Web Token（JWT）是目前最流行的跨域身份验证解决方案。[微服务常见的认证方案](https://www.jianshu.com/p/c007b8021d05)
## 一、跨域认证的问题
互联网服务离不开用户认证。一般流程是下面这样。

* 1、用户向服务器发送用户名和密码。

* 2、服务器验证通过后，在当前对话（session）里面保存相关数据，比如用户角色、登录时间等等。

* 3、服务器向用户返回一个 session_id，写入用户的 Cookie。

* 4、用户随后的每一次请求，都会通过 Cookie，将 session_id 传回服务器。

* 5、服务器收到 session_id，找到前期保存的数据，由此得知用户的身份。

这种模式的问题在于，扩展性（scaling）不好。单机当然没有问题，如果是服务器集群，或者是跨域的服务导向架构，就要求 session 数据共享，每台服务器都能够读取 session。

一种解决方案是 session 数据持久化，写入数据库或别的持久层。各种服务收到请求后，都向持久层请求数据。这种方案的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败。

另一种方案是服务器索性不保存 session 数据了，所有数据都保存在客户端，每次请求都发回服务器。JWT 就是这种方案的一个代表。

什么是JWT：一句话概括就是（通过客户端保存数据，而服务器根本不保存会话数据，每个请求都被发送回服务器。）
## 二、JWT
JSON Web Token（JWT）是一个非常轻巧的规范。这个规范允许我们使用JWT在用户和服务器之间传递安全可靠的信息。

一个JWT实际上就是一个字符串，它由三部分组成，头部、载荷与签名。
### 1、JWT的原则
JWT的原则是在服务器身份验证之后，将生成一个JSON对象并将其发送回用户，如下所示。
```
{

     "UserName": "少年闰土",

    "Role": "Admin",

    "Expire": "2019-12-21 09:15:56"

}
```
以后，用户与服务端通信的时候，都要发回这个 JSON 对象。服务器完全只靠这个对象认定用户身份。为了防止用户篡改数据，服务器在生成这个对象的时候，会加上签名。
### 2、JWT的数据结构
样例：
```
eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI4ODgiLCJzdWIiOiLlsI_nmb0iLCJpYXQiOjE1NTc5MDU4MDIsImV4cCI6MTU1NzkwNjgwMiwicm9sZXMiOiJhZG1pbiJ9.AS5Y2fNCwUzQQxXh_QQWMpaB75YqfuK-2P7VZiCXEJI
```
他是一个长字符串，中间用`.`进行分割，代表JWT的三个组成部分，如下：

* Header（头部）

* Payload（负载）
* Signature（签名）
![图片来自网络-仅供参考.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0xNGFkNzRiMDY3ZTI3YmQxLnBuZw?x-oss-process=image/format,png)
#### 2.1、头部（Header）
头部用于描述关于该JWT的最基本的信息，例如其类型以及签名所用的算法等。这也可以被表示成一个JSON对象。
`{"typ":"JWT","alg":"HS256"}`

这个json中的typ属性，用来标识整个token字符串是一个JWT字符串；它的alg属性，用来说明这个JWT签发的时候所使用的签名和摘要算法。typ跟alg属性的全称其实是type跟algorithm，分别是类型跟算法的意思。之所以都用三个字母来表示，也是基于JWT最终字串大小的考虑，同时也是跟JWT这个名称保持一致，这样就都是三个字符了…typ跟alg是JWT中标准中规定的属性名称

在头部指明了签名算法是HS256算法。 我们进行BASE64编码[http://base64.xpcha.com/](https://links.jianshu.com/go?to=http%3A%2F%2Fbase64.xpcha.com%2F)，编码后的字符串如下：
`eyJhbGciOiJIUzI1NiJ9`

#### 2.2、载荷（Playload）
Payload 部分也是一个 JSON 对象，用来存放实际需要传递的数据。JWT 规定了7个官方字段，供选用。
```
iss: jwt签发者
sub: jwt所面向的用户
aud: 接收jwt的一方
exp: jwt的过期时间，这个过期时间必须要大于签发时间
nbf: 定义在什么时间之前，该jwt都是不可用的.
iat: jwt的签发时间
jti: jwt的唯一身份标识，主要用来作为一次性token。
```
除了官方字段，你还可以在这个部分定义私有字段
样例：
`{"sub":"1234567890","name":"John Doe","admin":true}`
然后将其进行base64加密，得到Jwt的第二部分。
`eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9`
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1kNDM2MzY2NGU3ZTY3NjA1LnBuZw?x-oss-process=image/format,png)

#### 2.3、签名（Signature）
Signature 部分是对前两部分的签名，防止数据篡改。这个签证信息由三部分组成：
>header (base64后的)
 payload (base64后的)
secret

首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。这个部分需要base64加密后的header和base64加密后的payload使用.连接组成的字符串，然后通过header中声明的加密方式进行加盐secret组合加密，然后就构成了jwt的第三部分。
```
    HMACSHA256(
      base64UrlEncode(header) + "." +
      base64UrlEncode(payload),
      secret)
```
### 3、Base64URL
前面提到，Header 和 Payload 串型化的算法是 Base64URL。这个算法跟 Base64 算法基本类似，但有一些小的不同。

JWT 作为一个令牌（token），有些场合可能会放到 URL（比如 api.example.com/?token=xxx）。Base64 有三个字符+、/和=，在 URL 里面有特殊含义，所以要被替换掉：=被省略、+替换成-，/替换成_ 。这就是 Base64URL 算法。 
### 4、JWT 的使用方式
客户端收到服务器返回的 JWT，可以储存在 Cookie 里面，也可以储存在 localStorage。

此后，客户端每次与服务器通信，都要带上这个 JWT。你可以把它放在 Cookie 里面自动发送，但是这样不能跨域，所以更好的做法是放在 HTTP 请求的头信息Authorization字段里面。
`Authorization: Bearer <token>`

下图显示了如何获取JWT并将其用于访问API或资源：
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS05MDk3YjI4MDFlYTZjMTlmLnBuZw?x-oss-process=image/format,png)

## 三、JWT使用
### 1、添加依赖
```
       <dependency>
            <groupId>com.auth0</groupId>
            <artifactId>java-jwt</artifactId>
            <version>3.2.0</version>
        </dependency>
```
### 2、工具类

```
package com.example.demo.utils;

import com.auth0.jwt.JWT;
import com.auth0.jwt.JWTVerifier;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.exceptions.JWTDecodeException;
import com.auth0.jwt.interfaces.DecodedJWT;
import org.springframework.stereotype.Component;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

/**
 * @Author: 少年闰土
 * @Date: 2019/12/11 
 * @Time: 下午 4:12
 * @Version: v1.0
 * jwt工具类
 */
@Component
public class JwtUtils {

    /**
     * 解析token
     *
     * @param token token
     * @return 用户名
     */
    public static String getUserName(String token) {
        try {
            DecodedJWT jwt = JWT.decode(token);
            return jwt.getClaim("userName").asString();
        } catch (JWTDecodeException e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 签发token
     *
     * @param userName 用户名
     * @return token
     */
    public static String sign(String userName,String secret) {
        try {
            //token过期时间
            Date date = new Date(System.currentTimeMillis() + (60 * 60 * 1000));
            Algorithm algorithm = Algorithm.HMAC256(secret);
            // 附带username信息
            return JWT.create()
                    .withClaim("userName", userName)
                    .withExpiresAt(date)
                    .sign(algorithm);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 检验token是否过期
     *
     * @param token
     * @return
     */
    public static Map verify(String token,String userName, String secret) {
        Map result = new HashMap<String, Object>(2);
        try {
            Algorithm algorithm = Algorithm.HMAC256(secret);
            JWTVerifier verifier = JWT.require(algorithm)
                    .withClaim("userName", userName)
                    .build();
            DecodedJWT jwt = verifier.verify(token);
            result.put("isSuccess", true);
            result.put("exception", null);
        } catch (Exception exception) {
            result.put("isSuccess", false);
            result.put("exception", exception);
        }
        return result;
    }
}

```
### 3、使用
```
    @ApiOperation(value = "浏览器点击登录")
    @ApiImplicitParam(name = "user", value = "用户实体", required = true, paramType = "User")
    @PostMapping("/login")
    public R login(@RequestBody User user) {
        log.debug("------浏览器点击登录------");
        String userName = user.getUsername();
        String passWord = user.getPassword();
        User u = this.userService.getUser(userName);
        String passWordSalt = MD5.md5Salt(passWord, userName);
        if (u != null && u.getPassword().equals(passWordSalt)) {
            String token = JwtUtils.sign(userName, passWordSalt);
            return R.ok(R.SUCCESS, R.MSG_SUCCESS, token);
        } else {
            return R.error(R.MSG_LOGIN_ERROR);
        }
    }
```




