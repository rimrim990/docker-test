# Docker test
## 멀티 스테이지로 빌드하기

### 멀티 스테이지 예시
지금까지는 하나의 베이스 이미지만으로 이미지를 생성하였다.
```dockerfile
# syntax=docker/dockerfile:1

# 첫 번째 스테이지
FROM golang:1.21
WORKDIR /src
COPY <<EOF ./main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

# 두 번째 스테이지
FROM scratch
COPY --from=0 /bin/hello /bin/hello
CMD ["/bin/hello"]
```
- 다음과 2개 이상의 스테이지를 작성할 수 있다
- 멀티 스테이지로 작성한다는 것은 두 개 이상의 `FROM`을 사용함을 의미한다
- 각 `FROM`은 서로 다른 베이스 이미지를 사용 가능하며, `FROM`이 스테이지의 시작점이 된다

### 멀티 스테이지 살펴보기
**첫 번째 스테이지**
```dockerfile
FROM golang:1.21
WORKDIR /src
COPY <<EOF ./main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go
```
- 첫 번째 스테이지는 `go` 컴파일러를 사용하여 바이너리 파일을 빌드한다

**두 번째 스테이지**
```dockerfile
FROM scratch
COPY --from=0 /bin/hello /bin/hello
CMD ["/bin/hello"]
```
- 두 번재 스테이지는 이전 빌드에서 생성된 바이너리 파일을 복사해와 실행한다
- `/bin/hello`를 제외한 Go SDK 등의 모든 파일들은 최종 이미지에 포함되지 않는다

**스테이지에 이름 부여하기**
```dockerfile
FROM scratch
COPY --from=0 /bin/hello /bin/hello
CMD ["/bin/hello"]
```
- 기본적으로 각 스테이지에는 이름이 부여되지 않기 때문에 숫자로 이전 스테이지를 참조했었다
- 스테이지 번호는 0번부터 시작하는 정수이다

```dockerfile
FROM golang:1.21 as build
WORKDIR /src
COPY <<EOF /src/main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world)
}
EOF
```
- `FROM` 명령어에 `AS <NAME>`을 추가하면 스테이지에 이름을 부여할 수 있다

```dockerfile
COPY --from=build /bin/hello /bin/hello
```
- 스테이지에 이름이 부여되면 해당 이름을 사용하여 스테이지 참조가 가능하다
- 이름을 사용한다면 이후 도커 파일에서 빌드 순서가 바뀌어도 오류 없이 실행될 것이다

### 멀티 스테이지 빌드하기
```shell
$ docker build -t hello .
```
- 멀티 스테이지를 사용하여도 일반 도커 파일처럼 동일하게 빌드하면 된다

**특정 스테이지만 빌드하기**
```dockerfile
# 세 번째 스테이지
FROM node AS node
RUN ["echo", "hi"]
```
- 테스트를 위해 도커 파일에 `node`라는 스테이지를 하나 추가하였다

```shell
$ docker build --target node -t hello .
```
- `--target` 옵션을 추가하면 특정 스테이지만 빌드할 수 있다
- 앞서 생성한 `node` 스테이지만 빌드해보자

```
[+] Building 3.0s (13/13)    
 => [internal] load .dockerignore                              
 => CACHED [node 1/2] FROM docker.io/library/node:latest@sha256:1f937398bb  
 => [node 2/2] RUN ["echo", "hi"]  
 => exporting to image 
```
- 빌드 실행 결과를 살펴보면 `node` 스테이지만 수행된 것을 확인할 수 있다

```shell
$ docker history hello

IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
bffd1c162644   2 minutes ago   RUN echo hi # buildkit                          0B        buildkit.dockerfile.v0
<missing>      5 days ago      /bin/sh -c #(nop)  CMD ["node"]                 0B
...
```
- 이미지 레이어를 출력해보니 베이스 이미지 레이어와 `RUN echo hi`만 존재하였다
- 이러한 방식은 특정 스테이지를 디버깅할 때 사용하면 유용할 것 같다

**외부 이미지를 스테이지로 사용하기**
```dockerfile
COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```
- 지금까지 살펴본 예제에서는 도커 파일에 힘께 정의된 이전 스테이지를 참조하였다
- 그러나 위와 같이 `COPY --from`에 도커 저장소에 존재하는 외부 이미지 참조도 가능하다
