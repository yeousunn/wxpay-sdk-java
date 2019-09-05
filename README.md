WeChat Payment  Java SDK
------

对[WeChat payment developer documentation](https://pay.weixin.qq.com/wiki/doc/api/index.html) The API given in is encapsulated.

com.github.wxpay.sdk.WXPay The corresponding method is provided under the class：

|Method name| Description|
|--------|--------|
|microPay| 刷卡支付 Card payment|
|unifiedOrder | 统一下单 Unified order|
|orderQuery | 查询订单 checking order|
|reverse | 撤销订单 Cancel order|
|closeOrder|关闭订单 Close order|
|refund|申请退款 Request a refund|
|refundQuery|查询退款 Query refund|
|downloadBill|下载对账单 Download statement|
|report|交易保障 Transaction guarantee|
|shortUrl|转换短链接 Convert short links|
|authCodeToOpenid|授权码查询openid (
Authorization code query openid)|

* note:
* The certificate file cannot be placed in the web server virtual directory and should be placed in a directory with access control to prevent others from downloading.
* It is recommended to change the certificate file name to a file name that is complicated and not easy to guess.
* Merchant servers should be protected against viruses and Trojans, and they should not be stolen by hackers.
* Please keep the merchant payment key and public account SECRET in a safe place to avoid key leakage.
* The parameter is`Map<String, String>`Object, return type is also`Map<String, String>`
* The method internally converts the parameters into`appid`、`mch_id`、`nonce_str`、`sign\_type` with`sign` of XML
* Optional HMAC-SHA256 algorithm and MD5 algorithm signature
* After the returned data is obtained through the HTTPS request, it will be processed as necessary (for example, verifying the signature, and if the signature is incorrect, an exception is thrown)
* For downloadBill, the Map is returned regardless of success, and both contain `return_code` and `return_msg`. If successful, `return_code` is `SUCCESS`, and `data` corresponds to the statement data.


## Example
Configuration class MyConfig:
```java
import com.github.wxpay.sdk.WXPayConfig;
import java.io.*;

public class MyConfig implements WXPayConfig{

    private byte[] certData;

    public MyConfig() throws Exception {
        String certPath = "/path/to/apiclient_cert.p12";
        File file = new File(certPath);
        InputStream certStream = new FileInputStream(file);
        this.certData = new byte[(int) file.length()];
        certStream.read(this.certData);
        certStream.close();
    }

    public String getAppID() {
        return "wx8888888888888888";
    }

    public String getMchID() {
        return "12888888";
    }

    public String getKey() {
        return "88888888888888888888888888888888";
    }

    public InputStream getCertStream() {
        ByteArrayInputStream certBis = new ByteArrayInputStream(this.certData);
        return certBis;
    }

    public int getHttpConnectTimeoutMs() {
        return 8000;
    }

    public int getHttpReadTimeoutMs() {
        return 10000;
    }
}
```

Unified order：

```java
import com.github.wxpay.sdk.WXPay;

import java.util.HashMap;
import java.util.Map;

public class WXPayExample {

    public static void main(String[] args) throws Exception {

        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config);

        Map<String, String> data = new HashMap<String, String>();
        data.put("body", "腾讯充值中心-QQ会员充值 Tencent Recharge Center - QQ member recharge");
        data.put("out_trade_no", "2016090910595900000012");
        data.put("device_info", "");
        data.put("fee_type", "CNY");
        data.put("total_fee", "1");
        data.put("spbill_create_ip", "123.12.12.123");
        data.put("notify_url", "http://www.example.com/wxpay/notify");
        data.put("trade_type", "NATIVE");  // 此处指定为扫码支付 Designated here as a scan code payment
        data.put("product_id", "12");

        try {
            Map<String, String> resp = wxpay.unifiedOrder(data);
            System.out.println(resp);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

Order Tracking：
```java
import com.github.wxpay.sdk.WXPay;

import java.util.HashMap;
import java.util.Map;

public class WXPayExample {

    public static void main(String[] args) throws Exception {

        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config);

        Map<String, String> data = new HashMap<String, String>();
        data.put("out_trade_no", "2016090910595900000012");

        try {
            Map<String, String> resp = wxpay.orderQuery(data);
            System.out.println(resp);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

Refund inquiry：

```java
import com.github.wxpay.sdk.WXPay;

import java.util.HashMap;
import java.util.Map;

public class WXPayExample {

    public static void main(String[] args) throws Exception {

        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config);

        Map<String, String> data = new HashMap<String, String>();
        data.put("out_trade_no", "2016090910595900000012");

        try {
            Map<String, String> resp = wxpay.refundQuery(data);
            System.out.println(resp);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```


Download statement：

```java
import com.github.wxpay.sdk.WXPay;

import java.util.HashMap;
import java.util.Map;

public class WXPayExample {

    public static void main(String[] args) throws Exception {

        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config);

        Map<String, String> data = new HashMap<String, String>();
        data.put("bill_date", "20140603");
        data.put("bill_type", "ALL");

        try {
            Map<String, String> resp = wxpay.downloadBill(data);
            System.out.println(resp);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

The use of other APIs is similar to the above.


It is temporarily not supported to download a compressed format statement, but you can use the SDK to generate XML data for the request:
```java
import com.github.wxpay.sdk.WXPay;
import com.github.wxpay.sdk.WXPayUtil;

import java.util.HashMap;
import java.util.Map;

public class WXPayExample {

    public static void main(String[] args) throws Exception {

        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config);

        Map<String, String> data = new HashMap<String, String>();
        data.put("bill_date", "20140603");
        data.put("bill_type", "ALL");
        data.put("tar_type", "GZIP");

        try {
            data = wxpay.fillRequestData(data);
            System.out.println(WXPayUtil.mapToXml(data));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

When you receive a payment result notification, you need to verify the signature, you can do this:
```java

import com.github.wxpay.sdk.WXPay;
import com.github.wxpay.sdk.WXPayUtil;

import java.util.Map;

public class WXPayExample {

    public static void main(String[] args) throws Exception {

        String notifyData = "...."; // 支付结果通知的xml格式数据 Xml format data for payment result notification

        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config);

        Map<String, String> notifyMap = WXPayUtil.xmlToMap(notifyData);  // 转换成map Convert to map

        if (wxpay.isPayResultNotifySignatureValid(notifyMap)) {
            // 签名正确 Signed correctly
            // 进行处理。 Process it
            // 注意特殊情况：订单已经退款，但收到了支付结果成功的通知，不应把商户侧订单状态从退款改成支付成功 
            // Note the special case: the order has been refunded, but the payment result is successfully notified, and the order status of the merchant side should not be changed from refund to payment success.
        }
        else {
            // 签名错误，如果数据里没有sign字段，也认为是签名错误
            // Signature error, if there is no sign field in the data, it is also considered a signature error
        }
    }

}
```


HTTPS requests optional HMAC-SHA256 algorithm and MD5 algorithm signature:

```
import com.github.wxpay.sdk.WXPay;
import com.github.wxpay.sdk.WXPayConstants;

public class WXPayExample {

    public static void main(String[] args) throws Exception {
        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config, WXPayConstants.SignType.HMACSHA256);
        // ......
    }
}
```

If you need to use the sandbox environment:
```
import com.github.wxpay.sdk.WXPay;
import com.github.wxpay.sdk.WXPayConstants;

public class WXPayExample {

    public static void main(String[] args) throws Exception {
        MyConfig config = new MyConfig();
        WXPay wxpay = new WXPay(config, WXPayConstants.SignType.MD5, true);
        // ......
    }

}
```
