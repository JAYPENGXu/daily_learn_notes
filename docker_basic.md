#### Docker_notes

```shell
docker --version

#将当前用户加入安装中自动创建的docker用户组
sudo usermod -aG docker $USER

sudo chmod 666 /var/run/docker.sock
```

#### 镜像

```shell
docker pull xxx # 拉取一个镜像
docker images : #列出本地所有镜像
docker image rm ubuntu:20.04 或 docker rmi ubuntu:20.04 :#删除镜像ubuntu:20.04
docker save -o ubuntu_20_04.tar ubuntu:20.04 #将镜像ubuntu:20.04导出到本地文件 ubuntu_20_04.tar中
docker load -i ubuntu_20_04.tar :#将镜像ubuntu:20.04从本地文件ubuntu_20_04.tar中加载出来
docker run -p 20000:22 --name my_docker_server -itd docker_lesson:1.0  # 创建并运行docker_lesson:1.0镜像
```

#### 容器

```shell
docker create -it ubuntu:20.04：#利用镜像ubuntu:20.04创建一个容器。
docker ps -a：#查看本地的所有容器
docker start CONTAINERID：#启动容器
docker stop CONTAINERID：#停止容器
docker restart CONTAINERID：#重启容器
docker run -itd ubuntu:20.04：#创建并启动一个容器
docker attach CONTAINERID ：#进入容器
# 挂起容器: ctrl+p then ctrl+q
docker exec CONTAINERID command #在容器中执行命令
docker rm CONTAINERID # 删除容器
docker export -o xxx.tar CONTAINER:#将容器CONTAINER导出到本地文件 xxx.tar中
docker import xxx.tar image_name:#将本地文件xxx.tar导入成镜像，并将镜像命名为image_name
#与save / load 相比会丢弃历史记录，仅保留容器当时的快照状态
docker top CONTAINERID:#查看某个容器内的所有进程
docker cp xxx CONTAINERID:xxx 或 docker cp CONTAINERID:xxx xxx：#在本地和容器间复制文件
docker rename CONTAINER1ID CONTAINER2ID：#重命名容器
docker update CONTAINERID --memory 500MB：#修改容器限制
docker port CONTAINERID #查看容器的端口号及映射关系
```





























