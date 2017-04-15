---
title: Android微信支付踩坑全记录
date: 2017-04-05 00:00:00
tags: android
comments: false
---
首先吐槽一下，微信支付的文档和微信提供的demo实在是惨不忍睹，折腾了好久才终于搞定了，如果还是有问题，请仔细检查请求的参数是否正确，一般都是漏掉了某个参数的名称之类的。
<!--more-->

### 1. 首先第一步，获取应用的MD5签名信息
![这里写图片描述](http://img.blog.csdn.net/20170330180057671?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvU2VyX0JhZA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
如果你没有配置keytool的环境，那么也可以直接使用 [微信提供的获取包签名的信息](https://res.wx.qq.com/open/zh_CN/htmledition/res/dev/download/sdk/Gen_Signature_Android2.apk)
### 2.  申请过程就不提了，千万注意，应用包名和签名信息一定要填对，一开始我的签名信息弄错了，折腾了好久。

### 3. 申请好了之后，现在开始接入
- [首先在gradle中导入](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419319167&token=&lang=zh_CN)：
```grandle
dependencies {
   compile 'com.tencent.mm.opensdk:wechat-sdk-android-without-mta:+'
}
```
- 注册APPID
```gradle
final IWXAPI msgApi = WXAPIFactory.createWXAPI(context, null);
// 将该app注册到微信
msgApi.registerApp("wxd930ea5d5a258f4f");
```

- [统一下单](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=9_1)，目的是获取微信返回给我们的``prepay_id``
- URL地址：https://api.mch.weixin.qq.com/pay/unifiedorder
- 需要注意的是，该URL需要用post的方式发起请求
- 在[签名](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=4_3)的时候，参数名要按照ASCII码从小到大排序
```java
public String getOderXml() {
       Map<String, String> oderMap = new HashMap<>();
       oderMap.put("appid", appid); //
       oderMap.put("mch_id", mch_id);//
       oderMap.put("device_info", device_info);//-
       oderMap.put("nonce_str", nonce_str);//
       oderMap.put("trade_type", trade_type);
       oderMap.put("body", body);
       oderMap.put("detail", detail);
       oderMap.put("out_trade_no", out_trade_no);
       oderMap.put("total_fee", total_fee);
       oderMap.put("spbill_create_ip", spbill_create_ip);
       oderMap.put("time_start", time_start);
       oderMap.put("notify_url", notify_url);

       //拼接字符串加上key值，然后对sign进行md5解密，之后再转成大写
       //将集合M内非空参数值的参数按照参数名ASCII码从小到大排序
       StringBuilder sign = new StringBuilder();
       TreeSet<String> set = new TreeSet<>(new Comparator<String>() {
           @Override
           public int compare(String lhs, String rhs) {
               int i = rhs.compareTo(lhs);
               if (i > 0) {
                   return -1;
               } else if (i < 0) {
                   return 1;
               } else {
                   return i;
               }
           }
       });
       set.addAll(oderMap.keySet());

       for (String str : set) {
           sign.append(str).append("=").append(oderMap.get(str)).append("&");
       }
       sign.append("key").append("=").append(WechatUtil.KEY);
       oderMap.put("sign", StringUtil.md5(sign.toString()).toUpperCase());

       //拼接成xml格式的字符串
       StringBuilder stringBuilder = new StringBuilder();
       stringBuilder.append("<xml>");
       set.clear();
       set.addAll(oderMap.keySet());
       for (String s : set) {
           stringBuilder.append("<").append(s).append(">");
           stringBuilder.append(oderMap.get(s));
           stringBuilder.append("</").append(s).append(">");
       }
       stringBuilder.append("</xml>");
       return stringBuilder.toString();
   }

```
```java
//处理微信返回的xml数据
public OderResult xmlToResult(String xml) {
       XmlPullParser pullParser = Xml.newPullParser();
       InputStream ins = new ByteArrayInputStream(xml.getBytes());
       OderResult result = new OderResult();
       try {
           pullParser.setInput(ins, "utf-8");
           int eventType = pullParser.getEventType();
           while (XmlPullParser.END_DOCUMENT != eventType) {
               switch (eventType) {
                   case XmlPullParser.START_TAG:
                       if ("return_code".equals(pullParser.getName())) {
                           result.setReturn_code(pullParser.nextText());
                       }
                       if ("return_msg".equals(pullParser.getName())) {
                           result.setReturn_msg(pullParser.nextText());
                       }
                       if ("appid".equals(pullParser.getName())) {
                           result.setAppid(pullParser.nextText());
                       }
                       if ("mch_id".equals(pullParser.getName())) {
                           result.setMch_id(pullParser.nextText());
                       }
                       if ("device_info".equals(pullParser.getName())) {
                           result.setDevice_info(pullParser.nextText());
                       }
                       if ("nonce_str".equals(pullParser.getName())) {
                           result.setNonce_str(pullParser.nextText());
                       }
                       if ("sign".equals(pullParser.getName())) {
                           result.setSign(pullParser.nextText());
                       }
                       if ("prepay_id".equals(pullParser.getName())) {
                           result.setPrepay_id(pullParser.nextText());
                       }
                       if ("trade_type".equals(pullParser.getName())) {
                           result.setTrade_type(pullParser.nextText());
                       }
                       if ("result_code".equals(pullParser.getName())) {
                           result.setResult_code(pullParser.nextText());
                       }
                       break;
                   case XmlPullParser.END_TAG:
                       break;
                   default:
                       break;
               }
               eventType = pullParser.next();
           }
       } catch (XmlPullParserException e) {
           e.printStackTrace();
       } catch (IOException e) {
           e.printStackTrace();
       }
       return result;
   }
```
```java
//随机字符串，不长于32位
   public String getRandom() {
       SimpleDateFormat format = new SimpleDateFormat("yyMMddHHmmss", Locale.getDefault());
       Date date = new Date();
       String key = format.format(date);
       Random r = new Random();
       key = key + r.nextLong();
       key = StringUtil.md5(key);
       return key;
   }
```
- [调起支付](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=9_12)，签名的办法同上面的处理就好了
```java
IWXAPI api;
PayReq request = new PayReq();
request.appId = "wxd930ea5d5a258f4f";
request.partnerId = "1900000109";
request.prepayId= "1101000000140415649af9fc314aa427",;
request.packageValue = "Sign=WXPay";
request.nonceStr= "1101000000140429eb40476f8896f4c9";
request.timeStamp= "1398746574";
request.sign= "7FFECB600D7157C5AA49810D2D8F28BC2811827B";
api.sendReq(request);
```
- 支付结果回调，实现一个``.wxapi.WXPayEntryActivity``(包名或类名不一致会造成无法回调)。如果不需要显示这个，直接finish就好了。
```xml
<activity
           android:name=".wxapi.WXPayEntryActivity"
           android:exported="true" />
```
```java
public class WXPayEntryActivity extends Activity implements IWXAPIEventHandler {
    private IWXAPI api;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        api = WXAPIFactory.createWXAPI(this, WechatUtil.APPID, false);
        api.handleIntent(getIntent(), this);
    }

    @Override
    public void onReq(BaseReq baseReq) {

    }

    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        setIntent(intent);
        api.handleIntent(intent, this);
    }

    @Override
    public void onResp(BaseResp baseResp) {

        if (baseResp.getType() == ConstantsAPI.COMMAND_PAY_BY_WX) {
            if (baseResp.errCode == BaseResp.ErrCode.ERR_COMM) {
                showToast("微信支付失败");
            } else if (baseResp.errCode == BaseResp.ErrCode.ERR_USER_CANCEL) {
                showToast("用户取消");
            } else if (baseResp.errCode == BaseResp.ErrCode.ERR_OK) {
                showToast("微信支付成功");
            }
        }
    }
}
```
- 如果返回的是失败，请再仔细检查调起支付的参数是否正确，进行sign签名的参数是否正确，如果还是不行，请再检查一遍，微信支付的错误信息太少了，之后成功和失败，如果统一下单成功了的话，那么就是一定是你们调起支付的参数错误，请一定要的仔细检查，重要的事重复三遍。
- 如果是统一下单的时候有问题，请确认后台返回给你的订单号是否是唯一的，参数是否正确，sign的时候时候按照要求签名，微信平台填的是否是MD5签名，发起请求的方式是否为post，是否传送的是XML数据，如果是使用Retrofit发起请求，可以这样子：
```java
new Retrofit.Builder()
             .baseUrl("https://api.mch.weixin.qq.com/pay/")
             .addConverterFactory(StringConverterFactory.create())
             .build()
             .create(API.class)
             .oder(RequestBody.create(MediaType.parse("text/plain"), oder)).enqueue(new Callback<String>() {
         @Override
         public void onResponse(Call<String> call, Response<String> response) {
             if (response.body() != null) {
                 OderResult result = xmlToResult(response.body());
                 if (result.getReturn_code().equals("SUCCESS")) {
                     if (result.getResult_code().equals("SUCCESS")) {

                     }
                 } else {

                 }
             }

         }

         @Override
         public void onFailure(Call<String> call, Throwable t) {

         }
     });
```
```java
private interface API {
        @POST("unifiedorder")
        Call<String> oder(@Body RequestBody oder);
    }
```
