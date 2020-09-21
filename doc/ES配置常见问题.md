### ES配置常见问题

按着网上的乱七八糟的各种配制方法配置ES，结果启动的时候报了自检失败的错误

>ERROR: [X] bootstrap checks failed

这些错误都是因为某项检查自检没有通过而报的错，我自己的虚拟机出现的错误如下：

#### 1 "for elasticsearch process is too low, increase to at least [65536]"

原因：启动ES的用户权限过低，需要提升到65536。
解决方法：使用root用户，进入/etc/security/limits.conf，在最后加入`你的用户名 soft nofile 65536`和`你的用户名 hard nofile 65536`。保存后执行`sysctl -p`，重新启动ES即可。

#### 2 "system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk"

原因：Centos6不支持SeeComp，ES执行时进行检测，检测失败无法启动。
解决方法：修改conf/elasticsearch.yml，修改或者加入如下两条

```yml
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
```
#### 3 "max number of threads is too low, increase to at least [4096]；"

原因：当前用户最大线程数太低，需要提升到4096以上。
解决方法：进入/etc/security/limit.d/20-nproc.conf（可能不是20-xxx.conf，我的是90-nproc.conf），在里面加入`* soft nproc 4096`(" * "可以为指定的用户名)，重启后可以用`ulimit -a`来查看线程是否改为4096。

#### 4 “max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]”

原因：最大虚拟内存过小，需要提升到262144以上。
解决方法：进入/etc/sysctl.conf ，加入`vm.max_map_count=262144`，然后执行sysctl -p即可。

PS：ES要求的东西真多，挨个修改了这堆问题终于是能跑起来了。

#### 5 "max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]"

原因：每个进程最大同时打开文件数太小，可通过下面2个命令查看当前数量

解决方法：修改/etc/security/limits.conf文件，增加配置，用户退出后重新登录生效

```
*               soft    nofile          65536
*               hard    nofile          65536
```

#### 6 "max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]"

原因：

解决方法：

修改/etc/sysctl.conf文件，增加配置vm.max_map_count=262144

```linux
vi /etc/sysctl.conf
sysctl -p
```

#### 7 "集群部署备份后恢复NoSuchFileFoundException"

原因：集群的备份文件夹根目录必须是共享文件夹

解决方法：

1、安装NFS与RPC

```shell
yum install -y  nfs-utils
yum install -y rpcbind
```

2、设置服务并设为开机启动

```shell
systemctl start rpcbind
systemctl enable rpcbind
systemctl start nfs-server
systemctl enable nfs-server
```

3、修改配置

```shell
vi /etc/exports

内容:
	/home/galaxy/es/backup 192.168.245.0/24(rw)
	
systemctl reload nfs 
```

4、客户端配置

```shell
showmount -e 192.168.245.128 #查看nfs共享信息
vim /etc/fstab

内容:
192.168.245.128:/public  /mnt/public       nfs    defaults 0 0

mount -a
df -Th #检查完成情况
[[```]]