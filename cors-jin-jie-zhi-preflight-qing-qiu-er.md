#### 1. Access-Control-Allow-Origin

上一篇[从跨域到CORS(一)](http://www.rails365.net/articles/cong-kua-yu-dao-cors-yi)文章有说过CORS的基本使用，也实现了跨域的请求。本篇来讲讲CORS更高阶的用法。

首先来讲讲`Access-Control-Allow-Origin`的用法。

`Access-Control-Allow-Origin:*`表示的是允许任何的来源都可以访问，然而这并不符合大多数人的需求。假如我们只需要固定的一台或几台机器，或者是某个域名下的才可以访问，这又该如何呢？

还是按照先前的例子，从localhost:3000的服务跨域到nginx服务localhost:8080。现在来演示一下，只允许localhost:3000的访问，在nginx是这样配置的。

``` conf
location / {
  add_header 'Access-Control-Allow-Origin' 'http://localhost:3000';
}
```

![](http://aliyun.rails365.net/uploads/photo/image/104/preview_2016/43f90c5a06537be4efb8c794467564e9.png)

可见，是成功的。

也可以试一下把`http://localhost:3000`前面的`http://`去掉，会发现是不成功的，毕竟不同的协议也是不同的域的。

![](http://aliyun.rails365.net/uploads/photo/image/105/preview_2016/75ad5c8860d959a003e9dfa872a39bfd.png)

下面有个配置可以参考，在nginx中设置域名的访问。

``` conf
set $cors '';
if ($http_origin ~* 'https?://(localhost|www\.yourdomain\.com|www\.yourotherdomain\.com)') {
        set $cors 'true';
}

if ($cors = 'true') {
        add_header 'Access-Control-Allow-Origin' "$http_origin";
}
```

在上面的配置中，`$http_origin`表示的是获得来源域的域名，也就是请求头`Origin`的内容。

#### 2. Preflight请求

之前在浏览器模拟跨域请求的js是这样的：

```
var xhttp = new XMLHttpRequest();
xhttp.open("GET", "http://localhost:8080", true);
xhttp.send();
```

使用的方法是GET，CORS跟jsonp等方法不一样的，它可以支持像POST，DELETE方法。

先用POST方法来试一下，nginx那边需要改一下：

``` conf
server {
    error_page  405     =200 $uri;
}
```

再把js中的`GET`改成`POST`方法。

![](http://aliyun.rails365.net/uploads/photo/image/106/preview_2016/1da7ed01dd8a0f5fe79286ee10cde261.png)

可见，是成功的。如果改成DELETE方法呢？

![](http://aliyun.rails365.net/uploads/photo/image/107/preview_2016/f42ae930571be6286c2b0b0419fb622b.png)

报错了，大体上说DELETE方法是不被允许的。来看下产生的请求。

![](http://aliyun.rails365.net/uploads/photo/image/108/preview_2016/d308c2aef119f37cd83f6808fd8bbd50.png)

使用DELETE方法时，并不会真正的产生DELETE请求，而是先生成了一个叫OPTIONS的请求。

这个OPTIONS请求是怎么回事呢？

这个OPTIONS请求也叫Preflight请求，它是在发起DELETE真实请求时，先询问服务器是否支持DELETE方法，再用响应头信息的方式返回给浏览器，浏览器根据响应信息查看服务器是否支持DELETE方法，如果支持的话，就再发起真实的DELETE方法，不支持就报错了。所以出现了上面的报错信息。

我们先让服务器支持DELETE方法，再来尝试新的请求。

``` conf
location / {
  add_header 'Access-Control-Allow-Origin' 'http://localhost:3000';
  add_header 'Access-Control-Allow-Methods' 'DELETE';
}
```
![](http://aliyun.rails365.net/uploads/photo/image/112/preview_2016/99a728ec9a86a5bcc77e8bac688a8598.png)

果然，成功了。且发出了真正的DELETE请求。

![](http://aliyun.rails365.net/uploads/photo/image/113/preview_2016/c1eef7913693d7a990571e461b870a2e.png)

从前面的分析可以知道，`GET`，`POST`是不会发出Preflight请求的，而DELETE方法会，那还有什么情况下会呢？下面列举出来：

* 使用的方法不是GET, POST, 或 HEAD的任何一种
* Content-Type请求来不是下面任何一种
  * application/x-www-form-urlencoded
  * multipart/form-data
  * text/plain
* 不寻常的请求头，例如不是下面的几种：
  * Accept
  * Accept-Language
  * Content-Language
* XMLHttpRequest的upload事件

值得注意的是，为了避免困惑，在`Access-Control-Allow-Methods`都会显式地写上`GET`，`POST`方法，即使它是默认就支持的。

``` conf
add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, DELETE'
```

除此之外，频繁的Preflight请求可能会造成性能的消耗。我们可以用下面的方法处理一下：

``` conf
add_header 'Access-Control-Max-Age' 1728000;
add_header 'Content-Type' 'text/plain charset=UTF-8';
add_header 'Content-Length' 0;
return 204;
```

`Access-Control-Max-Age`可以设置请求的过期时间，单位是秒(second)。另外，返回204的状态码，是返回一个空内容的返回，毕竟，Preflight请求只需要响应的头部信息(Access-Control-Allow-Methods)，并不需要响应内容的。

下一篇: [CORS进阶之设置请求头信息(三)](http://www.rails365.net/articles/cors-jin-jie-zhi-she-zhi-qing-qiu-tou-xin-xi-san)

完结。
