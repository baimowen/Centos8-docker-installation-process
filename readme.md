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
$ yum clean all && yum makecache
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

安装所需的软件包。yum-utils 提供了 yum-config-manager ，并且 device mapper 存储驱动程序需要 device-mapper-persistent-data 和 lvm2。使用阿里云源

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
$ sudo docker -v
$ sudo docker run hello-world
```



5. **启动docker服务并设置开机自启**

```bash
$ sudo systemctl start docker
$ sudo systemctl enable docker-service
```





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



