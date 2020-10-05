# 持续集成基础

## Workshop 准备工作

1. Install sdkman: `curl -s "https://get.sdkman.io" | bash`
2. Use sdkman install gradle: https://gradle.org/install/
3. Use sdkman install spring-cli: https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started-installing-spring-boot.html#getting-started-sdkman-cli-installation
4. 安装Linux虚拟机, 最好Ubuntu16.04; 在虚拟机中 预安装以下
    1. docker 最新版本 https://docs.docker.com/install/linux/docker-ce/ubuntu/#upgrade-docker-ce
    2. 使用镜像加速 https://www.docker-cn.com/registry-mirror
    3. docker pull jenkins/jenkins:lts

## 主题

1. Create jenkins
2. CD setup
  1. Use spring-cli to create spring-boot project(可以用自己的项目)
  2. Dockerize spring-boot project
  3. Use Jenkinsfile to deploy
3. 简单介绍一下持续集成的实践

# 启动Jenkins

先安装 docker

## 简单模式(使用容器)

```

docker pull zhpooer/jenkins_z
mkdir -p /var/jenkins_home
sudo chmod 777 /var/jenkins_home
docker rm -f jenkins
docker run -d --name jenkins -p 8080:8080 -v /var/run/docker.sock:/var/run/docker.sock -v /var/jenkins_home:/var/jenkins_home zhpooer/jenkins_z
```

## 正常模式
使用Dockerfile构建,jenkins

* 新建一个目录, 写入Dockerfile
```
FROM jenkins/jenkins:lts

USER root

RUN set -x \
        && curl -fSL "https://download.docker.com/linux/static/stable/x86_64/docker-18.03.1-ce.tgz" -o docker.tgz \
        && tar -xzvf docker.tgz \
        && mv docker/* /usr/local/bin/ \
        && rmdir docker \
        && rm docker.tgz \
        && docker -v \
        && apt-get update \
        && apt-get install -y sudo \
        && curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose \
        && chmod +x /usr/local/bin/docker-compose \
        && rm -rf /var/lib/apt/lists/*

RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers
RUN groupadd -for -g 998 docker \
             && usermod -aG docker jenkins
USER jenkins
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt
Entrypiont bash
cmd echo
```
* 在同一个目录,写入plugin.txt
```
git:latest
workflow-job:2.11
antisamy-markup-formatter
matrix-auth
blueocean
```
* 运行命令 `docker build -t jenkins_z:latest .`
* 运行
```
mkdir -p /var/jenkins_home
sudo chmod 777 /var/jenkins_home
docker rm -f jenkins
docker run -d --name jenkins -p 8080:8080 -v /var/run/docker.sock:/var/run/docker.sock -v /var/jenkins_home:/var/jenkins_home jenkins_z ls
```

```
gpasswd -a xxx docker

docker build -t xxx:xx .
docker ps
docker ps -a
docker rm -f jenkins asdfsa
docker images
docker rmi
```

## 困难模式

在主机上安装ansible, vagrant
```
git clone https://github.com/bobzhangwz/toolbox.git
cd toolbox
vagrant up
ansible-playbook install_rancher.yml
```

# CD setup

## 创建一个 Springboot 项目(也可以复用自己的项目)

0. 在 github 上新建账号
1. 安装 spring-cli
2. `spring init -d=web,devtools --build=gradle hello-spring`
3. 修改 DemoApplication
  ```
    package com.example.hellospring;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.context.EnvironmentAware;
    import org.springframework.core.env.Environment;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;
    @SpringBootApplication
    @RestController
    public class DemoApplication implements EnvironmentAware {

        private String appEnv = "staging";

        @RequestMapping("/")
        public String home() {
            return "Hello Spring World: " + appEnv;
        }

        public static void main(String[] args) {
            SpringApplication.run(DemoApplication.class, args);
        }

        @Override
        public void setEnvironment(Environment environment) {
            appEnv = environment.getProperty("APP_ENV");
        }
    }
  ```
4. `gradle test` `gradle bootRun`
5. 创建库, 并提交
```
git add .
git commit -m ''
git push
```

## Dockerize spring-boot project

### Step1

1. 安装 docker 和 docker-compose(pip install docker-compose)
2. 创建 `docker-compose.yml`
    ```
    version: "2.2"

    services:
      _base:
        image: gradle:4.7.0-jdk8
        working_dir: /app
        environment:
          APP_ENV: staging
        volumes:
          - ".:/app"
          - "gradle-cache:/home/gradle/.gradle"

      dev:
        extends:
          service: _base
        ports:
          - "8080:8080"
        command: gradle bootRun

      build:
        extends:
          service: _base
        command: gradle test

    volumes:
      gradle-cache:
    ```
3. 尝试运行 `docker-compose run --rm --service-ports dev`, 访问 `http://localhost:8080`
4. `docker commit ...;docker push`

### Step 2

1. 在创建auto文件夹, 并在目录下创建 `dev` `release` `deploy` `test` 四个文件, 参考 https://github.com/bobzhangwz/hello-spring/tree/master/auto
2. `chmod a+x auto/*`
3. 可以尝试 `./auto/dev` 运行代码, `./auto/test` 测试代码
4. 提交

## 创建 Pipeline 以及 Jenkinsfile

```
pipeline {
  agent any
  stages {
    stage('Test') {
      steps {
        sh './auto/test'
      }
    }
    stage('Release') {
      when {branch 'master'}
      steps {
        sh './auto/release'
      }
    }
    stage('Staging') {
      when {branch 'master'}
      environment {
        PORT = '81'
        APP_ENV = 'staging'
      }
      steps {
        sh './auto/deploy'
      }
    }
    stage('Production') {
      when { branch 'master'}
      environment {
        APP_ENV = 'prod'
        PORT = '80'
      }
      steps {
        input 'Deploy?'
        sh './auto/deploy'
      }
    }
  }
}
```
