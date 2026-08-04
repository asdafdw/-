## 指纹

特征1：端口可能是5984，直接访问端口是`couchdb`

特征2：title="Project Fauxton"

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3ad53aeb449197d5da5364ebab704e3c.png)

### 垂直权限绕过漏洞（CVE-2017-12635）



[【vulhub】Couchdb 垂直权限绕过漏洞（CVE-2017-12635）漏洞复现_project fauxton-CSDN博客](https://blog.csdn.net/qq_45300786/article/details/120287521)

影响版本：小于 1.7.0 以及 小于 2.1.1

请求数据包里构造注册用户，403报错没有权限，可以通过burpsuite或postman修改请求数据包发送一次包含两个roles字段的数据包，即可绕过限制

注意点：

第二个roles字段必须为空，

已创建的用户，无法通过这种方法修改密码
想创建多个用户的话，url中的用户名必须和数据包中的用户名一样【url中的use:名字和表单中的name中的用户名要一致】

###  命令执行 （CVE-2017-12636）

影响版本：小于 1.7.0 以及 小于 2.1.1

原理：[couchdb 任意命令执行漏洞（cve-2017-12636）_couchdb 命令执行 (cve-2017-12636)-CSDN博客](https://blog.csdn.net/whatday/article/details/106618050)

https://github.com/vulhub/vulhub/blob/master/couchdb/CVE-2017-12636/exp.py

用python跑这个exp，把里面的target和command分别改为目标机和反弹命令的目的地址

在攻击机上监听：

nc -lvvp 9988(监听9988端口)

然后用python运行exp即可任意执行命令