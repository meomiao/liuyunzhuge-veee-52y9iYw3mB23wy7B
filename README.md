
最近给一个私募大佬帮忙做了一些股票交易有关的系统，其中涉及到行情数据抓取的问题，一番摸索之后，把成果在这里做个分享。


我把行情抓取的部分，和一个写手记的小功能，单独拿了出来放在一个小系统里面，可以免费使用：[https://rich.shengxunwei.com/](https://github.com)


先简单介绍下这个小系统的样子，然后我会详细的解释如何高性能实时抓取股票行情。


可以添加自己关注股票列表，支持股票、场内基金、可转债：


![](https://img2024.cnblogs.com/blog/78019/202410/78019-20241031112458382-1080177142.png)


股票详情数据：


![](https://img2024.cnblogs.com/blog/78019/202410/78019-20241031112947100-1378487712.png)


场内基金详情数据：


![](https://img2024.cnblogs.com/blog/78019/202410/78019-20241031113000753-1743803917.png)


可转债详情数据：


![](https://img2024.cnblogs.com/blog/78019/202410/78019-20241031113021813-2077399574.png)



> 在线演示：[https://rich.shengxunwei.com/](https://github.com)




---


## 行情抓取的数据来源


目前主要的数据来源是几个财经网站，比如东财：


![](https://img2024.cnblogs.com/blog/78019/202410/78019-20241031114345110-2114099510.png)


## 实时行情抓取方法


实时行情抓取的一个核心类是 HttpClient 类，然后只需访问东财的网站即可。


### 创建 HttpClient


下面的大多数示例都重复使用同一 HttpClient 实例，因此只需配置一次。 要创建 HttpClient，请使用 HttpClient 类构造函数。



```
// HttpClient lifecycle management best practices:
// https://learn.microsoft.com/dotnet/fundamentals/networking/http/httpclient-guidelines#recommended-use
private static HttpClient sharedClient = new()
{
    BaseAddress = new Uri("https://jsonplaceholder.typicode.com"),
};

```

前面的代码：


实例化新的 HttpClient 实例作为 static 变量。 根据准则，建议在应用程序的生命周期内重复使用 HttpClient 实例。
将 HttpClient.BaseAddress 设置为 "[https://jsonplaceholder.typicode.com](https://github.com):[westworld加速](https://tianchuang88.com)"。
发出后续请求时，此 HttpClient 实例将使用基址。 若要应用其他配置，请考虑：


设置 HttpClient.DefaultRequestHeaders。
应用非默认 HttpClient.Timeout。
指定 HttpClient.DefaultRequestVersion。


虽然面向 Android 设备（如 .NET MAUI 开发），但必须将 android:usesCleartextTraffic\="true" 添加到 AndroidManifest.xml 中的 。 这将启用明文流量，例如 HTTP 请求；否则由于 Android 安全策略，默认情况下会禁用。 请考虑以下示例 XML 设置：



```
xml version="1.0" encoding="utf-8"?
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
  <application android:usesCleartextTraffic="true">application>
  
manifest>

```

### HTTP 内容


HttpContent 类型用于表示 HTTP 实体正文和相应的内容标头。 对于需要正文的 HTTP 方法（或请求方法）POST、PUT 和 PATCH，可使用 HttpContent 类来指定请求的正文。 大多数示例演示如何使用 JSON 有效负载准备 StringContent 子类，但还有针对其他内容 (MIME) 类型的其他子类。


* ByteArrayContent：提供基于字节数组的 HTTP 内容。
* FormUrlEncodedContent：为使用 "application/x\-www\-form\-urlencoded" MIME 类型编码的名称/值元组提供 HTTP 内容。
* JsonContent：提供基于 JSON 的 HTTP 内容。
* MultipartContent：提供使用 "multipart/\*" MIME 类型规范进行序列化的 HttpContent 对象的集合。
* MultipartFormDataContent：为使用 "multipart/form\-data" MIME 类型进行编码的内容提供容器。
* ReadOnlyMemoryContent：提供基于 ReadOnlyMemory 的 HTTP 内容。
* StreamContent：提供基于流的 HTTP 内容。
* StringContent：提供基于字符串的 HTTP 内容。
* HttpContent 类还用于表示 HttpResponseMessage 的响应正文，可通过 HttpResponseMessage.Content 属性访问。


### HTTP Get


GET 请求不应发送正文，而是用于（如方法名称所示）从资源检索（或获取）数据。 要在给定 HttpClient 和 URI 的情况下发出 HTTP GET 请求，请使用 HttpClient.GetAsync 方法：



```
static async Task GetAsync(HttpClient httpClient)
{
    using HttpResponseMessage response = await httpClient.GetAsync("todos/3");
    
    response.EnsureSuccessStatusCode()
        .WriteRequestToConsole();
    
    var jsonResponse = await response.Content.ReadAsStringAsync();
    Console.WriteLine($"{jsonResponse}\n");

    // Expected output:
    //   GET https://jsonplaceholder.typicode.com/todos/3 HTTP/1.1
    //   {
    //     "userId": 1,
    //     "id": 3,
    //     "title": "fugiat veniam minus",
    //     "completed": false
    //   }
}

```

### HTTP Get from JSON


[https://jsonplaceholder.typicode.com/todos](https://github.com) 终结点返回“todo”对象的 JSON 数组。 这些对象的 JSON 结构如下所示



```
[
  {
    "userId": 1,
    "id": 1,
    "title": "example title",
    "completed": false
  },
  {
    "userId": 1,
    "id": 2,
    "title": "another example title",
    "completed": true
  },
]

```

本地计算机或应用程序配置文件可以指定使用默认代理。 如果指定了 Proxy 属性，则 Proxy 属性中的代理设置会覆盖本地计算机或应用程序配置文件，并且处理程序将使用指定的代理设置。 如果未在配置文件中指定代理，并且未指定 Proxy 属性，则处理程序将使用从本地计算机继承的代理设置。 如果没有代理设置，则请求将直接发送到服务器。


