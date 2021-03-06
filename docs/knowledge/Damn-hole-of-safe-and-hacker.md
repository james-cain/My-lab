# 网络安全

## 信息探测

### Google Hack

搜集Web信息

Google常用语法

| 关键字   | 说明                   |
| -------- | ---------------------- |
| Site     | 指定域名               |
| intext   | 正文中存在关键字的网页 |
| intitle  | 标题中存在关键字的网页 |
| info     | 一些基本信息           |
| inurl    | URL存在关键字的网页    |
| filetype | 搜索指定文件类型       |

## 同源策略（Same Origin Policy，SOP）

同源策略是Web应用程序的一种安全模型，控制了网页中DOM之间的访问。SOP影响范围包括：普通的HTTP请求、XMLHttpRequest、XSLT、XBL

### 如何判断同源？

给定一个页面，如果另外一个页面使用的协议、端口、主机名都相同，则认为两个页面具有相同的源。例如：

目标页面http://sub.eth.space/level/flower.html

| 是否同源 | URI                                                   |
| -------- | ----------------------------------------------------- |
| 同源     | http://sub.eth.space/level2/fruit.html                |
| 同源     | http://sub.eth.space:80/level/anotherlevel/fruit.html |
| 不同源   | https://sub.eth.space/level/flower.html               |
| 不同源   | http://sub.eth.space:81/level/flower.html             |
| 不同源   | http://mania.eth.space/level3/flower.html             |
| **同源** | http://red.sub.eth.space/level3/flower.html           |

**同源策略没有禁止脚本的执行，而是禁止读取HTTP回复。**SOP其实在防止CSRF上作用非常有限，CSRF的请求往往在发送出去的瞬间就已经达到攻击的目的，比如发送一段敏感数据，或请求一个具体的功能。一般静态资源通常不受同源策略限制，如js/css/jpg/png等

## SQL注入

常见的SQL注入类型包括：**数字型**和**字符型**

但不管注入类型如何，攻击者的目的只有一点，那就是绕过程序限制，使用户输入的数据带入数据库执行，利用数据库的特殊性获取更多的信息或者更大的权限。

### 数字型注入

当输入的参数为整形时，如果存在注入漏洞，即为数字型注入。数字型注入最多出现在PHP等弱类型语言中，弱类型语言会自动推导变量类型。对于Java等强类型语言，如果试图把一个字符串转换为int类型，则会抛出异常，语法继续执行。

### 字符型注入

当输入的参数为字符串时，称为字符型。字符串类型一般要使用单引号来闭合。e.g.

```sql
select * from table where username='admin'

/* 攻击者可以利用字符串闭合来发起注入 
	输入admin' and 1=1 -- */
select * from table where username = 'admin' and 1=1 --
```

一些常见的注入叫法

- POST注入：注入字段在POST数据中
- Cookie注入：注入字段在Cookie数据中
- 延时注入：使用数据库延时特性注入
- 搜索注入：注入处为搜索的地点
- base64注入：注入字符串需要经过base64加密

### 防止SQL注入

#### 严格的数据类型

Java等强类型语言几乎可以完全忽略数字型注入。即使攻击者想在代码中注入，也是不可能的，因为程序在接收参数后，做了一次数据类型的转换。如果接收的不是字符串，就会在转换的时候抛出异常。

而针对弱类型语言，其实防范也不难。只需要在程序中严格判断数据类型即可。如就用is_numeric()等函数判断数据类型，即可解决数字型注入。

#### 特殊字符转义

通过加强数据类型验证可以解决数字型的SQL注入，字符型的却不可以，最好的办法是**对特殊字符进行转义**。

防止SQL注入应该在程序中判断字符串是否存在敏感字符，如果存在，则根据相应的数据库进行转义。如MySQL使用"\\"转义。如果不知道需要转义哪些特殊字符，可以参考OWASP ESAPI，提供了专门对数据库字符转码的接口，还对不同的数据库实现了不同的编码器。

#### 使用预编译语句

Java等语言都提供了预编译语句，PreparedStatement表示预编译SQL语句的对象。可以有效的防御SQL注入。但是必须使用它提供的setter方法。

## 上传漏洞

上传漏洞比SQL注入的风险更大，如果Web应用程序存在上传漏洞，攻击者甚至可以直接上传一个WebShell到服务器上。

文件上传的基本流程是相同的，客户端使用JavaScript验证，服务器端采用随机数来重命名文件，以防止文件重复。

在防止上传漏洞时可以分为以下两种。

- 客户端检测：客户端使用JavaScript检测，在文件未上传时，就对文件进行验证
- 服务器端检测：服务器脚本一般会检测文件的MIME烈性，检测文件扩展名是否合法，甚至检测文件中是否嵌入恶意代码

第一种方式，客户端检测对一些普通用户防止上传错误还可以，但对于专业的技术人员来说，这是非常低级的验证。有多种方式可以绕过前端的检测，如

- 使用FireBug绕过。当单击"提交"按钮时，Form表单会触发onsubmit事件，事件中调用了检测文件是否合法的代码，并返回布尔值。如果为true，表达提交，反之提示失败。可以使用FireBug将onsubmit事件删除，就可以绕过了验证
- 中间人攻击。首先把木马文件扩展名改为一张正常图片的扩展名，比如JPG扩展名，在上传时使用Burp Suite拦截上传数据，再将其中的扩展名JPG修改为PHP，就可以绕过客户端验证。但这里如果修改的文件名的长度，需要将请求头中的Content-Length的长度也修改，否则上传可能会失败

因此，实际上，客户端验证是防止用户输入错误，减少服务器开销，但是预防攻击者还是必须用服务器端验证。

服务器端检测主要包含以下几点：

- 白名单与黑名单扩展名过滤
- 文件类型检测
- 文件重命名

### 白名单与黑名单验证

黑名单过滤是一种不安全的方式，服务器在接收文件后，与黑名单扩展名对比，如果发现文件扩展名与黑名单里的扩展名匹配，则认为文件不合法。攻击者可以使用很多方法绕过黑名单检测：

- 攻击者可以从黑名单中找到Web开发人员忽略的扩展名
- 在Windows系统下，如果文件名以"."或空格作为结尾，系统会自动去除"."或空格，利用此特性可以绕过黑名单验证

白名单是定义允许上传的扩展名，在获取到文件扩展名后判断命中，才认为合法允许上传。

### 修复上传漏洞

上传漏洞最终的形成原因主要有以下两点：

- 目录过滤不严，攻击者可能简历畸形目录
- 文件未重命名，攻击者可能利用web容器解析漏洞

修复步骤：

1. 接收文件及其文件临时路径
2. 获取扩展名与白名单做对比，如果没有命令，程序退出
3. 对文件进行重命名

## XSS(Cross site scripting)跨站点指令码

分为三种类型：反射型，存储型，DOM-based

- 反射型 - 当用户访问一个带有XSS代码的URL请求时，服务器端接收数据后处理，然后把带有XSS代码的数据发送到浏览器，浏览器解析这段带有XSS代码的数据后，最终造成XSS漏洞

- 存储型 - 允许用户存储数据的Web应用程序都可能会出现存储型XSS漏洞，当攻击者提交一段XSS代码后，被服务器端接收并存储，当攻击者再次访问某个页面时，这段XSS代码被程序读出来响应给浏览器，造成XSS跨站攻击

  XSS注入要先确定输入点与输出点。如果输出的数据在属性内，那么XSS代码是不会被执行的，那么需要换个方式注入，用带有闭合标签的方式，这样就造成了XSS跨站漏洞

  主要有三种方式的输入点：

  ```html
  // 1. 普通注入
  <script>alert(document.cookie)</script>
  // 2. 闭合标签注入
  " /><script>alert(document.cookie)</script>
  // 3. 闭合标签注入
  </textarea>'"><script>alert(document.cookie)</script>
  ```

- DOM-based - 不需要与服务器端交互，只发生在客户端处理数据阶段

### 带来危害

- XSS会话劫持(cookie)
- XSS蠕虫

### 如何防御

1. 转义输入输出的敏感内容，如对于引号，尖括号，斜杠转义

   ```js
   & -> &amp;
   < -> &lt;
   > -> &gt;
   " -> &quto;
   ' -> &##39;
   ` -> &##96;
   \/ -> &##x2F;
   ```

   对于显示富文本来说，通常采用白名单过滤的办法[ js-xss](https://github.com/leizongmin/js-xss)

2. **CSP 内容安全策略**

   CSP是一个额外的安全层，用于检测并削弱某些特定类型的攻击，包括跨站脚本（XSS）和数据注入攻击等。无论是数据盗取、网站内容污染还是散发恶意软件

   CSP本质是建立白名单，规定浏览器只能执行特定来源的代码

   通常可以通过HTTP header中的Content-Security-Policy来开启CSP

- 只允许加载本站资源

  ```
  Content-Security-Policy: default-src 'self'
  ```

- 只允许加载HTTPS协议图片

  ```
  Content-Security-Policy: img-src https://*
  ```

- 允许加载任何来源框架

  ```
  Content-Security-Policy: child-src 'none'
  ```

3. HttpOnly

   HttpOnly实际对防御XSS漏洞不起作用，而不要目的是为了解决XSS漏洞后续的Cookie劫持攻击

   在身份标识字段使用 HttpOnly可以有效的阻挡XSS会话劫持攻击，但却不能完全阻挡XSS攻击。因为XSS攻击的手段太多，仅靠HttpOnly是不够的，防御还是要靠过滤输入与输出

4. DOM型XSS攻击预防

   - 使用`.innerHTML`、`.outerHTML`、`document.write()`时要特别小心，不要把不可信的数据作为HTML插到页面上，应尽量使用`.textContent`、`.setAttribute()`等

   - DOM中的内联事件监听器，如location、onclick、onerror、onload、onmouseover等，< a >标签的href属性，javaScript的eval()、setTimeout()、setInterval()等，都能把字符串作为代码运行。

     ```html
     <!-- 内联事件监听器中包含恶意代码 -->
     <img onclick="UNTRUSTED" onerror="UNTRUSTED" src="data:image/png,">
     
     <!-- 链接内包含恶意代码 -->
     <a href="UNTRUSTED">1</a>
     
     <script>
     // setTimeout()/setInterval() 中调用恶意代码
     setTimeout("UNTRUSTED")
     setInterval("UNTRUSTED")
     
     // location 调用恶意代码
     location.href = 'UNTRUSTED'
     
     // eval() 中调用恶意代码
     eval("UNTRUSTED")
     </script>
     ```

5. 验证码：防止脚本冒充用户提交危险操作

## CSRF/XSRF(Cross-site request forgery)跨站请求伪造

是一种挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方式。其实就是利用用户的登录态发起的恶意请求

CSRF攻击是建立在会话之上的。也就是当被攻击站点(A)没有登出的情况，攻击者站点(B)发出了A中的请求，倘若该请求需要cookie认证信息，因为A站点已经登录此时浏览器存在cookie，浏览器无法分别该请求是从哪个站点发出的，造成了请求伪造，欺骗了用户。

CSRF无论是GET还是POST请求都能攻击。只是GET请求更为简单，POST请求要构造一个自动提交的form表单。

### 如何防御

遵循几种规则：

- 尽量不使用Get请求对数据进行修改

- 验证HTTP Referer字段，但这种方式存在缺陷，低版本浏览器可以篡改Referer

- 将token信息附加在form表单请求中，以hidden属性方式提交

- 请求时附带验证信息(**请求针对于会修改数据的请求，展示型的请求尽量不要带token**)，比如验证码或者token(**token一般不能直接使用已有的cookie，因为这样第三方站点进行提交请求发动CSRF攻击，该cookie也会被浏览器自动带到目标站点。token也不能显示的卸载URL链接的地方并且多次有效，因为这样第三方站点也可以直接拷贝该token后直接复制到第三方站点的某个URL，通过诱导用户打开某个链接而自动提交该token。一种方式是可以以参数的形式传递或者放在HTTP的请求头的自定义属性中，第二种方式可以统一所有请求，而不用一一给每个请求添加token。若一定要基于cookie的方式，可以依赖salt，该salt存储在cookie，然后再自己的站点动态执行脚本，对salt进行某种算法变换成一个token，并且该token通过脚本在自己站点提交的时候动态增加该参数进行提交，对于第三方站点，虽然发送CSRF请求浏览器会自动通过cookie带salt到目标站点，但第三方站点无法通过脚本读取该cookie并进行按照某种算法转换成token**)

  对于token的获取，可以由服务器生成一个随机的token，保存在session中，并在登录时返回给前端；前端在每次请求时带上token信息，服务器拦截请求，查看发送的token和服务器session中的是否一致。若一致，允许请求；否则拒绝请求。

  该种方式可以的前提是伪造者请求无法获取到token信息，若可以获取到，也形同虚设。

SameSite

> 可以对Cookie设置SameSite属性。该属性设置Cookie不随跨域请求发送，该属性可以很大程度减少CSRF的攻击，但不是所有浏览器都兼容

## 密码安全

### 加盐

通常需要对密码加盐，再进行几次不同加密算法的加密

```
// 加盐也就是给原密码添加字符串，增加原密码长度
sha256(sha1(md5(salt + password+ salt)))
```

