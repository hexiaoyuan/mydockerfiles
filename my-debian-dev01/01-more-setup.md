# 开发容器实例开启后的进一步设置

## 针对开发

### 修改apt源并更新

修改 /etc/apt/sources.list 到国内阿里云镜像:

```txt
deb http://mirrors.163.com/debian bullseye main
deb http://mirrors.163.com/debian-security bullseye-security main
deb http://mirrors.163.com bullseye-updates main
```

或者恢复出厂内容:

```txt
deb http://deb.debian.org/debian bullseye main
deb http://security.debian.org/debian-security bullseye-security main
deb http://deb.debian.org/debian bullseye-updates main
```

如果需要使用socks5代理访问:
ssh -CNg -D 127.0.0.1:1082 myproxyhostname
sudo apt -o Acquire::http::proxy="socks5h://127.0.0.1:1082" update

### Install oh-my-zsh

sudo apt install zsh curl
chsh -s /bin/zsh  # 改你的默认shell,重新登陆生效后安装ohyzsh,然后修改配置:
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
vi ~/.zshrc

### 安装docker(没有测试过哦)

详情请参考: https://docs.docker.com/engine/install/debian/

### 手工启动docker服务(没有测试过哦)

sudo /etc/init.d/docker start

如果想重启docker是自动启动docker,可以把上面行加到 ~/.local/auto.sh 文件中去. 建议需要时开启即可，用完关掉省点资源.

### 安装aws工具

```sh
python3 -m pip install --user awscli
aws s3 ls
python3 -m pip install --upgrade awscli  ## 升级
```

### 安装golang环境 (手动安装)

`#wget https://go.dev/dl/go1.18.4.linux-amd64.tar.gz`
wget https://golang.google.cn/dl/go1.18.4.linux-amd64.tar.gz
rm -rf ~/.local/go && tar -C ~/.local/ -xzf go1.18.4.linux-amd64.tar.gz
vi $HOME/.profile

```sh
export PATH=$PATH:$HOME/.local/go/bin:$HOME/go/bin
```

$ go version
$ go env

### 安装nodejs环境(手动安装)

NODEJS_VERSION=v16.16.0
wget https://nodejs.org/dist/$NODEJS_VERSION/node-$NODEJS_VERSION-linux-x64.tar.xz
tar -C ~/.local -xJvf node-$NODEJS_VERSION-linux-x64.tar.xz
cd ~/.local; rm -f nodejs; ln -s node-$NODEJS_VERSION-linux-x64 nodejs; cd -;
vi $HOME/.profile

```sh
export PATH=$PATH:$HOME/.local/nodejs/bin
```

$ node -v
$ npm version
$ npx -v

直接命令行运行:
$ npm install --location=global yarn

会安装到 /home/pete/.local/nodejs/bin/yarn 本地目录，
下次升级nodejs后记得重新安装即可。

### 安装rust-lang环境(手动安装)

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
rustup component add rustfmt
#
rustup update
#softwareupdate --install-rosetta
#
rustup install nightly
```

