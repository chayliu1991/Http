# HTTPS建立连接

当你在浏览器地址栏里键入“https”开头的URI，再按下回车，会发生什么呢？

因为协议名是“https”，所以浏览器就知道了端口号是默认的443，它再用DNS解析域名，得到目标的IP地址，然后就可以使用三次握手与网站建立TCP连接了。

在HTTP协议里，建立连接后，浏览器会立即发送请求报文。但现在是HTTPS协议，它需要再用另外一个“握手”过程，在TCP上建立安全连接，之后才是收发HTTP报文。

# TLS协议的组成

TLS包含几个子协议，你也可以理解为它是由几个不同职责的模块组成，比较常用的有记录协议、警报协议、握手协议、变更密码规范协议等。

记录协议（Record Protocol）规定了TLS收发数据的基本单位：记录（record）。它有点像是TCP里的segment，所有的其他子协议都需要通过记录协议发出。但多个记录数据可以在一个TCP包里一次性发出，也并不需要像TCP那样返回ACK。

警报协议（Alert Protocol）的职责是向对方发出警报信息，有点像是HTTP协议里的状态码。比如，protocol_version就是不支持旧版本，bad_certificate就是证书有问题，收到警报后另一方可以选择继续，也可以立即终止连接。

握手协议（Handshake Protocol）是TLS里最复杂的子协议，要比TCP的SYN/ACK复杂的多，浏览器和服务器会在握手过程中协商TLS版本号、随机数、密码套件等信息，然后交换证书和密钥参数，最终双方协商得到会话密钥，用于后续的混合加密系统。

变更密码规范协议（Change Cipher Spec Protocol），它非常简单，就是一个“通知”，告诉对方，后续的数据都将使用加密保护。那么反过来，在它之前，数据都是明文的。

![](./img/tls_handshake.png)

# ECDHE握手过程

![](./img/wk.png)

![](./img/tls_connection.png)

在TCP建立连接之后，浏览器会首先发一个“Client Hello”消息，也就是跟服务器“打招呼”。里面有客户端的版本号、支持的密码套件，还有一个随机数（Client Random），用于后续生成会话密钥。

![](./img/client_hello.png)

这个的意思就是：“我这边有这些这些信息，你看看哪些是能用的，关键的随机数可得留着。”

作为“礼尚往来”，服务器收到“Client Hello”后，会返回一个“Server Hello”消息。把版本号对一下，也给出一个随机数（Server Random），然后从客户端的列表里选一个作为本次通信使用的密码套件

![](./img/server_hello.png)

这个的意思就是：“版本号对上了，可以加密，你的密码套件挺多，我选一个最合适的吧，用椭圆曲线加RSA、AES、SHA384。我也给你一个随机数，你也得留着。”然后，服务器为了证明自己的身份，就把证书也发给了客户端（Server Certificate）。

接下来是一个关键的操作，因为服务器选择了ECDHE算法，所以它会在证书后发送“Server Key Exchange”消息，里面是椭圆曲线的公钥（Server Params），用来实现密钥交换算法，再加上自己的私钥签名认证。

![](./img/server_key_exchange.png)

这相当于说：“刚才我选的密码套件有点复杂，所以再给你个算法的参数，和刚才的随机数一样有用，别丢了。为了防止别人冒充，我又盖了个章。”

之后是“Server Hello Done”消息，服务器说：“我的信息就是这些，打招呼完毕。”

![](./img/server_hello_done.png)



这样第一个消息往返就结束了（两个TCP包），结果是客户端和服务器通过明文共享了三个信息：Client Random、Server Random和Server Params。

之后开始走证书链逐级验证，确认证书的真实性，再用证书公钥验证签名，就确认了服务器的身份：“刚才跟我打招呼的不是骗子，可以接着往下走。”

然后，客户端按照密码套件的要求，也生成一个椭圆曲线的公钥（Client Params），用“Client Key Exchange”消息发给服务器。

![](./img/client_key_exchange.png)

现在客户端和服务器手里都拿到了密钥交换算法的两个参数（Client Params、Server Params），就用ECDHE算法一阵算，算出了一个新的东西，叫“Pre-Master”，其实也是一个随机数。算法可以保证即使黑客截获了之前的参数，也是绝对算不出这个随机数的。

现在客户端和服务器手里有了三个随机数：Client Random、Server Random和Pre-Master。用这三个作为原始材料，就可以生成用于加密会话的主密钥，叫“Master Secret”。而黑客因为拿不到“Pre-Master”，所以也就得不到主密钥。

































