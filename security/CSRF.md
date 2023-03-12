# CSRF

CSRF（Cross-Site Request Forgery），中文名称：跨站请求伪造攻击

完成 CSRF 攻击必须要有三个条件：

1、用户已经登录了站点 A，并在本地记录了 cookie
2、在用户没有登出站点 A 的情况下（也就是 cookie 生效的情况下），访问了恶意攻击者提供的引诱危险站点 B (B 站点要求访问站点A)。
3、站点 A 没有做任何 CSRF 防御

## 防护策略

CSRF通常从第三方网站发起，被攻击的网站无法防止攻击发生，只能通过增强自己网站针对CSRF的防护能力来提升安全性。

上文中讲了CSRF的两个特点：

- CSRF（通常）发生在第三方域名。
- CSRF攻击者不能获取到Cookie等信息，只是使用。

针对这两点，我们可以专门制定防护策略，如下：

1、 阻止不明外域的访问

- 同源检测
- Samesite Cookie

2、 提交时要求附加本域才能获取的信息

- CSRF Token
   > 渲染表单的时候，为每一个表单包含一个 csrfToken，提交表单的时候，带上 csrfToken，然后在后端做 csrfToken 验证。
- 双重Cookie验证

### sameSite

在 HTTP 响应头中，通过 set-cookie 字段设置 Cookie 时，可以带上 SameSite 选项，如下：
SameSite 选项通常有 Strict、Lax 和 None 三个值。

Strict 最为严格。如果 SameSite 的值是 Strict，那么浏览器会完全禁止第三方Cookie。

Lax 相对宽松一点。在跨站点的情况下，从第三方站点的链接打开和从第三方站点提交Get 方式的表单这两种方式都会携带 Cookie。但如果在第三方站点中使用 Post 方法，或者通过 img、iframe 等标签加载的 URL，这些场景都不会携带 Cookie。

None 任何第三方都可携带cookie。
