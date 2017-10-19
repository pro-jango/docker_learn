### docker学习笔记2
#### 一、docker镜像操作
* 注意：把系统升级到centos7才可以安装docker。
* 查看当前系统中docker的版本号
```
#保存镜像到文件
[root@localhost ~]# docker save -o /home/xl_dev/mmm.tar golang:latest
#从文件导入到docker镜像列表
[root@localhost ~]# docker load < /home/xl_dev/mmm.tar
#删除名字叫golang的镜像。
[root@localhost ~]# docker rmi golang
[root@localhost test]# docker load < /home/xl_dev/mmm.tar
Loaded image: golang:latest
#多次导入不会多次出现
```

#### 二、docker容器操作
* 1、镜像和容器的区别是，镜像是类，而容器是实例化的对象。一个类可以有多个实例化的对象，也就是说有一个镜像时，可以开启多个容器。
```
#列出当前正在运行的容器。
[root@tw06873s6 ~]# docker ps -a
CONTAINER ID        IMAGE                                    COMMAND                  CREATED             STATUS                    PORTS                    NAMES
d9195b621c2e        web-app                                  "go-wrapper run"         About an hour ago   Up About an hour          0.0.0.0:8081->8080/tcp   test2
8b6ca4d67d78        web-app                                  "go-wrapper run"         About an hour ago   Up About an hour          0.0.0.0:8080->8080/tcp   test
cdd59bb9a1cc        golang:latest                            "-d"                     2 hours ago         Created                                            naughty_bose
04ea0aed7273        golang:latest                            "bash"                   2 hours ago         Exited (0) 2 hours ago                             hopeful_bardeen
de6130b3deb2        registry.xunlei.cn/xlvip/openvip:0.3.0   "/opt/openvip/xlvip.b"   19 hours ago        Exited (1) 19 hours ago                            boring_kilby
#停止名字为test的容器，此时的停止不是终止，容器还在列表之中。
[root@tw06873s6 ~]# docker stop test
#启动名字为test的容器
[root@tw06873s6 ~]# docker start test
#重启名字为test的容器
[root@tw06873s6 ~]# docker restart test
#除了使用名字，还可以使用容器id来操作，例如：
#重启容器id为d9195b621c2e的容器。
[root@tw06873s6 ~]# docker restart d9195b621c2e
```
* 2、删除容器，注意和停止容器是不同的，这个操作会把容器从列表中删除。
```
#根据名字删除容器
[root@localhost hello]# docker rm hopeful_bardeen
#根据容器id删除容器
[root@localhost hello]# docker rm cdd59bb9a1cc
```

### 三、使用Dockerfile创建容器。
* 参考 ：https://linux.cn/article-8113-1.html
* 本例我们使用go写一个简单的web应用，并把该应用打包到我们的自定义容器中，来介绍Dockerfile的用法。
* 1、创建一个main.go文件，里面放入简单的web应用代码，并把端口绑定到8080.
```
[root@localhost ~]# mkdir web-app 
[root@localhost ~]# cd web-app 
[root@localhost ~]# vi main.go
package main
import (
    "fmt"
    "net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello %s", r.URL.Path[1:])
}
func main() {
    http.HandleFunc("/World", handler)
    http.ListenAndServe(":8080", nil)
}
```
* 2、创建一个 dockerfile 文件，它会告诉 docker 如何容器化我们的 web 应用
* 命令参考：http://www.cnblogs.com/fengzheng/p/5181222.html
```
[root@localhost ~]# vi Dockerfile
# 得到最新的 golang docker 镜像
FROM golang:latest
# 在容器内部创建一个目录来存储我们的 web 应用，接着使它成为工作目录。
RUN mkdir -p /go/src/web-app
WORKDIR /go/src/web-app
# 复制 web-app 目录到容器中
COPY . /go/src/web-app
# 下载并安装第三方依赖到容器中
RUN go-wrapper download
RUN go-wrapper install
# 设置 PORT 环境变量
ENV PORT 8080
# 给主机暴露 8080 端口，这样外部网络可以访问你的应用
EXPOSE 8080
# 告诉 Docker 启动容器运行的命令
CMD ["go-wrapper", "run"]
```
* 注意1：CMD命令只能有一个，因为docker会用该命令的开启代表运行成功，如果文件中有多个CMD命令，则只会执行最后一个。
* 注意2：CMD命令不能是 service nginx start 这种及时退出的命令，因为这个命令执行完，控制权又交回给了容器，容器认为命令执行完成，然后就退出容器了。解决方法是使用nginx的非守护进程模式开启，例如：CMD ["nginx", "-g", "daemon off;"]
* 3、使用下面的命令构建你的 Go web-app，你会在成功构建后获得确认。
```
[root@localhost ~]# docker build -t web-app .
# -t 构建的镜像名字，通过:来区分tag
#  . 代表当前目录，是指Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径。

Sending build context to Docker daemon 3.072 kB
Step 1 : FROM golang:latest
 ---> 99e596fc807e
*****************************
Removing intermediate container 4c3dd4e855a6
Successfully built 07e5cd22caa1

#生成成功，接下来我们启动这个容器：
[root@localhost ~]# docker run -p 8080:8080 --name="test" -d web-app
d9195b621c2ec548750fea9bef4e976f6f4cbe2a69268901e40e4766741b007d
-p 匹配镜像内的网络端口号 支持格式：ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort。 
-d  后台运行容器，在后台运行容器，并且打印出容器ID
-t 表示让Docker分配一个伪终端并绑定到容器的标准输入上
-i 表示让容器的标准输入保持打开，-i -t 一般用来运行操作系统的命令时用。
-d 表示以守护方式打开（即非交互模式，后台运行），一般做服务器时使用。
--rm 告诉Docker CLI一旦容器退出，就自动发起一个docker rm命令。那样，不会留下任何东西，--rm 一般用来执行一次操作时使用，比如打包golang。

```
进入 http://localhost:8080/World 浏览你的 web 应用。你已经成功容器化了一个可重复的/确定性的 Go web 应用。使用前面介绍的命令来启动、停止并检查容器的状态。

#### 四、给docker配置hosts
```
[root@localhost ~]# docker run --add-host biaoge-ops:192.168.0.1 centos cat /etc/hosts
127.0.0.1   localhost
::1 localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
192.168.0.1 biaoge-ops
10.0.0.3    6ff3ea7114b4
```

* 1、利用docker编译GO应用程序。
* 参考文档：
> https://cr.console.aliyun.com/?spm=5176.100239.blogcont29941.12.yxm454#/imageDesc/1215/detail
