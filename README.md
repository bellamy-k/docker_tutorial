# Docker Tutorial

## 실행 환경
```
windows 10 x64 1904, WSL 2, Docker Engine v20.10.7
```

## 실습 내용
### Tutorial image 실행
```
docker run -d -p 80:80 docker/getting-started
```
```
-d : detached(백그라운드 실행)
-p 80:80 : host의 80 port와 container의 80 port를 mapping
docker/getting-started : 사용할 이미지
```

### 각종 커맨드 확인 및 분석
#### 이미지 조회
```
docker images
```
![image](https://user-images.githubusercontent.com/43736669/125201284-606c6900-e2a9-11eb-95fa-84e093304bc7.png)

#### 이미지 pull
```
docker pull <IMAGENAME>
```
![image](https://user-images.githubusercontent.com/43736669/125201783-74b16580-e2ab-11eb-9a30-16c3463190e2.png)

  - docker 이미지의 이름은 단순 문자열
  - 도커 이미지 이름의 구조는 `<NAMESPACE>/<IMAGE_NAME>:<TAG>`
  - 실습 화면에서 `library` 는 도커 hub의 공식 이미지 Namespace(보통은 사용자의 이름이 옴)
  - Namespace 앞의 `/` 는 도커 이미지 저장소의 주소를 나타냄. 이 경우 `docker.io` : Docker Hub(도커 클라이언트의 기본 도커 레지스트리)
  - `nginx:latest` 가 사실 기본 도커 레지스트리가 추가된 `index.docker.io/library/nginx:latest` 로 변환된 것
  - 마지막의 sha256 digest 값도 이미지 주소를 나타냄  

  ```
  b4d181a07f80: Pull complete
  66b1c490df3f: Pull complete
  d0f91ae9b44c: Pull complete
  baf987068537: Pull complete
  6bbc76cbebeb: Pull complete
  32b766478bc2: Pull complete
  ```

  - 5줄의 Pull complete 문장과 Hash 값은 Image Layer를 의미함  
  - Image를 Pull 받으면 레이어들은 독립적으로 저장, 컨테이너를 실행할 때 이 레이어들을 차례대로 쌓아올려 특정 위치에 Mount. Layer들은 Read-Only라 **변하지 않음**
  - 레이어들 위에 마지막으로 컨테이너 전용 쓰기 가능한 레이어를 쌓아 모든 변경 사항을 저장
  - 위의 이미지는 제일 아래의 `b4d181a07f80` 레이어부터 아래의 레이어를 쌓아서 만들어진 이미지

#### 이미지 Run & Diff & Commit 
```
docker run -it nginx bash
```
```
-i : stdin 활성화
-t : tty(Bash 사용 위해, 터미널)
```
- git bash가 열림
```
root@335a1125bb70:/#
```
docker diff로 변하는 부분 확인(아직까지는 출력 x)
```
docker diff 335a1125bb70
```
파일 하나 생성
```
root@335a1125bb70:/# cd tmp
root@335a1125bb70:/tmp# touch FILE
```
diff 확인
```
docker diff 335a1125bb70
C /tmp
A /tmp/FILE
C : 변경
A : 추가
D : 삭제
```
컨테이너 내부의 변경사항은 같은 이미지를 사용하는 다른 컨테이너에는 **아무 영향을 끼치지 않음!**
 
이미지 Commit
컨테이너 변경 사항을 이미지로 새로 저장
```
docker commit 335a1125bb70 nginx:modified
```
이 경우, 기존 nginx layer들에 새로운 레이어를 최상단에 새로 만들어 레이어 구조로 사용
만약 컨테이너를 다시 실행하면 이 레이어 위에 전용 쓰기 레이어가 다시 추가

docker history로 레이어 확인
```
docker history nginx:new
344e8fed361b   16 seconds ago   bash                                            0B
4cdc5dd7eaad   5 days ago       /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
<missing>      5 days ago       /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B
<missing>      5 days ago       /bin/sh -c #(nop)  EXPOSE 80                    0B
<missing>      5 days ago       /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B
<missing>      5 days ago       /bin/sh -c #(nop) COPY file:09a214a3e07c919a…   4.61kB
```

#### Docker Commit, Dockerfile로 이미지 만들기
우선 git이 없는 ubuntu image를 pull
```
docker pull ubuntu:focal
docker run -it ubuntu:focal /bin/sh -c 'git --version'
/bin/sh: 1: git: not found
- c : sh 실행시 문자열로부터 명령을 읽음
```
2단계에 걸쳐 git 설치
```
docker run ubuntu:focal /bin/sh -c 'apt-get update'
docker commit $(docker ps -alq) ubuntu:git-layer-1
docker run ubuntu:git-layer-1 /bin/sh -c 'apt-get install -y git'
docker commit $(docker ps -alq) ubuntu:git
$(docker ps -alq) : 가장 최근에 만들어진 컨테이너 ID
docker run -it ubuntu:git bash -c 'git --version'
git version 2.25.1
```
history 확인
```
docker history ubuntu:git
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
ea293bce76e1   3 minutes ago    /bin/sh -c apt-get install -y git               102MB
487906025bc4   38 minutes ago   /bin/sh -c apt-get update                       29.3MB
9873176a8ff5   3 weeks ago      /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>      3 weeks ago      /bin/sh -c #(nop) ADD file:920cf788d1ba88f76…   72.7MB
```
Dockerfile로 같은 환경 만들기
```
FROM ubuntu:focal
RUN apt-get update
RUN apt-get install -y git
```
해당 내용을 Dockerfile로 저장하고 build로 ubuntu:git2 라는 이름의 이미지 생성
