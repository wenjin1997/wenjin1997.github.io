---
layout:     post

title:      "发现负面新闻"
# subtitle:   "Java并发编程的艺术笔记"
# excerpt: "Java并发编程的艺术笔记"
author:     "谢文进"
date:       2022-05-08
description: "梁博云实习的第一个项目，发现负面新闻。"
# image: "/img/2022-03-28-The-Art-of-Java-Concurrency-Programming/background.jpg"
published: 2022-05-08 
tags:
     - Java
     - Spring Boot
     - 服务器

categories: [ Tech ]
URL: "/2022/05/08/liangbo-negative-news/"
---
## 任务
这是梁博云实习的第一个项目，发现负面新闻。

实习目标
1. 学习和掌握json代码的解析和生成
2. 学习一种http服务的方式
3. 学习如何使用开放API

第一步、学习和利用现有一个情感计算API
[http://www.pullword.com/baobian/](http://www.pullword.com/baobian/)

调用样例：Linux命令行下执行
```
curl -X POST 'http://baobianapi.pullword.com:9091/get.php' -d'十一月再见，十二月你好，2021年最后一个月总会有不期而遇的温暖，和生生不息的希望' –compressed
```

返回的json结果要能解析，大于0.5表示正面情感，小于-0.5表明负面情感。或者使用其他自己熟悉的API也可以。推荐使用梁博公开的服务。

第二步，学习提供http服务的工具

C语言选手可以通过搜狗工作流开源工具实现。不会C和C++的同学可以用php代码或者其他自己熟悉的语言实现。

第三步，学习和了解数据源
http://news.baidu.com/

从百度新闻首页获得当天热门新闻，用curl或者wget命令采集首页。然后提取其中的新闻标题和URL


第四步，将今日新闻标题调用情感计算API得到正负面评价，提取负面结果的新闻标题和URL，通过第二步掌握的http服务工具以json格式对外展示。展示的内容包括标题和URL的列表。在自己的linux机器上能访问即可，有公网服务器的可以开放公网URL供我们检查，没有公网服务器的截图即可。

## 爬取百度新闻
打开[百度新闻](http://news.baidu.com/)，F12查看页面源码，可以看到热点新闻的`class=mod-tab-pane active`。
![baidu-news](/img/2022-05-08-liangbo-negative-news/baidunews.png)
用如下方法得到新闻标题和链接：
```java
// 获取热点新闻HTML代码
Elements hotNews = doc.select("[class=mod-tab-pane active]").select("a");
// 获取新闻标题及链接
for (int i = 0; i < hotNews.size(); i++) {
     news.put(hotNews.get(i).text(), hotNews.get(i).attr("href"));
}
```
程序的完整逻辑是先得到百度新闻的HTML文件，再从`class`筛选得到相应的标题和链接，这里`Elements`类是实现了`List<Element>`接口，因此是可以遍历的，在遍历时加入存放新闻的标题和链接的哈希表。
```java
package com.jinjin.news.controller;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.select.Elements;

import java.io.IOException;
import java.util.HashMap;

/**
 * @author 文进
 * @version 1.0
 */
public class News {
    /**
     *
     * @param url 访问路径
     * @return
     */
    public Document getDocument (String url){
        try {
            //5000是设置连接超时时间，单位ms
            return Jsoup.connect(url).timeout(5000).get();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * @param url 访问网站的url
     * @return 返回热点新闻的标题和链接
     */
    public HashMap getNews (String url) {
        Document doc = getDocument(url);
        HashMap<String, String> news = new HashMap<>();
        // 获取热点新闻HTML代码
        Elements hotNews = doc.select("[class=mod-tab-pane active]").select("a");
        // 获取新闻标题及链接
        for (int i = 0; i < hotNews.size(); i++) {
            news.put(hotNews.get(i).text(), hotNews.get(i).attr("href"));
//            System.out.println(hotNews.get(i).text() + "   " + hotNews.get(i).attr("href"));
        }
        return news;
    }
}
```
参考资料：[Java爬取网页内容的简单例子](https://blog.csdn.net/sinat_35626559/article/details/78285007)

## 使用情感计算API
情感计算API链接是：http://www.pullword.com/baobian/

我们要做的是将爬取到的新闻标题用POST方法向上述链接发请求，最后得到一个json结果，提取其中的情感数值，将负的情感值对应的新闻标题保存下来，由于之前存取的新闻是HashMap，因此得到负面新闻的URL也不是难事了。

参考[java向指定URL发送GET或POST请求](https://blog.csdn.net/qq_34706514/article/details/82658380)，我们用到了UrlUtils工具类，我们需要用的是`public static String sendPost(String url, String param){}`这个带参数的发送POST请求的方法。
```java
package com.jinjin.news.controller;

import com.sun.org.slf4j.internal.Logger;
import com.sun.org.slf4j.internal.LoggerFactory;

import java.io.*;
import java.net.URL;
import java.net.URLConnection;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

/**
 * @author 文进
 * @version 1.0
 */
public class UrlUtils {
    private static final Logger log = LoggerFactory.getLogger(UrlUtils.class);

    /**
     * 向指定URL发送GET方法的请求
     *
     * @param url   发送请求的URL
     * @param param 请求参数，请求参数应该是 name1=value1&name2=value2 的形式。
     * @return URL 所代表远程资源的响应结果
     */
    public static String sendGet(String url, String param) {
        String result = "";
        BufferedReader in = null;
        try {
            String urlNameString = url + "?" + param;
            URL realUrl = new URL(urlNameString);
            // 打开和URL之间的连接
            URLConnection connection = realUrl.openConnection();
            // 设置通用的请求属性
            connection.setRequestProperty("accept", "*/*");
            connection.setRequestProperty("connection", "Keep-Alive");
            connection.setRequestProperty("user-agent",
                    "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
            // 建立实际的连接
            connection.connect();
            // 获取所有响应头字段
            Map<String, List<String>> map = connection.getHeaderFields();
            // 遍历所有的响应头字段
//            for (String key : map.keySet()) {
//                System.out.println(key + "--->" + map.get(key));
//            }
            // 定义 BufferedReader输入流来读取URL的响应
            in = new BufferedReader(new InputStreamReader(
                    connection.getInputStream()));
            String line;
            while ((line = in.readLine()) != null) {
                result += line;
            }
        } catch (Exception e) {
            log.error("发送GET请求出现异常！" + e);
            e.printStackTrace();
        }
        // 使用finally块来关闭输入流
        finally {
            try {
                if (in != null) {
                    in.close();
                }
            } catch (Exception e2) {
                e2.printStackTrace();
            }
        }
        return result;
    }

    /**
     * 向指定 URL 发送POST方法的请求
     *
     * @param url      发送请求的 URL
     * @param paramMap 请求参数
     * @return 所代表远程资源的响应结果
     */
    public static String sendPost(String url, Map<String, ?> paramMap) {
        PrintWriter out = null;
        BufferedReader in = null;
        String result = "";
        String param = "";
        Iterator<String> it = paramMap.keySet().iterator();
        while (it.hasNext()) {
            String key = it.next();
            param += key + "=" + paramMap.get(key) + "&";
        }
        try {
            URL realUrl = new URL(url);
            // 打开和URL之间的连接
            URLConnection conn = realUrl.openConnection();
            // 设置通用的请求属性
            conn.setRequestProperty("accept", "*/*");
            conn.setRequestProperty("connection", "Keep-Alive");
            conn.setRequestProperty("Accept-Charset", "utf-8");
            conn.setRequestProperty("user-agent", "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
            // 发送POST请求必须设置如下两行
            conn.setDoOutput(true);
            conn.setDoInput(true);
            // 获取URLConnection对象对应的输出流
            out = new PrintWriter(conn.getOutputStream());
            // 发送请求参数
            out.print(param);
            // flush输出流的缓冲
            out.flush();
            // 定义BufferedReader输入流来读取URL的响应
            in = new BufferedReader(new InputStreamReader(conn.getInputStream(), "UTF-8"));
            String line;
            while ((line = in.readLine()) != null) {
                result += line;
            }
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
        //使用finally块来关闭输出流、输入流
        finally {
            try {
                if (out != null) {
                    out.close();
                }
                if (in != null) {
                    in.close();
                }
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
        return result;
    }


    /**
     * 发送post请求 json格式
     *
     * @param url
     * @param param json字符串
     * @return
     */
    public static String sendPost(String url, String param) {
        PrintWriter out = null;
        BufferedReader in = null;
        StringBuilder result = new StringBuilder();
        try {
            URL realUrl = new URL(url);
//            // 忽略https功能，普通使用可以删除
//            if ("https".equalsIgnoreCase(realUrl.getProtocol())) {
//
//                SslUtils.ignoreSsl();
//            }
            // 打开和URL之间的连接
            URLConnection conn = realUrl.openConnection();
            // 设置通用的请求属性
            conn.setRequestProperty("accept", "*/*");
            conn.setRequestProperty("connection", "Keep-Alive");
            conn.setRequestProperty("Accept-Charset", "utf-8");
            conn.setRequestProperty("user-agent", "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
            // 发送POST请求必须设置如下两行
            conn.setDoOutput(true);
            conn.setDoInput(true);
            // 获取URLConnection对象对应的输出流
            //out = new PrintWriter(conn.getOutputStream());
            out = new PrintWriter(new OutputStreamWriter(conn.getOutputStream(), "UTF-8"));
            // 发送请求参数
            out.print(param);
            // flush输出流的缓冲
            out.flush();
            // 定义BufferedReader输入流来读取URL的响应
            in = new BufferedReader(new InputStreamReader(conn.getInputStream(), "UTF-8"));
            String line;
            while ((line = in.readLine()) != null) {
                //result += line;
                result.append(line);
            }
        } catch (Exception e) {
            System.out.println("发送 POST 请求出现异常!" + e);
            e.printStackTrace();
        }
        //使用finally块来关闭输出流、输入流
        finally {
            try {
                if (out != null) {
                    out.close();
                }
                if (in != null) {
                    in.close();
                }
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
        //log.info("url返回: " +result);
        return result.toString();
    }
}
```
## 创建Spring Boot项目
上述模块都测试好之后，最后就是返回JSON数据格式，参考[<后端初学者>Spring Boot 返回 JSON 数据及数据封装](https://juejin.cn/post/6873288147820609550)，有多种方法，其中一种用Spring Boot中的`@RestController`注解来返回JSON数据，那么我们创建一个Spring Boot项目，再把上述模块的代码迁移过来就好啦。

创建Spring Boot项目的步骤参考[2020入门教程macOS 使用Intellj IDEA 创建spring boot项目](https://blog.csdn.net/lxyoucan/article/details/111128415)。

## 返回JSON数据格式
首先为了方便返回JSON数据格式，我们先将`NegativeNews`封装成一个类，其主要的属性就两个：新闻的标题`title`和对应的`url`。剩下的就是常见的构造器以及get、set方法。
```java
package com.jinjin.news.controller;

/**
 * @author 文进
 * @version 1.0
 */
public class NegativeNews {
    private String title;
    private String url;


    public NegativeNews(String title, String url) {
        this.title = title;
        this.url = url;
    }


    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }
}
```
返回JSON数据格式可以选择多种形式，这里由于要返回的新闻有多条，因此我们返回一个`List<NegativeNews>`。

在返回JSON数据格式方法的前面加上注解`@RequestMapping("/negativeNews")`，在请求该页面时，就会执行其中的代码，因此我们将上述爬取百度新闻标题，向情感计算API发送POST请求，得到负面新闻，综合到该方法中一起执行。

```java
package com.jinjin.news.controller;

import com.sun.net.httpserver.HttpServer;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * @author 文进
 * @version 1.0
 */
@RestController
public class HttpServers {
    @RequestMapping("/negativeNews")
    public List<NegativeNews> returnJson( ) {
        // 获取百度新闻中热点新闻的标题和链接
        HashMap<String, String> news = new News().getNews("http://news.baidu.com/");
        // 负面新闻
        HashMap<String, String> negativeNews = new HashMap<>();

        // 通过新闻标题得到的情感值，将负面新闻添加到 negativeNews 中
        for (Map.Entry<String, String> entry : news.entrySet()) {
            // 得到情感API返回的结果，例如 data = {"result":0.739306}
            String data = UrlUtils.sendPost("http://baobianapi.pullword.com:9091/get.php", entry.getKey());
            // 提取结果中的分数值 data = {"result": score}
            String score = data.substring(10, data.length() - 1);
            // 将分数值 score 转换成double数值，方便找到负面情绪的新闻
            double emotion = Double.parseDouble(score);
            // 将负面新闻添加到 negativeNews 中
            if (emotion < -0.5) {
                negativeNews.put(entry.getKey(), entry.getValue());
            }
        }

        // 返回 JSON 数据格式
        List<NegativeNews> negativeNewsList = new ArrayList<>();
        for (Map.Entry<String, String> entry : negativeNews.entrySet()) {
            NegativeNews nNews = new NegativeNews(entry.getKey(), entry.getValue());
            negativeNewsList.add(nNews);
        }
        return negativeNewsList;
    }
}
```
## 部署到服务器
部署到服务器有两种方式，参考：
 * [spring boot 项目部署到服务器 两种方式](https://blog.csdn.net/qq_22638399/article/details/81506448)
 * [SpringBoot 项目部署到服务器的两种方式](https://blog.csdn.net/CSDN2497242041/article/details/106911182)

### maven项目打包
我们选择打包成jar包，其中，有可能遇到发生一些类找不到的错误，需要将本地jar包打包进去，参考[maven项目引入本地jar包史上最详细实践方法](https://cloud.tencent.com/developer/article/1746822)。

解决如下类找不到的情况：
```java
import com.sun.org.slf4j.internal.Logger;
import com.sun.org.slf4j.internal.LoggerFactory;
```
经过跟踪，这两个都在`rt.jar`类中，用引入本地jar包的方法还是不成功，最终参考[Idea本地maven打包，程序包不存在](https://blog.csdn.net/qq_33666602/article/details/103438713)解决了，将`rt.jar`复制到`jdk/jre/lib/ext`中。

引入`org.json`，从中获取相应版本的依赖代码，添加到pom.xml配置文件中。
```java
<dependency>
     <!-- jsoup HTML parser library @ http://jsoup.org/ -->
     <groupId>org.jsoup</groupId>
     <artifactId>jsoup</artifactId>
     <version>1.6.1</version>
</dependency>
```
添加后，在maven中按刷新按钮，如下图：

![maven](/img/2022-05-08-liangbo-negative-news/maven.png)

打包好后现在本地测试一下，终端进入到jar所在的路径，运行如下代码：
```
java -jar 包名.jar
```
### Linux上运行
参考[在Linux上运行springboot项目](https://blog.csdn.net/CSDg166/article/details/109386959)。

为了让服务端关闭SSH连接后依然运行，运行时执行如下命令：
```
nohup java -jar 包名.jar &
``` 
杀掉运行中的进程，首先查看：
```
ps aux|grep getCimiss-surf.jar
```
接着杀掉进程：
```
kill -9 30768
```

MacOS上查看对应端口的进程：
```
lsof -i tcp:端口号
```
杀掉对应进程：
```
kill -9 PID号
```
## 代码及公网地址
至此整个过程就完成了:)
* 代码：[negativeNews](https://github.com/wenjin1997/liangbo-internship/tree/main/negativeNews)
* 公网地址：[http://139.155.0.15/negativeNews](http://139.155.0.15/negativeNews)
* 公网请求负面新闻截图：
![negativeNews](/img/2022-05-08-liangbo-negative-news/negativeNews.png)