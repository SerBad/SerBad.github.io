---
title: Gson序列化
date: 2017-03-21 01:09:43
tags: Android
comments: false
---
最近项目中遇到一个问题，就是Gson解析的时候遇到一个为空的数组，这种时候如果忽略后台优雅地解决这个问题呢？答案就是——Gson 的序列化和反序列化：
<!--more-->
```java
public class ResultDeserializer implements JsonDeserializer<BaseResult> {
    @Override
    public BaseResult deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) throws JsonParseException {
        JsonObject obj = json.getAsJsonObject();
        //下面添加你要做的事情
        if (obj.get("result") != null) {
            if (obj.get("result").isJsonObject() && obj.getAsJsonObject("result").has("message") && obj.getAsJsonObject("result").entrySet().size() == 1) {
                obj.getAsJsonObject("result").remove("message");
            }
            if (obj.get("result").toString().equals("{}")) {
                obj.remove("result");
            }
        }

        BaseResult baseResult = new Gson().fromJson(json, typeOfT);
        if (obj.has("success")&&obj.get("success").getAsString().equals("true")) {
            baseResult.setSuccess(true);
        } else {
            baseResult.setSuccess(false);
        }
        if (obj.isJsonNull()) {
            baseResult.setSuccess(false);
        }
        baseResult.setErrorcode(obj.has("errorcode") ? obj.get("errorcode").getAsString() : "");
        baseResult.setErrorcode(obj.has("error") ? obj.get("errorcode").getAsString() : "");
        return baseResult;
    }
}
```
然后注册一下这个typeadapter
```java
Gson gson=new GsonBuilder()
      .registerTypeAdapter(BaseResult.class, new ResultDeserializer())
      .create();
```
然后就好了，你可以写一个范型的Bean的基础类，然后就可以在全局使用了,比如：
```java
public class BaseResult<T> implements Serializable {
    private Boolean success;
    private String errorcode;
    private String error;
    private T result;
}
```


``registerTypeAdapter`` 还支持多种类型的，找时间可以去研究一下：

- JsonSerializer &lt;?>
- JsonDeserializer &lt;?>
- InstanceCreator &lt;?>
- TypeAdapter &lt;?>
