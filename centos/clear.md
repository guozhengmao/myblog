## 关于centos磁盘清理

> **起因**<br>
> 前段时间买了一个腾讯云的服务器，因为平时忙没时间瞎折腾，今天突然想起来，想写一个爬虫的项目，在用jenkins的时候突然发现创建不了项目，报错的原因jenkins无法写入到/var/lib/jenkins/jobs

以前我也用jenkins构建过项目，没有遇到过这种问题，我的第一反应是jenkins没有这个文件的读写权限，后来想想好像不是，然后百度，大部分人都是说centos清除cache。

在命令行中我输入和yum有关的指令都会提示我Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
我按照百度所说的的一下两条指令尝试了一下

```
yum clean all   
yum makecache
```
有提示了我另一个错误

```
One of the configured repositories failed (未知),
 and yum doesn't have enough cached data to continue. At this point the only
 safe thing yum can do is fail. There are a few ways to work "fix" this:
     1. Contact the upstream for the repository and get them to fix the problem.
     2. Reconfigure the baseurl/etc. for the repository, to point to a working
        upstream. This is most often useful if you are using a newer
        distribution release than is supported by the repository (and the
        packages for the previous distribution release still work).
     3. Disable the repository, so yum won't use it by default. Yum will then
        just ignore the repository until you permanently enable it again or use
        --enablerepo for temporary usage:
            yum-config-manager --disable <repoid>
     4. Configure the failing repository to be skipped, if it is unavailable.
        Note that yum will try to contact the repo. when it runs most commands,
        so will have to try and fail each time (and thus. yum will be be much
        slower). If it is a very temporary problem though, this is often a nice
        compromise:
            yum-config-manager --save --setopt=<repoid>.skip_if_unavailable=true
Cannot find a valid baseurl for repo: base/7/x86_64
```

这个时候我整个人都不好了，大概的看了一下，应该是磁盘没有空间了，这个时候我又去找centos如何去清理磁盘，找到了一个办法。就是从根目录开始使用du -sh *去查看文件占用磁盘的空间

```
[root@VM_0_15_centos /]# du -sh *
0       bin
142M    boot
800K    data
0       dev
38M     etc
84M     home
0       lib
0       lib64
16K     lost+found
4.0K    media
4.0K    mnt
8.0K    opt
du: cannot access ‘proc/16149/task/16149/fd/4’: No such file or directory
du: cannot access ‘proc/16149/task/16149/fdinfo/4’: No such file or directory
du: cannot access ‘proc/16149/fd/4’: No such file or directory
du: cannot access ‘proc/16149/fdinfo/4’: No such file or directory
du: cannot access ‘proc/16150’: No such file or directory
du: cannot access ‘proc/16151’: No such file or directory
0       proc
171M    root
396K    run
0       sbin
4.0K    srv
0       sys
76K     tmp
2.9G    usr
46G     var
```
这是我根目录的占用情况发现var这个文件夹占用46g的空间，难怪jenkins创建不了项目

然后再到var这个文件夹下执行du -sh *就这样一直执行下去，最后在/var/log/jenkins这个文件夹下发现了两个20g+的日志文件


```
[root@VM_0_15_centos jenkins]# du -sh *
23G     jenkins.log
20G     jenkins.log-20180410
155M    jenkins.log-20180411
23M     jenkins.log-20180413
15M     jenkins.log-20180414
14M     jenkins.log-20180415
30M     jenkins.log-20180416
```
最后使用这两条之类去清空这两个文件

```
>jenkins.log
>jenkins.log-20180410
```
最后来看看磁盘的占用情况


```
[root@VM_0_15_centos jenkins]# du -sh *
0       jenkins.log
0       jenkins.log-20180410
155M    jenkins.log-20180411
23M     jenkins.log-20180413
15M     jenkins.log-20180414
14M     jenkins.log-20180415
30M     jenkins.log-20180416
```
磁盘终于被清理干净了。