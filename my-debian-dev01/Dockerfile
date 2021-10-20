#########
#
# docker build -f Dockerfile -t hexiaoyuan/my-debian-dev01:latest .
#
# 基于 debian-11+supervisor+sshd 用于开发的基础环境
#
# 注意：使用时不要apt更新系统，直接升级新的docker-image才对，因此个人开发环境
# 内容和工具请自己安装到自己目录下而不要安装到系统上。
#
#########
#
#
# 为了方便测试可以开启个临时环境进入测试用:
# docker run --rm -it -e container=docker -e LANG=C.UTF-8 --tmpfs /tmp --tmpfs /run --tmpfs /run/lock -v /sys/fs/cgroup:/sys/fs/cgroup:ro -p 60102:22 --hostname alice01 debian:11 bash
#
#
#########


FROM debian:11 AS debian-base

ENV container docker
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8
ENV DEBIAN_FRONTEND noninteractive

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# 替换仓库
RUN sed -i 's#deb.debian.org#mirrors.163.com#' /etc/apt/sources.list && sed -i 's#security.debian.org#mirrors.163.com#' /etc/apt/sources.list

# Install required tools ...
RUN apt-get -qq update && apt-get -qq -y dist-upgrade  \
    && apt-get -y -qq --no-install-recommends --no-install-suggests install \
        apt-utils ca-certificates apt-transport-https gnupg openssl curl wget jq \
    && apt-get -y -qq --no-install-recommends --no-install-suggests install \
        sudo supervisor openssh-server tmux lsof vim lsb-release \
        zsh bc iputils-ping iproute2 htop elinks less zip unzip git \
        python3 python3-pip python3-dev \
    && apt-get -y -qq --no-install-recommends --no-install-suggests install \
        build-essential patch cmake clang libclang-dev pkg-config libssl-dev 
##
# clean if you want ...
RUN apt-get -qq clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
#
# Configure sudo.
RUN ex +"%s/^%sudo.*$/%sudo ALL=(ALL:ALL) NOPASSWD:ALL/g" -scwq! /etc/sudoers
#
# Configure SSHD.
#RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
RUN bash -c 'install -m755 <(printf "#!/bin/sh\nexit 0") /usr/sbin/policy-rc.d' \
    && ex +'%s/^#\zeListenAddress 0.0.0.0/\1/g' -scwq /etc/ssh/sshd_config \
    && ex +'%s/^#\zeHostKey .*ssh_host_.*_key/\1/g' -scwq /etc/ssh/sshd_config \
    && ex +'%s/^#\zeUseDNS/\1/g' -scwq /etc/ssh/sshd_config \
    && sed -i 's!^#PasswordAuthentication yes!PasswordAuthentication no!' /etc/ssh/sshd_config
#
RUN RUNLEVEL=1 dpkg-reconfigure openssh-server
#RUN ssh-keygen -A -v
#RUN update-rc.d ssh defaults
#
# 删除掉生成的key在系统第一次启动时自动重新生成
RUN rm -f /etc/ssh/ssh_host_*key*
#
#
# 系统启动入口
COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN  chmod +x /docker-entrypoint.sh
COPY 55-supervisord.conf /etc/supervisor/conf.d/55-supervisord.conf
#
#
# Setup the default user.
RUN groupadd --gid 1000 alice
#  #  # --groups 'root,sudo,docker' 
RUN useradd --create-home --system -d /home/alice -s /bin/zsh --uid 1000 --gid 1000 --groups 'root,sudo' alice
RUN echo 'alice:passw0rd' | chpasswd
USER alice
WORKDIR /home/alice
#
# Generate and configure user keys.
USER alice
#RUN ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
COPY --chown=alice:alice "./authorized_keys" /home/alice/.ssh/authorized_keys
# Install oh-my-zsh
#RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended
#
# 用户自定义启动脚本
COPY --chown=alice:alice auto.sh /home/alice/.local/auto.sh
# RUN  chmod +x /home/alice/.local/auto.sh
#
USER root
EXPOSE 22
CMD ["/docker-entrypoint.sh"]

