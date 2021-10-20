# README

my-debian-dev01 是个人开发使用的基础 docker-image, 方便快速搭建开发环境. 基于debian稳定版本定制.

基于 debian:11 加上 supervisor+sshd 和一些自己常用的开发工具，建议搭配使用 vscode 装 ms-vscode-remote.remote-ssh 插件一起用.

通常建议一个项目一个本地容器，开发环境独立互不干扰.

+ 用户密码 alice/passw0rd; 默认zsh，请安装 oh-my-zsh 到自己目录;
+ 时区 UTC，编码 C.UTF-8;

## 编译发布

```sh
touch  authorized_keys
docker build -f Dockerfile -t hexiaoyuan/my-debian-dev01:latest .
docker tag hexiaoyuan/my-debian-dev01:latest hexiaoyuan/my-debian-dev01:v20211020
docker push hexiaoyuan/my-debian-dev01 -a
```

## 本地开启一个容器实例

```sh
docker pull hexiaoyuan/my-debian-dev01
docker run -it --rm --init -p 60101:22 --name alice01 --hostname alice01 hexiaoyuan/my-debian-dev01

#### 记录一些参数，以后在测试
docker run -d \
  --init \
  --privileged `#optional` \
  -v /var/run/docker.sock:/var/run/docker.sock `#optional` \
  -v /sys/fs/cgroup:/sys/fs/cgroup:ro `#optional`\
  --cap-add=SYS_ADMIN --env container=docker `#optional` \
  --tmpfs /tmp --tmpfs /run --tmpfs /run/lock `#optional`\
  --shm-size="1gb" `#optional` \
  -p 60101:22 \
  --name alice01 --hostname alice01 hexiaoyuan/my-debian-dev01

#
# 开发测试时：
docker run --rm -it \
  -e container=docker -e LANG=C.UTF-8  \
  --tmpfs /tmp --tmpfs /run --tmpfs /run/lock \
  -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
  -p 60102:22 --hostname alice01 debian:11

```

## 进入

```txt
$ docker exec -it alice01 zsh

alice01# passwd alice
New password:
Retype new password:
passwd: password updated successfully
alice01#

##把需要的key加入：
# su - alice
$ vi /home/alice/.ssh/authorized_keys

##根据自己需求调整zsh的配置
$ vi ~/.zshrc

$ exit

```

## 远程

```sh
ssh alice@127.0.0.1 -p 60101
```

## 使用 volume 来保存数据(建议)

```sh
docker volume create vol_alice01_home
docker volume inspect vol_alice01_home

docker run -d --init \
  -p 60101:22 \
  --mount source=vol_alice01_home,target=/home \
  --name alice01 --hostname alice01 hexiaoyuan/my-debian-dev01:v20211020

docker exec -it alice01 zsh

```
