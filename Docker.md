---
title: Docker
tags: Docker,Command
grammar_cjkRuby: true
---


## Command list
>Command list and introduction

 1. FROM        指定Container base image 来源
 2. MAINTAINER: 维护者&作者<MAINTAINER Mail> 
 3. RUN         Linux Command 
 4. ENV         设置环境变量。 
 5. EXPOSE      暴露端口
 6. ADD         添加文件至Container内
 7. COPY        复制本地主机的 <src> （为Dockerfile所在目录的相对路径）到容器中的 <dest>
 8. VOLUME      挂载Volume 持久化存储文件
 9. ENTRYPOINT  Dockerfile仅可指定一个ENTRYPOINT，若指定多个，仅一个有效
 10. CMD         镜像构建容器后被调用。 
 11. WORKDIR    WORKDIR命令用于设置CMD指明的命令的运行目录。
 12. ONBUILD    配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。


PS:RUN先于CMD/ENTRYPOINTRUN命令覆盖CMD 
例：CMD["echo"] 而docker run CONTAINER_NAME echo foo则会运行echo foo，
而忽略CMD 而ENTRYPOINT则会传递RUN命令，
例：ENTRYPOINT ["echo"]docker run CONTAINER_NAME echo foo则会运行echo echo foo 输出为echo foo 

*****************************************************************


>Introduction Command  usage and example;
### FROM 

    用法：ubuntu:xenial or tag (daocloud.io/library:ubuntu or tag) 

### MAINTAINER

    # Usage: MAINTAINER [name]
    MAINTAINER Xiekers <im@xieke.org>

## ENV

    #usage:ENV KEY Volue
    ENV PHP 5.5
    用于设置环境变更
    使用此dockerfile生成的image新建container，可以通过 docker inspect CONTAINER ID  看到这个环境变量
    也可以通过在docker run时设置或修改环境变量

### ADD

    #Usage ADD <src> <dest>
    ADD index.php /var/www/html
    ADD http://xxx.xxx/xxx.zip /var/www/html    //会被下载并复制到Container
    ADD package.zip /var/www/html                 //会被解压出来


### COPY

    #usage COPY src /destination
    COPY index.php /var/www/html

    

### USER

    比如指定 memcached 的运行用户，可以使用上面的 ENTRYPOINT or CMD来实现:
    ENTRYPOINT ["memcached", "-u", "daemon"]
    更好的方式：
    ENTRYPOINT ["memcached"]
    USER daemon

## VOLUME

    #usage VOLUME ["<mountpoint>"]
    如:`VOLUME ["/data"]`
    创建一个挂载点用于共享目录
    
## WORKDIR

    #usage WORKDIR /path/to/workdir
    配置RUN, CMD, ENTRYPOINT 命令设置当前工作路径
    可以设置多次，如果是相对路径，则相对前一个 WORKDIR 命令
    比如:`WORKDIR /a WORKDIR b WORKDIR c RUN pwd`
    其实是在 /a/b/c 下执行 pwd

### USER

    比如指定 memcached 的运行用户，可以使用上面的 ENTRYPOINT or CMD来实现:
    ENTRYPOINT ["memcached", "-u", "daemon"]
    更好的方式：
    ENTRYPOINT ["memcached"]
    USER daemon
    SER命令用于设置运行容器的UID。
    # Usage: USER [UID]
    USER 751

### EXPOSE

    #Usage EXPOSEd <port>
    在docker使用--link来链接两容器时会用到相关端口


### CMD
>和RUN命令相似，CMD可以用于执行特定的命令。和RUN不同的是，这些命令不是在镜像构建的过程中执行的，而是在用镜像构建容器后被调用。

    #Usage 1: CMD application "argument", "argument", ..
    CMD "echo" "Hello docker!"


### ONBUILD

    #usage:ONBUILD [INSTRUCTION] 。


# Dockerfile Example
```

    FROM centos6-base
    MAINTAINER Xiekers <im@xieke.org>
    RUN ssh-keygen -q -N "" -t dsa -f /etc/ssh/ssh_host_dsa_key
    RUN ssh-keygen -q -N "" -t rsa -f /etc/ssh/ssh_host_rsa_key
    RUN sed -ri 's/session    required     pam_loginuid.so/#session    required     pam_loginuid.so/g' /etc/pam.d/sshd
    RUN mkdir -p /root/.ssh && chown root.root /root && chmod 700 /root/.ssh
    EXPOSE 22
    RUN echo 'root:redhat' | chpasswd
    RUN yum install -y yum-priorities && rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm && rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
    RUN yum install tar gzip gcc vim wget -y
    ENV LANG en_US.UTF-8
    ENV LC_ALL en_US.UTF-8
    CMD /usr/sbin/sshd -D
    #End

```

    FROM       php
    MAINTAINER Xiekers <im@xieke.org>
    #将index.php复制到容器内的/var/www目录下
    ADD        index.php /var/www/
    #对外暴露8080端口
    EXPOSE     8080
    #设置Volume挂载目录
    VOLUME ["/data"]
    # 设置容器默认工作目录为/var/www
    WORKDIR    /var/www/
    # 容器运行后默认执行的指令
    ENTRYPOINT ["php", "-S", "0.0.0.0:8080"]
    CMD service tomcat7 start                     #无效命令运行几秒钟之后，容器就会退出
    ENTRYPOINT service tomcat7 start       #无效命令运行几秒钟之后，容器就会退出
    #正确写法 CMD相同
    ENTRYPOINT service tomcat7 start && tail -f /var/lib/tomcat7/logs/catalina.out
    OR
    ENTRYPOINT ["/usr/sbin/sshd"]
    CMD ["-D"]