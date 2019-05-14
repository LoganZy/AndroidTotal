---


---

<h1 id="android学习-之-http-client与httpurlconnection的区别">Android学习 之 Http Client与HttpURLConnection的区别</h1>
<p>文章转载自<a href="https://www.jianshu.com/p/a32d6980227b">https://www.jianshu.com/p/a32d6980227b  </a></p>
<p>很多Android初学者在接触到Http协议的时候，估计都会在选择HttpClient还是HttpURLConnection之间，这两个都是Android中包含的Http客户端类。那么，这两个有什么区别呢？<br>
　　咳，其实在之前对我而言并没有区别，因为我平时都是用的开源框架比如Async-http-client。<br>
　　虽然Android 6.0中已经抛弃了HttpClient转而使用OkHttp，不过既然很多面试都喜欢问这个问题，那么还是了解一下吧。</p>
<hr>
<h2 id="补充">补充</h2>
<h4 id="tcpip、socket、http简要介绍">TCP/IP、Socket、Http简要介绍</h4>
<p>1、TCP/IP中文名为传输控制协议/因特网互联协议，又名网络通讯协议，是Internet最基本的协议、Internet国际互联网络的基础，由网络层的IP协议和传输层的TCP协议组成。</p>
<p>2、Socket是支持TCP/IP协议的网络通信基本操作单元，许多操作系统为应用程序提供了一套调用接口（API)，方便开发者开发网络程序。注意，socket本身并不是协议，只是提供一个针对TCP或UDP的编程接口。</p>
<p>3、HTTP协议是一个web服务器和客户端通信的超文本传送协议，是建立在TCP协议上的一个应用层协议。HTTP连接最显著的特点是客户端发送的每次请求都需要服务器回送响应，在请求结束后，会主动释放连接。从建立连接到关闭连接的过程称为“一次连接”。</p>
<h4 id="http-1.0与1.1的区别">Http 1.0与1.1的区别</h4>
<p>最主要的区别就是连接时，在1.0中只能1个单独的连接只能处理1次请求，1.1中可以执行多个请求，并且多个请求可以叠加而不用等待一个请求结束再发送下一个请求。</p>
<hr>
<h2 id="背景">背景</h2>
<h4 id="http-client">Http Client</h4>
<p>Apache公司提供的库，提供高效的、最新的、功能丰富的支持HTTP协议工具包，支持HTTP协议最新的版本和建议，是个很不错的开源框架，封装了Http的请求，参数，内容体，响应等，拥有众多API。</p>
<h4 id="httpurlconnection">HttpURLConnection</h4>
<p>Sun公司提供的库，也是Java的标准类库java.net中的一员，但这个类什么都没封装，用起来很原始，若需要高级功能，则会显得不太方便，比如重访问的自定义，会话和cookie等一些高级功能。</p>
<hr>
<h2 id="区别">区别</h2>
<h4 id="功能上">功能上</h4>
<p>Http Client适用于web浏览器，拥有大量灵活的API，实现起来比较稳定，且其功能比较丰富，提供了很多工具，封装了http的请求头，参数，内容体，响应，还有一些高级功能，代理、COOKIE、鉴权、压缩、连接池的处理。<br>
　　但是，正因此，在不破坏兼容性的前提下，其庞大的API也使人难以改进，因此Android团队对于修改优化Apache Http Client并不积极。(并在Android 6.0中抛弃了Http Client，替换成OkHttp)</p>
<p>HttpURLConnection对于大部分功能都进行了包装，Http Client的高级功能代码会较复杂，另外，HttpURLConnection在Android 2.3中增加了一些Https方面的改进(包括Http Client，两者都支持https)。且在Android 4.0中增加了response cache。</p>
<h4 id="性能上">性能上</h4>
<p>HttpURLConnection直接支持<a href="https://link.jianshu.com?t=http://www.cnblogs.com/TankXiao/archive/2012/11/13/2749055.html">GZIP压缩</a>，默认添加"Accept-Encoding: gzip"头字段到请求中，并处理相应的回应，而Http Client虽然支持，但需要自己写代码处理。<br>
　　注意：但在2.3中，由于Http的Content-Length头字段返回的是压缩后的大小，直接使用getContentLength()方法得到的数据大小是错误的，应该使用InputStream.read()直到返回值是-1为止。</p>
<p>HttpURLConnection直接在系统层面做了缓存策略处理(Android 4.0以上)，Http请求将在以下三种中选择：<br>
　　1、完全的cache的response将直接从本地存储中获取。因为不需要网络连接，此类response可以立即得到。<br>
　　2、有条件cache的response必须在Web服务器验证一下cache的有效性。客户端发送一个请求，比如“如果/foo.png从昨天起有变化则给我新的图片” , 服务端的response要么是更新后的内容，要么是304 没有修改状态码。如果内容是没有改变的，就不需要下载了。<br>
　　3、没有cache的response将从服务器上获取。得到这些response之后会存储到cache以便将来使用。</p>
<p><a href="https://link.jianshu.com?t=http://wj98127.iteye.com/blog/617014">这篇文章</a>中对两者的速度做了一个对比，做法是两个类都使用默认的方法去请求百度的网页内容，测试结果是使用httpurlconnection耗时47ms，使用httpclient耗时641ms。httpURLConnection在速度有比较明显的优势，当然这跟压缩内容和缓存都有直接关系。</p>
<hr>
<h2 id="选用">选用</h2>
<p>HttpURLConnect是一个通用的、适合大多数应用的轻量级组件。这个类起步比较晚，很容易在主要API上做稳步的改善。但是HttpURLConnection在在Android 2.2及以下版本上存在一些令人厌烦的bug，尤其是在读取 InputStream时调用 close()方法，就有可能会导致连接池失效了。</p>
<p>因此，在2.2之前的版本，Http Client会是一个比较好的选择，而自2.3起，HttpURLConnection将是最佳选择，其API简单，小巧，非常适合于Android。透明的压缩及response cache减少了网络流量，改进了网络速度，也就更省电。</p>
<p>在Volley框架的源码中，会发现它在Http请求的使用上也是如此，在Android 2.3及以上的版本，使用的是HttpURLConnection，而在Android 2.2及以下版本，使用的是HttpClient。</p>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

