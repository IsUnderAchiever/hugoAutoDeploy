---
title: 08_AOP案例
description: 08_AOP案例
date: 2023-01-18
slug: 08_AOP案例
image: 202412212133331.png
categories:
    - Spring
---

## 08_AOP案例
```java
public class AIController {
    // AI自动回答
    public String getAnswer(String question) {
        // AI核心代码
        String str = question.replace("吗", "");
        str = str.replace("？", "！");
        str = str.replace("?", "!");
        return str;
    }
    // AI算命
    public String fortuneTelling(String name) {
        // AI核心代码
        String[] strs = {"女犯伤官把夫克，早地莲花栽不活，不是吃上两家饭，也要刷上三家锅。",
                "一朵鲜花头上戴，一年四季也不开，一心想要花开时，采花之人没到来。",
                "此命生来脾气暴，上来一阵双脚跳，对你脾气啥都好，经常与人吵和闹。"};
        int index=name.hashCode()%3;
        return strs[index];
    }
}
```
> `需求`：现在为了保证数据的安全性，传入加密后的参数给fortuneTelling方法，此时AOP进行解密后真正传给fortuneTelling方法，执行后的返回值再进行加密后返回
>
> ps：后期也可能让其他方法进行加密处理
```java
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.security.SecureRandom;
// 加密、解密工具类
public class CryptUtil {
    private static final String AES = "AES";
    private static int keysizeAES = 128;
    private static String charset = "UTF-8";
    public static String parseByte2HexStr(final byte[] b) {
        final StringBuilder stringBuffer = new StringBuilder();
        for (byte value : b) {
            String hex = Integer.toHexString(value & 0xFF);
            if (hex.length() == 1) {
                hex = '0' + hex;
            }
            stringBuffer.append(hex.toUpperCase());
        }
        return stringBuffer.toString();
    }
    public static byte[] parseHexStr2Byte(final String hexStr) {
        if (hexStr.length() < 1) {
            return null;
        }
        final byte[] result = new byte[hexStr.length() / 2];
        for (int i = 0; i < hexStr.length() / 2; i++) {
            int high = Integer.parseInt(hexStr.substring(i * 2, i * 2 + 1), 16);
            int low = Integer.parseInt(hexStr.substring(i * 2 + 1, i * 2 + 2), 16);
            result[i] = (byte) (high * 16 + low);
        }
        return result;
    }
    private static String keyGeneratorES(final String res, final String algorithm, final String key, final Integer keysize, final boolean bEncode) {
        try {
            final KeyGenerator g = KeyGenerator.getInstance(algorithm);
            if (keysize == 0) {
                byte[] keyBytes = charset == null ? key.getBytes() : key.getBytes(charset);
                g.init(new SecureRandom(keyBytes));
            } else if (key == null) {
                g.init(keysize);
            } else {
                byte[] keyBytes = charset == null ? key.getBytes() : key.getBytes(charset);
                SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
                random.setSeed(keyBytes);
                g.init(keysize, random);
            }
            final SecretKey secretKey = g.generateKey();
            final SecretKeySpec keySpec = new SecretKeySpec(secretKey.getEncoded(), algorithm);
            final Cipher cipher = Cipher.getInstance(algorithm);
            if (bEncode) {
                cipher.init(Cipher.ENCRYPT_MODE, keySpec);
                final byte[] result = charset == null ? res.getBytes() : res.getBytes(charset);
                return parseByte2HexStr(cipher.doFinal(result));
            } else {
                cipher.init(Cipher.DECRYPT_MODE, keySpec);
                return new String(cipher.doFinal(parseHexStr2Byte(res)));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
    public static String AESencode(final String res) {
        return keyGeneratorES(res, AES, "aAll*-%", keysizeAES, true);
    }
    public static String AESdecode(final String res) {
        return keyGeneratorES(res, AES, "aAll*-%", keysizeAES, false);
    }
    public static void main(String[] args) {
        System.out.println("加密后:" + AESencode("123456"));
        System.out.println("解密后:" + AESdecode("555B695B57E024EEA169EAE275B9D93B"));
    }
}
```
```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface EnCodeAndDeCode {
}
@Component
@Aspect
public class MyAOP {
    @Pointcut("execution(* com.example.controller..*.fortuneTelling(..))")
    public void pointCut() {
    }
    @Pointcut("@annotation(com.example.aspect.EnCodeAndDeCode)")
    public void annotationPointCut() {
    }
    @Around("annotationPointCut()")
    public Object around(ProceedingJoinPoint point) {
        Object proceed = null;
        Object[] args = point.getArgs();
        System.out.println("明文参数是:" + Arrays.toString(args));
        for (int i = 0; i < args.length; i++) {
            args[i] = CryptUtil.AESdecode((String) args[i]);
        }
        System.out.println("密文参数是:" + Arrays.toString(args));
        try {
            proceed = point.proceed(args);
            System.out.println("明文返回值是:" + proceed);
            System.out.println("密文返回值是:" + CryptUtil.AESencode((String) proceed));
        } catch (Throwable e) {
            e.printStackTrace();
        }
        return CryptUtil.AESencode((String) proceed);
    }
}
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        AIController aiController = context.getBean(AIController.class);
        //String answer = aiController.getAnswer("我帅吗？");
        String result = aiController.fortuneTelling(CryptUtil.AESencode("赵子龙"));
        //System.out.println(answer);
        System.out.println("返回值解密是:"+CryptUtil.AESdecode(result));
    }
}
```
> 执行结果：
>
> 明文参数是:[6F152DBC4B9D1B6B03EA4B858D6AE775]
> 密文参数是:[赵子龙]
> 明文返回值是:女犯伤官把夫克，早地莲花栽不活，不是吃上两家饭，也要刷上三家锅。
> 密文返回值是:930617239D652C6E7BD9510B919BB44CAE3286191754B494FBB76F52FC370E2336813515381DDE2463A91F2588898E9CB9C4D31EBD2D09BCFFFBF593C675A13DC8A8FDEADC905A791A9E72D03F38E6B3C410C8EF7683A5130175E0670AC17210A6032F8A8CBC8256C8E0A70D65DE0148
> 返回值解密是:女犯伤官把夫克，早地莲花栽不活，不是吃上两家饭，也要刷上三家锅。
