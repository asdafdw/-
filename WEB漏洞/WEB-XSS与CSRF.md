# xss

## 原理

通过拼接恶意的html代码，利用js语句来执行攻击，实现对用户浏览器的攻击 

Js是浏览器执行的前端语言，用户在存在xss漏洞的站点url后者能输入数据的部分插入js语言，服务器接收到此数据，认为是js代码，从而返回的时候执行。因此，攻击者可利用这个漏洞对站点插入任意js代码进行窃取用户的信息。

#### 相关函数

一般哪些函数会和这些漏洞有关系，常见的输出型函数会产生关系，比如：print、print_r、echo、printf、sprintf、die、var_dump、var_export

## 分类

​       -反射型：攻击代码在url里，输出在http响应中
​      \- 存储型：把用户输入的数据存储在服务器上
​      \- DOM型：通过修改页面的DOM结点形成xss

反射和dom的区别: DOM-XSS是javascript处理输出， 而反射性xss是后台程序处理 

## cookie与session区别

Cookie可以存储在浏览器或者本地，Session只能存在服务器
②session 能够存储任意的 java 对象，cookie 只能存储 String 类型的对象
③Session比Cookie更具有安全性（Cookie有安全隐患，通过拦截或本地文件找得到你的cookie后可以进行攻击）
④Session占用服务器性能，Session过多，增加服务器压力
⑤单个Cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个Cookie，Session是没有大小限制和服务器的内存大小有关。
原文链接：https://blog.csdn.net/weixin_45393094/article/details/104747360

- cookie和session结合使用
  web开发发展至今，cookie和session的使用已经出现了一些非常成熟的方案。在如今的市场或者企业里，一般有两种存储方式：

1、存储在服务端：通过cookie存储一个session_id，然后具体的数据则是保存在session中。如果用户已经登录，则服务器会在cookie中保存一个session_id，下次再次请求的时候，会把该session_id携带上来，服务器根据session_id在session库中获取用户的session数据。就能知道该用户到底是谁，以及之前保存的一些状态信息。这种专业术语叫做server side session。
2、将session数据加密，然后存储在cookie中。这种专业术语叫做client side session。flask采用的就是这种方式，但是也可以替换成其他形式。

# csrf

**原理**：用户访问恶意网站时运行恶意网站上加载的JS，然后攻击者就可以利用受害者的身份 对已经登陆的正常网站发送数据包，达到篡改信息、修改配置等功能

**成因**：Cookie不过期，没有进行进一步的验证用户信息，没有安全意识访问了恶意站点

**利用**：受害者必须依次完成两个步骤，登陆受信任网站A，并在本地生成cookie，在不登出A的情况下，访问危险网站B

**防御**：加token或者验证码；尽量使用POST，限制GET

1.当用户发送重要的请求时需要输入原始密码
2.设置随机Token
3.检测referer来源，请求时判断请求连接是否为当前管理员正在使用的页面（管理员在编辑文章，黑客发来恶意的修改密码链接，因为修改密码页面管理员并没有在操作，所以攻击失败）
4.设置验证码
5.限制请求方式只能为POST
