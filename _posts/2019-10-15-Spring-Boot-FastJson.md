---
layout: post
title: Spring Boot 使用FastJson实现Jsonp返回
tags: SpringBoot  
---

### JSONP返回实现思路
在Java Web项目中，Jsonp 是非常常用的东西，这里实现Jsonp返回是通过fastjson实现的，实现效果如下

测试访问接口：
localhost:8080/testJsonp?platform=efunenplatform&userId=435654&language=en-US&flag=paypal&jsonpcallback=jsonp_1571018438835_11

返回结果：
``` json
jsonp_1571018438835_11([
	{
		"code": "1000",
		"data": {},
		"message": "success"
	}
])
```
可以看到，返回的结果中，就已经使用请求传过来的 jsonpcallback 对象对返回的内容进行了包装

### 实现代码

```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.serializer.SerializerFeature;
import com.alibaba.fastjson.support.config.FastJsonConfig;
import com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter;

import org.springframework.context.annotation.Bean;
import org.springframework.core.MethodParameter;
import org.springframework.http.MediaType;
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.http.server.ServletServerHttpRequest;
import org.springframework.http.server.ServletServerHttpResponse;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.servlet.mvc.method.annotation.ResponseBodyAdvice;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.OutputStream;
import java.util.ArrayList;
import java.util.List;

/**
 * FastJsonJsonpHttpMessageConverter
 * json转换组件，Spring返回Json数据时，加上jsonpcallback
 *
 * @author Lenjor
 * @version 1.0
 * @date 2019/10/14 11:30
 */
@ControllerAdvice
public class FastJsonJsonpHttpMessageConverter extends FastJsonHttpMessageConverter implements ResponseBodyAdvice {
    protected String[] jsonpParameterNames = new String[]{"jsoncallback", "jsonpcallback", "jsonp", "callback"};
    //FastJson序列化策略
    private static SerializerFeature[] features = new SerializerFeature[]{
            SerializerFeature.WriteNullNumberAsZero,
            SerializerFeature.WriteNullStringAsEmpty,
            SerializerFeature.WriteMapNullValue,
            SerializerFeature.WriteEnumUsingToString,
            SerializerFeature.WriteNullListAsEmpty,
            SerializerFeature.WriteNullBooleanAsFalse,
            SerializerFeature.PrettyFormat,
            SerializerFeature.DisableCircularReferenceDetect
    };

    @Override
    public Object beforeBodyWrite(Object o, MethodParameter methodParameter, MediaType mediaType, Class aClass, ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse) {
        HttpServletRequest servletRequest = ((ServletServerHttpRequest) serverHttpRequest).getServletRequest();
        HttpServletResponse response = ((ServletServerHttpResponse) serverHttpResponse).getServletResponse();

        String callback = null;
        for (int i = 0; i < jsonpParameterNames.length; i++) {
            callback = servletRequest.getParameter(jsonpParameterNames[i]);
            if (callback != null) {
                break;
            }
        }
        if (callback != null) {
            String jsonString = JSON.toJSONString(o, features);
            jsonString = new StringBuilder(callback).append("([").append(jsonString).append("])").toString();

            //根据前端需要返回对应的编码，默认均是utf-8
            byte[] bytes = jsonString.getBytes(getCharset());
            try {
                OutputStream out = null;
                out = response.getOutputStream();
                out.write(bytes);
                out.flush();
                out.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return o;
    }

    @Override
    public boolean supports(MethodParameter methodParameter, Class aClass) {
        return true;
    }

    @Bean
    public FastJsonHttpMessageConverter fastJsonHttpMessageConverter() {
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        converter.setFastJsonConfig(fastjsonConfig());
        converter.setSupportedMediaTypes(getSupportedMediaType());
        return converter;
    }

    /**
     * FastJson 配置
     */
    public FastJsonConfig fastjsonConfig() {
        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        fastJsonConfig.setSerializerFeatures(features);
        fastJsonConfig.setDateFormat("yyyy-MM-dd HH:mm:ss");
        return fastJsonConfig;
    }

    /**
     * 支持的mediaType类型
     */
    public List<MediaType> getSupportedMediaType() {
        ArrayList<MediaType> mediaTypes = new ArrayList<>();
        mediaTypes.add(MediaType.APPLICATION_JSON_UTF8);
        return mediaTypes;
    }
}

```

### 序列化策略

    QuoteFieldNames 输出key时是否使用双引号,默认为true
    UseSingleQuotes 使用单引号而不是双引号,默认为false
    WriteMapNullValue   是否输出值为null的字段,默认为false  
    WriteEnumUsingToString  Enum输出name()或者original,默认为false
    UseISO8601DateFormat    Date使用ISO8601格式输出，默认为false  
    WriteNullListAsEmpty    List字段如果为null,输出为[],而非null  
    WriteNullStringAsEmpty  字符类型字段如果为null,输出为”“,而非null  
    WriteNullNumberAsZero   数值字段如果为null,输出为0,而非null
    WriteNullBooleanAsFalse Boolean字段如果为null,输出为false,而非null
    SkipTransientField  如果是true，类中的Get方法对应的Field是transient，序列化时将会被忽略。默认为true
    SortField   按字段名称排序后输出。默认为false
    WriteTabAsSpecial   把\t做转义输出，默认为false   不推荐
    PrettyFormat    结果是否格式化,默认为false
    WriteClassName  序列化时写入类型信息，默认为false。反序列化是需用到
    DisableCircularReferenceDetect  消除对同一对象循环引用的问题，默认为false
    WriteSlashAsSpecial 对斜杠’/’进行转义  
    BrowserCompatible   将中文都会序列化为\uXXXX格式，字节数会多一些，但是能兼容IE 6，默认为false
    WriteDateUseDateFormat  全局修改日期格式,默认为false。JSON.DEFFAULT_DATE_FORMAT = “yyyy-MM-dd”;JSON.toJSONString(obj, SerializerFeature.WriteDateUseDateFormat);
    DisableCheckSpecialChar 一个对象的字符串属性中如果有特殊字符如双引号，将会在转成json时带有反斜杠转移符。如果不需要转义，可以使用这个属性。默认为false
    NotWriteRootClassName   含义  
    BeanToArray 将对象转为array输出
    WriteNonStringKeyAsString   含义  
    NotWriteDefaultValue    含义  
    BrowserSecure   含义  
    IgnoreNonFieldGetter      
    WriteEnumUsingName  用枚举name()输出
