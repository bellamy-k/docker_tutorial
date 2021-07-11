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
  - 
