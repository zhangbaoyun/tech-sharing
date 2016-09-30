环境：ubuntu16.04 ，安装最新版本docker  
## docker 部署  
### docker 安装  
- 添加GPG key  
```
$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D  
```  
- 添加源 新建文件：/etc/apt/sources.list.d/docker.list
```  
$ sudo vim /etc/apt/sources.list.d/docker.list  
```  
在里面添加内容:  
> deb https://apt.dockerproject.org/repo ubuntu-xenial main  

- 更新源  
```  
$ sudo apt update  
```  
- 安装docker  
```  
$ sudo apt-get install docker-engine  
```  
- 启动docker，并将docker设置为开机启动项。  
```  
$ sudo service docker start  
$ sudo chkconfig docker on  
```  
- 使用新式 systemd 语法，启动docker并设置为开机启动项，如下:  
```  
$ sudo systemctl  start docker.service  
$ sudo systemctl  enable docker.service  
```  

### docker 桌面镜像的部署  （邱总制作镜像）  
- 镜像位置  
\\10.0.0.2\rnd_fileserver\Users\QiuYunpeng\Docker\ubuntu-xfce-vnc-novnc-ssh-v0.tar.gz  
- 加载镜像  
```  
# docker load < ubuntu-xfce-vnc-novnc-ssh-v0.tar.gz  
```  
- 创建容器  
```  
# docker run --env LANG=en_US.UTF-8 --name 容器名称 -d -p 5900:5900 -p 5822:22 -p 6080:6080 ubuntu-xfce-vnc-novnc-ssh-v0  
```  
- 访问方式  
    - vnc客户端连接ip:5900  
    - 浏览器中打开：http://ip:6080/vnc.html  
      密码：ubuntu  
    - ssh连接：10.0.0.5:5822  
      用户：ubuntu  
      密码：ubuntu  
- 镜像主要功能  
    - firefox浏览器，支持中文显示  
    - 使用noVNC、VNC客户端进行交互  
    - 搜狗拼音输入法  

### 基于ubuntu16.04发行版创建docker镜像模板  
说明：基于ubuntu16.04桌面版  
- 环境:  
安装ubuntu16.04-16.04-desktop-amd64.ios，中文，美式键盘  
- 安装 vim  
```  
$ sudo apt-get install vim  
```  
- 安装ssh服务端  
```  
$ sudo apt-get install openssh-server  
```  
- 安装,配置x11vnc  
    - 安装x11vnc  
    ```  
    $ sudo apt-get install x11vnc  
    ```  
    - 配置访问密码  
    ```  
    $ sudo x11vnc -storepasswd  
    ```  
    密码保存位置为：~/.vnc/passwd  
    - 创建服务  
    ```  
    $ sudo vim /lib/systemd/system/x11vnc.service  
    ```  
    添加如下内容:  
    > [Unit]  
    Description=Start x11vnc at startup.  
    After=multi-user.target  
    [Service]  
    Type=simple  
    ExecStart=/usr/bin/x11vnc -auth guess -forever -loop -noxdamage -repeat -rfbauth /home/用户名/.vnc/passwd -rfbport 5900 -shared  
    [Install]  
    WantedBy=multi-user.targe  

    配置防火墙  
    ```  
    $ sudo ufw allow 5900  
    ```  
    设置X11vnc服务随系统启动  
    ```  
    $ sudo systemctl enable x11vnc.service  
    $ sudo systemctl daemon-reload  
    ```  
- 配置完成后系统功能：  
    - 能够ssh远程登入  
    - 通过vnc连接桌面  
    - vim编辑器，中文显示，中文输入  
- 将系统所有文件打包  
```  
$ sudo tar zcvf Ubuntu-ssh-x11vnc-desktop.tar.gz 系统文件所在目录  
```  
- 将压缩的系统文件加载称为docker镜像  
```  
$ sudo cat tar zcvf Ubuntu-ssh-x11vnc-desktop.tar.gz |docker impor- ubuntu16.04-desktop:v1  
```  
- 启动容器  
```  
# Docker run –it - -name test1 - -env LANG=en_US.UTF-8 ubuntu16.04-desktop:v1 /bin/bash  
```  
新建test容器shell中能够显示中文，输入中文

### 问题  
新制作模板后台启动后直接退出，无法指定端口，远程连接。
