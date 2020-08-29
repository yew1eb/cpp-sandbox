
#####
## DockerFile编写最佳实践：
##    https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
##
#####
FROM ubuntu:20.04
LABEL maintainer="zhouhai<yew1eb@gmail.com>"

# tzdata设置为非交互方式
ENV DEBIAN_FRONTEND noninteractive

ARG workdir=/workspaces

# 将ubuntu默认的软件源替换成国内阿里云
RUN sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list && \
    apt clean && \
    apt update

# 安装必要的工具
RUN apt install -y --no-install-recommends \
    sudo \
    build-essential ninja-build \
    cmake  \
    gdb gdbserver \
    rsync \
    net-tools \
    vim \
    git \
    wget \
    curl \
    ca-certificates

# ssh配置
RUN apt install -y openssh-server
RUN mkdir -p /var/run/sshd && \
    mkdir -p /root/.ssh

# 修改 root 的密码为 root
RUN echo 'root:root' | chpasswd
RUN sed -ri 's/^#?PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config

# 允许无密码的账号登录
RUN sed -ri 's/^#?PermitEmptyPasswords\s+.*/PermitEmptyPasswords yes/' /etc/ssh/sshd_config

# SSH login fix. Otherwise user is kicked off after login
RUN sed -i 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' /etc/pam.d/sshd

ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile



# 创建一个个人账号
RUN useradd -ms /bin/bash zhouhai
RUN echo 'zhouhai:zhouhai' | chpasswd
RUN passwd -d zhouhai
RUN echo 'zhouhai  ALL=(ALL:ALL) ALL' >> /etc/sudoers

# c++开发环境
RUN apt install -y g++ gcc

# golang开发环境
ARG go_download_url=https://studygolang.com/dl/golang/go1.14.7.linux-amd64.tar.gz
RUN curl -fsSL $go_download_url -o golang.tar.gz \
    && tar -C /usr/local -xzf golang.tar.gz \
    && rm -f golang.tar.gz
ENV GOPROXY=https://goproxy.cn
ENV GOSUMDB="sum.golang.google.cn"
ENV GOROOT=/usr/local/go
ENV GOPATH=${workdir}/go
ENV PATH $GOPATH/bin:${GOROOT}/bin:$PATH
RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH" &&\
    echo "export GOROOT=${GOROOT}" >> /etc/profile && \
    echo "export GOPATH=${workdir}/go" >> /etc/profile  && \
    echo "export PATH=${GOROOT}/bin:${PATH}" >> /etc/profile  


# 删除 apt update 产生的缓存文件
# 因为 docker 的文件系统是层文件系统，上一个层中缓存有apt-get update的结果，
# 那么下次 Dockerfile 运行时就会直接使用之前的缓存，
# 这样 docker 中的 apt 软件源就不是最新的软件列表了，将会带来缓存过期的问题。
# 并且这些缓存将占用不少空间，导致最终生成的image非常庞大，
# 而这些垃圾文件是我们最终的image中无需使用到的东西，我们应当在Docker构建过程中予以删除。
RUN apt clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# for gdbserver
EXPOSE 2000


USER zhouhai
VOLUME ${workdir}
WORKDIR ${workdir}

# 22 for ssh server.
EXPOSE 22

CMD sudo service ssh start
