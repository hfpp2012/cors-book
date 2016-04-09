#### 1. Set-Cookie

在这一篇文章[CORS进阶之设置请求头信息(三)](http://www.rails365.net/articles/cors-jin-jie-zhi-she-zhi-qing-qiu-tou-xin-xi-san)中，可以使用CORS来设置自定义的请求头部信息，然而为了安全，默认情况下，浏览器的cookie也是不能作为信息传递给跨域的服务器的。

第一步，要做的是首先浏览器上有cookie。

第一步，是要测试把cookie发送给跨域的服务器。

先来完成第一步，设置cookies。

其实最简单的是通过javascript来设置，不过这里介绍另一种方法：通过响应头信息`Set-Cookie`来处理。

``` conf
add_header 'Set-Cookie' 'name=value';
```

上面表示的是响应一个头部信息叫`Set-Cookie`，浏览器得到它的值就会自动设置cookie信息的，键为`name`，值为`value`。

还是按照之前的例子，从localhost:3000跨域到nginx服务器localhost:8080。

先访问一下localhost:8080这台服务器。

![](http://aliyun.rails365.net/uploads/photo/image/119/preview_2016/ac85fc476b8d1c89bfef70140ef2f60b.png)

现在在localhost这个域上就有`name=value`这样的cookie信息了。

接着在localhost:3000上把这个cookie信息跨域发送给localhost:8080这台nginx服务器。

#### 2. Access-Control-Allow-Credentials

要发送带cookie的请求到跨域服务器中，只需要把`withCredentials`设为`true`即可。

``` javascript
var xhttp = new XMLHttpRequest();
xhttp.open("GET", "http://localhost:8080", true);
xhttp.withCredentials = true
xhttp.send();
```

效果图如下：

![](http://aliyun.rails365.net/uploads/photo/image/120/preview_2016/015f76c11d035a0ea1503bce34d47b6f.png)

大体的意思是这样，当`Access-Control-Allow-Origin`为`*`时，又把`withCredentials`设为了`true`，那是不被允许的。

我们先把`Access-Control-Allow-Origin`改为localhost:3000这台机器。

``` conf
add_header 'Access-Control-Allow-Origin' 'http://localhost:3000';
```

再重新发送跨域请求。

![](http://aliyun.rails365.net/uploads/photo/image/121/preview_2016/e1ceea6bc32920732e9b9de73e1173ab.png)

之前`Access-Control-Allow-Origin`为`*`的问题解决了，可是又出现了新的错误：在服务器端`Access-Control-Allow-Credentials`应该被设为`true`。

``` conf
add_header 'Access-Control-Allow-Origin' 'http://localhost:3000';
add_header 'Access-Control-Allow-Credentials' 'true';
```

再重新发起新的域跨请求。

![](http://aliyun.rails365.net/uploads/photo/image/122/preview_2016/c8fa40febbdc4d8b400092c97d64942e.png)

可见，请求是成功的。

但是，服务器能得到那些cookie信息的内容吗？

#### 3. echo指令

在nginx中还是可以轻易获取到cookie的内容的，那就是通过nginx的变量来获得。

`$cookie_XXX`这样的变量就是获取cookie内容，比如`name=value`这样的cookie信息可以通过`$cookie_name`变量获得。

为了验证要在nginx中把cookie的内容打印出来，我们在这里要利用一个模块：[echo-nginx-module](https://github.com/openresty/echo-nginx-module)。

它提供了`echo`指令可以输出变量的内容。

要编译这个模块，可以参照我之前的一篇文章：[nginx之编译第三方模块(六)](http://www.rails365.net/articles/nginx-zhi-bian-yi-di-san-fang-mo-kuai-liu)。

``` conf
add_header 'Access-Control-Allow-Origin' 'http://localhost:3000';
add_header 'Access-Control-Allow-Credentials' 'true';
echo "name = $cookie_name";
```

现在重新发起跨域请求，来看下响应的信息。

![](http://aliyun.rails365.net/uploads/photo/image/123/preview_2016/e15ec209c2e4649ef5ac567e813f2ab4.png)

果然，服务器上是能获得cookie的信息的。

完结。

