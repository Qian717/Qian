### Apache静态网站

```markdown
# yum install httpd     
# systemctl start httpd 
# systemctl enable httpd
```
  Linux系统中的配置文件
```markdown
主配置文件		   /etc/httpd/conf/httpd.conf 
网站数据目录		  /var/www/html 
访问日志			/var/log/httpd/access_log 
错误日志			/var/log/httpd/error_log 
```
配置httpd服务程序时最常用的参数
```markdown
ServerName 	      网站服务器的域名
DocumentRoot	  网站数据目录
Directory		  网站数据目录的权限
Listen		      监听的IP地址与端口号
DirectoryIndex    默认的索引页页面
ErrorLog 		  错误日志文件
CustomLog 		  访问日志文件
```

-------
#### 基于主机名
定义IP地址与域名之间对应关系的配置文件
```markdown
[root@localhost  ~]# vim /etc/hosts     
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    10.65.2.22 bbs.linuxjoker.com ccs.linuxjoker.com
```

创建用于保存不同网站数据的三个目录，并向其中分别写入网站的首页文件
```markdown
[root@localhost html]# vim /etc/httpd/conf/httpd.conf
[root@localhost html]# mkdir -p /var/www/html/bbs
[root@localhost html]# mkdir -p /var/www/html/ccs
[root@localhost html]# echo "bbs.linuxjoker.com" > /var/www/html/bbs/index.html
[root@localhost html]# echo "ccs.linuxjoker.com" > /var/www/html/ccs/index.html
```

追加写入三个基于主机名的虚拟主机网站参数
```markdown
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
    <VirtualHost 10.65.2.22>
     DocumentRoot "/var/www/html/bbs"
    ServerName "bbs.linuxjoker.com"
     <Directory "/var/www/html/bbs">
    AllowOverride None
    Require all granted
    </Directory>
    </VirtualHost>

    <VirtualHost 10.65.2.22>
    DocumentRoot "/var/www/html/ccs"
    ServerName "ccs.linuxjoker.com"
    <Directory "/var/www/html/ccs">
    AllowOverride None
    Require all granted
    </directory>
    </VirtualHost>
#重启服务
[root@localhost html]# systemctl restart httpd
```
-----
#### 基于端口
```markdown
#创建目录，写入网站的首页文件
[root@localhost ~]# mkdir -p /var/www/html/6111
[root@localhost ~]# mkdir -p /var/www/html/6222
[root@localhost ~]# echo "port:6111" > /var/www/html/6111/index.html
[root@localhost ~]# echo "port:6222" > /var/www/html/6222/index.html

#添加用于监听6111和6222端口的参数
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
#Listen 10.65.2.22:80 
    Listen 80 
    Listen 6111
    Listen 6222

#分别追加写入两个基于端口号的虚拟主机网站参数
	<VirtualHost 10.65.2.22:6111>
    DocumentRoot "/var/www/html/6111"
    ServerName "bbs.linuxjoker.com"
    <Directory "/var/www/html/6111">
    AllowOverride None
    Require all granted
    </Directory>
    </VirtualHost>

    <VirtualHost 10.65.2.22:6222>
    DocumentRoot "/var/www/html/6222"
    ServerName "ccs.linuxjoker.com"
    <Directory "/var/www/html/6222">
    AllowOverride None
    Require all granted
    </directory>
    </VirtualHost>

#写入两个基于端口号的虚拟主机网站参数
[root@localhost html]# semanage port -a -t http_port_t -p tcp 6111
[root@localhost html]# semanage port -a -t http_port_t -p tcp 6222
[root@localhost html]# systemctl start httpd
[root@localhost html]# systemctl restart httpd
[root@localhost html]# semanage port -l| grep http
    http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
    http_cache_port_t              udp      3130
    http_port_t                    tcp      6222, 6111, 80, 81, 443, 488, 8008, 8009, 8443, 9000
    pegasus_http_port_t            tcp      5988
    #重启服务
```