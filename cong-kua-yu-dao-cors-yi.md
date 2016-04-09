#### 1. 介绍

跨域，可能很多前端开发者都会遇到过，也可能知道有jsonp，iframe之类的跨域方法。不过要说这些方法之前，先得来说说什么叫跨域，为什么要跨域。

所谓跨域，顾名思义，跨到了另外的域，域不仅仅指的是不同的域名网站，可能同一个域名不同的端口号也算不同的域。浏览器是有规则的，只要协议、域名、端口有任何一个不同，都被当作是不同的域。协议指的是http，或者https等。

下面给出一个列表，指出不同域的情况：

![](http://aliyun.rails365.net/uploads/photo/image/98/preview_2016/8ed6cce9a01810beefba77ab39c268a5.png)

这个叫浏览器的同源策略(same-origin policy)，为什么要这样规定呢，原因之一就是为了安全。

假如你在A网站，用一段js脚本就能访问B网站，B网站凭什么让你访问，为什么浏览器能让你随便访问别的网站呢，说不定B网站存在着各种危险，比如盗取你的密码，跨域攻击(CSRF)等，一切都不太合理。

但也是有例外的情况，有些情况是要访问到别的网站的，比如加载一张图片，这张图片可能在别的网站，还有加载一个js文件，也是有可能在别的网站的，还有嵌入一个frame元素，也可以访问到别的网站的内容。

所以跨域正是利用了上面几点，比如jsonp就是利用的加载js文件的功能，比如在`<script src="..."></script>`中的src指定为目标网站的js，同理，还有跨域攻击也可以利用`<image src="..." />`的功能。还有iframe更能直接加载别的网站的内容到自己的网站里来。

这前面几种可以说是技巧，说句不好听的就是浏览器的漏洞，浏览器并未从正面上支持跨域，而且上面的几种跨域方法也是有各种局限，比如jsonp的方法，只能用GET方法，iframe方法是能直接加载内容到网站上，但是本网站和iframe的数据交互也是一个头疼的地方。

毕竟从iframe加载内容到本站后，是存在着数据交互的，可以用`document.domain`，`window.name`，不过这些方法都能用，且有效，不过总不尽完美，存在着各种各样的局限。

然而HTML5引进了一个叫window.postMessage方法来跨域传送数据，这个倒是不错，不过也是利用了iframe。

除此之外，还有flash，服务器代理等跨域方法，但是本章要介绍的是浏览器或服务器的跨域方法，它的名字叫CORS。

#### 2. CORS

CORS全称是Cross-Origin Resource Sharing，跨域资源共享，这是浏览器的标准，也算是协议，基本上现代浏览器都支持，除了奇葩浏览器，例如IE8、IE9，只支持部分特性。

使用它，需要服务器端和客户端两方面的准备，服务器端我们选择nginx作为测试，客户端只是js罢了。

nginx服务监听在localhost的8080端口，而现在有一个网站运行localhost的3000端口，需要跨域到nginx那台服务器。

现在开始测试之旅，要在浏览器模拟跨域请求，只需三行js代码。

``` javascript
var xhttp = new XMLHttpRequest();
xhttp.open("GET", "http://localhost:8080", true);
xhttp.send();
```

我使用的浏览器是chrome，打开它的开发者工具，在`console`里运行上面的代码，效果如下：

![](http://aliyun.rails365.net/uploads/photo/image/99/preview_2016/2772ecd0077ff793bd13f638427b6e31.png)

上面的报错已经提示得很明显了，主要是下面这句话：

```
XMLHttpRequest cannot load http://localhost:8080/. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:3000' is therefore not allowed access.
VM162:4 XHR failed loading: GET "http://localhost:8080/".
```

错误中说`Access-Control-Allow-Origin`这个头信息不存在于被请求的服务器中，来源域`http://localhost:3000`是不允许访问该服务器的。

`XMLHttpRequest`这个可能很多开发者都明白，那是ajax请求利用的对象，利用它能发起ajax请求，但它的功能不仅仅是发起ajax请求，还能用于跨域，还有设置时限，FormData对象管理表单数据，文件上传等功能，具体可以自行搜索相关的资料，在这里，它能发起跨域请求就可以了。

我们来看看这个请求相关的信息。

![](http://aliyun.rails365.net/uploads/photo/image/101/preview_2016/804c38c2a3c748d8fd61a9bd212875ce.png)

具体的头信息如下：

``` ruby
Request Headers
Accept:*/*
Accept-Encoding:gzip, deflate, sdch
Accept-Language:en-US,en;q=0.8,zh-CN;q=0.6,zh;q=0.4
Connection:keep-alive
Host:localhost:8080
Origin:http://localhost:3000
Referer:http://localhost:3000/
User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.97 Safari/537.36
```

最重要的是`Origin:http://localhost:3000`这一行，它标明的是来源的域，这是请求头信息，会传给服务器。

我们来看下服务器。

![](http://aliyun.rails365.net/uploads/photo/image/102/preview_2016/85cecc9322e4cdd43fd3afd2e69e6713.png)

可见，服务器还是正常响应200状态的。也就是说，服务器端该怎么应答就怎么应答，只是被浏览器给阻止了。

浏览器是如何阻止的呢。主要是看响应头部的信息。

```
Response Headers
Accept-Ranges:bytes
Connection:keep-alive
Content-Length:612
Content-Type:text/html
Date:Mon, 01 Feb 2016 07:17:54 GMT
ETag:"56988bc6-264"
Last-Modified:Fri, 15 Jan 2016 06:03:50 GMT
Server:nginx/1.8.0
```

它没有发现有发现`Origin:http://localhost:3000`这个来源域名是被服务器所允许的。

我们在服务器端设置一下，让`Origin:http://localhost:3000`这个来源域被允许访问。

``` conf
location / {
  add_header 'Access-Control-Allow-Origin' '*';
}
```

``add_header 'Access-Control-Allow-Origin' '*'`表示允许任何来源域访问nginx这台服务器。

用`sudo nginx -s reload`重新加载服务器配置。

再来看下效果。

![](http://aliyun.rails365.net/uploads/photo/image/103/preview_2016/713b3af4a8058d57333b92c6f248b0d3.png)

果然成功了，不再提示错误。

来看下响应的信息。

``` conf
Response Headers
Accept-Ranges:bytes
Access-Control-Allow-Origin:*
Connection:keep-alive
Content-Length:612
Content-Type:text/html
Date:Mon, 01 Feb 2016 07:25:25 GMT
ETag:"56988bc6-264"
Last-Modified:Fri, 15 Jan 2016 06:03:50 GMT
Server:nginx/1.8.0
```

响应信息中多了这一行`Access-Control-Allow-Origin:*`。

原来，浏览器在用`XMLHttpRequest`发起跨域请求的时候，它在请求头带了`Origin`这个项，而服务器，在响应头信息中是有响应`Access-Control-Allow-Origin`这项的，两者比较一下，如果匹配，则请求成功，不匹配就不成功，不过，服务器那边还是照常执行。

**在测试的时候需要注意的事，有可能会发现改了nginx的配置，浏览器发出的跨域请求却没生效，这可能是因为浏览器cache的原因，只要清除浏览器的cache，再重新发起请求就好了。**

下一节[CORS进阶之Preflight请求(二)](http://www.rails365.net/articles/cors-jin-jie-zhi-preflight-qing-qiu-er)会介绍CORS更高阶的内容，比如`Preflight请求`，`Access-Control-Allow-Methods`，`Access-Control-Allow-Headers`等。

完结。
