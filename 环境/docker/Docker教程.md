# 卸载当前docker环境(非必须)
如果Linux上已经安装过docker，想要安装最新版本的docker环境，就要先把已经安装好的docker环境卸载
执行以下命令：
```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
# 更新yum
```bash
sudo yum update
```

# 设置安装环境
## 安装docker需要的工具包
```bash
sudo yum install -y yum-utils
```

## 建立docker仓库
这里使用的是阿里云镜像
```bash
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

# 安装docker
```bash
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

# 启动和校验docker安装结果
## 启动docker
```bash
systemctl start docker
```

## 停止docker
```bash
systemctl stop docker
```

## 重启docker
```bash
systemctl restart docker
```

## 设置开机自启动
```bash
systemctl enable docker
```

## 执行docker命令 查看是否启动
```bash
docker ps
```

# docker中安装mysql
```bash
docker run -d --name mysql -p 3306:3306 -e TZ=Asia/Shanghai -e MYSQL_ROOT_PASSWORD=123456 mysql
```


# 配置镜像
要在/etc/docker下创建daemon.json文件，并且写入镜像地址，然后重载镜像配置并且重启docker服务
```bash
sudo vim /etc/docker/daemon.json
```
镜像地址
```json
{
"registry-mirrors": [
   "https://docker.registry.cyou",
   "https://docker-cf.registry.cyou",
   "https://dockercf.jsdelivr.fyi",
   "https://docker.jsdelivr.fyi",
   "https://dockertest.jsdelivr.fyi",
   "https://mirror.aliyuncs.com",
   "https://dockerproxy.com",
   "https://mirror.baidubce.com",
   "https://docker.m.daocloud.io",
   "https://docker.nju.edu.cn",
   "https://docker.mirrors.sjtug.sjtu.edu.cn",
   "https://docker.mirrors.ustc.edu.cn",
   "https://mirror.iscas.ac.cn",
   "https://docker.rainbond.cc"
 ]
}
```
重载镜像配置
```bash
sudo systemctl daemon-reload
```
重启docker服务
```bash
sudo systemctl restart docker
```

# 基础命令解释
```bash
docker run -d \
	--name mysql \
	-p 3306:3306 \
	-e TZ=Asia/Shanghai \
	-e MYSQL_ROOT_PASSWORD=123456 \
	mysql
```
docker run -d：创建并且运行一个容器，-d指明在后台运行
--name mysql：给当前容器命名，需要唯一
-p 3306:3306：设置容器的端口映射，第一个是宿主机端口也就是linux端口，第二个是容器端口
-e key=value：设置环境变量，这是由镜像制作者决定的，不同的镜像可能都不一样
mysql：指定运行镜像的名字，完整的写法：[repository]:[tag]，repository是镜像名，tag类似于版本，如果不写tag就默认找latest

# 常见命令
官方文档：https://docs.docker.com


# 数据卷
volume是一个虚拟目录，是docker容器和宿主机之间的映射
## 命令
```bash
# 创建数据卷
docker volume create

# 查看所有数据卷
docker volume ls

# 删除指定数据卷
docker volume rm

# 查看某个数据卷详情
docker volume inspect

# 清除数据卷
docker volume prune
```

## 如何挂载数据卷
命令：
```bash
# 数据卷名字可自定义 但不能重复
# -v 数据卷:容器目录
docker run -d --name nginx -p 80:80 -v html:/usr/share/nginx/html nginx
```
**只能在创建容器时挂载**

# 本地目录挂载到容器

和挂载数据卷的命令相同，只不过-v后面是本地路径:容器类路径,例如：
```bash
docker run -d \
	--name mysql \
	-p 3306:3306 \
	-e TZ=Asia/Shanghai \
	-e MYSQL_ROOT_PASSWORD=123456 \
	-v /usr/local/src/mysql/data:/var/lib/mysql \
	-v /usr/local/src/mysql/conf:/etc/mysql/conf.d \
	-v /usr/local/src/mysql/init:/docker-entrypoint-initdb.d \
	--network hm-net \
	mysql
```



# 创建网络环境

```bash
docker network create <网络名称>
```


# DockerFile语法
| 指令 | 说明 | 示例 |
| ---- | ---- | ---- |
| FROM | 指定基础镜像 | from centos:6 |
| ENV | 设置环境变量，可以在后面指令使用 | ENV KEY VALUE |
| COPY | 拷贝本地文件到镜像的指定目录 | COPY ./jre11.tar.gz /tmp |
| RUN | 执行Linux的shell命令 | RUN tar -zxvf |
| EXPOSE | 指定容器运行时的监听端口 | EXPOSE 8080 |
| ENTRYPOINT | 镜像中应用的启动命令，容器运行时调用 | ENTRYPOINT java -jar xxx.jar |

编写好了Dockerfile之后可以用下面的命令来构建一个镜像：
```bash
docker build -t imagename:imageversion .
```
- -t是指定镜像版本也就是tag
- .：是指定Dockerfile所在的目录