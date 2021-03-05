# redis.conf常用配置

### 服务器端设定

- 设置服务器以守护进程的方式运行

> deamonize yes|no

- 绑定主机地址

> bind 127.0.0.1

- 设置服务器端口号

> port 6379

- 设置数据库数量

> databases 16

**日志配置**

- 设置服务器以指定日志记录级别

> loglevel debug|verbose|notice|warning

- 日志记录文件名

> logfile 端口号.log

注意：日志级别开发期设置为verbose即可，生产环境中配置为notice，简化日志输出量，降低写日志IO的频度

**客户端配置**

- 设置同一时间最大客户链接数，默认无限制。当客户端连接到达上线，Redis会关闭新的链接

> maxclients 0

- 客户端限制等待最大时常，达到最大之后关闭连接。如需关闭该功能，设置为0

> timeout 300

多服务器快捷配置

- 导入并加载指定配置文件信息，用于快速创建redis公共配置较多的redis实例配置文件，便于维护

> include /path/server-端口号.conf