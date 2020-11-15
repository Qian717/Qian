### bind提供域名解析
##### 安装
    # yum install bind-chroot    --安装域名解析服务程序
修改主配置文件
```markdown
#vim /etc/named.conf 
options {
        listen-on port 53 { any; };  --表示服务器上的所有IP地址均可提供DNS域名解析服务
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { any; };   --以及允许所有人对本服务器发送DNS查询请求
```

-------
##### 正向解析
编辑区域配置文件
```markdown
# vim /etc/named.rfc1912.zones
zone "linuxjoker.com" IN {
type master;
file "linuxjoker.com.zone";
allow-update {10.65.2.32;};
 };
```
编辑数据配置文件
```markdown
#cd /var/named
# cp -a named.localhost linuxjoker.com.zone
# vim linuxjoker.com.zone
	$TTL 1D
	@       IN SOA  linuxjoker.com. root.linuxjoker.com. (
											0       ; serial
											1D      ; refresh
											1H      ; retry
											1W      ; expire
											3H )    ; minimum
			NS      ns.linuxjoker.com.
	ns      A       10.65.2.33
	www     A       10.65.2.33
	ftp     A       10.65.2.33

```

把Linux系统网卡中的DNS地址参数修改成本机IP地址(nmtui命令)
```markdown
#nmcli c up ens33
#nslookup --用于检测能否从DNS服务器中查询到域名与IP地址的解析记录
```

--------

##### 反向解析
编辑区域配置文件
```markdown
# vim /etc/named.rfc1912.zones
zone "2.65.10.in-addr.arpa" IN {
type master;
file "10.65.2.arpa";
allow-update {10.65.2.32;};
 };
```

编辑数据配置文件
```markdown
# cp -a named.loopback 10.65.2.arpa
# vim 10.65.2.arpa
	$TTL 1D
	@       IN SOA  linuxjoker.com. root.linuxjoker.com. (
											0       ; serial
											1D      ; refresh
											1H      ; retry
											1W      ; expire
											3H )    ; minimum
			NS      www.linuxjoker.com.
	www     A       10.65.2.33
	33      PTR     www.linuxjoker.com.
	33      PTR     mail.linuxjoker.com.
	20      PTR     ccc.linuxjoker.com.
```

### 部署主从服务器

主服务器区域配置文件
```markdown
# vim /etc/named.rfc1912.zones
	zone "linuxjoker.com" IN {
	type master;
	file "linuxjoker.com.zone";
	allow-update {10.65.2.32;};
	 };

	zone "2.65.10.in-addr.arpa" IN {
	type master;
	file "10.65.2.arpa";
	allow-update {10.65.2.32;};
	 };
	 
# systemctl restart named
```

从服务器区域配置文件
```markdown
	#vim /etc/named.rfc1912.zones
	zone "linuxjoker.com" IN {
	type slave;
	 masters { 10.65.2.33; };
	file "slaves/linuxjoker.com.zone"; };

	zone "2.65.10.in-addr.arpa" IN {
	type slave;
	masters { 10.65.2.33; };
	file "slaves/10.65.2.arpa"; };

# systemctl restart named
```
------
问题:

- 11月 15 12:27:02 centos7-2 named[2855]: transfer of 'linuxjoker.com/IN' from 10.65.2.33#53: failed to connect: host unreachable
- 11月 15 12:27:03 centos7-2 named[2855]: transfer of '2.65.10.in-addr.arpa/IN' from 10.65.2.33#53: failed to connect: host unreachable


从服务器获取不到主服务器的数据，状态显示主服务器不可达，但主从能互相ping通