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

yum%20install%20docker%0Asystemctl%20start%20docker%0Adocker%20pull%20drone%2Fdrone%3A2%20%20%0A%0A%0A%60%60%60%0Adocker%20run%20%5C%0A%20%20\-\-volume%3D%2Fvar%2Flib%2Fdrone%3A%2Fdata%20%5C%0A%20%20\-\-env%3DDRONE\_GITHUB\_CLIENT\_ID%3D54d4bfb6aa5dede20dce%20%5C%0A%20%20\-\-env%3DDRONE\_GITHUB\_CLIENT\_SECRET%3Dae5b5ae6d9a80948d4a42baf3ecaaba184844408%20%5C%0A%20%20\-\-env%3DDRONE\_RPC\_SECRET%3Dimzrpcsec%20%5C%0A%20%20\-\-env%3DDRONE\_SERVER\_HOST%3D59.110.167.206%20%5C%0A%20%20\-\-env%3DDRONE\_SERVER\_PROTO%3Dhttp%20%5C%0A%20%20\-\-publish%3D80%3A80%20%5C%0A%20%20\-\-env%3DDRONE\_TLS\_AUTOCERT%3Dfalse%20%5C%0A%20%20\-\-env%3DDRONE\_LOGS\_DEBUG%3Dtrue%20%5C%0A%20%20\-\-env%3DDRONE\_USER\_CREATE%3Dusername%3Amqzhifu%2Cadmin%3Atrue%20%5C%0A%20%20\-\-publish%3D443%3A443%20%5C%0A%20%20\-\-restart%3Dalways%20%5C%0A%20%20\-\-detach%3Dtrue%20%5C%0A%20%20\-\-name%3Ddrone%20%5C%0A%20%20drone%2Fdrone%3A2%0A%60%60%60%0A%20%20%0A%20%20%0A%E6%89%93%E5%BC%80%E7%BD%91%E9%A1%B5%EF%BC%9A%20%2059.110.167.206%0A%E6%AD%A3%E5%B8%B8%E8%AE%BF%E9%97%AEGITHUB%20%EF%BC%8C%E5%AE%83%E4%BC%9A%E8%87%AA%E5%8A%A8%E6%8E%88%E6%9D%83%EF%BC%8C%E5%B9%B6%E8%BF%94%E5%9B%9E%0A%E8%BF%9B%E5%85%A5UI%20%E6%8C%91%E4%B8%80%E4%B8%AA%E9%A1%B9%E7%9B%AE%EF%BC%8C%E4%BF%9D%E5%AD%98%EF%BC%8COK~%0A%0A%E9%A1%B9%E7%9B%AE%E4%B8%AD%E6%96%B0%E5%BB%BA%E4%B8%80%20.drone.yml%0A%0A%60%60%60%0Akind%3A%20pipeline%0Aname%3A%20default%0Asteps%3A%0A%20%20\-%20%20%20name%3A%20Hello%20World%0A%20%20%20%20%20%20image%3A%20centos%0A%20%20%20%20%20%20commands%3A%0A%20%20%20%20%20%20%20%20\-%20echo%20Hello%20World%20%20%20%20%20%20%0A%60%60%60%0A%0Adocker%20pull%20drone%2Fdrone\-runner\-docker%3A1%0A%0A%0A%60%60%60%0Adocker%20run%20\-d%20%5C%0A%20%20\-v%20%2Fvar%2Frun%2Fdocker.sock%3A%2Fvar%2Frun%2Fdocker.sock%20%5C%0A%20%20\-e%20DRONE\_RPC\_PROTO%3Dhttp%20%5C%0A%20%20\-e%20DRONE\_RPC\_HOST%3D59.110.167.206%20%5C%0A%20%20\-e%20DRONE\_RPC\_SECRET%3Dimzrpcsec%20%5C%0A%20%20\-e%20DRONE\_RUNNER\_CAPACITY%3D2%20%5C%0A%20%20\-e%20DRONE\_DEBUG%3Dtrue%20%5C%0A%20%20\-e%20DRONE\_RUNNER\_NAME%3Ddocker\-runner%20%5C%0A%20%20\-e%20DRONE\_RUNNER\_ENVIRON%3DTEST\_SERVER\_HOST%3A114.116.212.202%2CONLOINE\_SERVER\_HOST%3A0.0.0.0%2CPROJECT\_ORI\_DIR%3A%2Fwww%2Fproject%2CTARGET\_GIT\_PROJECT\_DIR%3A%2Fwww%2Fgit\_project%2CCONFIG\_PROJTECT\_NAME%3Aconfigcenter%2CGIT\_URL\_PROTOCOL%3Agit%2CGIT\_URL\_PREFIX%3A%2F%2Fgithub.com%2Fmqzhifu%2CENV\_FILE%3A%2Ftmp%2Fenv.conf%2CCICD\_INIT\_SH\_NAME%3Acicd\_init.sh%2CVOLUMES\_PATH%3A%2Fdata%2Fwww%2Fjs%2F%20%5C%0A%20%20\-p%203000%3A3000%20%5C%0A%20%20\-\-restart%20always%20%5C%0A%20%20\-\-name%20runner%20%5C%0A%20%20drone%2Fdrone\-runner\-docker%3A1%0A%60%60%60%0A%20%20%0A%0A%0A%20%20
