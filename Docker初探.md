# Docker 初探

官网：https://www.docker.com/  （不稳定，需要fq）

配置阿里云的镜像：https://cr.console.aliyun.com/cn-hangzhou/instances/repositories



docker ps

查看正在运行的容器



docker run -it ubuntu

建立并运行ubuntu的容器（会先去先查本地有无镜像，没有会去Docker Hub镜像仓库里面下载，类似Node的NPM）

 

docker start ID

启动容器但不进去容器控制台



docker attach ID

进去容器的控制台



docker stop ID

停止容器

docker rm ID

移除容器

docker image ls

查看本地下载的镜像容器

docker rmi xx

彻底移除镜像，释放空间



docker run --name web-server -d -p 8000:80 nginx

建立名为web-server的nginx服务器，以后台形式启动，本地8000端口映射nginx80端口



docker pull <image>

docker search <image>





**docker run 选项**

* -d , 后台运行容器
* -e，设置环境变量
* --expose/-p 宿主端口:容器端口
* --name，指定容器名称
* --link，链接不同容器
* -v 宿主目录:容器目录，挂载磁盘卷



**启动mongodb**

* docker run --name mongo -p 27017:27017 -v I:\program\mongo:data/db -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=admin -d mongo

  创建并启动mongo的容器。映射本地端口27017到容器端口27017，挂载本机I盘下的目录到data/db中。设置超级管理员的账号密码都为admin。



**登录到MongoDB容器中**

* docker exec -it mongo bash   进入mongo的控制台

**通过Shell 连接MongoDB**

* mongo -u admin -p admin









 