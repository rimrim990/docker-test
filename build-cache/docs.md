# Docker test
## 빌드 캐시 활용하기

### 빌드 캐시 살펴보기
```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:latest

RUN apt-get update && apt-get install -y build-essentials
COPY main.c Makefile /src/
WORKDIR /src/
RUN make build
```
- 도커 파일에 사용된 각 명령어들은 이미지에 하나의 레이어로 사용된다

<img width="500" src="https://docs.docker.com/build/images/cache-stack.png">

- 도커 이미지는 위와 같이 레이어들의 스택으로 구성되어 있다
- 각 레이어는 지금까지 쌓아온 이전 레이어에 새로운 내용을 추가해준다

```
# 다시 빌드
 => CACHED [1/5] FROM docker.io/library/ubuntu:latest@sha256:2b7412e6465c3 
```
- 도커는 빌드할 때마다 매번 레이어를 새로 생성하는게 아니라, 이전에 생성했던 레이어를 재사용한다
- 즉 기본적으로 이미지를 빌드할 때 레이어 캐시를 지원한다

<img  width="500" src="https://docs.docker.com/build/images/cache-stack-invalidated.png">

- 그런데 만약 도커 파일에 사용된 명령어의 내용이 바뀐다면 이는 이미지의 레이어가 변화함을 의미한다
- 예를 들어 `main.c` 파일을 수정했다고 가정하자. 해당 변화로 인해 `COPY` 명령어의 수행 결과가 바뀌므로 이전에 캐싱한 레이어를 재사용할 수 없다
- 이러한 경우에는 `COPY` 이후의 모든 레이어가 다시 빌드되어야 한다

### 빌드 캐시 활용하기

빌드 캐시를 효율적으로 사용한다는 것은, 이전에 캐싱된 레이어를 최대한 재활용 함을 의미한다
- 이미지 빌드시 레이어 캐시를 자동 적용하기 때문에, 이를 극대화 할 수 있도록 도커 파일을 잘 작성해야 한다

**레이어 순서 조정하기**
```dockerfile
# syntax=docker/dockerfile:1
FROM node
WORKDIR /app
COPY . .          # Copy over all files in the current directory
RUN npm install   # Install dependencies
RUN npm build     # Run build
```
- 예를 들어 위와 같은 `node` 기반의 이미지를 생성한다고 가정해보자
- 만약 `/app` 하위의 파일이 변화한다면 항상 `COPY` 이후 레이어를 다시 빌드해야 한다
- 이는 `package.json`이 아닌 `index.html`과 같은 파일이 변화하더라도 불필요하게 `npm install`을 다시 수행해야 함을 의미한다

```dockerfile
# syntax=docker/dockerfile:1
FROM node
WORKDIR /app
COPY package.json yarn.lock .    # Copy package management files
RUN npm install                  # Install dependencies
COPY . .                         # Copy over project files
RUN npm build                    # Run build
```
- 캐시를 최대한 재활용하기 위해 위와 같이 도커 파일의 순서를 조정해보자
- 만약 `index.html`이 변화하더라도 `npm install`은 다시 수행되지 않고, `COPY . .` 이후의 레이어만 다시 빌드하면 된다
- `COPY . .`와 같이 자주 변화하는 레이어를 최대한 도커 파일의 마지막에 배치하면 빌드 캐시를 효율적으로 사용할 수 있다

**레이어 작게 유지하기**
```dockerfile
# X
COPY . /src

# O
COPY ./src ./Makefile /src
```
- 레이어를 최대한 작게 유지해야 해당 레이어가 변화할 가능성을 줄여 캐시를 재활용 할 수 있다
- 예시와 같이 빌드 컨텍스트에서 불필요한 파일들을 함께 복사하기 보다는 필요한 파일만 복사해야 레이어를 작게 유지할 수 있다

```
# .dockerignore
unnecessary.c
```
- 또한 `.dockerignore`를 사용하여 빌드 컨텍스트에서 불필요한 파일을 제외하는 것도 좋은 방법이다

**번외 - 멀티 스테이지 빌드 사용하기**
```dockerfile
# syntax=docker/dockerfile:1

# stage 1
FROM alpine as git
RUN apk add git

# stage 2
FROM git as fetch
WORKDIR /repo
RUN git clone https://github.com/your/repository.git .

# stage 3
FROM nginx as site
COPY --from=fetch /repo/flask/app.py /usr/share/nginx
```
- 이전에 학습했던 멀티 스테이지 빌드를 사용하여 이미지 크기를 작게 유지할 수 있다
- `git clone`의 모든 결과가 아닌 필요한 파일만 최종 이미지에 포함된다

```dockerfile
RUN git clone https://github.com/your/repository.git .

# CACHED [fetch 2/2] RUN git clone https://github.com/rimrim990/docker-test 
```
- 주의할 점은 `RUN` 레이어가 캐싱되기 때문에 `git clone`의 결과가 항상 최신 상태를 유지함을 보장할 수 없다
- 만약 리포지토리가 수정되어 캐시를 무효화해야 한다면 `docker build prune`으로 빌드 캐시를 제거하거나 `--no-cache` 플래그를 사용할 수 있다
