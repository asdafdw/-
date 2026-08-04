![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cb1407fb2952567855ff8b574772ec2e.png#pic_center)

- 门户：综合类漏洞
- 电商：业务逻辑突出
- 论坛：xss逻辑突出
- 博客：漏洞较少
- 第三方：据功能决定

### 漏洞挖掘思路

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/54ae161526480b821d482e6f090e5bf6.png#pic_center)

#### 文件上传

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a5608a7c796bf5bdcf30b37a75444291.png)

#### xss

漏洞可能产生点：留言板，评论区，订单系统，反馈条件等

#### SSRF

常出现在：1.能够对外发起网络请求的地方 2.请求远程服务器资源的地方 3.数据库内置功能 4.邮件系统 5.文件处理 6.在线处理工具

   　1.分享：通过URL地址分享网页内容　　　　　　　　　　　　　　　　　　　　　　　　　　

　　2.转码服务（通过URL地址把原地址的网页内容调优，使其适合手机屏幕的浏览）

　　3.在线翻译

　　4.图片加载与下载：通过URL地址加载或下载图片

　　5.图片、文章收藏功能

　　6.未公开的api实现及调用URL的功能

　　7.从URL关键字中寻找

![img](https://img2018.cnblogs.com/i-beta/1646039/201911/1646039-20191106063648464-50325334.png)

#### 文件下载

白盒分析：

查看源码中是否存在：**上传类函数，删除类函数，下载类函数，目录操作函数，读取查看函数等**



看到下面的点，可以去想想是不是文件下载漏洞
read.xxx?filename=
down.xxx?filename=
readfile.xxx?file=
downfile.xxx?file=
…/ …\ .\ ./等
%00 ? %23 %20 .等
&readpath=、&filepath=、&path=、&inputfile=、&url=、&data=、&readfile=、&menu=、META-INF= 、
WEB-INF

##### 漏洞判断

参数f的参数值为PHP文件时：

1.文件被解析，则是文件包含漏洞

2.显示源代码，则是文件查看漏洞

3.提示下载，则是文件下载漏洞