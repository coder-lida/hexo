title: shiro 加密登录 密码加盐处理
tags:
  - shiro
categories:
  - 技术
date: 2019-07-26 11:25:00
cover: true

---
![](http://q6pznk9ej.bkt.clouddn.com/img%20%2814%29.jpeg)
<!-- more -->

密码加密登录是为了提高系统安全性，即使是管理员查看数据库也得不到密码
使用shiro可以很轻松的完成加密及登录操作

## 1.加密工具

`此工具用于注册时对密码进行加密`
```
public static final String md5(String password, String salt){
    //加密方式
    String hashAlgorithmName = "MD5";
    //盐：为了即使相同的密码不同的盐加密后的结果也不同
    ByteSource byteSalt = ByteSource.Util.bytes(salt);
    //密码
    Object source = password;
    //加密次数
    int hashIterations = 1024;
    SimpleHash result = new SimpleHash(hashAlgorithmName, source, byteSalt, hashIterations);
    return result.toString();
}
```
`测试一下`
```
public static void main(String[] args) {
    String password = md5("123456", "WHLH");
    System.out.println(password);
    //加密后的结果
    //3bcbb857c763d1429a24959cb8de2593
}
```
## 2.使用shiro登录
`Realm类`
```
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) {
    UsernamePasswordToken token=(UsernamePasswordToken) authenticationToken;
    String username = token.getUsername();
    //查询用户信息
    User user=userService.findByUsername(username);
    //取出盐并编码
    ByteSource salt = ByteSource.Util.bytes(user.getSalt());
    return new SimpleAuthenticationInfo(username, user.getPassword(),salt, getName());
}
```
## 3.修改自定义realm配置

`加密算法和加密次数要和加密工具参数保持一致`
```
<bean id="myRealm" class="cn.jaffreyen.web.shiro.MyRealm">
    <property name="credentialsMatcher">
        <bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
            <!-- 加密算法 -->
            <property name="hashAlgorithmName" value="MD5"></property>
            <!-- 加密次数 -->
            <property name="hashIterations" value="1024"></property>
        </bean>
    </property>
</bean>
```