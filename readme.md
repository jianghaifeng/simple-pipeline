# Setup CI/CD pipeline using Jenkins

## prerequisites

* jenkins

* gradle

* node

* nginx

* git

* jdk


## steps
1. In Jenkins, create a new job by clicking the "New Item"
2. Enter the name and choose "Freestyle project", then click "ok"
3. In "Source Code Management" tab, choose the "Git" and in the input "Repository URL", specify your url as your github repository.
4. In "Build Triggers", choose the "Poll SCM" and input in the "Schedule" as below:
    ```
    H/3 * * * *
    ```
5. Create a directory on your disk to store your artifacts and the launch script, for example: 
    ```
    D:\deploy
    ```
6. In "Build" tab, click "Add build step", int the "Command" input box, add the following command:
    ```
    gradle test
    ```
7. Add a new build step, with the following command:
    ```
    gradle build
    ```
8. Add a new build step, with the following command:
    ```
    copy build\libs\*.jar D:\deploy\
    cd D:\deploy
    run.bat
    ```
9. In directory `D:\deploy`, create a launch scirpt `run.bat`, in the file, add the following scripts:
    ```
    for /f "token=5" %%a in ('netstat -ano ^| findstr "8090"') do set pid=%%s
    if %pid% neq "" if %pid% neq "0" (taskkill /f /pid %pid%)
    start java -jar spring-boot-employee-0.0.1-SNAPSHOT.jar
    ```
10. Click "Save" and then "Build now", you can see the pipeline started to run.



=======


## 配置前端 Pipeline

我们采用freestyle方式创建一个流水线；
整体思路是通过 `jenkins`运行test、build，将构建产生的build文件放到nginx目录下，通过nginx将服务启动；

### 环境准备：

1. [Nginx](https://nginx.org/en/download.html)
    1.1 下载安装nginx最新稳定版本；
    1.2 解压zip包
    1.3 找到 `nginx.exe` 文件，双击，会发现一个窗口闪过；
    1.4 访问 `localhost`会进入到Nginx页面；
    
2. Node & Npm
    2.1 Node升级到最新版，通过从[官网](https://nodejs.org/en/download/)下载 `msi` 文件进行安装和升级；
    2.2 npm升级到最新版，通过 `npm install -g npm` 来更新；
    2.3 切换源：将npm镜像包切换到国内淘宝的地址；
        `npm config set registry https://registry.npm.taobao.org`

## Pipeline构建
1. 打开Jenkins，选择 `新建任务`,  输入任务名称，比如 `web-ui-build`,选择  `构建一个自由风格的软件项目`，点击确定；

2. 在General部分输入项目描述，
3. 源码管理选 `git`, 输入 `repository url`，指定分支master，
4. 构建触发器：选择轮询SCM，输入 `H/5 * * * *`，指定5分钟获取一次，如果git上有更新，则会开始build，如果没有，则跳过这次；
5. 构建环境：选择 `Add timestamps to the Console Output`
6. 构建部分：
   6.1：添加 `执行Windows批处理命令`， 在命令框中输入: `node -v && npm -v` 查看各自的版本
   6.2： 添加 `执行Windows批处理命令`， 在命令框中输入: `npm install && npm build`
   6.3: 将构建出来的build文件夹下面的内容复制到一个路径下，会引导nginx访问那个路径.比如我们将构建的文件放到 `D:\jenkins-build\web-ui`目录下，添加 `执行Windows批处理命令`， 在命令框中输入下面的命令：


	```
	rmdir /Q /S D:\jenkins-build\web-ui && mkdir D:\jenkins-build\web-ui && xcopy build D:\jenkins-build\web-ui /E/H/C/I
	```
    

7. 点击保存，然后 `立即构建`，检查 `D:\jenkins-build\web-ui`路径下构建好的文件是否存在；


## 配置Nginx

1. 打开nginx所在的目录， 编辑 `C:\Users\ITATEACH2\Downloads\nginx-1.18.0\conf\nginx.conf` 文件，找到 server 下面的location部分，将其改成下面的内容： 

    ```
    location / {
                root   D:/jenkins-build/web-ui/;
                index  index.html index.htm;
            }

    ```


2. 配置完成后，访问localhost应该可以看到前端页面；




