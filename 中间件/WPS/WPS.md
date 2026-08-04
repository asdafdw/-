wps影响范围为：WPS Office 2023 个⼈版 < 11.1.0.15120

WPS Office 2019 企业版 < 11.8.2.12085



目标主机先在1.html当前路径下启动http server并监听80端口，然后再到目标主机上修改hosts文件：127.0.0.1 clientweb.docer.wps.cn.cloudwps.cn

此时目标主机再打开poc.docx时就会弹出计算机

{可以把1.html中代码的shellcode换成cs生成的shellcode，这样之后打开poc.docx就会被cs远控}

实战：申请{xxxxx}wps.cn域名，

在目标主机host文件增加clientweb.docer.wps.cn.{xxxxx}wps.cn      ip

再到ip中架设1.html网站服务，同时提前修改好1.html中的shellcode成cs的shellcode

具体参考小迪2024第80天