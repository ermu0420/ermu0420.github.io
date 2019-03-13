---
layout: post
title: GitLib+Jenkins+Docker 持续集成方案
category: tool
tags: [GitLib，Jenkins,Docker]
keywords: GitLib，Jenkins,Docker,持续集成
---
# GitLib+Jenkins+Docker 持续集成方案

## 安装GitLib
[安装官方教程:https://www.gitlab.com.cn/installation/](https://www.gitlab.com.cn/installation/)
### 下载一些需要的依赖
~~~
sudo yum install -y curl policycoreutils-python openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd

sudo firewall-cmd --permanent --add-service=http
sudo systemctl reload firewalld
~~~
### 下载邮箱服务
~~~
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix

~~~
### 获取GitLib
~~~
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash

sudo EXTERNAL_URL="http://gitlab.example.com" yum install -y gitlab-ee
gitlab.example.com为自己网站地址以及端口例如
sudo EXTERNAL_URL="http://118.24.21.49:8088" yum install -y gitlab-ee
~~~
### 修改配置启动
~~~
vim  /etc/gitlab/gitlab.rb

gitlab-ctl reconfigure

gitlab-ctl restart
~~~
### 出现的问题
Job for postfix.service failed because the control process exited with error code. See "systemctl status postfix.service" and "journalctl -xe" for details.

修改 /etc/postfix/main.cf的设置  
~~~
inet_protocols = ipv4  
inet_interfaces = all  
~~~

## 安装Jenkins 
### 添加Jenkins库到yum库，下载Jenkins
~~~
wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
yum install -y jenkins
~~~
### 配置Jenkins的端口
~~~
 vi /etc/sysconfig/jenkins
~~~
### Root 用户启动
~~~
 vim /etc/sysconfig/jenkins
 
 JENKINS_USER="root"
 
 chown -R root:root /var/lib/jenkins
 chown -R root:root /var/cache/jenkins
 chown -R root:root /var/log/jenkins
~~~

### 启动
~~~
service jenkins start/stop/restart
~~~
初始密码在：/var/lib/jenkins/secrets/initialAdminPassword 


### 出错问题
JAVA 环境
vim  /etc/init.d/jenkins
candidates="
/usr/soft/jdk1.8.0_144/bin/java #此处为加入的java路径

## Docker
### 安装
[安装文档:https://docs.docker.com/install/](https://docs.docker.com/install/)<br>
1.移除老版本
~~~
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
~~~
2.安装需要的软件包
~~~
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
~~~
3.设置yum源
~~~
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
~~~
4.查看版本 并安装
~~~
yum list docker-ce --showduplicates | sort -r

yum install docker-ce # 直接下载
yum install <FQPN>  # 指定版本下载例如： yum install docker-ce-17.12.0.ce 
~~~
5.启动
~~~
systemctl start docker
systemctl enable docker
~~~


### 基本命令
- docker images # 所有镜像
- docker ps # 正在运行容器
- docker ps # 所有容器
- docker search # 搜索镜像
- docker pull # 拉取
- docker run # 运行一个容器
 -d 后台运行
 -i -t 可以交互式命令
- docker rm  # 删除容器
- docker rmi  # 删除镜像
- docker start # 重新启动
- docker logs # 容易起日志
- docker logs -f # tailf
- docker stop # 停止容器
- docker commit # 保存成一个新镜像
- docker push # 上传镜像

### Docker-Compose 
Windows:
PS C:\Windows\system32>[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
PS C:\Windows\system32>Invoke-WebRequest "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-Windows-x86_64.exe" -UseBasicParsing -OutFile $Env:ProgramFiles\Docker\Docker\resources\bin\docker-compose.exe
PS C:\Windows\system32>docker-compose --version

### Docker-Swarm
- docker swarm init --listen-addr 127.0.0.1:2377
- docker swarm join-token manager

### Docker Stack
- docker stack deploy -c docker-compose.yml [name]  # 部署
- docker stack ls # stack列表
- docker stack services [name] #  services  in stack
- docker stack rm [name]   # 删除stack
- docker stack ps [name]  # tasks in stack