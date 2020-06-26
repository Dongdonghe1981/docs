## 平台简介

* 前端采用Vue、Element UI。
* 后端采用Spring Boot、Spring Security、Redis & Jwt。
* 权限认证使用Jwt，支持多终端认证系统。
* 支持加载动态权限菜单，多方式轻松权限控制。
* 高效率开发，使用代码生成器可以一键生成前后端代码。
* 感谢[Vue-Element-Admin](https://github.com/PanJiaChen/vue-element-admin)，[eladmin-web](https://gitee.com/elunez/eladmin-web?_from=gitee_search)。


- admin/admin123  

## 部署过程

192.168.1.212    前端

192.168.1.210    后端-1

192.168.1.211    后端-2

+ 在前端安装node.js

下载安装包`http://nodejs.cn/download/`

解压缩

```shell
xz -d node-v12.18.0-linux-x64.tar.xz
tar -xvf node-v12.18.0-linux-x64.tar
```

 vi ~/.bash_profile，添加PATH

PATH=$PATH:/usr/local/node-v12.18.0-linux-x64/bin

运行`source ~/.bash_profile`

npm 就可以识别了

为npm添加淘宝镜像

`npm config set registry https://registry.npm.taobao.org`

`npm config get registry`

使用淘宝npm镜像的cnpm，安装依赖

npm install --unsafe-perm --registry=https://registry.npm.taobao.org

到ruoyi-ui目录下，打包npm run build:prod

dist目录是打包完的成品

+ 在后端安装JDK

  `tar -zxvf jdk-8u251-linux-x64.tar.gz`

  配置环境变量，将下面拷贝到~/.bash_profile

  ```shell
  export JAVA_HOME=/usr/local/jdk1.8.0_251 
  export CLASSPATH=$:CLASSPATH:$JAVA_HOME/lib/ 
  export PATH=$PATH:$JAVA_HOME/bin
  ```

  source ~/.bash_profile

+ 在后端安装Maven

`tar -zxvf apache-maven-3.6.3-bin.tar.gz`

修改/usr/local/apache-maven-3.6.3/conf/.settings文件

```xml
<localRepository>/usr/local/repo</localRepository>
<servers>
    <server>
      <id>nexus-public</id>
      <username>admin</username>
      <password>admin</password>
    </server>
    <server>
      <id>nexus-releases</id>
      <username>admin</username>
      <password>admin</password>
    </server>
    <server>
      <id>nexus-snapshots</id>
      <username>admin</username>
      <password>admin</password>
    </server>
</servers>

</mirrors>
    <mirror>
      <id>nexus-public</id>
      <mirrorOf>*</mirrorOf>
      <name>Nexus Public</name>
      <url>http://192.168.1.205:8081/repository/maven-public/</url> 
    </mirror>
</mirrors>
```

vi ~/.bash_profile

`PATH=$PATH:$HOME/bin:/usr/local/apache-maven-3.6.3/bin`

source ~/.bash_profile

+ 打包jar

  到ruoyi目录下执行`mvn package`，过一会生成的jar在 `target`目录下，将它拷贝到rouyi目录下。

+ 打war

  - 本地工程用IDEA打开，将工程下的pom.xml的`<packaging>`改成war

  - 将spring-boot内置的tomcat排除掉

    ```xml
    		<dependency>
    			<groupId>org.springframework.boot</groupId>
    			<artifactId>spring-boot-starter-tomcat</artifactId>
    			<scope>provided</scope> <!-- 发布时剔除 -->
    		</dependency>
    ```

  - 添加初始化Servlet容器类
  
    ```java
    package com.ruoyi;
    
    import org.springframework.boot.builder.SpringApplicationBuilder;
    import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
    
    public class SpringBootStartApplication extends SpringBootServletInitializer {
    
        @Override
        protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
            return builder.sources( RuoYiApplication.class );
        }
    }
    ```

  - 将修改的pom.xml和java类上传到后端服务器
  
  - 运行`mvn clean`
  
  - 运行`mvn package`
  
- 配置前端Niginx

  ```shell
  # /usr/local创建nginx目录
  mkdir nginx
  cd nginx
  # 下载依赖的库文件
  yum -y install gcc automake autoconf libtool make
  yum install gcc-c++
  yum install pcre
  yum install pcre-devel
  yum install zlib 
  yum install zlib-devel
  yum install openssl
  yum install openssl-devel
  # 下载
  wget http://nginx.org/download/nginx-1.17.10.tar.gz
  # 解压
  tar -zxvf nginx-1.17.10.tar.gz
  # 进行configure配置（使用默认配置）
  ./configure
  make && make install
  
  #设置nginx开机启动
  cd /lib/systemd/system/
  vim nginx.service
  # 添加内容
  [Unit]
  Description=nginx 
  After=network.target 
     
  [Service] 
  Type=forking 
  ExecStart=/usr/local/nginx/sbin/nginx
  ExecReload=/usr/local/nginx/sbin/nginx reload
  ExecStop=/usr/local/nginx/sbin/nginx quit
  PrivateTmp=true 
     
  [Install] 
  WantedBy=multi-user.target
  
  #设置开机自启动
  systemctl enable nginx.service
  
  #查看nginx状态
  systemctl status nginx.service
  
  # 如果是下面的状态
  [root@localhost system]# systemctl status nginx.service
  ● nginx.service - nginx
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
     Active: inactive (dead)
  
  #关闭服务器防火墙
  systemctl stop firewalld.service
  
  # 重启nginx
  pkill -9 nginx
  systemctl start nginx
  systemctl status nginx.service
  
  # 启动、停止nginx
  cd /usr/local/nginx/sbin/
  ./nginx #启动
  ./nginx -s stop #停止
  ./nginx -s quit #停止
  ./nginx -s reload 
  ```

- 配置到nginx

  修改nginx.conf
  
      第43行
          location / {
          	root   html;
          	index  index.html index.htm;
          }
      变为
      	location / {
              root   /usr/local/workspace/ruoyi-ui/dist;
              index  index.html index.htm;
          }
      第1行
          #user nobody;   ->  user root
  
- nginx重启

  ```shell
  cd /usr/local/nginx/sbin
  ./nginx -s stop
  ./nginx
  ```

- 部署后端jar

  ```shell
  # 在后台启动jar
  nohup java -jar ruoyi.jar &
  ```
  
- 配置nginx的转发路径，修改nginx.conf，添加后端的代理，然后`./nginx -s reload`

  ```conf
      location /prod-api/ {
  		proxy_set_header Host $http_host;
  		proxy_set_header X-Real-IP $remote_addr;
  		proxy_set_header REMOTE-HOST $remote_addr;
  		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  		proxy_pass http://192.168.1.210:8080/;
      }
  ```

- 改成部署war

  将war放到tomcat下的`webapps`目录下
  
- 设置根目录访问，启动tomcat，前端和后端可以连通
  
  ```xml
  <Host ....
  <Context path="/" docBase="/usr/local/apache-tomcat-8.5.56/webapps/ruoyi" reloadable="false"></Context>
  </Host>
  ```
  
- 部署后端2
  
- 修改nginx的nginx.conf
  
  ```
  # 配置集群
  upstream ruoyi{
     server 192.168.1.210:8080 weight=5;
     server 192.168.1.211:8080 weight=3;
  }
  # 修改转发到集群
  location /prod-api/ {
  	proxy_set_header Host $http_host;
  	proxy_set_header X-Real-IP $remote_addr;
  	proxy_set_header REMOTE-HOST $remote_addr;
  	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  	proxy_pass http://ruoyi/;
  }
  
  ```
  
- 启动

  - mysql和redis的服务先启动 192.168.1.202 两个docker镜像
  - 两个后端的tomcat `/usr/local/apache-tomcat-8.5.56/bin/startup.sh`
  - nginx服务 开机自动启动 启动目录`/usr/local/nginx/sbin`
  
  

