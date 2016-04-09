#### 1. getAllResponseHeaders

这一篇来讲讲如何获得从跨域的服务器端返回的响应头部信息。先用一段js获取响应的头部信息：

``` javascript
var xhttp = new XMLHttpRequest();
xhttp.onreadystatechange = function() {
    if (xhttp.readyState == 4 && xhttp.status == 200) {
        console.log(xhttp.getAllResponseHeaders());
    }
};
xhttp.open("GET", "http://localhost:8080", true);
xhttp.send();
```

效果图如下：

![](http://aliyun.rails365.net/uploads/photo/image/124/preview_2016/0f0de3ba991d1a78d1a1625ea16aaff0.png)

内容是下面这样的：

```
Last-Modified: Fri, 15 Jan 2016 06:03:50 GMT
Content-Type: text/html
```

这样是OK的，然而有时候会在服务器端自己添加响应的头部信息，比如：

``` conf
add_header 'X-Powered-By' 'rails365';
```

那又能否获得到呢，来试一下：

![](http://aliyun.rails365.net/uploads/photo/image/125/preview_2016/11d4c236a01533d618802be9914af31a.png)

没有生效，也没有报错。

#### 2. Access-Control-Expose-Headers

然而，我们需要服务器端处理一下，把`X-Powered-By`这个头部能够让客户端的js读取到。

这就是`Access-Control-Expose-Headers`这个指令所发挥的作用。

``` conf
add_header 'X-Powered-By' 'rails365';
add_header 'Access-Control-Expose-Headers' 'X-Powered-By';
```

重新发送跨域请求：

![](http://aliyun.rails365.net/uploads/photo/image/126/preview_2016/6f768c55238a8f481dd391f727bb5f70.png)

可见，成功输出了响应的头部信息`X-Powered-By`的内容了。

完结。
