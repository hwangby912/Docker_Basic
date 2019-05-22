# Ubuntu에서 Docker의 기초 다지기

- HTTP 요청에 대해 응답하는 Docker Image 생성 방법

  - Step1. 작업 Directory 생성

  ```shell
  mkdir docker
  cd docker
  vi main.go
  ```

  /root/docker/main.go

  ```go
  package main
  
  import (
          "fmt"
          "log"
          "net/http"
  )
  
  func main() {
          http.HandleFunc("/", func(w http.ResponseWriter, r * http.Request) {
                  log.Println("received request")
                  fmt.Fprintf(w, "Hello Docker!!! ^^7 Docker is difficult\n")
          })
  
          log.Println("Start server")
          server := &http.Server{Addr : ":8080"}
          if err := server.ListenAndServe(); err != nil {
                  log.Println(err)
          }
  }
  
  ```

  8080 port로 HTTP 요청을 대기하였다가 HandleFunc의 첫번째 인자값 주소로 요청이 들어오면 fmt.Fprintf의 두번째 인자값 내용으로 응답한다. 

  - Step2. Ubuntu 가상머신에서 main.go 테스트

  /root/docker

  ```shell
  apt install golang-go
  go run main.go
  ```

  main.go를 실행할 수 있는지 확인하기 위해 golang-go package 설치를 하고 main.go를 수행한다. 

  ```shell
  curl http://localhost:8080
  ```

  localhost의 8080 port로 접근하여 서버에 접근하는 code이며, Fprintf의 두번째 인자값이 나와야 정상적인 출력이다. 

  - Step3. Dockerfile 작성

  /root/docker/Dockerfile

  ```shell
  vi Dockerfile
  ```

  ```shell
  FROM golang:1.9
  RUN mkdir /echo
  COPY main.go /echo
  CMD ["go", "run", "echo/main.go"]
  ```

  FROM은 어떤 이미지를 기반으로 할지 설정. <이미지 이름>:<태그> 형식을 취함

  RUN는 Shell Script 혹은 명령을 실행함

  CMD는 컨테이너가 시작되었을때, 실행할 실행파일 혹은 Script

  Script형식인 경우 list형태로 쓴다. 

  - Step4. Docker Image Build

  /root/docker

  명령어 형식

  ```shell
  docker image build -t IMAGE_NAME[:TAG] DOCKERFILE_PATH
  ```

  IMAGE_NAME[:TAG]을 DOKERFILE_PATH에서 Dockerfile을 찾아 build함

  ```shell
  docker image build -t example/echo:latest .
  ```

  -t: 이미지의 이름을 지정

  -f: 기본적으로 build를 할 때에는 Dockerfile을 이용한다. 하지만 사용자가 원하는 file을 이용하는 경우 "-f 파일이름"으로 옵션을 설정한다. 

  "."(현재 Directory)를 써주는 것이 엄청 중요함(왜냐하면 Dockerfile이 "."에 있기 때문)

  - Step5. main.go 내용을 수정 후 다시 빌드할 경우 일어나는 일

  ```shell
  docker image ls
  root@server:~/docker# docker image ls
  REPOSITORY           TAG                 IMAGE ID            CREATED                  SIZE
  example/mysql-data   latest              7132851e4633        Less than a second ago   1.2MB
  example/echo         latest              f989794dfabf        31 seconds ago           750MB
  busybox              latest              64f5d945efcc        11 days ago              1.2MB
  nginx                latest              53f3fd8007f7        13 days ago              109MB
  mysql                5.7                 7faa3c53e6d6        13 days ago              373MB
  golang               1.9                 ef89ef5c42a9        10 months ago            750MB
  ```

  기존의 docker image

  ```shell
  root@server:~/docker# docker image build -t example/echo:latest .
  Sending build context to Docker daemon  10.24kB
  Step 1/4 : FROM golang:1.9
   ---> ef89ef5c42a9
  Step 2/4 : RUN mkdir /echo
   ---> Using cache
   ---> 9adcf10c3cb4
  Step 3/4 : COPY main.go /echo
   ---> be560cd5e5ad
  Removing intermediate container 1d1cd97e0e38
  Step 4/4 : CMD go run /echo/main.go
   ---> Running in 4ffd70ee9d7a
   ---> c04e7a73ae06
  Removing intermediate container 4ffd70ee9d7a
  Successfully built c04e7a73ae06
  
  ```

  main.go를 수정한 뒤 example/echo:latest로 build하였으며 DOCKERFILE_PATH를 "."(현재 디렉토리)로 해주었다. 

  ```shell
  root@server:~/docker# docker image ls
  REPOSITORY           TAG                 IMAGE ID            CREATED                  SIZE
  example/mysql-data   latest              7132851e4633        Less than a second ago   1.2MB
  example/echo         latest              c04e7a73ae06        7 seconds ago            750MB
  <none>               <none>              f989794dfabf        2 minutes ago            750MB
  busybox              latest              64f5d945efcc        11 days ago              1.2MB
  nginx                latest              53f3fd8007f7        13 days ago              109MB
  mysql                5.7                 7faa3c53e6d6        13 days ago              373MB
  golang               1.9                 ef89ef5c42a9        10 months ago            750MB
  
  ```

  main.go를 수정하기 전에 build를 한 example/echo는 정상적으로 되어있다. 하지만 main.go를 수정한 뒤, 그대로 build를 하였을 때, f989794dfabf에 대한 repository와 size가 none으로 변한 것을 볼 수 있다. 

  위의 해결 방안 Code 형식

  ```shell
  docker image tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
  ```

  SOURCE_IMAGE[:TAG]를 참조하는 TARGET_IMAGE[:TAG] 태그를 만듭니다. 

  ```
  root@server:~/docker# docker image tag example/echo:latest example/echo:1.0
  root@server:~/docker# docker image ls
  REPOSITORY           TAG                 IMAGE ID            CREATED                  SIZE
  example/mysql-data   latest              7132851e4633        Less than a second ago   1.2MB
  example/echo         1.0                 c04e7a73ae06        17 minutes ago           750MB
  example/echo         latest              c04e7a73ae06        17 minutes ago           750MB
  <none>               <none>              f989794dfabf        19 minutes ago           750MB
  busybox              latest              64f5d945efcc        11 days ago              1.2MB
  nginx                latest              53f3fd8007f7        13 days ago              109MB
  mysql                5.7                 7faa3c53e6d6        13 days ago              373MB
  golang               1.9                 ef89ef5c42a9        10 months ago            750MB
  ```

  해당 명령어로 인해 기존 example/echo:latest는 살아있으며, tag가 1.0인 example/echo가 생성되었다. 또한, 이 2개의 Image ID는 동일하다. 

- Docker 실행, 정지, 재실행

  - Docker container run 

  명령어 형식

  ```shell
  docker container run OPTION IMAGE_NAME EXEC_FILE
  ```

  ```shell
  root@server:~/docker# docker container run -it -d -p 9000:8080 example/echo:latest
  8135079a7ab294476bb2093ff345db7ba35c8e9558126254e125d98e373d6d0d
  root@server:~/docker# curl http://localhost:9000
  Hello Docker!!! ^^ Docker is difficultl
  root@server:~/docker#docker container ls
  CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                    NAMES
  8135079a7ab2        example/echo:latest   "go run /echo/main.go"   50 seconds ago      Up 49 seconds       0.0.0.0:9000->8080/tcp   clever_goldstine
  ```

  run의 속성들에 대해 알아보자. 

  -d(-detach): Docker의 container를 background process로 실행하는 옵션이다. 

  -p(-port): Host의 port와 docker container가 노출한 port를 binding 시킨다. code에서는 "9000:8080"으로 9000 port를 8080으로 binding 시켰지만 "8080"만 쓴다면 port는 임의로 설정이 된다. 

  --name: docker container에 대한 NAMES를 사용자가 지정할 수 있는 옵션. 이 옵션을 지정 안하면 알아서 지정된다. 

  -i(-interactive), -t(-psuedo-tty): 옵션을 사용하면 실행할 파일에 입력 및 출력이 가능하게 된다. 

  - Docker Container 정지

  명령어 형식

  ```shell
  docker container stop CONTAINER_ID(= NAMES)
  ```

  ```shell
  root@server:~/docker# docker container stop 8135
  8135
  root@server:~/docker# docker container ls -a
  CONTAINER ID        IMAGE                 COMMAND                  CREATED              STATUS                     PORTS               NAMES
  8135079a7ab2        example/echo:latest   "go run /echo/main.go"   About a minute ago   Exited (2) 8 seconds ago                       clever_goldstine
  ```

  docker container ls -a의 명령어로 중지(Exit)가 된 것을 볼 수 있다. 

  - Docker Container 재실행

  명령어 형식

  ```shell
  docker container restart CONTAINER_ID
  ```

  ```shell
  root@server:~/docker# docker container restart 8135
  8135
  root@server:~/docker# docker container ls
  CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                    NAMES
  8135079a7ab2        example/echo:latest   "go run /echo/main.go"   6 minutes ago       Up 5 seconds        0.0.0.0:9000->8080/tcp   clever_goldstine
  ```

  docker container ls 명령어로 확인한 결과, 다시 실행되는 것을 확인할 수 있다. 

  - Docker Attach 

  명령어 형식

  ```shell
  docker attach CONTAINER_ID(= NAMES)
  ```

  수행중인 CONTAINER_ID와 붙는 역할을 한다. 

  ```shell
  root@server:~/docker# docker container run -d -it busybox /bin/sh
  5a11f22747d37129645ef329669ef93fbb7a0800bae7e821219f56d38d03ef12
  root@server:~/docker# docker container ls
  CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                    NAMES
  5a11f22747d3        busybox               "/bin/sh"                19 seconds ago      Up 18 seconds                                pedantic_hodgkin
  root@server:~/docker# docker attach 5a11
  / # ls
  bin   dev   etc   home  proc  root  sys   tmp   usr   var
  / # exit
  root@server:~/docker# docker container ls -a
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
  5a11f22747d3        busybox             "/bin/sh"           9 minutes ago       Exited (0) 6 seconds ago                       pedantic_hodgkin
  ```

  busybox를 기본 이미지로 하는 container를 실행한 뒤, shell을 실행한다.  그 후, 잘 수행되는지 확인한다. 

  수행중인 container와 attach를 하며 그 안에서의 명령어 ls를 친 결과가 나온다. 바로 exit를 하게 되면 해당 container가 중지 상태로 넘어가 있는 것을 알 수 있다. 

- Docker Container 명령어 실행, 파일 복사, 현황 조회

  - 실행중인 Container에 명령어 실행 방법

  명령어 형식

  ```shell
  docker container exec CONTAINER_ID CMD 
  ```

  수행중인 CONTAINER_ID에 대해서 CMD를 수행한다. 

  ```shell
  root@server:~/docker# docker restart 5a11
  5a11
  root@server:~/docker# docker container exec 5a11 ls
  bin
  dev
  etc
  home
  proc
  root
  sys
  tmp
  usr
  var
  ```

  위에서 재실행한 container에 대해서 ls를 전달한 결과이다. 

  - Host와 Container 간의 File 복사 방법

  명령어 형식

  ```shell
  docker container cp Directory CONTAINER_ID:/Directory
  ```

  여기서 Directory는 File도 가능하다. 

  ```shell
  root@server:~/docker# touch test
  root@server:~/docker# docker container cp ./test 5a11:/
  root@server:~/docker# docker container exec 5a11 ls
  bin
  dev
  etc
  home
  proc
  root
  sys
  test
  tmp
  usr
  var
  ```

  "./test"(현재 Directory/test File)을 CONTAINER_ID(5a11)의 "/" Directory에 복사하였다. 그 후 exec을 사용하여 test File이 제대로 복사가 되었다는 것을 확인할 수 있다.

  역으로 Container의 File(혹은 Directory) 를 Host로 복사하는 방법은 역으로 하면 된다. 

  ```shell
  root@server:~/docker# docker container cp 5a11:/home ./chome
  root@server:~/docker# ls
  Dockerfile  chome               index.html  nginx    test
  a-ban       docker-compose.yml  main.go     runc.sh
  ```

  다음과 같이 chome이 복사된 것을 알 수 있다. 

  - Container 사용 현황 조회

  해당 명령어는 현재 수행중인 Container들의 resource 사용 통계를 표시해 줍니다. 

  명령어 형식

  ```shell
  root@server:~/docker# docker stats
  ```

- Docker-Composer

  - docker-compose란 한번에 여러개의 Container를 실행하고 통합 관리 할 수 있게 하는 툴이다. 
  - Docker Compose 정의는 YAML 파일 형식으로 작성하며, 기본 파일명은 docker-compose.yml이다. 
  
  여러 Container를 실행하기 위한 YAML File은 다음과 같다. 
  
  ```shell
  root@server:~/docker# vi docker-compose.yml
  version: "3"
  services:
      echo1:
          build: .
          ports:
             - 8080
      echo2:
          build: .
          ports:
             - 8080
      hello1:
          build: .
          ports:
             - 8080
      hello2:
          build: .
          ports:
             - 8080
  ```
  
  echo1, echo2, hello1, hello2는 Container에 해당하며, build대신 image를 쓴다면 Docker Hub에 등록한 IMAGE_NAME[:TAG]로 하면 된다. 
  
  ```shell
  root@server:~/docker# docker-compose up
  Building hello1
  Step 1/4 : FROM golang:1.9
   ---> ef89ef5c42a9
  Step 2/4 : RUN mkdir /echo
   ---> Using cache
   ---> a51f9d1873ed
  Step 3/4 : COPY main.go /echo
   ---> Using cache
   ---> b3f2e8e0c8d0
  Step 4/4 : CMD go run /echo/main.go
   ---> Using cache
   ---> a246325d63f4
  Successfully built a246325d63f4
  Successfully tagged docker_hello1:latest
  WARNING: Image for service hello1 was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
  Building hello2
  Step 1/4 : FROM golang:1.9
   ---> ef89ef5c42a9
  Step 2/4 : RUN mkdir /echo
   ---> Using cache
   ---> a51f9d1873ed
  Step 3/4 : COPY main.go /echo
   ---> Using cache
   ---> b3f2e8e0c8d0
  Step 4/4 : CMD go run /echo/main.go
   ---> Using cache
   ---> a246325d63f4
  Successfully built a246325d63f4
  Successfully tagged docker_hello2:latest
  WARNING: Image for service hello2 was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
  Starting docker_echo1_1  ... done
  Creating docker_hello1_1 ... done
  Starting docker_echo2_1  ... done
  Creating docker_hello2_1 ... done
  Attaching to docker_echo1_1, docker_hello1_1, docker_hello2_1, docker_echo2_1
  hello2_1  | 2019/05/22 01:59:03 Start server
  hello1_1  | 2019/05/22 01:59:03 Start server
  echo1_1   | 2019/05/22 01:59:04 Start server
  echo2_1   | 2019/05/22 01:59:04 Start server
  ```
  
  명령어를 수행하면 다음과 같이 해당 Container들이 수행되는 것을 알 수 있다. 
  
  - 여러 Container 생성
  
  명령어 형식
  
  ```shell
  root@server:~/docker# docker-compose up --scale NAMES=NUMBER
  ```
  
  ```shell
  root@server:~/docker# docker-compose up --scale echo1=10
  Starting docker_hello1_1 ... done
  Starting docker_echo2_1  ... done
  Starting docker_echo1_1  ... done
  Starting docker_hello2_1 ... done
  Creating docker_echo1_2  ... done
  Creating docker_echo1_3  ... done
  Creating docker_echo1_4  ... done
  Creating docker_echo1_5  ... done
  Creating docker_echo1_6  ... done
  Creating docker_echo1_7  ... done
  Creating docker_echo1_8  ... done
  Creating docker_echo1_9  ... done
  Creating docker_echo1_10 ... done
  Attaching to docker_hello1_1, docker_hello2_1, docker_echo2_1, docker_echo1_1, docker_echo1_6, docker_echo1_3, docker_echo1_9, docker_echo1_4, docker_echo1_10, docker_echo1_7, docker_echo1_2, docker_echo1_8, docker_echo1_5
  hello1_1  | 2019/05/22 02:15:56 Start server
  hello2_1  | 2019/05/22 02:15:56 Start server
  echo2_1   | 2019/05/22 02:15:57 Start server
  echo1_1   | 2019/05/22 02:15:56 Start server
  echo1_6   | 2019/05/22 02:16:08 Start server
  echo1_9   | 2019/05/22 02:16:08 Start server
  echo1_4   | 2019/05/22 02:16:09 Start server
  echo1_2   | 2019/05/22 02:16:10 Start server
  echo1_7   | 2019/05/22 02:16:10 Start server
  echo1_10  | 2019/05/22 02:16:10 Start server
  echo1_3   | 2019/05/22 02:16:11 Start server
  echo1_8   | 2019/05/22 02:16:11 Start server
  echo1_5   | 2019/05/22 02:16:11 Start server
  ```
  
  기존 YAML File Container를 수행하면서 echo1=10을 주면서 echo1_1 ~ echo1_10까지 수행 되는것을 볼 수 있다. 
  
  - 여러 Container의 시작 / 정지 / 재시작 / 일시정지 /재개
  
  명령어 형식
  
  ```shell
  root@server:~/docker# docker-compose stop
  root@server:~/docker# docker-compose restart
  root@server:~/docker# docker-compose pause
  root@server:~/docker# docker-compose unpause
  root@server:~/docker# docker-compose down
  ```
  
  수행되고 있는 Container가 많으므로 Container 정지 등에 대한 자동으로 나오는 Code는 생략하며 위의 명령어를 수행하면 정상적으로 수행할 수 있는 것을 확인할 수 있다. 
  
- Image Docker Hub 등록 및 Commit

  - 등록을 위해 Log-In을 해야함 

  명령어 형식

  ```
  root@server:~/docker# docker login -u USER_ID
  ```

  - Docker Image 등록

  명령어 형식

  ```shell
  root@server:~/docker# docker image push USER_ID/testing:latest
  ```

  명령어 형식을 적용한 Code는 다음과 같다. 

  ```shell
  root@server:~/docker# docker image push rnlfus/testing:latest
  The push refers to a repository [docker.io/rnlfus/testing]
  415308c86f3e: Pushed 
  99e48ac59bb5: Pushed 
  186d94bd2c62: Mounted from rnlfus/echo 
  24a9d20e5bee: Mounted from rnlfus/echo 
  e7dc337030ba: Mounted from rnlfus/echo 
  920961b94eb3: Mounted from rnlfus/echo 
  fa0c3f992cbd: Mounted from rnlfus/echo 
  ce6466f43b11: Mounted from rnlfus/echo 
  719d45669b35: Mounted from rnlfus/echo 
  3b10514a95be: Mounted from rnlfus/echo 
  latest: digest: sha256:195763ad0578b50ef71cb40b2de0c0efb115154840ef9d96ec7afff98a596fbf size: 2417
  ```

  명령어를 수행한 뒤, Docker Hub에 올라간 것을 확인할 수 있다. 

  ![1558498208580](C:\Users\HBY\AppData\Roaming\Typora\typora-user-images\1558498208580.png)
  - Docker Commit Container에서 Image 생성

  명령어 형식

  ```shell
  docker container commit OPTION CONTAINER_ID USER_ID/IMAGE_NAME[:TAG]
  ```

  CONTAINER_ID에 대하여 USER_ID/IMAGE_NAME[:TAG]의 Image를 만들어 낸다. 

  이 명령어 형식을 적용한 Code는 다음과 같다

  ```shell
  root@server:~/docker# docker container commit --author "rnlfus" --message "echo web" 3ec0 rnlfus/hellodocker:1.0
  sha256:0e1e26ff45cf7ae4b19ef87f9199da61ab074b23a3bc501f8bba38e50ab5913d
  root@server:~/docker# docker image ls
  REPOSITORY           TAG                 IMAGE ID            CREATED              SIZE
  rnlfus/hellodocker   1.0                 0e1e26ff45cf        About a minute ago   754MB
  example/echo         latest              60986152610e        35 minutes ago       750MB
  rnlfus/testing       latest              60986152610e        35 minutes ago       750MB
  nginx                latest              53f3fd8007f7        2 weeks ago          109MB
  golang               1.9                 ef89ef5c42a9        10 months ago        750MB
  ```

  3eco에 대해서 rnlfus/hellodocker:1.0의 Image를 만들어 낸 결과이다. 

- Docker 배운 것을 기반으로 실습한 것
  - nginx를 Image로 하여 hello.html 파일을 test Directory에서 띄우기
- nginx에서 실행되는 root Directory는 /usr/share/nginx/html이다. 
  - port는 7988로 하며 80으로 Binding하자. 
  - 주소를 http://localhost:7988/testdirectory/hello.html로 주었을때, 다음과 같이 나오게 해보자. 
  
  ![1558535024353](C:\Users\HBY\AppData\Roaming\Typora\typora-user-images\1558535024353.png)
  
  - root/docker/testdirectory/hello.html의 Code는 다음과 같다. 
  
  ```html
  <html>
  <head>
  	<meta charset="utf-8">
  </head>
  <body>
	<h1>Hello This is hello file</h1>
  </body>
  </html>
  ```
  
  Answer Code
  
  ```shell
  root@server:~/docker# docker image pull nginx:latest
  latest: Pulling from library/nginx
  743f2d6c1f65: Pull complete 
  6bfc4ec4420a: Pull complete 
  688a776db95f: Pull complete 
  Digest: sha256:23b4dcdf0d34d4a129755fc6f52e1c6e23bb34ea011b315d87e193033bcd1b68
  Status: Downloaded newer image for nginx:latest
  root@server:~/docker# docker container run -it -d -p 7988:80 nginx
  dfeed76e6e3bfc6a467451284b76572ccec86004f76d7845b15f45290050785a
  root@server:~/docker# docker container ls
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
  dfeed76e6e3b        nginx               "nginx -g 'daemon ..."   2 minutes ago       Up 2 minutes        0.0.0.0:7988->80/tcp   loving_ride
  root@server:~/docker# docker container exec dfeed ls -l /usr/share/nginx/html/
  total 8
  -rw-r--r-- 1 root root 494 Apr 16 13:08 50x.html
  -rw-r--r-- 1 root root 612 Apr 16 13:08 index.html
  root@server:~/docker# docker container cp ./testdirectory dfee:/usr/share/nginx/html/
  root@server:~/docker# docker container exec dfee ls -l /usr/share/nginx/html/
  total 12
  -rw-r--r-- 1 root root  494 Apr 16 13:08 50x.html
  -rw-r--r-- 1 root root  612 Apr 16 13:08 index.html
  drwxr-xr-x 2 root root 4096 May 22 14:19 testdirectory
  root@server:~/docker# docker container exec dfee ls -l /usr/share/nginx/html/testdirectory
  total 4
  -rw-r--r-- 1 root root 104 May 22 14:19 hello.html
  ```
  
  Answer Code KeyPoint Review
  
  ```shell
  root@server:~/docker# docker container exec dfeed ls -l /usr/share/nginx/html/
  # dfee Container의 /usr/share/nginx/html의 ls 결과 index.html을 볼수 있다. 
  ```
  
  밑의 사진의 파일 경로는 Container_ID/usr/share/nginx/html/index.html이다. 
  
  ![1558535682829](C:\Users\HBY\AppData\Roaming\Typora\typora-user-images\1558535682829.png)
  
  testdirectory가 아니라 usr이라는 Directory인 경우에 실수를 할 가능성은 다음과 같다. 
  
  ```shell
  root@server:~/docker# docker container cp ./usr dfee:/usr
  ```
  
  위의 Code는 어찌보면 맞는 것 같지만 이렇게 하면 403 Forbidden 오류를 일으킨다. dfee:/usr은 Window에서 따지면 '사용자 계정'에 해당한다. 이 명령어는 현재 Directory안의 일반 Directory인 'usr'을 dfee Container의 사용자 Directory로 복사한다는 뜻이 된다. 유의해서 쓰도록 하자!!!!!!!!!!
  
  결과물의 url에서 http://localhost:7988까지가 nginx의 root Directory인 CONTAINER_ID/usr/share/nginx/html/ 경로까지 들어와 있다는 것을 다시 한번 유의하자(default는 index.html로 수행된다)