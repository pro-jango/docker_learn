### docker学习笔记1
#### 一、安装docker
* 注意：把系统升级到centos7才可以安装docker。
* 查看当前系统中docker的版本号
```
[root@localhost ~]#docker version
Client:
 Version:         1.12.6
 API version:     1.24
 Package version: docker-1.12.6-32.git88a4867.el7.centos.x86_64
 Go version:      go1.7.4
 Git commit:      88a4867/1.12.6
 Built:           Mon Jul  3 16:02:02 2017
 OS/Arch:         linux/amd64

Server:
 Version:         1.12.6
 API version:     1.24
 Package version: docker-1.12.6-32.git88a4867.el7.centos.x86_64
 Go version:      go1.7.4
 Git commit:      88a4867/1.12.6
 Built:           Mon Jul  3 16:02:02 2017
 OS/Arch:         linux/amd64
```
如果没有安装docker，则需要执行安装。
```
[root@localhost ~]# yum install docker docker-client -y
#安装完成之后执行 docker version会发现client 已经存在，但是提示：
Cannot connect to the Docker daemon. Is the docker daemon running on this host?
#是因为没有开启docker 服务。
[root@localhost ~]# service docker start
Redirecting to /bin/systemctl start docker.service
[root@localhost ~]#docker version
#会显示Client 和Server的版本。
```
启动，停止，重启命令
```
[root@localhost ~]# service docker restart
Redirecting to /bin/systemctl restart docker.service
[root@localhost ~]# service docker stop
Redirecting to /bin/systemctl stop docker.service
[root@localhost ~]# service docker start
Redirecting to /bin/systemctl start docker.service
```
#### 二、镜像操作
* 1、镜像列表
```
[root@tw06873s6 ~]# docker images
REPOSITORY                                        TAG                 IMAGE ID            CREATED             SIZE
registry.xxx.com/vipproject/hello                  0.3.0               3f68de0483fc        5 weeks ago         210.5 MB

```
* 2、docker信息
```
[root@tw06873s6 ~]# docker info
Containers: 1 # 容器个数 Images: 22 # 镜像个数 Storage Driver: devicemapper # 存储驱动 Pool Name: docker-8:17-3221225728-pool
```
* 3、删除一个或者多个镜像
```
[root@tw06873s6 ~]# docker rmi xxx

```

### 三、私有仓库操作
* 1、下载私有仓库
* 下载时遇到了一个https的问题：
* 解决方法是在/etc/docker/daemon.json中添加 insecure-registries 列表
```
[root@localhost ~]# docker pull registry.xxxx.com/xxx/xxx:0.3.0 
Trying to pull repository registry.xxxxx.com/xxx/xxx ... 
Get https://registry.xxx.com/v1/_ping: x509: certificate is valid for *.xxx.com, xxx.com, not registry.xxx.com
报证书错误的问题。

[root@localhost ~]# vim /etc/docker/daemon.json
{
"insecure-registries" : ["registry.xxxx.com"]
}

[root@localhost ~]# systemctl restart docker
```
* 2、运行私有镜像
```
[root@localhost ~]# docker run registry.xxxx.com/xxxx/xxxx /opt/hello/vip.b
Unable to find image 'registry.xxxx.com/xxxx/xxxx:latest' locally
Trying to pull repository registry.xxxx.com/xxxx/xxxx ... 
Pulling repository registry.xxxx.com/xxxx/xxxx
/usr/bin/docker-current: Error: image xxxx/xxxx:latest not found.
See '/usr/bin/docker-current run --help'.
# 提示找不到latest版本的镜像，原来是镜像在运行时需要加上版本号。
[root@localhost ~]# docker run registry.xxxx.com/xxxx/xxxx:0.3.0 /opt/hello/vip.b
#运行成功
```
* 3、安装国内阿里云的镜像
* >https://yq.aliyun.com/articles/29941
```
[root@localhost ~]# vi  /etc/docker/daemon.json
{
    "registry-mirrors": ["<your accelerate address>"]
}
```

#### 四、高级应用
```
- 当利用 docker run 来创建容器时，Docker 在后台运行的标准操作包括： 
- 检查本地是否存在指定的镜像，不存在就从公有仓库下载 
- 利用镜像创建并启动一个容器 
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层 
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去 
- 从地址池配置一个 ip 地址给容器 
- 执行用户指定的应用程序 
- 执行完毕后容器被终止
```

* 1、利用docker编译GO应用程序。
* 参考文档：
> https://cr.console.aliyun.com/?spm=5176.100239.blogcont29941.12.yxm454#/imageDesc/1215/detail
