# Docker
요즘 세대는 개발 시작하면서 도커를 빼고 시작하면 어색하달까

옛날사람 되는건 한순간이여

사용법은 대충 알지만, 조금 더 자세히 공부해보자며 점심스터디로 시작하게 됨 



## 공부할때 참고한 책
```
시작하세요! 도커 : 기초 개념부터 최신 도구 활용법까지 망라한
```
이 책 구매시 주의할 점은 불과 몇 달 사이에 수정본이 나와서 2장쯤에 추가된 내용이 있으니 꼭 최신판 사셈 #난실패


## 일단 근본없는 막정리

docker -v

docker run -t -i ubuntu:14.04
```
ctrl + D 그냥 종료
ctrl + P, Q 컨테이너 실행은 그대로 두고 쉘에서만 빠져나옴
```

docker pull centos:7

컨테이너 실행 방법에 대해
```
docker run -t -i ubuntu:14.04
docker create -t -i --name mycentos centos:7
docker start mycentos
docker attatch mycentos
docker start dd06c5cb6df4
docker start dd0             이렇게만쳐도 실행됨 앞에 3,4자만 맞아도.
```

컨테이너 목록확인
```
docker ps
docker ps -a
```

```
docker run -i -t ubuntu:14.04 echo hello world
```
```
docker rename angry_morse my_container
docker rm angry_morse
docker rm mycentos
docker stop mycentos
docker rm mycentos
docker rm -f mycentos
```

컨테이너 다 싹 지워버리는
```
docker container prune
```
```
docker ps -a -q
```
위 명령을 이용해 이런식으로 할수도 있다. 결국 prune 과 같은 명령 아님?
```
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```

컨테이너 외부 노출
```
docker run -i -t --name network_test ubuntu:14:04
docker run -i -t --name network_test ubuntu:14:04 -p 80:80 ubuntu:14.04
docker run -i -t --name network_test ubuntu:14:04 -p 80:80 -p 192.168.0.100:7777:80 ubuntu:14.04
```

```
docker run -d \
--name wordpressdb \
-e MYSQL_ROOT_PASSWORD=password \
-e MYSQL_DATABASE=wordpress \
mysql:5.7

docker run -d \
-e WORDPRESS_DB_PASSWORD=password \
--name wordpress \
--link wordpressdb:mysql \
-p 80 \
wordpress

docker port wordpress
```

-i, -t 는 내부로 진입하여 상호작용 하려는 attach 모드라면

-d 는 detached 모드로 컨테이너를 백그라운드에서 실행하는것
```
docker exec -i -t wordpressdb /bin/bash
```
```
docker exec wordpressdb ls /
```

--link 컨테이너간 연결. 내부 IP 알거 없이 별명으로 하면 된다
```
docker run -d \
--name wordpressdb_hostvolnume \
-e MYSQL_ROOT_PASSWORD=password \
-e MYSQL_DATABASE=wordpress \
-v /home/wordpress_db:/var/lib/mysql \
mysql:5.7

docker run -d \
-e WORDPRESS_DB_PASSWORD=password \
--name wordpress_hostvolume \
--link wordpressdb_hostvolume:mysql \
-p 80 \
wordpress
```

-v [호스트의 공유 디렉토리]:[컨테이너의 공유 디렉토리]

docker 실행한데서 /home/wordpress_db 에 가보면 /var/lib/mysql 이 들어 있다.

```
echo hello >> /home/hello && echo hello2 >> /home/hello2 >> /home/hello2
docker run -i -t \
--name file_volume \
-v /home/hello:/hello \
-v /home/heelo2:/hello2 \
ubuntu:14:04
```
-v 옵션시 호스트에 디렉토리가 없더라도 생성됨. 컨테이너 파일이 복사되는.

만약 호스트, 컨테이너 모두 디렉토리가 존재할 경우 컨테이너의 내용이 호스트의 디렉토리를 덮어쓴다. 

사실 호스트의 디렉토리를 컨테이너의 디렉토리에  마운트 하는것.
```
docker run -i -t \
--name volume_overide \
-v /home/wordpress_db:/home/testdir_2 \
alicek106/volume_test
```

볼륨컨테이너
```
docker run -i -t 
--name volumes_from_container 
--volumes-from volume_overide 
ubuntu:14.04
```

도커 볼륨
```
docker volume create --name myvolume

docker volume ls

[볼륨이름]:[컨테이너의 공유 디렉토리]


docker run -i -t --name myvolume_1 \
-v myvolume:/root/ \
ubuntu:14.04



docker run -i -t --name myvolume_2 \
-v myvolume:/root/ \
ubuntu:14.04

docker inspect --type volume myvolume

docker run -i -t --name volume_auto \
-v /root \
ubuntu:14.04

docker volume ls

create container inspect volume_auto
```
볼륨을 싸그리 지움
```
docker volume prune
```

ifconfig
veth... 이거가 가상을 의미 virtual eth

stateless 로 설계 하는것이 좋음
(컨테이너 외부에 데이터를 저장하고 컨테이너는 그 데이터로 동작)

docker0 이라는 도커 브리지

brctl show dodcker0


도커 네트워크 기능
```
docker network ls 

docker network inspect bridge
```

브리지타입 네트워크 생성. 생성된 네트워크는 --net 으로 사용
```
docker network create --driver bridge mybridge

docker run -i -t --name mynetwork_container \
--net mybridge \
ubuntu:14.04
```

서브넷, 게이트웨이, IP 할당 범위 등 설정
```
docker network create --driver=bridge \ 
--subnet=172.72.0.0/16 \
--ip-range=172.72.0.0/24 \
--gateway=172.72.0.1 \
my_custom_network
```

호스트네트워크
```
docker run -i -t --name network_host \
--net host \
ubuntu:14.04
```

아무런 네트워크를 쓰지 않는 컨테이너. 외부와 연결이 단절
```
docker run -i -t --name network_none \
--net none \
ubuntu:14.04
```


--net 옵션으로 container 를 입력하면 다른 컨테이너 네트워크 환경을 공유
```
docker run -i -t -d --name network_container_1 ubuntu:14.04

docker run -i -t -d --name network_container_2 \
--net container:network_container_1 \
ubuntu:14.04

docker exec network_container_1 ifconfig
docker exec network_container_2 ifconfig
이렇게 각각 컨테이너에 명령 날려보면 같은 eth0 정보를 쓰고 있음
```



브리지 네트워크, --net-alias

특정 호스트 이름으로 컨테이너 여러개 접근 가능(컨테이너를 한개의 도메인에 묶어버릴때 유용)
```
docker run -i -t -d --name network_alias_container1 \
--net mybridge \
--net-alias alicek106 ubuntu:14.04

docker run -i -t -d --name network_alias_container2 \
--net mybridge \
--net-alias alicek106 ubuntu:14.04

docker run -i -t -d --name network_alias_container3 \
--net mybridge \
--net-alias alicek106 ubuntu:14.04

docker inspect network_alias_container1 | grep IPAddress

docker run -i -t --name network_alias_ping \
--net mybridge \
ubuntu:14.04

ping -c 1 alicek106
ping -c 1 alicek106
ping -c 1 alicek106
```

차례로 때려보면 3개 IP 로 RR 돌림
```
--net-alias 옵션으로 alicek106설정한 컨테이너로 변환

--net-alias, --link 옵션과 비슷하게 동작

dig alicek106
```

컨테이너 로깅
```
jong-file log

docker run -d --name mysql \
-e MYSQL_ROOT_PASSWORD=1234
mysql:5.7
```

컨테이너 표준 출력확인
```
docker logs mysql
```

일부러 -e 옵션없이 mysql 이 뜨지 않게 실행 해봄
```
docker run -d --name no_passwd_mysql \
mysql 5.7
```

docker ps 로 mysql 이 만들어졌지만 docker start 로 실행안됨 이때 로그명령으로 확인
```
docker logs no_passwd_mysql
docker logs --tail 2 mysql
docker logs --since 1474765979 mysql
docker logs -f -t mysql

docker logs -i -t --name logstest ubuntu:14.04
echo test!

docker logs logstest

cat /var/lib/docker/containers/${CONTAINER_ID}/${CONTAINER_ID}-json.log
```

syslog 로그
```
docker run -d --name syslog_container \
--log-driver=syslog \
ubuntu:14.04 \
echo syslogtest

tail /var/log/syslog
```


환경설정에서 기본 로깅 드라이버를 변경할 수도 있음
```
DOCKER_OPTS="--log-driver=syslog"
```
syslog로그를 원격서버에 설치해서 로그정보를 원격으로도 보낼 수 있음

rsyslog 중앙컨테이너 로그로 저장

```
docker run -i -t \ 
-h rsyslog \
--name rsyslog_server \
-p 514:514 -p 514:514/udp \
ubuntu:14.04
```

컨테이너 내부 rsyslog.conf 파일 syslog 설정변경
```
vi /etc/rsyslog.conf
ModLod
UPDServerRun 514
InputTCPServerRun 514

service rsyslog restart

docker run -i -t \
--log-driver=syslog \
--log-opt syslog-address=tcp://192.168.0.100:514 \
--log-opt tag="mylog" \
ubuntu:14.04

echo test
```

--log-opt 로깅 드라이버의 추가 옵션

syslog-address rsyslog 컨테이너에 접근할 수 있는 주소 입력 

tag 는 로그 데이터가 기록될때 함께 저장될 태그 로그 분류를 위해 사용

```
tail /var/log/syslog

docker run -i -t \
--log-driver=syslog \
--log-opt syslog-address=udp://192.168.0.100:514 \
--log-opt tag="mylog" \
ubuntu:14.04

docker run -i -t \
--log-driver syslog \
--log-opt syslog-address=tcp://192.168.0.100:514 \
--log-opt tag="maillog" \
--log-opt syslog-facility="mail" \
ubuntu:14.04

ls /var/log
```

fluentd 로깅

기본구성으로 도커 서버 fluentd, mongo
```
docker run --name mongoDB -d \
-p 27017:27017 \
mongo
```

fluentd 서버의 호스트에서 fluent.conf 파일로 저장
```
<source>
  @type forward
</source>

<match docker.**>
  @type mongo
  database nginx
  collection access
  host 192.168.0.102
  port 27017
  flush_interval 10s

  몽고에 인증정보가 있다면 아래 기술
  user alicek106
  password mypw
</match>
```

파일 저장 후
```
docker run -d --name fluentd -p 24224:24224 \
-v $(pwd)/fluent.conf:/fluentd/etc/fluent.conf \
-e FLUENTD_CONF=fluent.conf
alicek106/fluentd:mongo
```

fluentd 설정대로라면 docker.로 시작하는 태그로깅들을 저장
```
docker run -p 80:80 -d \
--log-driver=fluentd \
--log-opt fluentd-address=192.168.0.101:24224 \
--log-opt tag=docker.nginx.webserver \
nginx
```

위 로그 태그가 docker. 로 시작하므로 위 fleuntd 를 통해 저장됨
```
docker exec -it mongoDB mongo


AWS
docker run -i -t \
--log-driver=awslogs \
--log-opt awslogs-region=ap-northeast-e \
--log-opt awslogs-group=mylogs \
--log-opt awslogs-stream=mylogstream \
ubuntu:14.04
```

2.3.1 도커 이미지 생성
``` 
docker run -i -t --name commit_test ubuntu:14.04
echo test_first! >> first

docker commit [OPTION] CONTAINER [REPOSITORY[:TAG]]

docker commit \
-a "alicek106" -m "my first commit" \
commit_test \
commit_test:first
```

-a 는 저자 author

```
docker run -t -i --name commit_test2 commit_test:first
echo test_second! >> second


docker commit \
-a "alicek106" -m "my second" \
commit_test2 \
commit_test:second

docker inspect ubuntu:14.04
docker inspect commit_test:first
docker inspect commit_test:second

docker save -o ubuntu_14_04.tar ubuntu:14.04
docker load -i ubuntu_14_04.tar

docker export -o rootFS.tar mycontainer
docker import rootFS.tar myimage:0.0
```

저장소에 올릴때는
```
docker commit commit_container1 myimagename:0.0
```

tag 를 따서 저장소이름(사용자의 이름) 을 이미지 앞에 접두어로 추가해야함
```
docker tag myimagename:0.0 601kecila/myimagename:0.0
```
```
docker login
docker push 601/myimagename:0.0
docker pull 601/myimagename:0.0
```





본격 Dockerfile

이미지 생성시 필요한 인련의 과정을 기록해두고 수행할 수 있는 빌드를 제공

dockerfile 예
```
vi Dockerfile

FROM ubuntu:14.04
MAINTAINER alicek106
LABEL "purpose"="practice"
RUN apt-get update
RUN apt-get install apache2 -y
ADD test.html /var/www/html
WORKDIR /var/www/html
RUN ["/bin/bash", "-c", "echo hello >> test2.html"]
EXPOSE 80
CMD apachetl -DFORGROUND
```


WORKDIR 명령어를 여러번 사용하면 cd 명령어를 여러번 사용한것과 같음 예를들어.

```
WORKDIR /var
WORKDIR www/html
```
은 WORK /var/www/html 과 동일함


CMD 는 컨테이너가 시작될떄마다 실행할 명령어이며 Dockerfile 내에서는 한번만 사용할 수 있음

그러나 docker run 명령어에서 커맨드 명령줄 인자를 입력하면 Dockerfile 내에서 사용한 CMD 명령어는 run 커맨드로 덮어쓰여짐.


Dockerfile 빌드
```
docker build -t mybuild:0.0 ./
```

```
docker run -d -P --name myserver mybuild:0.0
```

-P 옵션은 이미지에 설정된 EXPOSE 의 모든 포트를 호스트에 연결하도록 설정
```
docker port myserver
```

.dockerignore
```
vi .dockerignore
test2.html
*.html
*/*.html
test.htm?

*.html
!test*.html
```

레이어 캐시를 사용하지 않게 하려면 쓰는 명령어
```
docker build --no-cache -t mybuild:0.0 .
```

캐시로 부터 이미지를 생성하는 방법
```
docker build --cache-from nginx my_extend_nginx:0.0 .
```


기타 Dockerfile 명령어들
```
vi Dockerfile
FROM ubuntu:14.04
ENV test /home
WORKDIR $test
RUN touch $test/mytouchfile
```

docker run 실행시 -e 옵션을 사용할 경우 기존 값은 덮어써짐
```
docker build -t myenv:0.0 ./
docker run -t -t --name env_test myenv:0.0 /bin/bash
docker run -t -i --name env_test_override \
-e test=myvalue \
myenv:0.0 /bin/bash 

vi Dockerfile 
FROM ubuntu:14.04
ENV my_env my_value
RUN echo ${my_env:-value} / ${my_env:+value} / ${my_env:-value} / ${my_env2:+value}

docker build .
```


VOLUME : 이미지 생성시 호스트와 공유할 내부의 디렉토리 설정

VOLUME ["/home/dir","/home/dir2"] 처럼 JSON 배열의 형식으로 여러개를 사용하거나

VOLUME /home/dir /home/dir2 처럼 사용가능


```
vi Dockerfile

FROM ubuntu14.04
RUN mkdir /home/volume
RUN echo test >> /home/volume/testfile
VOLUME /home/volume

dockr build -t myvolume:0.0 .

docker run -i -t -d --name volume_test myvolume:0.0

docker volume ls
```


ARG
```
build 명령 실행시 추가로 입력받아 dockerfile 내에서 사용될 변수의 값을 설정
```


ENTRYPOINT 와 CMD 의 차이점
```
차이는 차차 차마시면서 차례차례 찾아보자..... 는 TODO: 다음에 업뎃 
```


 