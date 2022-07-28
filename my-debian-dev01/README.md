# README

my-debian-dev01 是个人开发使用的基础 docker-image, 方便快速搭建开发环境. 基于debian稳定版本定制.

基于 debian:11 加上 supervisor+sshd 和一些自己常用的开发工具，建议搭配使用 vscode 装 ms-vscode-remote.remote-ssh 插件一起用.

通常建议一个项目一个本地容器，开发环境独立互不干扰.

+ 用户密码 pete/passw0rd; 默认zsh，请安装 oh-my-zsh 到自己目录;
+ 时区 UTC，编码 C.UTF-8;

## 编译发布

```sh
touch  authorized_keys
docker build -f Dockerfile -t hexiaoyuan/my-debian-dev01:latest .
docker tag hexiaoyuan/my-debian-dev01:latest hexiaoyuan/my-debian-dev01:v20220728
docker push hexiaoyuan/my-debian-dev01 -a
```

## 本地开启一个容器实例

```sh
docker pull hexiaoyuan/my-debian-dev01
docker run -it --rm --init -p 60101:22 --name pete01 --hostname pete01 -e container=docker -e LANG=C.UTF-8  hexiaoyuan/my-debian-dev01

#### 记录一些参数，以后在测试
docker run -d --init    \
  --privileged                                         `#optional` \
  -v /var/run/docker.sock:/var/run/docker.sock         `#optional` \
  -v /sys/fs/cgroup:/sys/fs/cgroup:ro                  `#optional` \
  --cap-add=SYS_ADMIN                                  `#optional` \
  --env container=docker -e LANG=C.UTF-8               `#optional` \
  --tmpfs /tmp --tmpfs /run --tmpfs /run/lock          `#optional` \
  --shm-size="1gb"                                     `#optional` \
  -p 60102:22 --name pete01 --hostname pete01 hexiaoyuan/my-debian-dev01
#
# 发现 /tmp 导致 sshd 启不来.


#
# 开发测试时：
docker run --rm -it \
  -e container=docker -e LANG=C.UTF-8  \
  --tmpfs /tmp --tmpfs /run --tmpfs /run/lock \
  -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
  -p 60102:22 --hostname pete01 debian:11

```

## 进入

```txt
$ docker exec -it pete01 zsh

pete01# passwd pete
New password:
Retype new password:
passwd: password updated successfully
pete01#

##把需要的key加入：
# su - pete
$ vi /home/pete/.ssh/authorized_keys

##根据自己需求调整zsh的配置
$ vi ~/.zshrc

$ exit

```

## 远程

```sh
ssh pete@127.0.0.1 -p 60101
```

## 使用 volume 来保存数据(建议)

```sh
docker volume create vol_pete01_home
docker volume inspect vol_pete01_home
#
docker run -d --init \
  -p 60101:22 \
  -e container=docker -e LANG=C.UTF-8  \
  --mount source=vol_pete01_home,target=/home \
  --name pete01 --hostname pete01 hexiaoyuan/my-debian-dev01:v20220728
#
docker exec -it pete01 zsh
#
#
#
#
#### 记录当前我的pete02开发环境启动命令
docker run -d --init --env container=docker --env LANG=C.UTF-8 \
  --mount source=vol_pete02_home,target=/home                  \
  -p 60102:22 --name pete02 --hostname pete02 hexiaoyuan/my-debian-dev01:v20220728
#
#
```

