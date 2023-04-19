# 在服务器上部署sentry
## ssh免密登录
1. `ssh-keygen` 生成公私钥
2. `ssh-copy-id -i ~/.ssh/id_rsa.pub root@xxx.xx.x.x` 这条命令是写到服务器上的ssh目录下去了
`cd ~/.ssh`
`vim authorized_keys`
3. `vim ~/.ssh/config`
```
Host sentry
	HostName xxx.xx.x.x
	Port 22
	User root
	IdentityFile ~/.ssh/id_rsa
```
4. `ssh sentry`免密登录
## 安装docker
* `curl -sSL https://get.daocloud.io/docker | sh`
## openvpn
## git clone repo
## ./install.sh
* `service docker start`
