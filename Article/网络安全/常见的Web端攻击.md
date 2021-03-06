# 常见的Web端攻击

* XSS
* CSRF
* 点击劫持（click jacking）
* SQL注入
* 网络劫持
* DDOS

## XSS

> 跨站脚本攻击（Cross-Site Scripting，XSS）指通过存在安全漏洞的Web网站注册用户的浏览器内运行非法的非本站点HTML标签或JavaScript进行的一种攻击。

XSS 攻击可理解为在别人的站点执行自己的脚本代码，从而窃取用户 cookie，冒充用户，为所欲为。XSS 漏洞的存在一般是直接将用户的输入直接插入到页面中，如果其中包含代码，就会被执行。XSS 攻击可分为反射型，存储型和 DOM 型三种。

**反射性 XSS**

攻击者构造包含恶意代码的链接，诱骗用户点击。恶意代码将从 URL 中取出并执行。一般发生在搜索页面，攻击者构造恶意链接后，通过将链接转成短链接，配合赌博小短信等方式，诱骗用户点击。

**存储型 XSS**

攻击者在个人信息，或者文章评论等地方提交恶意代码，如果开发者没有验证直接存入数据库，恶意代码就会被存入数据库。当别的用户访问该页面时，就会执行恶意代码。这种 XSS 破坏范围非常大，容易造成大量用户的信息被窃取。

**DOM型 XSS**

开发者获取 DOM 中的数据本地执行，或将用户输入直接插入文档中，就会产生 DOM型 XSS 漏洞。攻击者同样需要通过诱骗用户操作达成攻击的目的。与上面不同的是，DOM 型 XSS 是前端过滤不严导致，与后端无关。

### 防御

#### 黑名单

防御 XSS 最有效的方法是字符转义，过滤危险字符，或者不让用户输入。

```js
const escapeHTML = (str) => 
    str.replace(/[&<>'"]/g, (tag) => (
        {
            '&': '&amp;',
            '<': '&lt;',
            '>': '&gt;',
            "'": '&#39;',
            '"': '&quot;'
        }[tag] || tag)
    )
```

#### 白名单

黑名单一棍子打死在一些情况下就不适用了，比如富文本。现在有一些库有更完备的过滤规则，只处理危险的标签。

```js
const xss = require('xss')
let html = xss('<h1 id="title">XSS Demo</h1><script>alert("xss");</script>')
 
 console.log(html)
 // <h1>XSS Demo</h1>&lt;script&gt;alert("xss");&lt;/script&gt;
```

#### CSP

> 内容安全策略 (CSP, Content Security Policy) 是⼀个附加的安全层，⽤于帮助检测和缓解某些类型的攻击，包括跨站脚本 (XSS) 和数据注⼊等攻击。 这些攻击可⽤于实现从数据窃取到⽹站破坏或作为恶意软件分发版本等⽤途。

CSP 是防御 XSS 的另一种思路。字符过滤不让攻击者输入危险字符，CSP 使得即使有危险标签，也不允许执行。CSP 本质上就是建⽴⽩名单，开发者明确告诉浏览器哪些外部资源可以加载和执⾏。我们只需要配置规则，如何拦截是由浏览器⾃⼰实现的。我们可以通过这种⽅式来尽量减少 XSS 攻击。

```js
// 只允许加载本站资源
Content-Security-Policy: default-src 'self'
// 只允许加载 HTTPS 协议图⽚
Content-Security-Policy: img-src https://*
// 不允许加载任何来源框架
Content-Security-Policy: child-src 'none'
```

#### 其他方法

1. 限制输入长度。每个输入框都应该限制输入长度，一方面可以增加攻击难度，另一方面，数据库也有字段长度限制。
2. 数据格式校验。对手机号，邮箱等进行格式校验。
3. 验证码。防止脚本冒充用户操作。
4. http-only Cookie。让脚本不能读取和操作 Cookie，即使完成 XSS 攻击也无法窃取用户信息。

## CSRF

> 跨站请求伪造(Cross Site Request Forgery，CSRF)，它利⽤⽤户已登录的身份，在⽤户毫不知情的情况下，以⽤户的名义完成⾮法操作。

CSRF 利用浏览器会自动发送 Cookie 的特点。举个例子，你登录了银行后，去访问了攻击者的钓鱼网站，此时钓鱼网站可能就会向银行网站发送转账的请求，因为你已经登录过银行的网站，所以此时浏览器会自动发送一行网站的 Cookie，绕过银行网站的检测。

CSRF 一般有以下的特点：
1. CSRF 一般发生在第三方域名
2. CSRF 不能获取 Cookie（比 XSS 弱一点），只是使用

根据这两点，可以针对性地进行防御。

### 防御

#### 同源检测

CSRF 一般发生在第三方域名，可以通过请求的 Origin 或 Referer 字段检测是否是信任的源发出的请求。但这两个字段都可能被修改，无法完全防御。且 https 的请求中不携带 Referer 字段。

#### Samesite Cookie

Google 起草的新提案中新增了一个 Cookie 属性 Samesite，用来限制第三方 Cookie 的发送。但目前兼容性不好。

#### CSRF Token

网站无法识别请求是攻击发出还是用户自己发出，那么可以让用户请求时携带一个攻击者无法获取的 Token，从而防范 CSRF 攻击。

CSRF Token分三个步骤：
1. 进入页面时发送一个 CSRF Token 到页面
2. 页面上发送的请求都携带这个 Token
3. 服务器验证 Token

#### JWT

既然 CSRF 攻击是因为浏览器请求自动携带 Cookie，那我们可以换一种用户的认证方案。JWT（Json Web Token）是一种基于 Token 的鉴权机制，流程如下：
1. 用户发送账号密码到服务器
2. 服务器验证通过后发送一个 Token 给用户
3. 客户端保存这个 Token，并在每次请求中附带这个 Token
4. 服务器验证这个 Token，返回请求的结果

## 点击劫持（click jacking）

> 点击劫持是⼀种视觉欺骗的攻击⼿段。攻击者将需要攻击的⽹站通过 iframe 嵌套的⽅式嵌⼊⾃⼰的⽹⻚中，并将 iframe 设置为透明，在⻚⾯中透出⼀个按钮诱导⽤户点击。

### 防御

点击劫持是比 CSRF 攻击更弱一级的攻击方式，需要诱导用户点击页面上内容。点击劫持需要将页面用 iframe 嵌入，因此可以限制页面被 iframe 引入。

#### X-FRAME-OPTIONS

* DENY，表示⻚⾯不允许通过 iframe 的⽅式展示
* SAMEORIGIN，表示⻚⾯可以在相同域名下通过 iframe 的⽅式展示
* ALLOW-FROM，表示⻚⾯可以在指定来源的 iframe 中展示

#### js

通过脚本判断当前页面是否是顶级窗口

```html
<html>
<body>
    <style id="click-jacking">
        html{
            display: none !important;
        }
    </style>
    <script>
        if (self === top) {
            var style = document.getElementById('click-jacking')
            document.body.removeChild(style)
        } else {
            top.location = self.location
        }
    </script>
</body>
</html>
```

## SQL 注入

SQL 注入是因为没有检验就将用户输入作为 SQL 语句的一部分拼接 SQL。SQL 注入可以绕过系统的登录验证，让攻击者进入网站后台，窃取系统信息，对系统造成破坏。

### 防御

1. 不要相信前端发送过来的任何信息，对参数的类型和格式进行校验。
2. 对进入数据库的特殊字符（如`', ", \, <, >, &, *`）进行转义或者编码转换。
3. 绑定变量，使用预编译语句，而不是直接拼接 SQL。

## 网络劫持

网络劫持分为两种：DNS 劫持和 HTTP 劫持。一般是运营商干的。

### DNS 劫持

如输入淘宝被强制跳转到京东就属于 DNS 劫持。

* DNS 强制解析：通过修改运营商本地 DNS 记录，来引导用户流量到缓存服务器。
* 302 跳转：通过监控网络出口的流量，分析判断哪些内容是可以进行劫持处理的。再对劫持的内存发起302跳转的回复，引导用户获取内容。

### HTTP 劫持

HTTP 明文传输，运营商可以修改 HTTP 的响应内容。比如插个贪玩蓝月的广告给你。

### 防御

HTTP 劫持可以升级为 HTTPS。DNS 劫持只能报警了。

## DDOS

分布式拒绝访问（distributed denial of service，DDOS）是一大类攻击的总称。网站运行的各个环节都可以是它的攻击目标。只要有一个环节被攻破，就会使得整个系统陷入瘫痪，无法访问。

### 常见的攻击方式

* SYN Flood

> 此攻击通过向⽬标发送具有欺骗性源 IP 地址的⼤量 TCP “初始连接请求” SYN 数据包来利⽤ TCP 握⼿。⽬标机器响应每个连接请求，然后等待握⼿中的最后⼀步，这⼀步从未发⽣过，耗尽了进程中的⽬标资源。

* HTTP Flood

> 此攻击向服务器发送大量的 HTTP 请求，⼤量 HTTP 请求泛滥服务器，导致拒绝服务。

### 防御

* 备份网站。不必要做到全功能，最低限度可以显示公告，告知用户正在抢修。
* HTTP 拦截。发出攻击的电脑的 IP 地址或 UserAgent 字段某些特征。如恶意请求都是从某个 IP 段中发出了，那把这个 IP 段封掉就行。
* 带宽扩容 / CDN。通过扩大网站的承载能力，使得普通用户也可以访问，提高攻击的成本。