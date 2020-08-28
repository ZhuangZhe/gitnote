# 域名系统

解析大致过程如图

![](../.gitbook/assets/image%20%2887%29.png)

当一个用户在地址栏输入www.taobao.com时，DNS解析有大致十个过程：

1. 浏览器先检查自身缓存中有没有被解析过的这个域名对应的ip地址，如果有，解析结束。同时域名被缓存的时间也可通过TTL属性来设置。
2. 如果浏览器缓存中没有\(专业点叫还没命中\)，浏览器会检查操作系统缓存中有没有对应的已解析过的结果。而操作系统也有一个域名解析的过程。在windows中可通过c盘里一个叫hosts的文件来设置，如果你在这里指定了一个域名对应的ip地址，那浏览器会首先使用这个ip地址。但是这种操作系统级别的域名解析规程也被很多黑客利用，通过修改你的hosts文件里的内容把特定的域名解析到他指定的ip地址上，造成所谓的域名劫持。所以在windows7中将hosts文件设置成了readonly，防止被恶意篡改。
3. 如果至此还没有命中域名，才会真正的请求本地域名服务器（LDNS）来解析这个域名，这台服务器一般在你的城市的某个角落，距离你不会很远，并且这台服务器的性能都很好，一般都会缓存域名解析结果，大约80%的域名解析到这里就完成了。
4. 如果LDNS仍然没有命中，就直接跳到Root Server域名服务器请求解析。
5. 根域名服务器返回给LDNS一个所查询域的主域名服务器\(gTLD Server，国际顶尖域名服务器，如.com .cn .org等\)地址。
6. 此时LDNS再发送请求给上一步返回的gTLD。
7. 接受请求的gTLD查找并返回这个域名对应的Name Server的地址，这个Name Server就是网站注册的域名服务器。
8. Name Server根据映射关系表找到目标ip，返回给LDNS。
9. LDNS缓存这个域名和对应的ip。
10. LDNS把解析的结果返回给用户，用户根据TTL值缓存到本地系统缓存中，域名解析过程至此结束。

域名系统是分层次的，一些DNS服务器位于顶层。当查询\(域名\)IP 时，路由或ISP提供连接DNS服务器的信息。较底层的DNS服务器缓存映射，它可能会因为DNS传播延时而失效。DNS结果可以缓存在浏览器或操作系统中一段时间，时间长短取决于[存活时间 TTL](https://en.wikipedia.org/wiki/Time_to_live)。

* **NS记录\(域名服务\)：**指定解析域名或子域名的 DNS 服务器。
* **MX记录\(邮件交换\)：**指定接收信息的邮件服务器。
* **A记录\(地址\)：**指定域名对应的 IP 地址记录。
* **CNAME\(规范\)：**一个域名映射到另一个域名或 `CNAME` 记录（ example.com 指向 [www.example.com](http://www.example.com/) ）或映射到一个 `A` 记录。

[CloudFlare](https://www.cloudflare.com/dns/) 和 [Route 53](https://aws.amazon.com/route53/) 等平台提供管理 DNS 的功能。某些 DNS 服务通过集中方式来路由流量:

* [加权轮询调度](http://g33kinfo.com/info/archives/2657)
  * 防止流量进入维护中的服务器
  * 在不同大小集群间负载均衡
  * A/B 测试
* 基于延迟路由
* 基于地理位置路由

#### 缺陷:

* 虽说缓存可以减轻DNS延迟，但连接DNS服务器还是带来了轻微的延迟。
* 虽然它们通常由[政府，网络服务提供商和大公司](http://superuser.com/questions/472695/who-controls-the-dns-servers/472729)管理，但DNS服务管理仍可能是复杂的。
* DNS服务最近遭受[DDoS攻击](http://dyn.com/blog/dyn-analysis-summary-of-friday-october-21-attack/)，阻止不知道Twitter IP地址的用户访问Twitter。





**参考资料：**

* \*\*\*\*[**详解DNS域名解析全过程**](https://blog.csdn.net/m0_37812513/article/details/78775629)\*\*\*\*

