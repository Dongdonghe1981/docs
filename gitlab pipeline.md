# gitlab pipeline



## 说明

每个项目下有个一个.gitlab-ci.yml文件

这个文件就是pipeline运行时要读取的配置文件

这个文件制定了stage和job

job是有gitlab-runner运行

gitlab-runner需要安装在其他的机器上，并配置相应的runner，在gitlab里注册

当runner运行时，要从gitlab上下载代码到gitlab-runner的机器上，并执行stages和job

将执行结果返回给gitlab，并显示

## 在ubuntu上使用gitlab-runner

添加gitlab的官方仓库

```shell
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
```

下载安装

```shell
# For Debian/Ubuntu/Mint
sudo apt-get install gitlab-runner

```

**注册runner**

```shell
sudo gitlab-runner register
```

填写相关注册信息

```txt
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
http://192.168.1.203:8090/  
--从gitlab的Settings->CI/CD->Runners->Specific Runners -> Set up a specific Runner manually取得

Please enter the gitlab-ci token for this runner
xxx  
--从gitlab的Settings->CI/CD->Runners->Specific Runners -> Set up a specific Runner manually取得
--但生成的config.toml里的token会跟这个不同，不需要修改。

Please enter the gitlab-ci description for this runner
shell-runner

Please enter the gitlab-ci tags for this runner (comma separated):
shell

Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:
shell

Please enter the Docker image (eg. ruby:2.1):
alpine:latest
```

查看注册信息

```shell
# 如果以root执行runner
cat /etc/gitlab-runner/config.toml 
```



注册信息在

/etc/gitlab-runner/config.toml

```toml
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "shell-runner"
  url = "http://192.168.1.203:8090/"
  token = "_s677L86MF4g-uzsDMAc"
  executor = "shell"
  #需要手动添加
  clone_url = "http://192.168.1.203:8090/"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]

```



在项目根目录下创建.gitlab-ci.yml

```yaml
stages:
  - build
  - test
  - deploy

job1:
  stage: build
  script:
    - echo "I am job1"
    - echo "I am in build stage"

job2:
  stage: test
  script:
    - echo "I am job2"
    - echo "I am in test stage"
```



### pipelines设置

`Settings` -> `CI/CD` 

-> `Auto DevOps` -

​	-> check `Default to Auto DevOps pipeline`

​	     check `Deployment strategy`    `Continuous deployment to production`

->`Runners`

​    -> `Shared Runners` -> `Enable shared Runners`

​    ->`Specific Runners` -> 上面配置好的Runner `shell-runner` -> `Edit`

​         -> check `Run untagged jobs` -> `Save changes`





