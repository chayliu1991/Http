# HTTP 报文首部  

HTTP 报文的结构：

![](./img/http_message.png)

HTTP 协议的请求和响应报文中必定包含 HTTP 首部。  

在请求中， HTTP 报文由方法、 URI、 HTTP 版本、 HTTP 首部字段等部分构成：

![](./img/http_request_message.png)

在响应中， HTTP 报文由 HTTP 版本、状态码（数字和原因短语）、HTTP 首部字段 3 部分构成：

![](./img/http_response_message.png)

# HTTP 首部字段  

在客户端与服务器之间以 HTTP 协议进行通信的过程中，无论是请求还是响应都会使用首部字段，它能起到传递额外重要信息的作用。  

HTTP 首部字段是由首部字段名和字段值构成的，中间用冒号“:”分隔：

```
首部字段名: 字段值
```

字段值对应单个 HTTP 首部字段可以有多个值。

## 4 种 HTTP 首部字段类型  

HTTP 首部字段根据实际用途被分为以下 4 种类型：

- **通用首部字段（General Header Fields）**，请求报文和响应报文两方都会使用的首部
- **请求首部字段（Request Header Fields）**，从客户端向服务器端发送请求报文时使用的首部。补充了请求的附加内容、客户端信息、响应内容相关优先级等信息
- **响应首部字段（Response Header Fields）**，从服务器端向客户端返回响应报文时使用的首部。补充了响应的附加内容，也会要求客户端附加额外的内容信息
- **实体首部字段（Entity Header Fields）**，针对请求报文和响应报文的实体部分使用的首部。补充了资源内容更新时间等与实体有关的信息

## HTTP/1.1 首部字段一览  

### 通用首部字段一览

![](./img/general_header.png)

### 请求首部字段一览

![](./img/request_header.png)

### 响应首部字段一览

![](./img/response_header.png)

### 实体首部字段一览

![](./img/entity_header.png)

## 非 HTTP/1.1 首部字段  

首部字段，不限于 RFC2616 中定义的 47 种首部字段。还有 Cookie、 Set-Cookie 和 Content-Disposition等在其他 RFC 中定义的首部字段，它们的使用频率也很高。这些非正式的首部字段统一归纳在 RFC4229 HTTP Header Field Registrations 中。  

 ## End-to-end 首部和 Hop-by-hop 首部  

HTTP 首部字段将定义成缓存代理和非缓存代理的行为，分成 2 种类型：

- **端到端首部（End-to-endHeader）**，分在此类别中的首部会转发给请求 / 响应对应的最终接收目标，且必须保存在由缓存生成的响应中，另外规定它必须被转发
- **逐跳首部（Hop-by-hopHeader）**，分在此类别中的首部只对单次转发有效，会因通过缓存或代理而不再转发。 HTTP/1.1 和之后版本中，如果要使用 hop-by-hop 首部，需提供 Connection 首部字段

HTTP/1.1 中，除下面 8 个首部字段之外，其他所有字段都属于端到端首部：

- Connection
- Keep-Alive
- Proxy-Authenticate
- Proxy-Authorization
- Trailer
- TE
- Transfer-Encoding
- Upgrade

# HTTP/1.1 通用首部字段  

通用首部字段是指，请求报文和响应报文双方都会使用的首部。  

## Cache-Control  

通过指定首部字段 Cache-Control 的指令，就能操作缓存的工作机制。  

![](./img/cache_conntrol.png)

指令的参数是可选的，多个指令之间通过“,”分隔。首部字段 Cache-Control 的指令可用于请求及响应时：

```
Cache-Control: private, max-age=0, no-cache
```

### Cache-Control 指令一览  

**缓存请求指令:**

![](./img/cache_request_cmd.png)

**缓存响应指令:**

![](./img/cache_response_cmd.png)

### 表示是否能缓存的指令  

#### public 指令

当指定使用 public 指令时，则明确表明其他用户也可利用缓存：

```
Cache-Control: public
```

#### private 指令

当指定 private 指令后，响应只以特定的用户作为对象，这与 public 指令的行为相反  ：

```
Cache-Control: private
```

缓存服务器会对该特定用户提供资源缓存的服务， 对于其他用户发送过来的请求，代理服务器则不会返回缓存。  

#### no-cache 指令  

```
Cache-Control: no-cache
```

使用 no-cache 指令的目的是为了防止从缓存中返回过期的资源。  

![](./img/no_cache.png)

- 客户端发送的请求中如果包含 no-cache 指令，则表示客户端将不会接收缓存过的响应。于是，“中间”的缓存服务器必须把客户端请求转发给源服务器
- 如果服务器返回的响应中包含 no-cache 指令，那么缓存服务器不能对资源进行缓存。源服务器以后也将不再对缓存服务器请求中提出的资源有效性进行确认，且禁止其对响应资源进行缓存操作

```
Cache-Control: no-cache=Location
```

由服务器返回的响应中，若报文首部字段 Cache-Control 中对 nocache 字段名具体指定参数值， 那么客户端在接收到这个被指定参数值的首部字段对应的响应报文后， 就不能使用缓存：

- 换言之，无参数值的首部字段可以使用缓存
- 只能在响应指令中指定该参数

### 控制可执行缓存的对象的指令  

#### no-store 指令  

```
Cache-Control: no-store
```

当使用 no-store 指令 A 时，暗示请求（和对应的响应）或响应中包含机密信息。  因此，该指令规定缓存不能在本地存储请求或响应的任一部分。  

no-cache 代表不缓存过期的资源，缓存会向源服务器进行有效期确认后处理资源， 也许称为 do-not-serve-from-cache-without-revalidation更合适。

no-store才是真正地不进行缓存。

### 指定缓存期限和认证的指令  

#### s-maxage 指令  

```
Cache-Control: s-maxage=604800（单位 ：秒）
```

s-maxage 指令的功能和 max-age 指令的相同，它们的不同点是 s-maxage 指令只适用于供多位用户使用的公共缓存服务器 A。也就是说，对于向同一用户重复返回响应的服务器来说，这个指令没有任何作用。

另外，当使用 s-maxage 指令后，则直接忽略对 Expires 首部字段及 max-age 指令的处理。  

#### max-age 指令  

````
Cache-Control: max-age=604800（单位 ：秒）
````

![](./img/max_age.png)

当客户端发送的请求中包含 max-age 指令时，如果判定缓存资源的缓存时间数值比指定时间的数值更小，那么客户端就接收缓存的资源。另外，当指定 max-age 值为 0，那么缓存服务器通常需要将请求转发给源服务器。  

当服务器返回的响应中包含 max-age 指令时，缓存服务器将不对资源的有效性再作确认，而 max-age 数值代表资源保存为缓存的最长时间。

应用 HTTP/1.1 版本的缓存服务器遇到同时存在 Expires 首部字段的情况时，会优先处理 max-age 指令，而忽略掉 Expires 首部字段。而 HTTP/1.0 版本的缓存服务器的情况却相反， max-age 指令会被忽略掉。  

#### min-fresh 指令  

```
Cache-Control: min-fresh=60（单位 ：秒）
```

![](./img/min_fresh.png)

min-fresh 指令要求缓存服务器返回至少还未过指定时间的缓存资源。比如，当指定 min-fresh 为 60 秒后，过了 60 秒的资源都无法作为响应返回了。  

#### max-stale 指令  

```
Cache-Control: max-stale=3600（单位 ：秒）
```

使用 max-stale 可指示缓存资源，即使过期也照常接收。

如果指令未指定参数值，那么无论经过多久，客户端都会接收响应；如果指令中指定了具体数值，那么即使过期，只要仍处于 max-stale 指定的时间内，仍旧会被客户端接收。  

#### only-if-cached 指令  

```
Cache-Control: only-if-cached
```

使用 only-if-cached 指令表示客户端仅在缓存服务器本地缓存目标资源的情况下才会要求其返回。 换言之，该指令要求缓存服务器不重新加载响应，也不会再次确认资源有效性。若发生请求缓存服务器的本地缓存无响应，则返回状态码 504 Gateway Timeout。  

#### must-revalidate 指令  

```
Cache-Control: must-revalidate
```

使用 must-revalidate 指令，代理会向源服务器再次验证即将返回的响应缓存目前是否仍然有效。

若代理无法连通源服务器再次获取有效资源的话， 缓存必须给客户端一条 504（ Gateway Timeout）状态码。

另外，使用 must-revalidate 指令会忽略请求的 max-stale 指令（即使已经在首部使用了 max-stale，也不会再有效果）。  

#### proxy-revalidate 指令  

```
Cache-Control: proxy-revalidate
```

proxy-revalidate 指令要求所有的缓存服务器在接收到客户端带有该指令的请求返回响应之前，必须再次验证缓存的有效性。  

#### no-transform 指令  

```
Cache-Control: no-transform
```

使用 no-transform 指令规定无论是在请求还是响应中，缓存都不能改变实体主体的媒体类型。这样做可防止缓存或代理压缩图片等类似操作。  

### Cache-Control  扩展  

#### cache-extension token  

```
Cache-Control: private, community="UCI"
```

通过 cache-extension 标记（ token），可以扩展 Cache-Control 首部字段内的指令。

如上例， Cache-Control 首部字段本身没有 community 这个指令。借助 extension tokens 实现了该指令的添加。如果缓存服务器不能理解 community 这个新指令， 就会直接忽略。因此， extension tokens 仅对能理解它的缓存服务器来说是有意义的。  





