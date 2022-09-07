当购买了一台云服务器时(ubuntu系统)，如阿里云、腾讯云、华为云等

如何配置免密登录：

（1）：首先登录到该云服务器上，`ssh root/(ubuntu)@xxx.xxx.xxx.xxx` ,注意腾讯云的用户是ubuntu

（2）：在云服务器上创建一个用户，`adduser xiaoming` 

（3）：给用户分配`sudo`权限 `usermod -aG sudo xiaoming`

（4）：配置免密登录，回到本地，创建文件 `~/.ssh/config`,在该文件中输入

```shell
Host myserver
	HostName xxx.xxx.xxx.xxx #云服务器IP地址
	User xiaoming
```

（5）：创建密钥： `ssh-keygen`，然后一直回车

（6）：将密钥中的公钥传到云服务器上 `ssh-copy-id myserver`

（7）：结束，此时便可通过 `ssh myserver`登录到自己的云服务器上。