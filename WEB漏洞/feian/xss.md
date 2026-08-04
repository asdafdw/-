## 原理

Js是浏览器执行的前端语言，用户在存在xss漏洞的站点url后者能输入数据的部分插入js语言，服务器接收到此数据，认为是js代码，从而返回的时候执行。因此，攻击者可利用这个漏洞对站点插入任意js代码进行窃取用户的信息。

## 分类

### DOM



###  反射



### 存储

## 绕过检测方式

1.双写绕过

2.大小写绕过

3.事件闭合标签：eg；鼠标点击事件onclick='alert(1)

4注入点在内置属性中被标签嵌套，先闭合标签

5.标签的js伪协议实现href属性支持javascript:伪协议构造poc 产生一个链接

"> <a href=javascript:alert('xss') > xss</a> //

6.Unicode编码

### Httponly绕过

如果您在cookie中设置了HttpOnly属性，那么通过js脚本将无法读取到cookie信息，这样能有效的防止XSS攻击，但是并不能防止xss漏洞只能是防止cookie被盗取。

绕过 httponly：
浏览器未保存帐号密码：需要 xss 产生登录地址，利用表单劫持
浏览器保存帐号密码：浏览器读取帐号密码

### WAF检测绕过

[第二十八天waf绕过](https://www.yuque.com/weiker/xiaodi/ay5ige0mva8qme4v)