## 下载离线软件包及依赖

1. 安装 yum-utils

   ```
   yum install -y yum-utils \
     device-mapper-persistent-data \
     lvm2
   ```

2. 配置 docker-ce yum 源

   官方：

   ```
   yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
   ```

   阿里：

   ```
   yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   ```

3. 下载 createrepo 及依赖

   createrepo 软件可以为本地 yum 库生成索引。

   创建 yum/local 文件夹

   ```
   $ mkdir -p yum/local
   ```

   下载 createrepo 软件包及其依赖

   ```
   repotrack -a x86_64 -p yum/local createrepo
   ```

4. 下载 libgudev1 和 systemd-sysv

   ```
   repotrack -a x86_64 -p yum/local libgudev1
   repotrack -a x86_64 -p yum/local systemd-sysv
   repotrack -a x86_64 -p yum/local audit
   ```

5. 下载 docker-ce 及依赖

   ```
   repotrack -a x86_64 -p yum/local docker-ce
   ```

6. 打包 yum 软件

   ```
   $ tar -zcvf docker-ce-yum.tgz yum/
   ```

## 安装离线包

1. 解压

   ```
   tar -zxvf docker-ce-yum.tgz -C /root/
   ```

1. 进入 yum/local 文件夹

   ```
   cd /root/yum/local
   ```

1. 安装 createrepo

   ```
   rpm -ivh createrepo-0.9.9-26.el7.noarch.rpm
   ```

   可能会提示 deltarpm 和 python-deltarpm 版本不够，可以升级这些包

   ```
   # rpm -Uvh deltarpm-3.6-3.el7.x86_64.rpm
   # rpm -Uvh python-deltarpm-3.6-3.el7.x86_64.rpm
   ```

1. 配置本地源文件

   添加文件/etc/yum.repos.d/CentOS-Local.repo

   ```
   vi /etc/yum.repos.d/CentOS-Local.repo
   ```

   写入以下内容

   ```
   [Local]
   name=Local Yum
   baseurl=file:///root/yum/
   gpgcheck=0
   ```

1. 生成 yum 源的索引及缓存

   ```
   # createrepo /root/yum
   # yum makecache
   ```

1. 安装 docker-ce

   ```
   yum --disablerepo=* --enablerepo=Local install docker-ce
   ```

   17.03.2:

   ```
   yum --disablerepo=* --enablerepo=Local --setopt=obsoletes=0 install docker-ce-17.03.2.ce
   ```

1. 安装 docker-compose

   复制 docker-compose-Linux-x86_64 文件到/usr/bin 目录下，重命名为 docker-compose

1. 启动 Docker

   ```
   systemctl start docker
   ```

1. 设置开机启动

   ```
   systemctl enable docker
   ```

## nexus yum 软件源配置

`vi /etc/yum.repos.d/nexus.repo`

```
[nexusrepo]
name="Nexus Repository"
baseurl="http://192.168.10.200:8081/repository/yum-hosted/"
gpgcheck=0
```

`vi /etc/yum.repos.d/CentOS-Base.repo`

```
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

#released updates
[updates]
name=CentOS-$releasever - Updates
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

`yum makecache`
