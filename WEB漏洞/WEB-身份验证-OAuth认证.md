## 基础知识

### 什么是OAuth？

OAuth：一种常用的授权框架，它允许网站和Web应用程序请求对另一个应用程序上的用户帐户的有限访问权限，像那种允许使用第三方账号（QQ、微信等）登录的网站，可能就是使用的OAuth框架。

eg：一些网站登录时支持第三方微信,QQ等其余应用账户登录

### 实战情况

采用OAuth认证的一般都是大企业网站，因为小企业网站大多数情况下没多少人用，没必要,还会扩大自己的被攻击面，而且还要向微信qq申请接口，麻烦

## 漏洞发现

抓到OAuth包发现没有state字段，可以尝试CSRF

### -实战案例CSRF

BP靶场，先用给的用户进行第三方登录，然后抓包，发现没有State字段，尝试CSRF

BP靶场只让使用他们自己提供的攻击机，用对方给的，构造一个页面，获取第三方成功用户登录后授权服务器回显给资源服务的响应数据包中的CODE码（code字段）

然后钓鱼受害者访问该页面，获取到对方的code码，拿到code码后就可以通过第三方登录免密登录受害者账户

https://blog.csdn.net/weixin_39190897/article/details/139885599

```
就是你用第三方登录的时候会发送一个code，然后通过返回的东西去绑定账户。这里拦截丢弃后，在利用这个code绑定受害者的账户
```



先自己登录，第三方登录，抓到OAuth响应数据包，发到构造的CSRF页面中的恶意代码的请求体中【当用户访问该页面时，会去发送恶意代码请求体中数据包】

## 分类

参考文章：

https://mp.weixin.qq.com/s/TSsRNZtpttqXBviLwtYT9A

https://mp.weixin.qq.com/s/ATjdIxSOruY-_lCCs2kcGg

### 授权码模式

#### 具体过程

1、授权请求：

- 百度通过微信登录，微信为授权服务器，百度为资源服务器

客户端向授权服务器发送授权请求，如下：

```
GET /authorize?client_id=123
              &redirect_uri=http://client-app.com/test
              &response_type=code
              &scope=profile
              &state=abcd1234 HTTP/1.1
Host: authorization-server.com
```

**client_id`：客户端在授权服务器的ID（公开）。**

**`redirect_uri`：授权完成后，授权服务器回调客户端的地址。【比如百度登录微信，这里就是百度的地址】**

**`response_type`：声明授权类型，code是授权码授权。**

**`scope`：告诉授权服务器，客户端要访问哪些用户数据，profile是仅请求用户的基本信息（如用户名、头像）。**

**【比如用户可以自己选择是否勾选微信上获取头像当做当前网站的头像的功能】**

**`state`：随机字符串，防止CSRF。【与token异曲同工】**

````
2、用户登录与授权：

用户登录后，会跳到授权页面，显示客户端需要请求哪些信息，并提示是否允许授权。

3、返回授权码：

允许授权后，授权服务器会生成一个授权码（短期有效），通过重定向将授权码返回给客户端，重定向地址就是第一步请求时指定的回调地址。

```
GET /callback?code=123456&state=abcd1234 HTTP/1.1
Host: client-app.com
```

`code`：授权码。

4、令牌交换：

客户端后端向授权服务器发送请求，用授权码换取访问令牌。

```
POST /token HTTP/1.1
Host: authorization-server.com
Content-Type: application/x-www-form-urlencoded

code=123456
&client_id=123
&client_secret=secret_key
&redirect_uri=http://client-app.com/test
&grant_type=authorization_code
```

`client_secret`：客户端密钥，用来验证客户端身份。

`grant_type`：指定授权类型，此处固定为 `authorization_code`，表示使用授权码流程。

授权服务器验证客户端身份，通过后返回访问令牌。

```
{
  "access_token": "123456789",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

`access_token`：访问令牌。

`token_type`：令牌类型，告诉客户端如何携带令牌访问资源服务器，Bearer是持有者令牌。

`expires_in`：令牌有效期，以秒为单位。

5、访问资源：

客户端携带访问令牌向资源服务器发起请求，获取用户数据。

```
GET /userinfo HTTP/1.1
Host: resource-server.com
Authorization: Bearer 123456789
```

6、返回数据：

资源服务器验证令牌后，返回用户数据。
````

```
{
  "username": "hack",
  "email": "hack@example.com"
  }
```

### 隐式授权类型

- 与授权码模式的区别在于隐式授权没有第三步

## 利用

### 无State字段

### URL重定向

## 实战案例

https://mp.weixin.qq.com/s/TSsQ_mWGsFYZiF_RBdfbKg

https://mp.weixin.qq.com/s/NuNkzax8nb72qb-S1RvTnQ

https://mp.weixin.qq.com/s/QuhNuVyb2uy2T-br-mxAJw



 