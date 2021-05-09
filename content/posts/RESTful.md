---
title: "Jersey 的RESTful API"
date: 2019-07-28T09:00:50+08:00
draft: false
categories: ["Java相关"]
tags: ["Java","Jersey","RESTful","Web"]
author: "Jason·Gan"
---
* REST 是英文 Representational State Transfer 的缩写，有中文翻译为“表述性状态传输”。REST 这个术语是由 Roy Fielding 在他的博士论文 《 Architectural Styles and the Design of Network-based Software Architectures 》中提出的。REST 并非标准，而是一种开发 Web 应用的架构风格，可以将其理解为一种设计模式。REST 基于 HTTP，URI，以及 XML 这些现有的广泛流行的协议和标准，伴随着 REST，HTTP 协议得到了更加正确的使用。相较于基于 SOAP 和 WSDL 的 Web 服务，REST 模式提供了更为简洁的实现方案。目前，越来越多的 Web 服务开始采用 REST 风格设计和实现，真实世界中比较著名的 REST 服务包括：Google AJAX 搜索 API、Amazon Simple Storage Service (Amazon S3)等。  
* 系统中的每一个对象或是资源都可以通过一个唯一的 URI 来进行寻址，URI 的结构应该简单、可预测且易于理解，比如定义目录结构式的 URI。
* 以遵循 RFC-2616 所定义的协议的方式显式地使用 HTTP 方法，建立创建、检索、更新和删除（CRUD：Create, Retrieve, Update and Delete）操作与 HTTP 方法之间的一对一映射：
  * 若要在服务器上创建资源，应该使用 POST 方法；
  * 若要检索某个资源，应该使用 GET 方法；
  * 若要更改资源状态或对其进行更新，应该使用 PUT 方法；
  * 若要删除某个资源，应该使用 DELETE 方法。
* URI 所访问的每个资源都可以使用不同的形式加以表示（比如 XML 或者 JSON），具体的表现形式取决于访问资源的客户端，客户端与服务提供者使用一种内容协商的机制（请求头与MIME类型）来选择合适的数据格式，最小化彼此之间的数据耦合。  
* 要开始使用 Jersey 客户端 API，你首先需要创建一个 com.sun.jersey .api.client.Client 类的实例。下面是最简单的方法：  
```  

import com.sun.jersey .api.client.Client;
Client client = Client.create();
```
* Client 类是创建一个 RESTful Web Service 客户端的主要配置点。你可以使用它来配置不同的客户端属性和功能，并且指出使用哪个资源提供者。创建一个 Client 类的实例是一个比较昂贵的操作，所以尽量避免创建一些不需要的客户端实例。比较好的方式是尽可能地复用已经存在的实例。
* 当你创建完一个 Client 类的实例后，你可以开始使用它。无论如何，在发出请求前，你需要创建一个 Web Resource 对象来封装客户端所需要的 Web 资源。
* Web 资源创建了一个 WebResponse 对象：  
```  

import com.sun.jersey .api.client.WebResource;
Web Resource webResource = c.resource("http://example.com/base");
```  

* 通过使用 WebResource 对象来创建要发送到 Web 资源的请求，以及处理从 Web 资源返回的响应。例如，你可以使用 WebResource 对象来发送 HTTP GET、PUT、POST 以及 DELETE 请求。
* GET 请求：使用 WebResource 类的 get() 方法来提交一个 HTTP GET请求到 Web 资源：
``` String s = webResource.get(String.class); ```这表示如果 WebResource 对象的 URL 是 http://example.com/base，那么一个 HTTP GET 请求将会发送到地址为 http://example.com/base 的资源。
* 你还可以指定 get() 请求时的查询参数。例如，下面的代码在 get() 请求中指定了两个查询参数：  
```  

MultivaluedMap queryParams = new MultivaluedMapImpl();
queryParams.add("param1", "val1");
queryParams.add("param2", "val2");
String s = webResouce.queryParams(queryParams).get(String.class);
```  
* 可以指定响应所能接受的 MIME 类型。例如，下面的代码指定了响应的 MIME 类型只能为文本：
``` String s = webResource.accept("text/plain").get(String.class); ```
* 还可以获取对应请求的 HTTP 状态码，例如下面这个例子展示获取一个请求所返回的文本实体与状态码：  
```  

ClientResponse response = webResource.accept("text/plain").get(ClientResponse.class);
int status = response.getStatus();
String textEntity = response.getEntity(String.class);
```  
* ClientResponse 对象代表了一个客户端收到的 HTTP 响应。     
* PUT 请求 ：使用 WebResource 类的 put() 方法来提交一个 HTTP PUT 请求到 Web 资源。例如下面的代码展示了请求发送一个文本实体 foo:bar 到指定的 Web 资源:  
```  

ClientResponse response = webResource.type("text/plain").put(ClientResponse.class, "foo:bar");    
```
* 同样，也可以使用put()方法发送请求时指定查询参数，方法与使用get()方法时指定查询参数一样：  
```

MultivaluedMap formData = new MultivaluedMapImpl();
formData.add("name1", "val1");
formData.add("name2", "val2");
ClientResponse response = webResource.type("application/x-www-form-urlencoded").post(ClientResponse.class, formData);
```  
* Delete请求：使用Web Resource.path类的delete()方法来发送Http Delete请求到指定的Web资源。例如，下面的例子展示删除一个URL为http://example.com/base/user/123 资源：  
``` 

ClientResponse response = webResource.path("user/123").delete(ClientResponse.class);  
```
* Web Resource.path() 方法可以在所有 HTTP 请求中使用，它可以让你给要请求的 Web 资源指定一个额外的路径。另一个 WebResouce 类的方法 header() 可以给你的请求添加 HTTP 头部信息。
* 以下是一个Jersey客户端的示例：  
```


import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import com.rimi.medical.common.domain.ResultPojo;
import com.sun.jersey.api.client.Client;
import com.sun.jersey.api.client.ClientResponse;
import com.sun.jersey.api.client.WebResource;
import com.sun.jersey.core.util.MultivaluedMapImpl;

import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.MultivaluedMap;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

/**
 * JerseyAPi客户端
 * Created by libt on 2015/01/30.
 */
public class JerseyClientUtil {

    private static final String BIGDATA_API_URL = ReadSettingProperties.getValue("bigdata_api_url");

    /**
     * post方法
     *
     * @param method 方法名
     * @param param  参数
     * @return 返回值
     */
    public static ResultPojo postMethod(String method, String param) {
        ResultPojo resultPojo = new ResultPojo();
        ClientResponse response = null;
        try {
            Client client = Client.create();
            WebResource resource = client.resource(BIGDATA_API_URL + method);
            response = resource.type(MediaType.APPLICATION_JSON_TYPE).post(ClientResponse.class, param);
            int status = response.getStatus();
            String data = response.getEntity(String.class);
            if (status == 200) {
                JSONObject jsonObject = JSON.parseObject(data);
                resultPojo.setStatus(jsonObject.getInteger("status"));
                resultPojo.setData(data);
            } else {
                resultPojo.setStatus(response.getStatus());
                resultPojo.setData(data);
            }
        } catch (Exception e) {
            resultPojo.setStatus(500);//服务器异常
            resultPojo.setErrorMsg(e.getMessage());
        } finally {
            if (response != null) {
                response.close();
            }
        }
        return resultPojo;
    }


    /**
     * get方法
     * 例如：consultation/recommend?startDate=201412030253&endDate=201412020253
     * @param method 方法名
     * @param param  参数
     * @return 返回值
     */
    public static ResultPojo getMethod(String method, String param) {
        ResultPojo resultPojo = new ResultPojo();
        ClientResponse response = null;
        try {
            Client client = Client.create();
            WebResource resource = client.resource(BIGDATA_API_URL + method);
            response = resource.queryParams(parseJSON2Map(param)).accept(MediaType.APPLICATION_JSON_TYPE).get(ClientResponse.class);
            int status = response.getStatus();
            String data = response.getEntity(String.class);
            if (status == 200) {
                JSONObject jsonObject = JSON.parseObject(data);
                resultPojo.setStatus(jsonObject.getInteger("status"));
                resultPojo.setData(data);
            } else {
                resultPojo.setStatus(response.getStatus());
                resultPojo.setData(response.getEntity(String.class));
            }
        } catch (Exception e) {
            e.printStackTrace();
            resultPojo.setStatus(500);//服务器异常
            resultPojo.setErrorMsg(e.getMessage());
        } finally {
            if (response != null) {
                response.close();
            }
        }
        return resultPojo;
    }

    /**
     * get方法
     * 例如：consultation/recommend/A1000037B04B8C
     * @param method 方法名
     * @param param  参数
     * @return 返回值
     */
    public static ResultPojo getMethodOnly(String method, String param) {
        ResultPojo resultPojo = new ResultPojo();
        ClientResponse response = null;
        try {
            Client client = Client.create();
            WebResource resource = client.resource(BIGDATA_API_URL + method + param);
            response = resource.accept(MediaType.APPLICATION_JSON_TYPE).get(ClientResponse.class);
            int status = response.getStatus();
            String data = response.getEntity(String.class);
            if (status == 200) {
                JSONObject jsonObject = JSON.parseObject(data);
                resultPojo.setStatus(jsonObject.getInteger("status"));
                resultPojo.setData(data);
            } else {
                resultPojo.setStatus(response.getStatus());
                resultPojo.setData(response.getEntity(String.class));
            }
        } catch (Exception e) {
            e.printStackTrace();
            resultPojo.setStatus(500);//服务器异常
            resultPojo.setErrorMsg(e.getMessage());
        } finally {
            if (response != null) {
                response.close();
            }
        }
        return resultPojo;
    }

    public static MultivaluedMap parseJSON2Map(String jsonStr) {
        MultivaluedMap queryParams = new MultivaluedMapImpl();
        //最外层解析
        JSONObject json = JSON.parseObject(jsonStr);
        for (Map.Entry<String, Object> entry : json.entrySet()) {
            Object v = entry.getValue();
            //如果内层还是数组的话，继续解析
            if (v instanceof JSONArray) {
                List<Map<String, Object>> list = new ArrayList<Map<String, Object>>();
                Iterator<Object> it = ((JSONArray) v).iterator();
                while (it.hasNext()) {
                    JSONObject json2 = (JSONObject) it.next();
                    list.add(parseJSON2Map(json2.toJSONString()));
                }
                queryParams.add(entry.getKey(), list);
            } else {
                queryParams.add(entry.getKey(), v);
            }
        }
        return queryParams;
    }


    public static void main(String[] args) {

//        ResultPojo resultPojo = postMethod("bfr/bfr_choices", "{\"userid\":\"00004\",\"createTime\":\"2014-09-23 16:19:23\",\"bmiScore\":\"80\",\"imageNum\":\"01\",\"type\":\"0\",\" info \":\"个人身体质量分析正常\"}");
        ResultPojo resultPojo = getMethod("recommendInfo/query", "{\"endDate\":\"201412020253\",\"startDate\":\"201410010253\"}");
//        ResultPojo resultPojo = getMethodOnly("consultation/recommend/", "A1000037B04B8C");
        System.out.println(resultPojo.getStatus());
        System.out.println(resultPojo.getErrorMsg());

    }
}
```
