---
author: "jarvan"
title: "十个实验熟练掌握Docker"
date: 2021-05-21
description: "Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从 Apache2.0 协议开源。让打包应用以及依赖包到一个轻量级、可移植的容器中，发布到任何Linux机器上。"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👻
tags:
- docker
- categories:

---


## 目标
1. 用Docker搭建一套三层架构的Web系统（nodejs+PHP+MySQL）
2. 掌握日常使用Docker进行程序开发、配置和发布的方法与原理

## 准备
1. 安装[docker desktop](https://docs.docker.com/get-docker/)
2. 克隆git项目:
```sh
git clone http://git.tmeoa.com/TMEIT/container-step-by-step
```
3. 切换目录:
```sh
cd ${PWD}/container-step-by-step/k8s
```

## 实验一 容器基本操作
### 目标
1. 用公共镜像启动一个Nginx服务，并能通过浏览器访问
### 目的
1. 掌握容器的启动、停止和删除操作
### 实验步骤
#### 运行nginx
1. 前台运行
    ```shell
    k8s run nginx
    ```
2. 后台运行
    ```shell
    k8s run -ti -d nginx
    ```
#### 查看容器运行状况
```shell
k8s ps
# 不加参数，查看正在运行的容器
# -a 查看全部容器，包括已停止的
```
#### 停止和启动容器
```shell
 # 停止容器
 k8s stop <name or id>
 # 强制停止
 # k8s kill <name or id>
 # 启动容器
 k8s start <name or id>
```
#### 暴露容器端口
1. 启动新的nginx容器
    ```shell
    k8s run -ti -d -p 8080:80 nginx
    ```
2. 打开浏览器，访问：http://127.0.0.1:8080
   @startuml
   agent 浏览器 as br
   node Docker服务器{
   port 8080
   node Nginx容器{
   port 80
   rectangle nginx
   }
   8080 --> 80
   ' 80 ... nginx
   }
   br --> 8080
   @enduml

#### 删除容器
```shell
# 容器停止状态下执行
k8s rm <name or id>
```

## 实验二 在容器内使用指定文件
### 目标
1. 通过容器内Nginx，用浏览器访问指定的静态文件
### 目的
1. 了解如何拷贝本地文件到容器
1. 了解如何拷贝容器文件到本地
3. 掌握如何挂载本地文件到容器
### 实验步骤
#### 文件拷贝方式
1. 启动nginx
    ```bash
    k8s run -d -ti -p 8081:80 nginx
    ```
2. 拷贝静态文件到容器指定目录
    ```bash
    k8s cp dist/ <id or name>:/usr/knowledge/nginx/html/
    ```
4. 浏览器访问http://127.0.0.1:8081
#### 挂载文件方式
1. 启动nginx并挂载文件夹(或文件)
    ```bash
    cp -R exp2/* data/
    k8s run -d -ti -p 8082:80  -v ${PWD}/data/dist:/usr/knowledge/nginx/html nginx
    # 或
    #  k8s run -d -ti -p 8082:80 -v ${PWD}/data/dist/index.html:/usr/knowledge/nginx/html/index.html nginx
    ```
4. 浏览器访问http://127.0.0.1:8082
3. 修改dist/index.html
4. 刷新浏览器

## 实验三 容器间通信
### 目标
1. 启动后端服务，通过Nginx转发HTTP请求
2. 通过客户端访问后端服务
### 目的
1. 理解Docker的容器网络环境
2. 了解容器间通信方式
### 实验步骤
1. 创建自定义网络
    ```bash
    k8s network create my_network
    # 查看现有网络列表
    k8s network ls
    ```
2. 启动php-fpm容器，并挂载代码到指定目录
    ```bash
    cp -R exp3/* data/
    k8s run -d -ti --name php-fpm-svr -v ${PWD}/data/php:/app/php --network my_network php:fpm  
    ```
2. 启动nginx容器，并挂载配置文件和静态文件
    ```bash
    k8s run -d -ti --name nginx-front -v ${PWD}/data/nginx_configs/default.conf:/etc/nginx/conf.d/default.conf -v ${PWD}/data/dist:/app/dist --network my_network -p 8080:80 nginx 
    ```
4. 浏览器访问：http://localhost:8080/api/getCurTime http://localhost:8080/

@startuml
agent 浏览器 as br
node Docker服务器{
cloud 主机网络 as hn
port "port_8080" as hnp
rectangle bridge网桥 as bridge {
portout port1
portout port2
}
node contanier1{
portin cp1
port1 -- cp1
}
node contanier2{
portin cp2
port2 -- cp2
}

    rectangle my_network网桥 as my_network {
        portout port3
        portout port4
    }
    node Nginx容器 as nginx{
        portin cp3
        port3 -- cp3
    }
    node php-fpm容器 as fpm{
        portin cp4
        port4 -- cp4
    }

    hnp -- hn
    hn -- bridge
    hn -- my_network
}
br -[#red,dotted]- hnp
hnp -[#red,dotted]- hn
hn -[#red,dotted]- my_network
my_network -[#red,dotted]- cp3
' nginx -[#blue,dotted]- port4
@enduml

## 实验四 使用临时容器执行任务
### 目标
1. 启动MySQL数据库
2. 通过MySQL容器初始化数据库
3. 数据持久化
### 目的
1. 了解如何查找和获取公共镜像版本
2. 理解容器环境隔离性和临时性
2. 理解容器存储原理
### 实验步骤
#### 临时容器
1. 浏览器访问 hub.docker.com，搜索mysql，进入mysql镜像页面
2. 下载指定版本镜像
    ```bash
    k8s pull mysql:5.7
    # 查看镜像列表
    k8s images
    ```
3. 启动mysql
    ```bash
    k8s run -d -ti -e MYSQL_ROOT_PASSWORD=rootpass mysql:5.7
    #查看容器IP
    k8s inspect <id or name>
    # 或者 k8s inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <id or name>
    ```
4. 启动临时容器，导入数据
    ```sh
    cp -R exp4/* data/
    k8s run --rm -ti -v ${PWD}/data/exp4.sql:/root/exp4.sql mysql:5.7 bash
    mysql -h<ip> -uroot -p
    # mysql shell
    source /root/exp4.sql
    quit
    # bash
    exit
    ```
   或者
    ```sh
    k8s run --rm -ti -v ${PWD}/data/exp4.sql:/root/exp4.sql mysql:5.7 mysql -h<ip> -uroot -p
    # mysql shell
    source /root/exp4.sql
    quit
    ```
5. 查看容器
    ```bash
    k8s ps -a
    ```
#### 数据持久化
启动新的mysql容器，指定数据存储目录，并导入数据
```sh
k8s run -d -ti --name mysql_svr -e MYSQL_ROOT_PASSWORD=rootpass -v ${PWD}/data/mysql_data:/var/lib/mysql mysql:5.7
# 将mysql容器加入到my_network网络中，以便使用容器名进行访问
k8s network connect my_network mysql_svr
# 导入数据
k8s run --rm -ti -v ${PWD}/data/exp4.sql:/root/exp4.sql --network my_network mysql:5.7 mysql -hmysql_svr -uroot -p
# mysql shell
source /root/exp4.sql
quit
```

@startuml
node Docker服务器{
node mysql容器 {
rectangle rootfs #aliceblue;line:blue;line.dotted;text:blue {
folder "/" as root
folder "/" as root
folder "/var" as var
folder "/usr" as usr
folder "..." as other
folder "/var/lib/mysql" as var_lib_mysql #aliceblue;line:grey;line.dotted;text:grey
folder "/var/lib/..." as var_lib_other

            root -- var
            root -- usr
            root -- other
            var -- var_lib_other
            var -- var_lib_mysql
        }
    }
folder mysql_data
mysql_data <--> var_lib_mysql
}
@enduml

#### 配置后端访问
1. 为PHP安装mysql驱动
    ```sh
    # 进入容器shell
    k8s exec -ti php-fpm-svr bash
    # 容器内 bash
    k8s-php-ext-install pdo pdo_mysql
    exit
    # 重启php-fpm-svr容器
    k8s stop php-fpm-svr
    k8s start php-fpm-svr 
    ```
2. 浏览器访问：http://localhost:8080/api/getDbData

## 实验五 创建容器镜像
### 目标
1. 固化程序版本，不再依赖本地文件和容器
2. 可同时访问新旧版本API
### 目的
1. 掌握Dockerfile基本使用方法

### 实验步骤
1. 修改 data/php/api.php 文件 第23行
    ```php
     $response = ['time' => $currentTime, 'version' => '1.0'];
    ```
2. 浏览器访问：http://127.0.0.1:8080/api/getCurTime 确认修改已生效
3. 拷贝Dockerfile
    ```sh
    cp exp5/Dockerfile data/php/
    ```
4. 构建镜像
    ```sh
    cd data/php
    k8s build -t php-fpm-svr:v1.0 .
    ```
5. 运行1.0版本镜像
    ```sh
    k8s run -d -ti --name php-fpm-svr-v1 --network my_network php-fpm-svr:v1.0
    ```
6. 修改nginx_configs/default.conf文件，添加如下配置：
    ```nginx
    location /api/v1/{
        if ($uri ~ "^/api/v1/(.*)$") {
            set $newuri /api/$1;
        }
        fastcgi_pass php-fpm-svr-v1:9000; # 新的PHP后端服务地址
        fastcgi_param SCRIPT_FILENAME /app/php/api.php; 
        include fastcgi_params;
        fastcgi_param REQUEST_URI $newuri;
    }
    ```
7. reload nginx使配置生效
    ```sh
    k8s exec -ti nginx-front nginx -s reload
    ```
1. 修改 data/php/api.php 文件 第23行
    ```php
     $response = ['time' => $currentTime, 'version' => '1.1'];
    ```
8. 分别访问：  
   http://127.0.0.1:8080/api/getCurTime  
   http://127.0.0.1:8080/api/v1/getCurTime

@startuml
agent 浏览器 as br
node Docker服务器{
port "port_8080" as hnp
node nginx-front as nginx{
portin port_80
}
node php-fpm-svr as fpm{
portin portA_9000
}
node php-fpm-svr-v1 as fpmv1{
portin portB_9000
}
node mysql_svr as mysql{
portin portC_3306
}
hnp -- port_80
nginx -- portB_9000
nginx -- portA_9000
fpm -- portC_3306
fpmv1 -- portC_3306
}
br -- hnp

@enduml

## 实验六 自动化编译打包
### 目标
1. 通过Dockerfile构建前端镜像
2. 前端镜像中只包含必要的静态文件
### 目的
1. 掌握如何利用容器构建编译环境方法
2. 掌握Dockerfile高级使用技巧
### 实验步骤
#### 手动编译vuejs
1. 运行nodejs容器，并初始化vuejs项目
    ```sh
    k8s run -ti --rm -v ${PWD}/data/vuejs:/app node:19 bash
    # 容器shell
    npm install -g @vue/cli
    cd /app
    vue create my-vue-app
    ```
3. 拷贝vue源文件
    ```sh
    cp -vR exp6/vue_src/* data/vuejs/my-vue-app/src 
    ```
3. 编译vuejs
    ```sh
    # 容器shell
    npm run build
    exit
    ```
4. 拷贝静态文件
    ```sh
    cp -vR data/vuejs/my-vue-app/dist/* data/dist/
    ```
5. 浏览器访问： http://127.0.0.1:8080/

#### 自动编译打包
1. 拷贝Dockerfile并构建镜像
    ```sh
    cp exp6/Dockerfile data/
    ```
2. 制作镜像
    ```sh
    cd data 
    k8s build -t front:v1.0 .
    ```
3. 运行前端镜像
    ```sh
    k8s run -d -ti --name front-1.0 -p 8081:80 --network my_network front:v1.0 
    ```
4. 浏览器访问： http://127.0.0.1:8081/

@startuml
agent 浏览器 as br
node Docker服务器{
port "port_8080" as hnp
node nginx-front as nginx{
portin portA_80
}
port "port_8081" as hnp1
node nginx-front-v1 as nginxv1{
portin portB_80
}
node php-fpm-svr as fpm{
portin portA_9000
}
node php-fpm-svr-v1 as fpmv1{
portin portB_9000
}
node mysql_svr as mysql{
portin portC_3306
}
hnp -- portA_80
hnp1 -- portB_80
nginx -- portB_9000
nginx -- portA_9000
nginxv1 -- portB_9000
nginxv1 -- portA_9000
fpm -- portC_3306
fpmv1 -- portC_3306
}
br -- hnp1

@enduml

## 实验七 镜像的分发与镜像仓库的使用
### 目标
1. 通过镜像导出、导入实现镜像分发
2. 通过镜像上传、下载到镜像仓库实现镜像分发
### 目的
1. 了解如何使用镜像私有化镜像仓库
2. 理解容器化软件分发方式

### 实验步骤
#### 镜像导出和导入
```shell
#导出镜像
k8s save -o front-v1.0.tar front:v1.0
# 删除正在运行的容器和镜像
k8s kill front-1.0
k8s rm front-1.0 && k8s rmi front:v1.0
#导入镜像
k8s load -i front-v1.0.tar
```
#### 利用镜像仓库上传下载镜像
<i>以腾讯云的个人镜像仓库为例</i>

```shell
# 登录镜像仓库
k8s login ccr.ccs.tencentyun.com --username=<username>
# 设置镜像别名
k8s tag front:v1.0 ccr.ccs.tencentyun.com/consbs/front:v1.0
# 上传镜像
k8s push ccr.ccs.tencentyun.com/consbs/front:v1.0
# 下载镜像
k8s pull ccr.ccs.tencentyun.com/consbs/front:v1.0
```

## 实验八 限制容器使用资源
### 目标
1. 通过设置参数实现容器最大能使用的CPU和内存资源数量
### 目的
1. 了解如何限制容器计算资源使用的方法
### 实验步骤
#### 限制CPU资源
1. 启动主机压测程序stress
```bash
# 建议设置一个超时时间，防止参数错误将服务器压垮
k8s run --rm -it progrium/stress --cpu 1 --timeout 60s
# --cpu=1 程序会让一个CPU核心使用率到达100%
```
2. 登录docker服务器，用top命令观察stress进程消耗资源
```sh
# 打开另一个终端窗口
# （可选）mac上登录docker服务器
k8s run --rm -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
top
# 按pid排序
N
# 展开CPU信息
1
```
3. 重新启动stress，并设置CPU=0.5核
```shell
# ctl+C 结束原来的street容器，重新启动
k8s run --cpus=0.5 --rm -it progrium/stress  --cpu 1  --timeout 60s
# --cpus=0.5 限制容器最大可用CPU资源为0.5个
#
```
4. 重复步骤2
#### 限制内存资源
1. 启动主机压测程序stress
```bash
# 建议设置一个超时时间，防止参数错误将服务器压垮
k8s run --rm -it progrium/stress --vm 1 --vm-bytes 256M --timeout 60s
# --vm 1 --vm-bytes 256M 程序会启动一个进程，申请256MB内存
```
2. 登录docker服务器，用top命令观察stress进程消耗资源
```sh
# 打开另一个终端窗口
# （可选）mac上登录docker服务器
k8s run --rm -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
top
# 按pid排序
N
```
3. 重新启动stress，并设置CPU=0.5核
```shell
# ctl+C 结束原来的street容器，重新启动
k8s run --rm -m 128m -it progrium/stress --vm 1 --vm-bytes 256M --timeout 60s
# -m 128m 限制容器最大可用内存为128MB
# 会发现容器由于无法申请到足够的内存，启动后马上退出
```

## 实验九 利用gitlabCI自动化构建和上传镜像
### 目标
1. 提交代码后，自动构建镜像并推送至镜像仓库
### 目的
1. 利用工具实现软件自动容器化过程
### 实验步骤
1. 拷贝文件 `.gitlab-ci.yml`和`Dockerfile` 文件到git仓库项目目录下。
    ```sh
    cp exp9/.gitlab-ci.yml you-git-project-dir
    cp exp9/Dockerfile you-git-project-dir
    ```
3. 在 GitLab 项目设置中打开 "CI/CD" 设置，找到 "Variables" 部分，并添加 镜像仓库、用户名、密码和镜像这四个变量，变量名称分别为:`CI_REGISTRY` `CI_REGISTRY_USER` `CI_REGISTRY_PASSWORD` `CI_REGISTRY_IMAGE`
4. 提交代码到 GitLab 仓库。每次提交时，GitLab CI/CD 将自动运行构建流程，构建 Docker 镜像并上传至 GitLab 容器仓库。详细信息可以在GitLab项目的CI/CD--Jobs页面查看。

### demo：
#### Dockerfile
请使用 Dockerfile 构建容器镜像。

Dockerfile 应与应用代码一起保存在 Git 仓库中，以便进行版本控制和分支管理。
#### 镜像构建与上传
推荐使用 GitLabCI 进行自动化镜像构建和上传。示例如下：
1. 在 GitLab 项目仓库中创建文件 .gitlab-ci.yml:
```yml
# .gitlab-ci.yml # 更多语法规则参考：https://docs.gitlab.com/ee/ci/ 
stages:
   - build

docker-build:
   stage: build
   only:
      - tags
   before_script:
      - k8s login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" $CI_REGISTRY
   # Default branch leaves tag empty (= latest tag)
   # All other branches are tagged with the escaped branch name (commit ref slug)
   script:
      - |
         pwd
         ls
         imageUrl="$CI_REGISTRY_KGAPI/api_build:$CI_COMMIT_TAG" 
         echo "Running on branch '$CI_COMMIT_BRANCH': commit_tag = '$CI_COMMIT_TAG'"
         echo "******"
         echo "${imageUrl}"
         echo "******"
      - k8s build -f ./Dockerfile -t "${imageUrl}" .
      - k8s push "${imageUrl}"
      - k8s rmi "${imageUrl}"
   tags:
      - tmeit-knowledge
```

2. 设置镜像仓库、镜像名称、用户名和密码等变量（GitLab 项目群组可使用同一套变量，如需独立的用户，可向运维申请）
```text
变量设置：
设置 -> CI/CD -> Variables

CI_REGISTRY_KGAPI -> tme-it-hr.tencentcloudcr.com/kgkpi
```

3. Dockerfile的内容如下：
```dockerfile
FROM tme-it-hr.tencentcloudcr.com/kgkpi/api_mid:v1.1
COPY . /var/www/html/TME_KG_STOCK_POINTS_API/
RUN chmod -R 777 /var/www/html/TME_KG_STOCK_POINTS_API/public/
RUN chmod -R 777 /var/www/html/TME_KG_STOCK_POINTS_API/storage/
RUN chmod -R 777 /var/www/html/TME_KG_STOCK_POINTS_API/bootstrap/
WORKDIR  /var/www/html/TME_KG_STOCK_POINTS_API
EXPOSE 9000
```
api_mid:v1.1的镜像的Dockerfile如下：
```dockerfile
FROM php:7.4-fpm
LABEL maintainer="kugou_stock_points api Maintainers <jarvenwang@kugou.net>"
RUN echo "deb http://mirrors.aliyun.com/debian/ bullseye main non-free contrib \n \
    deb-src http://mirrors.aliyun.com/debian/ bullseye main non-free contrib \n \
    deb http://mirrors.aliyun.com/debian-security/ bullseye-security main \n \
    deb-src http://mirrors.aliyun.com/debian-security/ bullseye-security main \n \
    deb http://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib \n \
    deb-src http://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib \n \
    deb http://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib \n \
    deb-src http://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib" > /etc/apt/sources.list
RUN apt-get update && apt-get install -y vim
RUN apt-get install -y zlib1g-dev
RUN apt-get install -y libzip-dev
RUN apt-get -y install gcc make autoconf libc-dev pkg-config\
    && apt-get -y install libmcrypt-dev
RUN apt-get install -y git
RUN docker-php-ext-install mysqli\
    && docker-php-ext-install sockets\
    && docker-php-ext-install pdo_mysql\
    && docker-php-ext-install zip\
    && pecl install swoole-4.4.17\
    && docker-php-ext-enable swoole\
    && pecl install redis-5.2.1\
    && docker-php-ext-enable redis\
    && pecl install mcrypt\
    && docker-php-ext-enable mcrypt
RUN php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');"\
    && php composer-setup.php\
    && mv composer.phar /usr/local/bin/composer\
    && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

4. git合并时忽略.gitlab-ci.yml
```text
1、git合并时忽略某个文件(代码片段)
2、git合并两个分支的某个文件

因为开发现场跟部署的环境不同,有很多ip地址每次都要改来改去;于是开两个分支master(用来保存部署现场的ip)和dev(开发环境的ip),开发功能时在dev分支,然后使用master合并,每个分支都保存着自己的config配置文件,不想dev分支被master合并时config文件也合并.

创建自定义merge driver
git config --global merge.kg.driver true
在要被merge的分支上创建.gitattributes文件,并且在文件中置顶不merge的文件名
echo ‘.gitlab-ci.yml merge=kg merge=ours‘ >> .gitattributes
之后提交到远程仓库
之后合并分支
```

## 实验十 问题排查
### 目的
1. 掌握常用容器运行时调试方法
2. 了解容器运行常见错误
### 实验步骤
#### 查看镜像或容器详细信息
```sh
k8s inspect <container/image id or name>
```
#### 查看容器的标准输出
```sh
k8s logs <container id or name> -n 10 -f
# -n <num> 显示最新num行日志
# -f 自动滚动更新
```
#### 进入容器运行时控制台
```sh 
docker attach <container id or name>
# ctl+p ctl+q 退出（docker run没有加 -ti 参数无效）
```
#### 在容器中执行命令
```sh
k8s exec -ti <container id or name> <command> <args>
```
#### 无法启动或启动后退出
分别构建exp10/Dockerfile中注释的两种错误镜像
```Dockerfile
# 正常启动 
CMD [ "nginx", "-g", "daemon off;" ]
# 错误1：启动脚本中带 &
# ENTRYPOINT [ "/mystart.sh" ]
# 错误2：后台运行主程序
# ENTRYPOINT [ "nginx" ]
```
分别启动这两种错误镜像：
```sh
k8s run -ti <image name>
```

