---
title:      kaldi sge集群和nfs网络文件系统
author:     liuyan
catalog:    true
tags:
  - kaldi
  - 语音识别
  - sge
  - nfs
  - 工具
date:       2018-09-09 16:42:26
urlname:
categories: 语音识别
---

### sge安装

#### qmaster sge安装
```shell
install_rpm.sh                    #qmaster节点上运行
install_qmaster.sh <admin-user>   #qmaster节点上运行
```

#### 使用nfs将qmaster上的sge部署到exec节点

- 在qmaster和exec节点安装nfs。需要保证各机器上的nfs版本一致。
```shell
sudo yum -y install nfs-utils   #安装
sudo yum -y upgrade nfs-utils   #更新
rpm -qa | grep nfs-utils        #查看版本
```

<!-- more -->

- qmaster节点配置nfs。在qmaster节点的/etc/exports中添加 `/opt/sge 10.40.0.0/16(rw,root_squash)`，其中10.40是exec节点的ip前缀。修改 `/etc/sysconfig/nfs`中的参数RPCNFSDCOUNT=64，将线程数设为64. 在文件`/proc/net/rpc/nfsd`中可以查看线程数。这样qmaster节点上的/opt/sge文件可以共享给所有ip以10.40开头的exec。
```shell
sudo systemctl start rpcbind.service
sudo systemctl start nfs.service        #启动服务
sudo exportfs -ra                       #使配置生效
sudo systemctl status nfs.service       #查看status
```

- exec节点挂载nfs。nfs日志文件放在 `/var/log/message` 和 `/var/log/cron`中，出现故障的时候可查看日志。
```shell
sudo mount -t nfs -o rw,vers=3,acdirmin=5,acdirmax=8,hard,proto=tcp xx.xx.xx.xx:/opt/nfs/train1 /opt/nfs/train1		#在exec挂载qmaster的目录，挂载之前exec和qmaster目录都需要存在。xx.xx.xx.xx是qmaster的ip
nfsstat -m          #查看nfs版本
umount -f           #卸载exec上的nfs，可添加-l强制卸载
```

- 错误解决。qmaster节点上启动服务`sudo systemctl start rpcbind.service`时出现问题`A dependency job for rpcbind.service failed. See 'journalctl -xe' for details.`主要是因为ipv6被禁用了，打开`/etc/systemd/system/sockets.target.wants/rpcbind.socket`，注释掉`ListenStream=[::]:111`即可。

#### 启动sge服务
- 在所有qmaster和exec节点执行下面的操作：
```shell
qconf -ah $node_name
qconf -as $node_name
```

- on exec node run:
```shell
install_deps.sh
install_rpm.sh
install_execd.sh
```

- start on boot
```shell
systemctl enable sgemaster.p6444            #启动qmaster上的sge
#sudo /etc/init.d/sgeexecd.p6444 start  
systemctl enable sgeexecd.p6444
#sudo /etc/init.d/sgemaster.p6444 start 
systemctl status sgeexecd.p6444             #查看sge状态
```

#### keeping grid stable
保证集群中内存不会崩，当内存接近使用完的时候自动杀死当前占用内存最多的非系统应用。
```shell
sudo bash
yum install -y subversion
svn cat https://svn.code.sf.net/p/kluster/code/trunk/scripts/sbin/mem-killer.pl > /sbin/mem-killer.pl
chmod +x /sbin/mem-killer.pl
cd /etc/init.d/ 
svn cat https://svn.code.sf.net/p/kluster/code/trunk/scripts/etc/init.d/mem-killer >mem-killer
chmod +x mem-killer 
chkconfig --add mem-killer
systemctl start mem-killer
systemctl status mem-killer
```

#### 配置
qconf -me dx-ai-speechoffline-training1
```shell
complex_values        ram_free=188G,gpu=2   #没有gpu不写gpu，配置共享内存
```

cephfs挂载
```shell
sudo fusermount -uz /opt/meituan/cephfs 
sudo ceph-fuse -c /etc/ceph/ceph.conf -n client.ceph-speech --client_mountpoint /ceph-speech /opt/meituan/cephfs
```

### sge使用
```shell
qhost -q        #查看sge节点信息
qstat -u user   #按用户查看    
qstat -u "*"    #查看所有任务
qstat -f        #查看所有任务 
qstat -j jobid  #按任务id查看 
qdel jobID      #删除某个任务
qdel -u user    #删除某个用户的所有任务

任务状态
qw      #表示等待状态 
eqw     #投递任务出错 
r       #表示任务正在运行 
dr      #节点挂了之后，删除任务就会出现这个状态，只有节点重启之后，任务才会消失
```

df -h	查看nfs是否配置成功
working on centos 6/7
以上所有安装脚本以及相应的资源文件无法提供

参考：
[Parallelization in Kaldi](http://kaldi-asr.org/doc/queue.html)
