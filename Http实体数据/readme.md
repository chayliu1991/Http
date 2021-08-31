# 数据类型与编码

MIME(Multipurpose Internet Mail Extensions)，多用途互联网邮件扩展，MIME 是一个很大的标准规范，但 HTTP 只取了其中的一部分，用来标记 body 的数据类型，这就是 “MIME type”。MIME 把数据分成了八大类，每个大类下再细分出多个子类，形式是 “type/subtype” 的字符串，巧得很，刚好也符合了 HTTP 明文的特点，所以能够很容易地纳入 HTTP 头字段里。

HTTP 里经常遇到的几个类别：

- **text**：即文本格式的可读数据，常见的有 text/html(超文本文档)，text/plain(纯文本)，text/css(样式表)
- **image**：即图像文件，常见的有 image/gif、image/jpeg、image/png 等
- **audio/video**：音频和视频数据，常见的有 audio/mpeg、video/mp4 等
- **application**：数据格式不固定，可能是文本也可能是二进制，必须由上层应用程序来解释。常见的有：application/json，application/javascript、application/pdf 等，另外，如果实在是不知道数据是什么类型，就会是 application/octet-stream，即不透明的二进制数据

MIME type 还不够，因为 HTTP 在传输时为了节约带宽，有时候还会压缩数据，还需要有一个 “Encoding type”，告诉数据是用的什么编码格式，这样对方才能正确解压缩，常用的有：

- **gzip**：GNU zip 压缩格式，也是互联网上最流行的压缩格式
- **deflate**：zlib（deflate）压缩格式，流行程度仅次于gzip
- **br**：一种专门为 HTTP 优化的新压缩算法（Brotli）

## 数据类型使用的头字段

有了 MIME type 和 Encoding type，无论是浏览器还是服务器就都可以轻松识别出body的类型，也就能够正确处理数据了。

HTTP 协议为此定义了两个 Accept 请求头字段和两个 Content 实体头字段，用于客户端和服务器进行“内容协商”

- 客户端用 Accept 头告诉服务器希望接收什么样的数据
- 服务器用 Content 头告诉客户端实际发送了什么样的数据

![](./img/accept_content.png)

