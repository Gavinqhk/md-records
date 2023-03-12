# SQL注入

考虑以下简单的管理员登录表单：

```r
<form action="/login" method="POST">
    <p>Username: <input type="text" name="username" /></p>
    <p>Password: <input type="password" name="password" /></p>
    <p><input type="submit" value="登陆" /></p>
</form>
```

后端的 SQL 语句可能是如下这样的:

```r
let querySQL = `
    SELECT *
    FROM user
    WHERE username='${username}'
    AND psw='${password}'
`;
// 接下来就是执行 sql 语句...
```

目的就是来验证用户名和密码是不是正确，按理说乍一看上面的 SQL 语句也没什么毛病，确实是能够达到我们的目的，可是你只是站在用户会老老实实按照你的设计来输入的角度来看问题，如果有一个恶意攻击者输入的用户名是 zoumiaojiang' OR 1 = 1 --，密码随意输入，就可以直接登入系统了。

恶意攻击者的奇怪用户名将你的 SQL 语句变成了如下形式：

```r
SELECT * FROM user WHERE username='zoumiaojiang' OR 1 = 1 --' AND psw='xxxx'
```

在 SQL 中，-- 是注释后面的内容的意思，所以查询语句就变成了：

```r
SELECT * FROM user WHERE username='zoumiaojiang' OR 1 = 1
```

这条 SQL 语句的查询条件永远为真，所以意思就是恶意攻击者不用我的密码，就可以登录进我的账号，然后可以在里面为所欲为，

### 防御 SQL 注入的几点注意事项

- 严格限制Web应用的数据库的操作权限，给此用户提供仅仅能够满足其工作的最低权限，从而最大限度的减少注入攻击对数据库的危害

- 后端代码检查输入的数据是否符合预期，严格限制变量的类型，例如使用正则表达式进行一些匹配处理。

- 对进入数据库的特殊字符（'，"，\，<，>，&，*，; 等）进行转义处理，或编码转换。基本上所有的后端语言都有对字符串进行转义处理的方法，比如 lodash 的 lodash._escapehtmlchar 库。

- 所有的查询语句建议使用数据库提供的参数化查询接口，参数化的语句使用参数而不是将用户输入变量嵌入到 SQL 语句中，即不要直接拼接 SQL 语句。例如 Node.js 中的 mysqljs 库的 query 方法中的 ? 占位参数。

```r
mysql.query(`SELECT * FROM user WHERE username = ? AND psw = ?`, [username, psw]);
```

- 在应用发布之前建议使用专业的 SQL 注入检测工具进行检测，以及时修补被发现的 SQL 注入漏洞。网上有很多这方面的开源工具，例如 sqlmap、SQLninja 等。

- 避免网站打印出 SQL 错误信息，比如类型错误、字段不匹配等，把代码里的 SQL 语句暴露出来，以防止攻击者利用这些错误信息进行 SQL 注入。

- 不要过于细化返回的错误信息，如果目的是方便调试，就去使用后端日志，不要在接口上过多的暴露出错信息，毕竟真正的用户不关心太多的技术细节，只要话术合理就行。
