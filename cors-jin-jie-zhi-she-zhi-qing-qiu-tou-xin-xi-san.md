#### 1. Access-Control-Allow-Headers

在这一篇文章[CORS进阶之Preflight请求(二)](http://www.rails365.net/articles/cors-jin-jie-zhi-preflight-qing-qiu-er)中，有说过，哪些情况下会造成Preflight请求，其中之一，就是下面的情况：

不寻常的请求头，例如不是下面的几种：

* Accept
* Accept-Language
* Content-Language

下面来用浏览器模拟一个任意的请求头。

``` javascript
var xhttp = new XMLHttpRequest();
xhttp.open("GET", "http://localhost:8080", true);
xhttp.setRequestHeader('Sample-Source','CORS in Action');
xhttp.send();
```

`setRequestHeader`是设置请求头的意思，在这里，设置了一个头信息`Sample-Source`，它的值是`CORS in Action`。

来看下请求的情况：

![](http://aliyun.rails365.net/uploads/photo/image/114/preview_2016/5405db86c78d4d975fe9950a8ecf5db8.png)

果然，报错了，大体的意思就是`Sample-Source`这个请求头不被允许。

且也产生了Preflight请求，如下：

![](http://aliyun.rails365.net/uploads/photo/image/115/preview_2016/46e77f7762c35421ad703046b6463c1b.png)

我们在服务器端设置一下，让这个请求头信息可以被接受。

``` conf
add_header 'Access-Control-Allow-Headers' 'Sample-Source';
```

再重新发起请求。

![](http://aliyun.rails365.net/uploads/photo/image/116/preview_2016/a716ad93cd7d4372146a15b37d43f28e.png)

成功了。也产生了两个请求，其中之一是Preflight请求，还有实际的GET请求。

![](http://aliyun.rails365.net/uploads/photo/image/117/preview_2016/cd7c0e93680218e28315fe03cfc1bade.png)

也返回了真正的头信息：

![](http://aliyun.rails365.net/uploads/photo/image/118/preview_2016/8b67facc02592f4ac252e4a4691129dc.png)

跟上面的例子一样，`Access-Control-Allow-Headers`也有自己的一套推荐写法，如下：

``` conf
add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Sample-Source';
```

完结。
