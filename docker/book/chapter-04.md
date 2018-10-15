# 04 도커 머신
### What is Docker Machine?
Docker Machine is a tool that lets you install Docker Engine on virtual hosts,
and manage the hosts with docker-machine commands. You can use Machine to create
Docker hosts on your local Mac or Windows box, on your company network, in your data center,
or on cloud providers like Azure, AWS, or Digital Ocean.


## 4.1 도커 머신을 사용하는 이유

### 도커 환경 어따 마련하지?
- 온프레미스 환경
- 개인 서버
- 클라우드 ex AWS
- on virtual machine

vm 은 비추. vm도 다중 가상 환경이 가능한데 것다가 도커까지 올리면 pm 장비가 어케되갓어

그래서 도커 머신을 제공

### 도커 머신이 해주는일
- 가상 머신 관리
- 각종 클라우드 서비스를 이용하여 새로운 도커 서버 생성
- AWS, Digital Ocean 등 에서 도커 서버를 동적으로 설치
- 물론 개인 서버도 도커머신에 등록해 사용 가능


## 4.2 도커 머신 사용
설치는 이미 되어 있어서 패스 - 언제 설치했지
```
generalkim@Generalui-MacBook-Pro-2 ~ > docker-machine version
docker-machine version 0.15.0, build b48dc28d
```


도커 서버 확인 - 도커 머신으로 띄우야 보이나 보다. 도커로 직접 띄운 애들은 안보임
```
generalkim@Generalui-MacBook-Pro-2 ~ > docker-machine ls
NAME   ACTIVE   DRIVER   STATE   URL   SWARM   DOCKER   ERRORS
```


도커 머신으로 재미로 만들어 봄 - default 드라이버는 virtualbox 인가봄
```
generalkim@Generalui-MacBook-Pro-2 ~ > docker-machine create machinetest
Creating CA: /Users/generalkim/.docker/machine/certs/ca.pem
Creating client certificate: /Users/generalkim/.docker/machine/certs/cert.pem
Running pre-create checks...
(machinetest) Image cache directory does not exist, creating it at /Users/generalkim/.docker/machine/cache...
(machinetest) No default Boot2Docker ISO found locally, downloading the latest release...
(machinetest) Latest release for github.com/boot2docker/boot2docker is v18.06.1-ce
(machinetest) Downloading /Users/generalkim/.docker/machine/cache/boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v18.06.1-ce/boot2docker.iso...
(machinetest) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
Creating machine...
(machinetest) Copying /Users/generalkim/.docker/machine/cache/boot2docker.iso to /Users/generalkim/.docker/machine/machines/machinetest/boot2docker.iso...
(machinetest) Creating VirtualBox VM...
(machinetest) Creating SSH key...
(machinetest) Starting the VM...
(machinetest) Check network to re-create if needed...
(machinetest) Found a new host-only adapter: "vboxnet1"
(machinetest) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env machinetest
```


다시 확인 - 아 역시 도커 머신으로 띄운 애들만 보이는구나. 근데 ACTIVE 가 * 이어야 쓸수 있는 도커
```
 generalkim@Generalui-MacBook-Pro-2 ~ > docker-machine ls
NAME          ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
machinetest   -        virtualbox   Running   tcp://192.168.99.100:2376           v18.06.1-ce
```


현재 선택된 도커 확인
```
docker-machine active
No active host found
```


active 시키려면 eval 명령을 사용하여 아래와 같이 변경
```
generalkim@Generalui-MacBook-Pro-2 ~ > eval $(docker-machine env machinetest)
generalkim@Generalui-MacBook-Pro-2 ~ > docker-machine active
machinetest
generalkim@Generalui-MacBook-Pro-2 ~ > docker-machine ls
NAME          ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
machinetest   *        virtualbox   Running   tcp://192.168.99.100:2376           v18.06.1-ce
```


삭제
```
docker-machine rm machinetest
```

정지
```
docker-machine stop machinetest
```

시작 - name 을 명시하지 않으면 default 가 시작 (난 default 가 없었어서 에러)
```
docker-machine start machinetest
```




## 4.2.3 클라우드에 도커 서버 생성
docker-machine create 명령으로 vm, 각종 클라우드 공급자로 부터 서버 할당 받아 도커 서버 생성 가능함다. 아래는 되는 애들.
- AWS
- Digital Ocean
- Azure
- Openstack



### 4.2.3.1 아마존 웹 서비스에 도커 서버 생성
이 명령어로 한다
```
docker-machine create \
--driver amazonec2 \
--amazone2-access-key ... \
--amazone2-secret-key ... \
--amazone2-region ap-northeast-2 \
myaws
```


### 4.2.3.2 에저에 도커 서버 생성
이 명령어로 한다
```
docker-machine create \
--driver azure \
--azure-subscriptioin-id [구독 ID] \
myazure
```


## 4.2.4 온프레미스 환경에 연결
기존 개발 서버에 연결해서 사용 가능. 도커머신에 등록한 뒤에 가상 머신이나 클라우드의 도커 서버와 같이 도커 머신에서 사용하면 됨

순서
- 연결할 서버에 docker 명령어를 사용할 수 있는 즉 root 나 sudo 권한을 가진 계정이고 서버에는 도커 엔진이 설치 되지 않은 상태
- 계정의 ~/.ssh 디렉토리에 ssh 키 쌍 파일이 있어야 하며, 이 키 쌍 파일은 도커 머신을 사용하는 호스트에도 있어야 함
- 도커 머신은 해당 SSH 키와 계정으로 서버에 접속해 도커 엔진 설치한 뒤 제어. 앤서블과 비슷


우선 키젠부터
```
ssh-keygen
```

키 복사하고, 머신 생성
```
docker-machine create --driver generic --generic-ip-address 111.111.111.111 myserver
```

키가 특정 위치에 있다면 요렇게
```
docker-machine create --driver generic \
--generic-ip-address 111.111.111.111 \
--generic-ssh-user myuser \
--generic-ssh-key ~/path/my-key.pub \
myserver
```















