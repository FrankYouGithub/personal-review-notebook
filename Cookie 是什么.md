## Cookie 是什么

1. Cookie 是浏览器访问服务器后，服务器传给浏览器的一段数据。Cookie 实际上是一小段的文本信息（key-value 格式）
2. 浏览器需要保存这段数据，不得轻易删除。
3. 此后每次浏览器访问该服务器，都必须带上这段数据。

## cookie 属性项

| 属性项     | 属性项介绍                                                                     |
| ---------- | ------------------------------------------------------------------------------ |
| NAME=VALUE | 键值对，可以设置要保存的 Key/Value，注意这里的 NAME 不能和其他属性项的名字一样 |
| Expires    | 过期时间，在设置的某个时间点后该 Cookie 就会失效                               |
| Domain     | 生成该 Cookie 的域名，如 domain="www.baidu.com"                                |
| Path       | 该 Cookie 是在当前的哪个路径下生成的，如 path=/wp-admin/                       |
| Secure     | 如果设置了这个属性，那么只会在 SSL 连接时才会回传该 Cookie                     |

**Expires**
该属性用来设置 Cookie 的有效期。Cookie 中的 maxAge 用来表示该属性，单位为秒。Cookie 中通过 getMaxAge()和 setMaxAge(int maxAge)来读写该属性。maxAge 有 3 种值，分别为正数，负数和 0。
如果 maxAge 属性为正数，则表示该 Cookie 会在 maxAge 秒之后自动失效。浏览器会将 maxAge 为正数的 Cookie 持久化，即写到对应的 Cookie 文件中（每个浏览器存储的位置不一致）。无论客户关闭了浏览器还是电脑，只要还在 maxAge 秒之前，登录网站时该 Cookie 仍然有效。下面代码中的 Cookie 信息将永远有效。

```
Cookie cookie = new Cookie("mcrwayfun",System.currentTimeMillis()+"");
// 设置生命周期为MAX_VALUE，永久有效
cookie.setMaxAge(Integer.MAX_VALUE);
resp.addCookie(cookie);
```

当 maxAge 属性为负数，则表示该 Cookie 只是一个临时 Cookie，不会被持久化，仅在本浏览器窗口或者本窗口打开的子窗口中有效，关闭浏览器后该 Cookie 立即失效。

```
Cookie cookie = new Cookie("mcrwayfun",System.currentTimeMillis()+"");
// MaxAge为负数，是一个临时Cookie，不会持久化
cookie.setMaxAge(-1);
resp.addCookie(cookie);
```

当 maxAge 为 0 时，表示立即删除 Cookie

那么 maxAge 设置为负值和 0 到底有什么区别呢？
maxAge 设置为 0 表示立即删除该 Cookie，如果在 debug 的模式下，执行上述方法，可以看见 cookie 立即被删除了。
maxAge 设置为负数，能看到 Expires 属性改变了，但 Cookie 仍然会存在一段时间直到关闭浏览器或者重新打开浏览器。

值得注意的是，从客户端读取 Cookie 时，包括 maxAge 在内的其他属性都是不可读的，也不会被提交。浏览器提交 Cookie 时只会提交 name 和 value 属性，maxAge 属性只被浏览器用来判断 Cookie 是否过期，而不能用服务端来判断。
我们无法在服务端通过 cookie.getMaxAge()来判断该 cookie 是否过期，maxAge 只是一个只读属性，值永远为-1。当 cookie 过期时，浏览器在与后台交互时会自动筛选过期 cookie，过期了的 cookie 就不会被携带了。

**Cookie 的域名**
Cookie 是不可以跨域名的，隐私安全机制禁止网站非法获取其他网站的 Cookie。

正常情况下，同一个一级域名下的两个二级域名也不能交互使用 Cookie，比如 test1.mcrwayfun.com 和 test2.mcrwayfun.com，因为二者的域名不完全相同。如果想要 mcrwayfun.com 名下的二级域名都可以使用该 Cookie，需要设置 Cookie 的 domain 参数为.mcrwayfun.com，这样使用 test1.mcrwayfun.com 和 test2.mcrwayfun.com 就能访问同一个 cookie

**Cookie 的路径**
path 属性决定允许访问 Cookie 的路径。比如，设置为"/"表示允许所有路径都可以使用 Cookie

## 如何使用 Cookie

Cookie 一般有两个作用。

**第一个作用是识别用户身份。**
比如用户 A 用浏览器访问了 http://a.com，那么 http://a.com 的服务器就会立刻给 A 返回一段数据「uid=1」（这就是 Cookie）。当 A 再次访问 http://a.com 的其他页面时，就会附带上「uid=1」这段数据。
同理，用户 B 用浏览器访问 http://a.com 时，http://a.com 发现 B 没有附带 uid 数据，就给 B 分配了一个新的 uid，为 2，然后返回给 B 一段数据「uid=2」。B 之后访问 http://a.com 的时候，就会一直带上「uid=2」这段数据。
借此，http://a.com 的服务器就能区分 A 和 B 两个用户了。

**第二个作用是记录历史。**
假设 http://a.com 是一个购物网站，当 A 在上面将商品 A1 、A2 加入购物车时，JS 可以改写 Cookie，改为「uid=1; cart=A1,A2」，表示购物车里有 A1 和 A2 两样商品了。
这样一来，当用户关闭网页，过三天再打开网页的时候，依然可以看到 A1、A2 躺在购物车里，因为浏览器并不会无缘无故地删除这个 Cookie。
