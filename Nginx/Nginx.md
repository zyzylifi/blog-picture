---
title: Nginx
date: 2023-05-10
categories:
  - Nginx
tags:
  - Nginx
---

## 前提准备

### 虚拟机安装

注意：安装虚拟机是为了在本地虚拟化Linux环境，便于学习。对应了实际应用场景中，企业的云服务器

- 本地安装虚拟机Vmware

- 系统CentOS 7.4 [官网 (opens new window)](https://www.centos.org/download/)下载适合自己的版本

  ![image-20220430161537719](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220430161537719.png)

  ![image-20220430161625233](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220430161625233.png)

  ![image-20220430161855527](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220430161855527.png)

  | 版本           | 说明                                                         |
  | -------------- | ------------------------------------------------------------ |
  | DVD ISO        | 标准安装版                                                   |
  | Everything ISO | 对完整版安装盘的软件进行补充，集成所有软件。(包含[centos7 (opens new window)](https://so.csdn.net/so/search?q=centos7&spm=1001.2101.3001.7020)的一套完整的软件包，可以用来安装系统或者填充本地镜像) |
  | Minimal ISO    | 精简版，自带的软件最少                                       |
  | NetInstall ISO | 网络安装镜像(从网络安装或者救援系统)                         |

- 使用虚拟机，安装CentOS镜像

  第一步、新建虚拟机
  ![image-20220430161855527](https://aeiherumuh10.gitee.io/blog-picture/Nginx/企业微信截图_20230424145341.png)

  - 选择典型

  ![image-20220430161855527](https://aeiherumuh10.gitee.io/blog-picture/Nginx/企业微信截图_20230424145722.png)

  - 选择下载好的镜像文件

  ![image-20220430161855527](https://aeiherumuh10.gitee.io/blog-picture/Nginx/企业微信截图_20230424150518.png)

  - 命名并选择存储位置

  ![image-20220430161855527](https://aeiherumuh10.gitee.io/blog-picture/Nginx/企业微信截图_20230424150249.png)

  - 指定磁盘大小

  ![image-20220430161855527](https://aeiherumuh10.gitee.io/blog-picture/Nginx/企业微信截图_20230424150259.png)

  - 完成

  ![image-20220430161855527](https://aeiherumuh10.gitee.io/blog-picture/Nginx/企业微信截图_20230424150310.png)


  第二步、Vmware安装好镜像，进入CentOS系统配置

  - 选择语言

    ![image-20220430164248258](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220430164248258.png)

  - 确认系统安装位置

    ![image-20220430164412467](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220430164412467.png)

    ![image-20220430164443903](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220430164443903.png)

  - 开始安装，安装过程中设置root密码，安装完成后重启

    ![image-20220430164506588](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220430164506588.png)

    ![image-20220430164600537](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220430164600537.png)

    ![image-20220430165140257](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220430165140257.png)

  - 账号root，密码就是上一步设置的密码

    ![image-20220430165302826](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220430165302826.png)

    登陆后

    ![image-20220430165355053](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220430165355053.png)

### 配置虚拟机网络

```
ip addr #查看网络
```

可以看到两个网卡 `lo`和`ens33`（lo是本地网卡）

![image-20220430165742369](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220430165742369.png)

使用vi编辑器打开`ens33`网络的配置文件，修改启动配置（如何使用vi编辑器：`i`修模式改数据，`esc`退出，`:wq`保存退出vi编辑器）

```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

修改ONBOOT为yes

![image-20220430170248402](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220430170248402.png)

**重启网络**

```
systemctl restart network
```

就能看到ens33被分配在了网络IP（192.168.174.128），但是这个网路是动态分配的内网地址，重启后会一直变化。

![image-20220430172307651](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220430180620463.png)

### 设置静态ip地址

​        在使用虚拟机的时候，默认情况下使用的[DHCP协议](https://so.csdn.net/so/search?q=DHCP%E5%8D%8F%E8%AE%AE&spm=1001.2101.3001.7020)分配的动态IP地址，使得每次打开虚拟机后当前的IP地址都会发生变化，这样不方便管理。为了能够给当前虚拟机设置一个静态IP地址，方便后期使用XShell连接工具进行连接，以及配置各种服务。所以，我们需要为虚拟机设置一个静态IP地址。

- 第一步：打开网卡配置文件

  ```shell
  vi /etc/sysconfig/network-scripts/ifcfg-ens33(网卡名称可能不同)
  ```


- 增加一下配置

  ```shell
  IPADDR=192.168.174.120 # 设置IP
  NETMASK=225.225.225.0 #子网掩码
  GATEWAY=192.168.174.1 #网关
  DNS1=8.8.8.8 # DNS服务器地址
  ```

对应ip通过vmware软件中编辑->虚拟网络编辑器->NAT设置中查看，如图

![image-20220430172307651](https://aeiherumuh10.gitee.io/blog-picture/Nginx/企业微信截图_20230424154118.png)

- 修改文件中BOOTPROTO字段为static后，后IP就固定为了IPADDR的值

  ```shell
  BOOTPROTO=static #原来值是dhcp，就是动态获取ip的一个协议
  ```

- 修改完成的配置如图

![image-20220430172307651](https://aeiherumuh10.gitee.io/blog-picture/Nginx/企业微信截图_20230424154346.png)



## Nginx介绍

- Nginx开源版 http://nginx.org/en/

  官方原始的Nginx版本

- Nginx plus商业版

  开箱即用，集成了大量功能

- Open Resty https://openresty.org/cn/

  OpenResty是一个基于Nginx与 Lua 的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。**更适用于需要大量二次开发的场景，有极强的扩展性**

- Tengine https://tengine.taobao.org/

  由淘宝网发起的Web服务器项目。它在[Nginx](http://nginx.org/)的基础上，针对大访问量网站的需求，添加了很多高级功能和特性。Tengine的性能和稳定性已经在大型的网站如[淘宝网](http://www.taobao.com/)，[天猫商城 ](http://www.tmall.com/)等得到了很好的检验。相比于Open Resty，扩展性不够强，但是能够满足绝多数使用场景

## Nginx安装

**第一种方式安装：**

```
使用epel源安装
yum install epel-release -y
yum install nginx -y
systemctl start nginx.service //启动nginx
systemctl enable nginx.service //开机启动
```

- 检查nginx是否启动成功 执行`netstat -lntup`

  ![企业微信截图_20231025162122](https://aeiherumuh10.gitee.io/blog-picture/Nginx/企业微信截图_20231025162122.png)

- 根据虚拟机ip，查看是否能访问nginx

![企业微信截图_20231025162306](https://aeiherumuh10.gitee.io/blog-picture/Nginx/企业微信截图_20231025162306.png)

- 如果无法访问，执行`systemctl stop firewalld.service`命令关闭防火墙

**第二种方式安装：**

- **下载Nginx包**

  [官网下载地址](http://nginx.org/en/download.html)

![image-20220501140833867](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220501140833867.png)

- 把压缩包放到服务器root目录中

![微信图片编辑_20231027142444](https://aeiherumuh10.gitee.io/blog-picture/Nginx/微信图片编辑_20231027142444.jpg)

- 解压Nginx包，并安装

  ```shell
  tar -zxvf  nginx-1.21.6.tar.gz #解压到当前目录

  cd nginx-1.21.6 #进入解压后的文件夹
  ls #文件夹中的文件：auto CHANGES.ru  configure  html man src CHANGES conf contrib LICENSE README
  ```


- 安装依赖库

  ```shell
  #安装C编译器
  yum install -y gcc

  #安装pcre库
  yum install -y pcre pcre-devel

  #安装zlib
  yum install -y zlib zlib-devel
  ```

- 安装

  ```shell
  ./configure --prefix=/usr/local/nginx #使用prefix选项指定安装的目录
  make
  make install
  ```

- 启动

  ```shell
  cd /usr/local/nginx/sbin

  ls # 里面是一个nginx的可执行文件

  ./nginx # 启动这个可执行
  ```


- 关闭防火墙

  ```shell
  systemctl stop firewalld
  ```


- 补充Nginx命令

  ```shell
  ./nginx -s stop #快速停止
  ./nginx -s quit #完成已接受的请求后，停止
  ./nginx -s reload #重新加载配置
  ./nginx -t #检查nginx配置是否正确
  ```


- 查看nginx状态

  ```shell
  ps -ef|grep nginx
  ```

  启动时：

![image-20220501205303265](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220501205303265.png)

  停止时：

![image-20220501205333304](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220501205333304.png)

- 注册系统服务

  通过系统服务的方式启动nginx

  ```shell
  vim usr/lib/systemd/system/nginx.service
  ```

  ```shell
  [Unit] 
  Description=nginx
  After=network.target remote-fs.target nss-lookup.target

  [Service]
  Type=forking
  PIDFile=/usr/local/nginx/logs/nginx.pid
  ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
  ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
  ExecReload=/usr/local/nginx/sbin/nginx -s reload
  ExecStop=/usr/local/nginx/sbin/nginx -s stop
  ExecQuit=/usr/local/nginx/sbin/nginx -s quit 
  PrivateTmp=true
     
  [Install]   
  WantedBy=multi-user.target  # 多用户
  ```

## Nginx 目录

![企业微信截图_20231025173540](https://aeiherumuh10.gitee.io/blog-picture/Nginx/企业微信截图_20231025173540.png)

```
conf #配置文件
	｜-nginx.conf # 主配置文件
	｜-其他配置文件 # 可通过那个include关键字，引入到了nginx.conf生效
	
html #静态页面

logs
	｜-access.log #访问日志(每次访问都会记录)
	｜-error.log #错误日志
	｜-nginx.pid #进程号
	
sbin
	｜-nginx #主进程文件
	
*_temp #运行时，生成临时文件
```

## Nginx基础配置

```shell
worker_processes  1; # 启动的worker进程数

events {
    worker_connections  1024; #每个worker进程的连接数
}

http {
    include       mime.types; #include是引入关键字，这里引入了mime.types这个配置文件（同在conf目录下，mime.types是用来定义，请求返回的content-type）
    default_type  application/octet-stream; #mime.types未定义的，使用默认格式application/octet-stream

    sendfile        on; #详情，见下文
    keepalive_timeout  65; #长链接超时时间
	
		#一个nginx可以启用多个server（虚拟服务器）
    server {
        listen       80;#监听端口80
        server_name  localhost;  #接收的域名

        location / { 
            root   html; #根目录指向html目录
            index  index.html index.htm; #域名/index 指向 index.html index.htm文件
        }

        error_page   500 502 503 504  /50x.html; # 服务器错误码为500 502 503 504，转到"域名/50x.html"
        location = /50x.html {# 指定到html文件夹下找/50x.htm
            root   html;
        }

    }
}
```

**sendfile**

打开sendfile，用户请求的数据不用再加载到nginx的内存中，而是直接发送

![image-20220502113913235](https://aeiherumuh10.gitee.io/blog-picture/Nginx/image-20220502113913235.png)

## Nginx进阶配置及应用场景

### nginx多站点配置

**不同二级域名，映射到不同静态网页**

可以写多个server字段，从前向后匹配，先匹配到哪个就用哪个

用户访问`pro.hedaodao.ltd`，就会走到第一个server配置；`test.hedaodao.ltd`走到第二个配置

```nginx
 http {
 		#....其他属性
 		server {
        listen       80;
        server_name  pro.hedaodao.ltd;

        location / { 
            root   html/pro; 
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
		}

 		server {
        listen       80;
        server_name  test.hedaodao.ltd;

        location / { 
            root   html/test; 
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
		}
}
```

**不同域名，映射到同一静态页面**

server_name

- 可以写多个，用空格分开
- 使用通配符（*）
- 使用正则表达式（https://blog.csdn.net/yangyelin/article/details/112976539）

```nginx
http{ 		
 		server {
            listen       80;
            server_name  *.hedaodao.ltd  ~^[0-9]+\.hedaodao\.ltd$; # "\."是转译"."

            location / { 
                root   html/test; 
                index  index.html index.htm;
            }

            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
		}
}
```

### include配置文件

**1、应用场景：**

> 当存在多个域名时，如果所有配置都写在 nginx.conf 主配置文件中，难免会显得杂乱与臃肿。
>
> 为了方便配置文件的维护，所以需要进行拆分配置。

**2、在 nginx 的 conf 目录下 创建 vhost 文件夹：**

```shell
[root@Centos conf]# pwd

/usr/local/nginx/conf

[root@Centos conf]# mkdir vhost
```

**3、在 vhost 文件夹中创建 test1.com.conf 和 test2.com.conf 文件：**

（1）test1.com.conf 文件内容：

```nginx
server {
        listen 8000;
        server_name test1.com;
        location / { 
                root   html/test; 
                index  index.html index.htm;
        }
}
```

（2）test2.com.conf 文件内容：

```nginx
   server {
        listen 8000;
        server_name test2.com;
        location / { 
                root   html/test; 
                index  index.html index.htm;
        }
   }
```

**4、在 nginx.conf 主配置文件 http{…} 段中加入以下内容：**

```shell
include vhost/*.conf;　　　　# include 指令用于包含拆分的配置文件
```

### nginx日志

> nginx日志分为访问日志`access.log` 和 错误日志`error.log`，两者都是存放在nginx安装目录的log文件夹中。例如`/usr/share/nginx/logs/`

![1698635245015](https://aeiherumuh10.gitee.io/blog-picture/Nginx/1698635245015.png)

1、access.log日志文件

access文件用于存放每个用户访问网站的请求日志，开发运维人员通过访问日志来分析用户的浏览器行为。默认情况下，nginx会在`log`目录下生成该文件，无需用户配置。

**自定义请求日志文件**

> `access.log` 允许自定义日志格式、内容、文件路径等

```nginx
http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    server {
        listen       80;
        server_name  localhost;
        access_log logs/mine_access.log main;
        location / {
            root   html;
            index  index.html index.htm;
        }
    }
}

# log_format 定义日志格式 
# main 格式名称
# access_log logs/mine_access.log main;  在server中声明 文件目录和定义的log_format名称
```

**日志格式变量**

| 参数                    | 说明                                         | 示例                                                         |
| ----------------------- | -------------------------------------------- | ------------------------------------------------------------ |
| $remote_addr            | 客户端地址                                   | 211.28.65.253                                                |
| $remote_user            | 客户端用户名称                               | --                                                           |
| $time_local             | 访问时间和时区                               | 18/Jul/2012:17:00:01 +0800                                   |
| $request                | 请求的URI和HTTP协议                          | "GET /article-10000.html HTTP/1.1"                           |
| $http_host              | 请求地址，即浏览器中你输入的地址（IP或域名） | [www.wang.com](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.wang.com) 192.168.100.100 |
| $status                 | HTTP请求状态                                 | 200                                                          |
| $upstream_status        | upstream状态                                 | 200                                                          |
| $body_bytes_sent        | 发送给客户端文件内容大小                     | 1547                                                         |
| $http_referer           | url跳转来源                                  | [https://www.baidu.com/](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.baidu.com%2F) |
| $http_user_agent        | 用户终端浏览器等信息                         | "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; SV1; GTB7.0; .NET4.0C; |
| $ssl_protocol           | SSL协议版本                                  | TLSv1                                                        |
| $ssl_cipher             | 交换数据中的算法                             | RC4-SHA                                                      |
| $upstream_addr          | 后台upstream的地址，即真正提供服务的主机地址 | 10.10.10.100:80                                              |
| $request_time           | 整个请求的总时间                             | 0.205                                                        |
| $upstream_response_time | 请求过程中，upstream响应时间                 | 0.002                                                        |

### URL重写(rewrite)

```shell
rewrite是URL重写的关键指令，根据regex（正则表达式）部分内容，重定向到replacement，结尼是flag标记。

rewrite    <regex>   <replacement>  [flag];
关键字				正则				替代内容     flagt标记

正则：per1森容正则表达式语句进行规则匹配
替代内容：将正则匹配的内容替换成replacement

flag标记说明：
last  #本条规则匹配完成后，继续向下匹配新的1ocation URI规则
break #本条规则匹配完成即终止，不再匹配后面的任何规则

redirect #返回302临重定向，游览器地址会显示跳转后的URL地址
permanent #返回301永久重定向，测览器地址栏会显示跳转后的URL地址
```

浏览器地址栏访问 `xxx/123.html`实际上是访问`xxx/index.jsp?pageNum=123`

```nginx
server {
        listen       80;
        server_name  localhost;
				
				location / { 
						rewrite ^/([0-9]+).html$ /index.jsp?pageNum=$1  break;
        		proxy_pass http://xxx;
        }
      

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}
```

### 反向代理+负载均衡

**nginx.conf配置文件**

启用proxy_pass，root和index字段就会失效

proxy_pass后的地址必须写完整 `http://xxx`，不支持https

当访问localhost时（Nginx服务器），网页打开的是`http://xxx`（应用服务器），网页地址栏写的还是localhost

```nginx
http{ 		
 		server {
            listen       80;
            server_name  localhost;
            location / { 
                    proxy_pass http://xxx;
                #root   html/test; 
                #index  index.html index.htm;
            }
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
		}
}
```

**定义地址别名 **

使用upstream定义一组地址【在server字段下】

访问localhost，访问都会代理到`192.168.174.133:80`和`192.168.174.134:80`这两个地址之一，每次访问这两个地址轮着切换（后面讲到，因为默认权重相等）

```nginx
http{
	upstream httpds{
		server 192.168.174.133:80; #如果是80端口，可以省略不写
		server 192.168.174.134:80;
	}
	server {
        listen       80;
        server_name  localhost;

        location / { 
        		proxy_pass http://httpds;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
	}
}
```

**设置权重**

访问使用哪个地址的权重

```nginx
upstream httpds{
		server 192.168.174.133:80 weight=10;
		server 192.168.174.134:80 weight=80;
}
```

**关闭**

```nginx
upstream httpds{
		server 192.168.174.133:80 weight=10 down;
		server 192.168.174.134:80 weight=80;
}
```

**备用机**

如果`192.168.174.133:80`出现故障，无法提供服务，就用使用backup的这个机器

```nginx
upstream httpds{
		server 192.168.174.133:80 weight=10;
		server 192.168.174.134:80 weight=80 backup;
}
```

### 动静分离

当用户请求时，动态请求分配到Tomcat业务服务器，静态资源请求放在Nginx服务器中

例子：

- 如果请求的资源地址是`location/`，`/`的优先级比较低，如果下面的location没匹配到，就会走http://xxx这个地址的机器
- 如果请求的资源地址是`location/css/*`，就会被匹配到nginx的html目录下的css文件夹中（我们把css静态资源放在这个位置）

```nginx
server {
        listen       80;
        server_name  localhost;
				location / { # /的优先级比较低，如果下面的location没匹配到，就会走http://xxx这个地址的机器
        		proxy_pass http://xxx;
        }
        
        location /css {  # root指的是html，location/css指的是root下的css，所以地址就是html/css
        		root html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}
```

使用正则

```
location ~*/(js|css|img){
	root html;
  index  index.html index.htm;
}
```

### ssl证书配置

::: warning

由于版本问题，配置文件可能存在不同的写法。例如：Nginx 版本为 `nginx/1.15.0` 以上请使用 `listen 443 ssl` 代替 `listen 443` 和 `ssl on`。

:::

```nginx
server {
     #SSL 默认访问端口号为 443
     listen 443 ssl; 
     #请填写绑定证书的域名
     server_name cloud.tencent.com; 
     #请填写证书文件的相对路径或绝对路径
     ssl_certificate cloud.tencent.com_bundle.crt; 
     #请填写私钥文件的相对路径或绝对路径
     ssl_certificate_key cloud.tencent.com.key; 
     ssl_session_timeout 5m;
     #请按照以下协议配置
     ssl_protocols TLSv1.2 TLSv1.3; 
     #请按照以下套件配置，配置加密套件，写法遵循 openssl 标准。
     ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; 
     ssl_prefer_server_ciphers on;
     location / {
         #网站主页路径。此路径仅供参考，具体请您按照实际目录操作。
         #例如，您的网站主页在 Nginx 服务器的 /etc/www 目录下，则请修改 root 后面的 html 为 /etc/www。
         root html; 
         index  index.html index.htm;
     }
 }
```

http转https

```nginx
server {
    listen 80;
    server_name 10.20.10.62;
    rewrite ^(.*)$ https://$host$1 permanent;
}
```

### gzip压缩

```nginx
 gzip on; #启用gzip压缩功能。
 gzip_min_length  1k; #设置最小压缩文件大小，小于此大小的文件不会被压缩。
 gzip_buffers     4 16k; #内存缓冲
 gzip_http_version 1.1; #http版本
 gzip_comp_level 2; #设置压缩级别，取值范围为1-9，数值越大压缩比越高，但同时也会更消耗CPU资源。
 gzip_types   text/plain application/javascript application/x-javascript text/javascript text/css application/xml;  #配置要进行gzip压缩的文件类型，与文件扩展名相关
 gzip_vary on; #在header头里面添加压缩标识
 gzip_proxied  expired no-cache no-store private auth; #根据客户端请求中的"Accept-Encoding"头部决定是否压缩响应，取值可以是 “off”、“expired”、“no-cache”、“no-store”、“private”、“no_last_modified”、“no_etag”、“auth” 或 “any”。
 gzip_disable   "MSIE [1-6]\."; #ie6以下不启用gzip
```

在前端项目中可以打包成gzip文件，可以使用`gzip_static  on` ,使用该配置后可以直接寻找同名的`**.js.gz`文件，节省资源和性能








