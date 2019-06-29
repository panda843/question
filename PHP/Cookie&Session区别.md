# 1、存储位置不同
- cookie的数据信息存放在客户端浏览器上。
- session的数据信息存放在服务器上。

# 2、存储容量不同
- 单个cookie保存的数据<=4KB，一个站点最多保存20个Cookie。
- 对于session来说并没有上限，但出于对服务器端的性能考虑，session内不要存放过多的东西，并且设置session删除机制。

# 3、存储方式不同
- cookie中只能保管ASCII字符串，并需要通过编码方式存储为Unicode字符或者二进制数据。
- session中能够存储任何类型的数据，包括且不限于string，integer，list，map等。

# 4、隐私策略不同
- cookie对客户端是可见的，别有用心的人可以分析存放在本地的cookie并进行cookie欺骗，所以它是不安全的。
- session存储在服务器上，对客户端是透明对，不存在敏感信息泄漏的风险。

# 5、有效期上不同
- 开发可以通过设置cookie的属性，达到使cookie长期有效的效果。
- session依赖于名为JSESSIONID的cookie，而cookie JSESSIONID的过期时间默认为-1，只需关闭窗口该session就会失效，因而session不能达到长期有效的效果。

# 6、服务器压力不同
- cookie保管在客户端，不占用服务器资源。对于并发用户十分多的网站，cookie是很好的选择。
- session是保管在服务器端的，每个用户都会产生一个session。假如并发访问的用户十分多，会产生十分多的session，耗费大量的内存。

# 7、浏览器支持不同
1. 假如客户端浏览器不支持cookie：
    - cookie是需要客户端浏览器支持的，假如客户端禁用了cookie，或者不支持cookie，则会话跟踪会失效。关于WAP上的应用，常规的cookie就派不上用场了。
    - 运用session需要使用URL地址重写的方式。一切用到session程序的URL都要进行URL地址重写，否则session会话跟踪还会失效。

2. 假如客户端支持cookie：
    - cookie既能够设为本浏览器窗口以及子窗口内有效，也能够设为一切窗口内有效。
    - session只能在本窗口以及子窗口内有效。

# 8、跨域支持上不同
- cookie支持跨域名访问。
- session不支持跨域名访问。

# Cookie的工作机制
1. 服务器向客户端响应请求的时候，会在响应头中设置set-cookie的值，其值的格式通常是name = value的格式
2. 浏览器将 cookie 保存下来
3. 每次请求浏览器都会自动将 cookie 发向服务器
4. cookie最初是在客户端用于存储会话信息的。

# Session的工作机制
1. 当客户端第一次请求session对象时，服务器会创建一个session，并通过特殊算法算出一个session的ID，用来标识该session对象，然后将这个session序列放置到set-cookie中发送给浏览器
2. 浏览器下次发请求的时候，这个sessionID会被放置在请求头中，和cookie一起发送回来
3. 服务器再通过内存中保存的sessionID跟cookie中保存的sessionID进行比较，并根据ID在内存中找到之前创建的session对象，提供给请求使用，也就是服务器会通过session保存一个状态记录，浏览器会通过cookie保存状态记录，服务器通过两者的对比实现跟踪状态，这样的做，也极大的避免了cookie被篡改而带来的安全性问题
4. 由于cookie可以被人为的禁止，必须有其他机制以便在cookie被禁止时仍然能够把session id传递回服务器。经常被使用的一种技术叫做URL重写，就是把session id直接附加在URL路径的后面，附加方式也有两种，一种是作为URL路径的附加信息，另一种是作为查询字符串附加在URL后面