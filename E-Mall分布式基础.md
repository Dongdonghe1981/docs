# E-Mall分布式基础

## 一、环境搭建

#### 1. 安装Linux虚拟机 

1. 安装Vagrant

2. 初始化centos7系统
	
	到`https://app.vagrantup.com/boxes/search` 官方镜像，查找centos版本名，
	
	打开cmd窗口，
	
	- 运行`vagrant init centos/7`
	- 运行`vagrant up`
	- 连接虚拟机 `vagrant ssh`
	- 退出`exit`
	- 启动虚拟机`vagrant up`
	
3. 虚拟机网络配置

   - windows主机`ipconfig`

     ```cmd
     以太网适配器 VirtualBox Host-Only Network:
     
        连接特定的 DNS 后缀 . . . . . . . :
        本地链接 IPv6 地址. . . . . . . . : fe80::8c01:9e48:a45a:e57c%9
        IPv4 地址 . . . . . . . . . . . . : 192.168.137.1
        子网掩码  . . . . . . . . . . . . : 255.255.255.0
        默认网关. . . . . . . . . . . . . :
     ```

   - 修改Vagrantfile文件，跟VirtualBox一个网段的IP设置

     ```txt
       # Create a private network, which allows host-only access to the machine
       # using a specific IP.
        config.vm.network "private_network", ip: "192.168.137.10"
     ```
     
   - 重启虚拟机`vagrant reload`
     
   - 确认IP生效 `ip addr`
   
   - windows主机跟虚拟机互相ping通
   
     ```cmd
     # 主机 -> 虚拟机
     C:\Users\HP>ping 192.168.137.10
     
     正在 Ping 192.168.137.10 具有 32 字节的数据:
     来自 192.168.137.10 的回复: 字节=32 时间<1ms TTL=64
     ```
   
     ```shell
     # 虚拟机 -> 主机
     [vagrant@localhost ~]$ ping 192.168.1.105
     PING 192.168.1.105 (192.168.1.105) 56(84) bytes of data.
     64 bytes from 192.168.1.105: icmp_seq=1 ttl=63 time=0.713 ms
     ```
   
4. 安装Docker

   centos安装docker的社区版的URL  `https://docs.docker.com/engine/install/centos/`

   - 先卸载已安装的Docker

     ```shell
     sudo yum remove docker \
                       docker-client \
                       docker-client-latest \
                       docker-common \
                       docker-latest \
                       docker-latest-logrotate \
                       docker-logrotate \
                       docker-engine
     ```

   - 安装依赖包，设置docker安装地址
   
     ```shell
     #安装依赖包
     $ sudo yum install -y yum-utils
     #设置docker安装地址
     $ sudo yum-config-manager \
         --add-repo \
         https://download.docker.com/linux/centos/docker-ce.repo
     ```
   
   - 安装docker
   
     ```shell
     sudo yum install docker-ce docker-ce-cli containerd.io
     ```
   
   - 启动docker
   
       ```shell
       $ sudo systemctl start docker
       ```
   
   - 确认docker版本
     
     ```shell
     docker -v
     ```
     
   - 设置开机自动启动
     
     ```shell
     sudo systemctl enable docker
     ```
     
   - 配置镜像加速
     
     ```shell
     sudo mkdir -p /etc/docker 
     cd /etc/docker
     sudo vi daemon.json
     {
     	"registry-mirrors": [
     		"http://hub-mirror.c.163.com",
     		"https://k7da99jp.mirror.aliyuncs.com/",
     		"https://dockerhub.azk8s.cn",
     		"https://registry.docker-cn.com"
      	]
     }
     sudo systemctl restart docker
     docker info
     ```
   
5. 安装mysql
   
   - docker镜像仓库URL   `https://hub.docker.com/`

     mysql容器是一个完整的linux
   
     ```shell
      sudo docker pull mysql:5.7
      su root #密码是vagrant
      docker run -p 3306:3306 --name mysql \ #映射端口 宿主机端口：容器端口 ，--name 容器名 mysql
      -v /mydata/mysql/log:/var/log/mysql \  #将日志文件挂载到主机
      -v /mydata/mysql/data:/var/lib/mysql \  #将数据文件挂载到主机
      -v /mydata/mysql/conf:/etc/mysql \  #将配置文件挂载到主机
   -e MYSQL_ROOT_PASSWORD=123456 \  #root的密码
      -d mysql:5.7
      #mysql容器自动启动
   docker update [容器id] --restart=always
     ```
   
   - 配置mysql
   
     ```shell
     [root@localhost conf]# vi /mydata/mysql/conf/my.conf
     [client]
     default-character-set=utf8
     
     [mysql]
     default-character-set=utf8
     
     [mysqld]
     init_connect='SET collation_connection=utf8_unicode_ci'
     init_connect='SET NAMES utf8'
     character-set-server=utf8
     collation-server=utf8_unicode_ci
     skip-character-set-client-handshake
     skip-name-resolve
     ```
   
     重启mysql   `docker restart mysql`
   
     确认容器里的配置文件修改
   
     ```shell
     root@eb964df2adfc:/etc/mysql# cat my.conf
     ```
   
6. 安装redis

   ```shell
   docker pull redis
   ```

   启动redis镜像

   ```shell
   mkdir -p /mydata/redis/conf
   #提前创建好redis.conf
   touch /mydata/redis/conf/redis.conf
   
   docker run -p 6379:6379 --name redis \
   -v /mydata/redis/data:/data \
   -v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
   -d redis redis-server /etc/redis/redis.conf
   
   #进入redis的容器的客户端
   docker exec -it redis redis-cli
   
   #mysql容器自动启动
   docker update [容器id] --restart=always
   
   # redis持久化配置，否则重启redis后，redis内存中的数据会丢失
   vi redis.conf
   appendonly yes  #加入该行
   
   ```

#### 2. 开发环境配置

   1. maven

      D:\Environment\apache-maven-3.6.3\conf\settings.xml

      ```xml
          <mirror>
            <id>nexus-aliyun</id>
            <mirrorOf>central</mirrorOf>
            <name>Nexus aliyun</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
          </mirror> 
      
      
          <profile>
            <id>jdk-1.8</id>
            <activation>
      		<activeByDefault>true</activeByDefault>
              <jdk>1.8</jdk>
            </activation>
      	  <properties>
      		<maven.compiler.source>1.8<maven.compiler.source>
      		<maven.compiler.target>1.8<maven.compiler.target>
      		<maven.compiler.compilerVersion>1.8<maven.compiler.compilerVersion>
      	  </properties>
          </profile>
      ```

2. IDEA配置
   
   - 配置Maven
   - 安装插件 `lombok`和`mybatisx`和`gitee`
   - 安装vscode插件
   
8. 配置git

   配置用户名和邮箱

   码云 `gitee.com`，配置公钥

   测试是否成功：`ssh -T git@gitee.com`

   ```shell
   $ ssh -T git@gitee.com
   The authenticity of host 'gitee.com (212.64.62.174)' can't be established.
   ECDSA key fingerprint is SHA256:FQGC9Kn/eye1W8icdBgrQp+KkGYoFgbVr17bmjey0Wc.
   Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
   Warning: Permanently added 'gitee.com,212.64.62.174' (ECDSA) to the list of known hosts.
   Hi Dongdonghe! You've successfully authenticated, but GITEE.COM does not provide shell access.
   
   ```

#### 3.微服务项目创建

使用IDEA的`spring initalizer`创建微服务项目`emall`，并创建子模块`emall-product`等。

修改`.gitignore`，不必要的文件不进行版本管理。

配置IDEA的VCS，连接到码云，进行版本控制。

#### 4. 生成微服务数据库



## 二、快速开发

#### 1.搭建管理控制台

使用码云上的`人人开源`项目`https://gitee.com/renrenio`，进行快速搭建。

1. 克隆`renren-fast`和`renren-fast-vue`

2. 导入renren-fast使用的数据库，修改数据源配置

3. 安装nodejs，配置npm镜像(已有不需安装，镜像已经配置)，在项目目录下运行`npm install`安装依赖，再运行`npm run dev`

4. 登录管理控制台admin/admin

#### 2.使用`renren-generator`生成代码

#### 3.整合mybatis-plus

## 三、SpringCloud Alibaba

Nacos :`http://127.0.0.1:8848/nacos`  nacos/nacos

GitHub:`https://github.com/alibaba/spring-cloud-alibaba`

#### Nacos服务注册与发现

1. 引用OpenFeign和nacos

   ```xml
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-openfeign</artifactId>
           </dependency>
   
   ```

   

2. 编写一个接口，该接口用于调用远程服务

3. 声明接口的方法，调用哪个远程服务的哪个请求

   ```java
   @FeignClient("emall-coupon")
   public interface CouponFeignService {
   
       @RequestMapping("/coupon/coupon/member/list")
       public R membercoupons();
   }
   
   ```

   

4. 开启远程调用功能

   ```java
   @EnableFeignClients(basePackages = "com.wh.emall.member.feign")
   @SpringBootApplication
   @EnableDiscoveryClient
   public class EmallMemberApplication {
   }
   ```

#### Nacos配置中心

1. 引入依赖

   ```xml
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
           </dependency>
   ```

2. 创建`bootstrap.properties`

      ```properties
      spring.application.name=emall-coupon
      spring.cloud.nacos.config.server-addr=127.0.0.1:8848
      ```

3. 给配置中心添加一个数据集`emall-coupon.properties`。默认规则，[应用名.properties]

4. 给[应用名.properties]添加配置

5. 动态获取配置

      @RefreshScop：给Controller类动态获取配置

      @Value("${配置相的名}")：获取得到配置

         如果配置中心和应用的配置文件都配置了相同的项目，优先使用配置中心的配置

#### Nacos相关概念

##### 一、命名空间

配置隔离

默认是public（保留空间），默认新增的所有配置都在public空间。

1. 开发、测试、生产环境可以利用命名空间进行隔离

   注意：在bootstrap.properties里指定使用的命名空间

```proper
#命名空间的id
spring.cloud.nacos.config.namespace=0c71b54f-c1aa-49de-a66c-68c0419dd9a3
```

2. 每个微服务之间的相互隔离，每个微服务只加载自己的命名空间下的配置。

##### 二、配置集

所有配置的集合

##### 三、配置集ID

类似文件名  Data ID

##### 四、配置分组

默认所有的配置集都属于：DEFAULT_GROUP;

```prop
spring.cloud.nacos.config.group=dev
```

最佳实践：每个微服务创建自己的命名空间，使用配置分组区分环境dev,test,prod

##### 五、同时加载多个配置集

1. 微服务任何配置信息，都可以放到配置中心

2. 在bootstrap.properties说明加载配置中心的哪些配置文件

   ```properties
   spring.cloud.nacos.config.namespace=coupon_ns
   spring.cloud.nacos.config.group=dev
   
   spring.cloud.nacos.config.extension-configs[0].data-id=datasource.yml
   spring.cloud.nacos.config.extension-configs[0].group=dev
   spring.cloud.nacos.config.extension-configs[0].refresh=true
   
   spring.cloud.nacos.config.extension-configs[1].data-id=mybatis.yml
   spring.cloud.nacos.config.extension-configs[1].group=dev
   spring.cloud.nacos.config.extension-configs[1].refresh=true
   
   spring.cloud.nacos.config.extension-configs[2].data-id=other.yml
   spring.cloud.nacos.config.extension-configs[2].group=dev
   spring.cloud.nacos.config.extension-configs[2].refresh=true
   ```

3. 可以通过@Value,@ConfigurationProperties获取配置文件的信息，配置中心优先使用

Nacos官方文档`https://nacos.io/zh-cn/docs/what-is-nacos.html`

#### SpringCloud Gateway