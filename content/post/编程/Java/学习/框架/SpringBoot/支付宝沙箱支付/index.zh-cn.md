---
title: 支付宝沙箱支付
description: 支付宝沙箱支付
date: 2023-03-27
slug: 支付宝沙箱支付
image: 202412212138297.png
categories:
    - 沙箱支付
---

支付宝沙箱支付
=======
> [参考博客1](https://blog.csdn.net/hhb442/article/details/123304287)
>
> [参考博客2](https://blog.csdn.net/xqnode/article/details/124457790)
>
> [网站](https://open.alipay.com/develop/sandbox/app)
>
> [参考视频](https://www.bilibili.com/video/BV12e4y1o7qj/?spm_id_from=333.337.search-card.all.click&vd_source=0de2cc95ad0adc3e9ec15f6dcede1a20),[青戈的demo项目](https://gitee.com/xqnode/alipay-demo)
![步骤1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152337963.png)
![image-20230326134943080](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152337090.png)
[密钥](https://opendocs.alipay.com/common/02kipk#%E5%B7%A5%E5%85%B7%E4%B8%8B%E8%BD%BD)
![image-20230326203237561](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152337207.png)
> 导入依赖
```xml
<dependency>
    <groupId>com.alipay.sdk</groupId>
    <artifactId>alipay-sdk-java</artifactId>
    <version>4.22.110.ALL</version>
</dependency>
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.7.20</version>
</dependency>
```
> application.properties里需要配置这四个
![image-20230326212330514](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152337844.png)
> 这里复制第一个appid
![appId](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152337268.png)
> 复制第二个私钥
![image-20230326212621235](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152337560.png)
![image-20230326212657145](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152337177.png)
> 复制第三个支付宝公钥
![image-20230326212806008](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152337241.png)
> 至于第四个url，需要下载一个工具
natapp地址
-----------------------------------------------------------
官网：[https://natapp.cn/](https://natapp.cn/)
下载：[https://pan.baidu.com/s/1L99Ibawylnck4b4c0NtmXg?pwd=685x](https://pan.baidu.com/s/1L99Ibawylnck4b4c0NtmXg?pwd=685x)
> 下载后，去官网注册账号
![image-20230326213755393](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152337915.png)
![image-20230326213914822](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152337442.png)
![image-20230326213940611](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152338191.png)
![image-20230326214106638](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152337401.png)
![image-20230326214209342](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152338219.png)
![image-20230326214411891](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152338689.png)
![image-20230326214512806](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152338811.png)
> 配好之后双击start.bat
![image-20230326214657488](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152338582.png)
![image-20230326214742135](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152338306.png)
> 还需要配置controller的地址
![image-20230326214859892](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152338384.png)
> 配置到此结束
![image-20230326215056683](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152338458.png)
```java
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;
@Data
@Component
@ConfigurationProperties(prefix = "alipay")
public class AliPayConfig {
    private String appId;
    private String appPrivateKey;
    private String alipayPublicKey;
    private String notifyUrl;
}
```
![image-20230326215215436](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152338705.png)
```java
import lombok.Data;
@Data
public class AliPay {
    private String traceNo;
    private double totalAmount;
    private String subject;
    private String alipayTraceNo;
}
```
> 继续新建`AliPayController`，把报错的`AliPayConfig`包换成自己项目里的路径，还有`Order`
```java
import cn.hutool.json.JSONObject;
import com.alipay.api.AlipayApiException;
import com.alipay.api.AlipayClient;
import com.alipay.api.DefaultAlipayClient;
import com.alipay.api.internal.util.AlipaySignature;
import com.alipay.api.request.AlipayTradePagePayRequest;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.example.alipay.common.AliPayConfig;
import com.example.alipay.dao.OrdersMapper;
import com.example.alipay.entity.Orders;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
// xjlugv6874@sandbox.com
// 9428521.24 - 30 = 9428491.24 + 30 = 9428521.24
@RestController
@RequestMapping("/alipay")
public class AliPayController {
    private static final String GATEWAY_URL = "https://openapi.alipaydev.com/gateway.do";
    private static final String FORMAT = "JSON";
    private static final String CHARSET = "UTF-8";
    //签名方式
    private static final String SIGN_TYPE = "RSA2";
    @Resource
    private AliPayConfig aliPayConfig;
    @Resource
    private OrdersMapper ordersMapper;
    @GetMapping("/pay") // &subject=xxx&traceNo=xxx&totalAmount=xxx
    public void pay(AliPay aliPay, HttpServletResponse httpResponse) throws Exception {
        // 1. 创建Client，通用SDK提供的Client，负责调用支付宝的API
        AlipayClient alipayClient = new DefaultAlipayClient(GATEWAY_URL, aliPayConfig.getAppId(),
                aliPayConfig.getAppPrivateKey(), FORMAT, CHARSET, aliPayConfig.getAlipayPublicKey(), SIGN_TYPE);
        // 2. 创建 Request并设置Request参数
        AlipayTradePagePayRequest request = new AlipayTradePagePayRequest();  // 发送请求的 Request类
        request.setNotifyUrl(aliPayConfig.getNotifyUrl());
        JSONObject bizContent = new JSONObject();
        bizContent.set("out_trade_no", aliPay.getTraceNo());  // 我们自己生成的订单编号
        bizContent.set("total_amount", aliPay.getTotalAmount()); // 订单的总金额
        bizContent.set("subject", aliPay.getSubject());   // 支付的名称
        bizContent.set("product_code", "FAST_INSTANT_TRADE_PAY");  // 固定配置
        request.setBizContent(bizContent.toString());
        // 执行请求，拿到响应的结果，返回给浏览器
        String form = "";
        try {
            form = alipayClient.pageExecute(request).getBody(); // 调用SDK生成表单
        } catch (AlipayApiException e) {
            e.printStackTrace();
        }
        httpResponse.setContentType("text/html;charset=" + CHARSET);
        httpResponse.getWriter().write(form);// 直接将完整的表单html输出到页面
        httpResponse.getWriter().flush();
        httpResponse.getWriter().close();
    }
    @PostMapping("/notify")  // 注意这里必须是POST接口
    public String payNotify(HttpServletRequest request) throws Exception {
        if (request.getParameter("trade_status").equals("TRADE_SUCCESS")) {
            System.out.println("=========支付宝异步回调========");
            Map<String, String> params = new HashMap<>();
            Map<String, String[]> requestParams = request.getParameterMap();
            for (String name : requestParams.keySet()) {
                params.put(name, request.getParameter(name));
                // System.out.println(name + " = " + request.getParameter(name));
            }
            String outTradeNo = params.get("out_trade_no");
            String gmtPayment = params.get("gmt_payment");
            String alipayTradeNo = params.get("trade_no");
            String sign = params.get("sign");
            String content = AlipaySignature.getSignCheckContentV1(params);
            boolean checkSignature = AlipaySignature.rsa256CheckContent(content, sign, aliPayConfig.getAlipayPublicKey(), "UTF-8"); // 验证签名
            // 支付宝验签
            if (checkSignature) {
                // 验签通过
                System.out.println("交易名称: " + params.get("subject"));
                System.out.println("交易状态: " + params.get("trade_status"));
                System.out.println("支付宝交易凭证号: " + params.get("trade_no"));
                System.out.println("商户订单号: " + params.get("out_trade_no"));
                System.out.println("交易金额: " + params.get("total_amount"));
                System.out.println("买家在支付宝唯一id: " + params.get("buyer_id"));
                System.out.println("买家付款时间: " + params.get("gmt_payment"));
                System.out.println("买家付款金额: " + params.get("buyer_pay_amount"));
                // 查询订单
                QueryWrapper<Orders> queryWrapper = new QueryWrapper<>();
                queryWrapper.eq("order_id", outTradeNo);
                Orders orders = ordersMapper.selectOne(queryWrapper);
                if (orders != null) {
                    orders.setAlipayNo(alipayTradeNo);
                    orders.setPayTime(new Date());
                    orders.setState("已支付");
                    ordersMapper.updateById(orders);
                }
            }
        }
        return "success";
    }
}
```
![image-20230326220353675](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152338134.png)
```url
http://localhost:15000/alipay/pay?subject=香蕉&traceNo=202208221661165525068&totalAmount=6.99
```
> 可以看到这里是502，奇怪，视频上明明是这样做的，上网搜了一下，发现[博客](https://blog.csdn.net/qq_46978751/article/details/111577414)`每周日中午12点至每周一中午12点沙箱环境进行维护`，今天是周日
>
> 官方也说了，请[查看](https://opendocs.alipay.com/support/01razf)
![image-20230327203933130](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152338575.png)
> ok，过了时间就可以打开了
![image-20230327205301864](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152338417.png)
```java
商家信息
商户账号rilvmb5932@sandbox.com
登录密码111111
商户PID2088621987551356
账户余额2132135.00
买家信息
买家账号xesgkm3338@sandbox.com
登录密码111111
支付密码111111
用户UID2088622987164881
用户名称xesgkm3338
证件类型IDENTITY_CARD
证件账号308282195800026310
账户余额99999.00 
```
> 这里我遇到了一个问题，没有执行回调，我上网查了一下，找到这篇[博客](https://blog.csdn.net/qq_45000856/article/details/125671735)，但是我不是这个原因
![image-20230327213001891](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152338042.png)
这个url，复制粘贴到`alipay.notifyUrl`这里，`这个url每次启动都会改变，所以应该更改properties`
