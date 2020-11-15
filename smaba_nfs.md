#### Samba 文件共享服务
##### 配置共享资源

    #yum install smaba    --安装smaba

1.配置访问共享资源的用户
###### 注意：用于访问配置文件的密码在此处配置，区分于系统用户linuxprobe的密码。
```markdown
# id linuxprobe 
uid=1000(linuxprobe) gid=1000(linuxprobe) groups=1000(linuxprobe) 
#pdbedit -a -u linuxprobe   --pdbedit命令把账户信息写入到数据库
new password:linuxprobe
retype new password:linuxprobe
```

2.创建用于共享资源的文件目录。在创建时，不仅要考虑到文件读写权限的问题，而且由于/home目录是系统中普通用户的家目录，因此还需要考虑应用于该目录的SELinux安全上下文所带来的限制。
```markdown
# mkdir /home/database   --创建目录
# chown -Rf linuxprobe:linuxprobe /home/database   --赋权限
//修改 database 的SELinux安全上下文策略
# semanage fcontext -a -t samba_share_t /home/database 
//应用于目录的新SELinux安全上下文立即生效
# restorecon -Rv /home/database
```

3.设置SELinux服务与策略，使其允许通过Samba服务程序访问普通用户家目录。执行getsebool命令，筛选出所有与Samba服务程序相关的SELinux域策略
```markdown
//查看Samba 服务程序相关的 SELinux 域策略
# getsebool -a | grep samba 
//允许通过 Samba 服务程序访问普通用户家目录
# setsebool -P samba_enable_home_dirs on 
```

4.修改Samba 服务程序的主配置文件
```markdown
# vim /etc/samba/smb.conf
	[global]
	workgroup = MYGROUP
	server string = Samba Server Version %v
	log file = /var/log/samba/log.%m
	max log size = 50
	security = user
	passdb backend = tdbsam
	load printers = yes
	cups options = raw

	[database]
	comment = Do not arbitrarily modify the database file
	path = /home/database
	public = no
	writable = yes
```

5.重启 smb 服务,清空 iptables 防火墙
```markdown
# systemctl restart smb 
# systemctl enable smb 
# iptables -F    --清空iptables防火墙
# service iptables save   --对iptables服务进行保存
```
   在centos7系统以上，service不起作用，输入# service iptables save之后提示"The service command supports only basic LSB actions (start, stop, restart, try-restart, reload, force-reload, status). For other actions, please try to use systemctl."
##### 解决办法：
```markdown
1.先关闭防火墙：
systemctl stop firewalld
systemctl mask firewalld

2.安装iptables services
yum install iptables-services

3.设置开机启动
systemctl enable iptables

4.重启iptables service
systemctl restart iptables

5.执行保存配置命令（成功）
service iptables save
```

-----
##### Windows 访问文件共享服务
要在Windows系统中访问共享资源，只需在Windows的“运行”命令框中输入两个反斜杠，然后再加服务器的IP地址即可，使用pdbedit命令设置的账号密码登录。
#####  Linux 访问文件共享服务
1、在客户端安装支持文件共享服务的软件包（cifs-utils）
    # yum install cifs-utils 

2、按照Samba服务的用户名、密码、共享域的顺序将相关信息写入到一个认证文件中。并把这个认证文件的权限修改为仅root管理员才能够读写
```markdown
[root@localhost ~]# vim auth.smb 
	username=linuxprobe 
	password=linuxprobe 
	domain=MYGROUP 
[root@localhost ~]# chmod 600 auth.smb 
```
3、创建一个用于挂载Samba服务共享资源的目录
```markdown
# mkdir /database 
# echo "//10.65.2.22/database /database cifs credentials=/root/auth.smb 0 0"  >> /etc/fstab
# mount -a
```
------
### NFS
##### 服务端配置
    # yum install nfs-utils
清空NFS服务器上面iptables防火墙的默认策略，以免默认的防火墙策略禁止正常的NFS共享服务
```markdown
[root@localhost ~]# iptables -F 
[root@localhost ~]# service iptables save
```

NFS服务程序的配置文件为/etc/exports，定义要共享的目录与相应的权限
```markdown
[root@localhost ~]# vim /etc/exports
	/nfsfile 10.65.2.*(rw,sync,root_squash)
```


启动和启用NFS服务程序，由于在使用NFS服务进行文件共享之前，需要使用RPC（Remote Procedure Call，远程过程调用）服务将NFS服务器的IP地址和端口号等信息发送给客户端。因此，在启动NFS服务之前，还需要顺带重启并启用rpcbind服务程序，并将这两个服务一并加入开机启动项中。
```markdown
# systemctl restart rpcbind
# systemctl enable rpcbind
# systemctl start nfs-server 
# systemctl enable nfs-server
```


##### 客户端配置

使用showmount命令查询NFS服务器的远程共享信息
		-e 显示NFS服务器的共享列表
		-a 显示本机挂载的文件资源的情况
		-v 显示版本号
        showmount -e 10.65.2.22

在NFS客户端创建一个挂载目录。使用mount命令并结合-t参数，指定要挂载的文件系统的类型，并在命令后面写上服务器的IP地址、服务器上的共享目录以及要挂载到本地系统（即客户端）的目录。
```markdown
# mkdir /nfsfile
# mount -t nfs 10.65.2.22:/nfsfile /nfsfile
```


