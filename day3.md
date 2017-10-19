### docker学习笔记3
#### 一、将镜像保存(push)到私有仓库。
* push到仓库时需要先登录。
```
#保存镜像到文件
[root@localhost ~]# docker login --help

Usage:	docker login [OPTIONS] [SERVER]

Log in to a Docker registry.

Options:
      --help              Print usage
  -p, --password string   Password
  -u, --username string   Username
[root@localhost ~]# docker login -u **** -p **** registry.****.com
Login Succeeded
#登录成功之后，会在~/.docker/config.json 保存创建一个文件，用来保存仓库的验证信息。
[root@localhost ~]# cat ~/.docker/config.json 
{
	"auths": {
		"registry.****.com": {
			"auth": "****"
		}
	}
}
```
* 注意：push之前需要把镜像的REPOSITORY改一下，使用 docker tag 命令完成。
* REPOSITORY的规则是 ：私有库域名/project/name:tag，例如：registry.xxx.com/vipproject/hello-world:1.0.1
* 如果不设置私有库的域名作为前缀，在你push的时候会默认push到公有仓库，当然共有仓库因为你没有验证会导致失败。
```
#错误情形1：没有给镜像设置project导致提交镜像时提交到了root目录被拒绝。
[root@localhost ~]# docker push webapp:1.0.2 
Error response from daemon: You cannot push a "root" repository. Please rename your repository to docker.io/<user>/<repo> (ex: docker.io/<user>/webapp)

#错误情形2：给镜像设置了project，但是没有设置域名，导致提交到公有库(docker.io)被拒绝。
[root@localhost ~]# docker tag 7b810c81aa2a vipproject/webapp:1.0.2
[root@localhost ~]# docker push vipproject/webapp:1.0.2 
The push refers to a repository [docker.io/vipproject/webapp]

#修改镜像名格式为：namespace/project/name:tag。
[root@localhost ~]# docker tag 7b810c81aa2a registry.xxx.com/vipproject/webapp:1.0.2
#再次提交镜像到私有库。
[root@localhost ~]# docker push registry.xxx.com/vipproject/webapp:1.0.2 
The push refers to a repository [registry.xxx.com/vipproject/webapp]
88aed28da0da: Pushed 
f818831208cc: Pushed 
c659aa2f8133: Pushed 
304bd43dfb47: Pushed 
f4afc468f101: Pushed 
d3b309921607: Pushed 
5b6ca6b17923: Pushed 
d575b6d13cac: Pushed 
c2dca236d8e6: Pushed 
d4417cb76edb: Pushed 
0dc1ec77adb3: Pushed 
a75caa09eb1f: Pushed 
1.0.2: digest: sha256:38c749c3f9a770e2db558f77938832d9f1bb42be9c79ff1940c33d98f55e5529 size: 2840
#成功。
```
```

#更多说明请看帮助手册。
[root@localhost ~]# docker push --help
[root@localhost ~]# docker tag --help

```