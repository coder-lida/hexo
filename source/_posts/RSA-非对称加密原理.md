title: RSA 非对称加密原理
tags:
  - 加密解密
categories:
  - 技术
date: 2020-03-05 14:22:00
cover: true

---
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0xMjFjNDJkZGU1OGRjZTdlLnBuZw?x-oss-process=image/format,png )
<!-- more -->

## RSA 加密原理
| 步骤| 说明| 描述|备注|
|-----|-----|------|------|
|1| 找出质数| P 、Q| -|
|2 | 计算公共模数| N = P * Q| -|
|3| 欧拉函数| φ(N) = (P-1)(Q-1)| -|
|4| 计算公钥E| 1 < E < φ(N)| E的取值必须是整数E 和 φ(N) 必须是互质数|
|5| 计算私钥D| E * D % φ(N) = 1| -|
|6| 加密| C ＝ M E mod N| C：密文 M：明文|
|7| 解密| M ＝C D mod N| C：密文 M：明文|

公钥＝(E , N)
私钥＝(D, N)

对外，我们只暴露公钥。

## 示例
### 1、找出质数 P 、Q
```
P = 3  
Q = 11
```
### 2、计算公共模数
```
N = P * Q = 3 * 11 = 33
N = 33
```
### 3、 欧拉函数
```
φ(N) = (P-1)(Q-1) = 2 * 10 = 20
φ(N) = 20
```
### 4、计算公钥E
```
1 < E < φ(N)
1 <E < 20
```
E 的取值范围 {3, 7, 9, 11, 13, 17, 19}
E的取值必须是整数, E 和 φ(N) 必须是互质数
为了测试，我们取最小的值 E =3
3 和 φ(N) =20 互为质数，满足条件
### 5、计算私钥D
```
E * D % φ(N) = 1
3 * D  % 20 = 1  
```
根据上面可计算出 D = 7
### 6、公钥加密
我们这里为了演示，就加密一个比较小的数字 M = 2
>公式：C ＝ ME mod N
```
M = 2
E = 3
N = 33
```
>C = 23 % 33 = 8

明文 “2” 经过 RSA 加密后变成了密文 “8”
### 7、私钥解密
>M ＝CD mod N
```
C = 8
D = 7
N = 33
```
>M = 87 % 33
8 * 8 * 8 * 8 * 8 * 8 * 8=2097152
8 * 8 * 8 * 8 * 8 * 8 * 8 % 33 = 2

密文 “8” 经过 RSA 解密后变成了明文 2。

## JDK 自带的 RSA 算法
```
import java.security.KeyFactory;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.PrivateKey;
import java.security.PublicKey;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;

import javax.crypto.Cipher;

import org.apache.commons.codec.binary.Base64;

/**
 * 非对称加密 唯一广泛接受并实现 数据加密&数字签名 公钥加密、私钥解密 私钥加密、公钥解密
 * 
 * @author jjs
 *
 */
public class RSADemo {

    private static String src = "infcn";

    private static RSAPublicKey rsaPublicKey;
    private static RSAPrivateKey rsaPrivateKey;

    static {
        // 1、初始化密钥
        KeyPairGenerator keyPairGenerator;
        try {
            keyPairGenerator = KeyPairGenerator.getInstance("RSA");
            keyPairGenerator.initialize(512);// 64的整倍数
            KeyPair keyPair = keyPairGenerator.generateKeyPair();
            rsaPublicKey = (RSAPublicKey) keyPair.getPublic();
            rsaPrivateKey = (RSAPrivateKey) keyPair.getPrivate();
            System.out.println("Public Key : " + Base64.encodeBase64String(rsaPublicKey.getEncoded()));
            System.out.println("Private Key : " + Base64.encodeBase64String(rsaPrivateKey.getEncoded()));
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }
    }

    /**
     * 公钥加密，私钥解密
     * @author jijs
     */
    public static void pubEn2PriDe() {
        //公钥加密
        X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(rsaPublicKey.getEncoded());
        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        PublicKey publicKey = keyFactory.generatePublic(x509EncodedKeySpec);
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.ENCRYPT_MODE, publicKey);
        byte[] result = cipher.doFinal(src.getBytes());
        System.out.println("公钥加密，私钥解密 --加密: " + Base64.encodeBase64String(result));

        //私钥解密
        PKCS8EncodedKeySpec pkcs8EncodedKeySpec = new PKCS8EncodedKeySpec(rsaPrivateKey.getEncoded());
        keyFactory = KeyFactory.getInstance("RSA");
        PrivateKey privateKey = keyFactory.generatePrivate(pkcs8EncodedKeySpec);
        cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.DECRYPT_MODE, privateKey);
        result = cipher.doFinal(result);
        System.out.println("公钥加密，私钥解密 --解密: " + new String(result));
    }


    /**
     * 私钥加密，公钥解密
     * @author jijs
     */
    public static void priEn2PubDe() {

        //私钥加密
        PKCS8EncodedKeySpec pkcs8EncodedKeySpec = new PKCS8EncodedKeySpec(rsaPrivateKey.getEncoded());
        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        PrivateKey privateKey = keyFactory.generatePrivate(pkcs8EncodedKeySpec);
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.ENCRYPT_MODE, privateKey);
        byte[] result = cipher.doFinal(src.getBytes());
        System.out.println("私钥加密，公钥解密 --加密 : " + Base64.encodeBase64String(result));

        //公钥解密
        X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(rsaPublicKey.getEncoded());
        keyFactory = KeyFactory.getInstance("RSA");
        PublicKey publicKey = keyFactory.generatePublic(x509EncodedKeySpec);
        cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.DECRYPT_MODE, publicKey);
        result = cipher.doFinal(result);
        System.out.println("私钥加密，公钥解密   --解密: " + new String(result));
    }

    public static void main(String[] args) {
        pubEn2PriDe();  //公钥加密，私钥解密
        priEn2PubDe();  //私钥加密，公钥解密
    }
}
```