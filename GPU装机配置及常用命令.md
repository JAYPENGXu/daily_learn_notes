装机配置步骤：

nvtop小插件：实时查看GPU使用率

​	$ `sudo apt install nvtop`

​	$ `man nvtop`

​	$ `nvtop -h`

$ `nvidia-smi -l 1` :每隔一秒刷新一次信息

$ `watch -n 1 nvidia-smi`

$ `gpustat -i`

$ `nvidia-smi topo -m`

$ `cd gpu-burn`

$ `./gpu_burn -tc 1200`

![image-20220916161439049](G:\typora_image_store\image-20220916161439049.png)





$ `nvcc`

李沐的测机代码： 

https://github.com/mli/transformers-benchmarks/blob/main/micro_bench.ipynb



   1.优先使用bf16。bf16和fp16性能差不多。bf16的范围不变，不需要使用loss scaling。

2. 使用apex的fuse adamw，避免无谓的中间变量读写。

3. 模型更新也有性能成本，可以用梯度累加减少。

4. megatron的性能比huggingface好，重写了ln,softmax,gelu等算子。

---

1.安装ssh，配置远程登录

$ `sudo apt-get install openssh-client`

2.查看当前的系统是否安装了ssh-server服务，默认只安装了ssh-client。

$ `dpkg-l | grep ssh`

3.安装ssh-server服务

$ `sudo apt-get install openssh-server`

4.查看是否安装了ssh-server

$ `dpkg -l | grep ssh`

**5.确认ssh-server是否启动**

$ `ps -e | grep ssh`

6.重新启动（非必须）：

$ `sudo service ssh start` **or** `sudo/etc/init.d/ssh start`

7.登录ssh

`ssh username@ip address`

8.配置别名登录

创建文件 `~/.ssh/config`

```shell
Host gpuserver

	HostName IP地址

	User 用户名
```

9.免密登录

`ssh-keygen`

10.一键添加公钥

`ssh-copy-id gpuserver`

~~`jupyter notebook --no-browser --port=8888 --ip=0.0.0.0`~~

11.服务器8888端口映射到本地8888端口

$`ssh -L 8888:localhost:8888 xp@222.24.22.87`

---

ubuntu系统防火墙和添加端口：

$`sudo ufw status`

$`sudo ufw allow 9999`

$ `sudo ufw deny 9999`

$`sudo ufw reload`

查看开启的端口是否有程序监听：

$ `netstat -ap | grep 9999`

---

ubuntu系统通过ssh关机

$ `sudo shutdown -r now` 立刻重启

$ `sudo shutdown -r 10` 在过10分钟重启

$ `sudo poweroff` 立刻关机

$ `sudo shutdown -c` 取消重启

$ `sudo shutdown -h now` 立刻关机

$ `sudo shutdown -h 10` 在过10分钟关机

---

scp传文件到gpu服务器上

$ `scp 文件名 gpu:/home/xp`

传文件夹到gpu服务器上

$ `scp -r ~/test gpu:home/xp/`

将gpu服务器中的~/work/文件夹复制到本地的当前路径下。

$ `scp -r gpu:work .`

$ `scp myserver: a.txt .`

----

SQL:

登录： $`mysql -u root -p`

[安装参考链接](https://kalacloud.com/blog/ubuntu-install-mysql/)

重启mysql: $`sudo service mysql restart`

查看是否对某用户授权成功：$ `SELECT user,host FROM mysql.user;`

刷新数据库 ： $`flush privileges;`

本地测试是否可连接数据库： $`mysql -h127.0.0.1 -u root -p`

---

docker 安装mysql

拉取镜像：$`docker pull mysql:8.0.15`

安装mysql：$`docker run -p 3306:3306 --name MySql -itd mysql:8.0.15`

启动docker运行mysql(<u>记得开放3306端口</u>):$ `docker run --name mysqlserver -v PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -i -p 3306:3306 mysql:8.0.15`

进入mysql： $`docker exec -it mysqlserver bash`

登录mysql: $`mysql -u root -p`

开启远程服务：

$`select now();`

$  `use mysql;`

$ `select host,user from user;`

$ `ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';`

$ `flush privileges;`

使用 MySQL Workbench连接运行在服务器docker中的数据库

Anaconda环境管理：

`conda create -n py36 python=3.6`

`conda create -n tf tensorflow`

`conda remove -n py36 --all`

`conda deactivate`

windows通过WSL创建虚拟linux环境，通过vscode登录虚拟环境

首先linux端需要开启ssh服务

`sudo service ssh start`

之后通过如下命令查看是否开启

`ps -e | grep ssh`


