# Drone

yum install docker

systemctl start docker

docker pull drone/drone:2

```
docker run \
  --volume=/var/lib/drone:/data \
  --env=DRONE_GITHUB_CLIENT_ID=54d4bfb6aa5dede20dce \
  --env=DRONE_GITHUB_CLIENT_SECRET=ae5b5ae6d9a80948d4a42baf3ecaaba184844408 \
  --env=DRONE_RPC_SECRET=imzrpcsec \
  --env=DRONE_SERVER_HOST=59.110.167.206 \
  --env=DRONE_SERVER_PROTO=http \
  --publish=80:80 \
  --env=DRONE_TLS_AUTOCERT=false \
  --env=DRONE_LOGS_DEBUG=true \
  --env=DRONE_USER_CREATE=username:mqzhifu,admin:true \
  --publish=443:443 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:2
```

打开网页： 59.110.167.206

正常访问GITHUB ，它会自动授权，并返回

进入UI 挑一个项目，保存，OK~

项目中新建一 .drone.yml

```
kind: pipeline
name: default
steps:
  -   name: Hello World
      image: centos
      commands:
        - echo Hello World      
```

docker pull drone/drone\-runner\-docker:1

```
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e DRONE_RPC_PROTO=http \
  -e DRONE_RPC_HOST=59.110.167.206 \
  -e DRONE_RPC_SECRET=imzrpcsec \
  -e DRONE_RUNNER_CAPACITY=2 \
  -e DRONE_DEBUG=true \
  -e DRONE_RUNNER_NAME=docker-runner \
  -e DRONE_RUNNER_ENVIRON=TEST_SERVER_HOST:114.116.212.202,ONLOINE_SERVER_HOST:0.0.0.0,PROJECT_ORI_DIR:/www/project,TARGET_GIT_PROJECT_DIR:/www/git_project,CONFIG_PROJTECT_NAME:configcenter,GIT_URL_PROTOCOL:git,GIT_URL_PREFIX://github.com/mqzhifu,ENV_FILE:/tmp/env.conf,CICD_INIT_SH_NAME:cicd_init.sh,VOLUMES_PATH:/data/www/js/ \
  -p 3000:3000 \
  --restart always \
  --name runner \
  drone/drone-runner-docker:1
```

