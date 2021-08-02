# Jenkins + Gitlab + Docker for git-flow based deployment

[ ] rerun the whole process again

## Jenkins container installation

> instruction: https://www.jenkins.io/doc/book/installing/docker/

1. create docker in docker container

```sh
docker network create jenkins

docker run \
  --name jenkins-docker \
  --rm \
  --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind \
  --storage-driver overlay2
```

2. build a custom jenkins blueocean image with docker-ce-cli installed

```Dockerfile
FROM jenkins/jenkins:2.289.3-lts-jdk11
USER root
RUN apt-get update && apt-get install -y apt-transport-https \
       ca-certificates curl gnupg2 \
       software-properties-common
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
RUN apt-key fingerprint 0EBFCD88
RUN add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/debian \
       $(lsb_release -cs) stable"
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean:1.24.7 docker-workflow:1.26"
```

```sh
docker build -t my-jenkins-blueocean:1.1 .
```

3. create the jenkins-blueocean container from custom image

```sh
docker run \
  --name jenkins-blueocean \
  --rm \
  --detach \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  my-jenkins-blueocean:1.1
```

4. initialize jenkins by initial root password and admin account and etc

### Key concepts explanation

jenkins-docker: one option to allow jenkins to run docker in its jobs. it's a *docker in docker* container that provides docker service through port *2376*.

by defining `DOCKER_HOST` environment variable in container *jenkins-blueocean* running command, the container could call docker service from this url. 

### Pitfall

since when jenkins invokes a job calling docker which actually calling docker in docker through tcp, the docker performance is not ideal.




## Gitlab installation

> instruction: https://docs.gitlab.com/ee/install/docker.html#install-gitlab-using-docker-engine

1. create gitlab container

```sh
export GITLAB_HOME=/srv/gitlab

docker run --detach \
  --hostname gitlab.example.com \
  --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  gitlab/gitlab-ee:latest
```

> Note: since https is not ideal for localhost, 443 port binding is removed

2. \[optional\] use docker-compose to create the container

```yaml
services:
    web:
        image: 'gitlab/gitlab-ee:latest'
        restart: always
        hostname: 'gitlab.example.com'
        environment:
            GITLAB_OMNIBUS_CONFIG: |
            external_url 'https://gitlab.example.com'
            # Add any other gitlab.rb configuration here, each on its own line
        ports:
            - '80:80'
            - '22:22'
        volumes:
            - '$GITLAB_HOME/config:/etc/gitlab'
            - '$GITLAB_HOME/logs:/var/log/gitlab'
            - '$GITLAB_HOME/data:/var/opt/gitlab'
```

3. initialize gitlab by initial root password and etc


## integration

To allow jenkins and gitlab to communicate with each other, we need to add both container into same network, and update the related urls configuration.

> ref: https://juejin.cn/post/6844903847383547911


## Optimization

For better documentation, I transform those commands into `docker-compose.yaml`.


## Conclusion

performance matters.
