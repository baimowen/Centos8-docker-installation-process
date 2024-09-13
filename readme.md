# 更新yum源

1. **备份**

```bash
$ sudo rename '.repo' '.repo.bak' /etc/yum.repos.d/*.repo
```

2. **下载最新的repo文件**

```bash
$ wget https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo -O /etc/yum.repos.d/Centos-vault-8.5.2111.repo
$ wget https://mirrors.aliyun.com/repo/epel-archive-8.repo -O /etc/yum.repos.d/epel-archive-8.repo
```

3. **修改源地址**

```bash
$ sudo sed -i 's/mirrors.cloud.aliyuncs.com/url_tmp/g'  /etc/yum.repos.d/Centos-vault-8.5.2111.repo &&  sed -i 's/mirrors.aliyun.com/mirrors.cloud.aliyuncs.com/g' /etc/yum.repos.d/Centos-vault-8.5.2111.repo && sed -i 's/url_tmp/mirrors.aliyun.com/g' /etc/yum.repos.d/Centos-vault-8.5.2111.repo

$ sudo sed -i 's/mirrors.aliyun.com/mirrors.cloud.aliyuncs.com/g' /etc/yum.repos.d/epel-archive-8.repo

$ sudo sed -i 's/mirrors.cloud.aliyuncs.com/mirrors.aliyun.com/g'  /etc/yum.repos.d/Centos-vault-8.5.2111.repo 

$ sudo sed -i 's/mirrors.cloud.aliyuncs.com/mirrors.aliyun.com/g'  /etc/yum.repos.d/epel-archive-8.repo
```

4. **重建缓存**

```bash
$ sudo yum clean all && yum makecache
```





# SSH

```bash
$ sudo yum update && yum install openssh-server -y
$ sudo vim /etc/ssh/sshd_config 
$ sudo sed -i 's/#PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config
$ sudo sed -i '/^#Portt 22/s/^#//' /etc/ssh/sshd_config
$ sudo sed -i '/^#ListenAddress 0.0.0.0/s/^#//' /etc/ssh/sshd_config
$ sudo sed -i '/^#ListenAddress ::/s/^#//' /etc/ssh/sshd_config
$ sudo systemctl restart sshd
```

>  *使用Xshell远程连接时使用普通用户登录*





# 安装docker

1. **卸载旧版本组件**

```bash
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```



2. **设置仓库**

安装所需的软件包。`yum-utils` 提供了 `yum-config-manager` ，并且 `device mapper` 存储驱动程序需要 `device-mapper-persistent-data` 和 `lvm2`。使用阿里云源

```bash
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
$ sudo yum-config-manager \
  --add-repo \
  http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```



3. **前提准备**

直接安装docker会报错缺少containerd.io以及podman包冲突和runc版本冲突

安装container.io时，会下载一个podman的包，podman包依赖runc包，runc包要>=1.0.0-57 ，container.io会和centos8自带的runc版本包冲突

```bash
$ yum install https://download.docker.com/linux/fedora/30/x86_64/stable/Packages/containerd.io-1.2.13-3.2.fc30.x86_64.rpm
$ yum erase podman buildah -y
$ yum erase runc -y
```



4. **安装并验证**

```bash
$ sudo yum install docker-ce docker-ce-cli containerd.io
$ docker -v
$ docker run hello-world
```



5. **启动docker服务并设置开机自启**

```bash
$ sudo systemctl start docker
$ sudo systemctl enable docker-service
```



# 创建自己的镜像

1. 编写`DockerFile`

```dockerfile
# 使用官方 Python 
FROM python:3.6

# 复制到镜像的/src /code
ADD /src /code

# 设置工作目录
WORKDIR /code

# 将当前目录内容复制到工作目录
COPY . /app

# 安装依赖
RUN pip install --no-cache-dir -r requirements.txt

# 运行应用
CMD ["python", "/code/app.py"]
```



2. 创建`requirements.txt`


```bash
pip freeze > requirements.txt
```



3. 执行创建命令`docker build`

```bash
$ docker build -f DockerFile -t <镜像名称> .
```

> 注意代理！！！否则报错





# 启动容器

```bash
$ docker run -it [-d] [-v <主机目录>:<容器目录>] [-w <容器目录>] <镜像名称> <容器内运行的进程>
```

> `-i`、`-t` 参数一般组合使用
>
> `-d` 参数为后台运行，无`-d`参数退出容器后自动删除容器
>
> `-v $PWD/myapp:/usr/src/myapp` ：将主机中当前目录下的 myapp 挂载到容器的 /usr/src/myapp。
>
> `-w /usr/src/myapp` ：指定容器的 /usr/src/myapp 目录为工作目录。
>
> `[容器内运行的进程]` eg：python main.py、/bin/bash、/usr/sbin/init   | 容器内需保持一个进程运行，否则容器停止





# 卸载docker

列出你安装过的包

```bash
$ yum list installed | grep docker
```

移除安装包

```bash
$ sudo yum -y remove docker-engine.x86_64
```

删除所有的镜像，容器和卷

```bash
$ sudo rm -rf /var/lib/docker
```





# 一些报错 eg:Centos8



* docker build创建镜像报错`Temporary failure in name resolution`

> 无法解析域名

1. 检查主机网络设置

查看系统是否打开ip地址转发功能：

```bash
$ cat /proc/sys/net/ipv4/ip_forward
1
```

返回`0`则说明未打开，开启ip地址转发：

```bash
$ sudo sed -i '/^#net.ipv4.ip_forward = 1/s/^#//' /etc/sysctl.conf
```

保存修改后，重启系统或输入以下命令使修改生效：

```bash
$ sudo sysctl -p /etc/sysctl.conf
$ sudo systemctl restart network
```

2. 检查主机防火墙配置

查看防火墙状态（若防火墙为关闭状态，可跳过防火墙有关设置）：

```bash
$ sudo firewall-cmd --state
```

若返回`runging`，则防火墙为开启状态，查看防火墙是否开启ip地址转发（ip地址伪装）：

```bash
$ sudo firewall-cmd --query-masquerade
1
```


若返回`no`，则输入以下命令开启ip地址转发：

```bash
$ sudo firewall-cmd --add-masquerade --permanent
1
```


然后输入以下命令使修改生效：

```bash
$ sudo firewall-cmd --reload
```

3. 设置Docker指定DNS服务器

打开Docker相关设置文件（主机内），没有就新建一个，输入下列命令会打开或自动新建：

```bash
$ sudo vim /etc/docker/daemon.json
1
```


在文件中输入以下内容：

```json
{
	"dns": ["8.8.8.8","114.114.114.114"]
}
```

然后重启Docker：

```bash
$ sudo systemctl restart docker
```



此处报错参考：https://blog.csdn.net/qq_43743460/article/details/105648139





* ...
