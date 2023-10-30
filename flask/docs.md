# Docker test
## Dockerfile 사용하여 이미지 빌드하기

### Dockerfile 예시
테스트에 사용된 도커 파일은 아래와 같다
```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y python3 python3-pip
RUN pip install flask==2.1.*
COPY hello.py /
ENV FLASK_APP=hello
EXPOSE 8000
CMD flask run --host 0.0.0.0 --port 8080
```
- Dockerfile을 사용하여 플라스크 애플리케이션 수행을 위한 이미지를 빌드할 수 있다

### Dockerfile 살펴보기
**FROM**
```dockerfile
FROM ubuntu:22.04
```
- 생성될 이미지에서 사용할 베이스 이미지를 설정한다
- `name:tag` 형식을 갖는다

**RUN**
```dockerfile
RUN apt-get update && apt-get install -y python3 python3-pip
RUN pip install flask==2.1.*
```
- 베이스 이미지 내에서 실행될 쉘 커맨트들을 정의할 수 있다
  - 기본 설정은 `/bin/sh`를 사용하여 쉘 커맨드 실행
- 베이스 이미지인 `ubuntu` 환경에서 명령어들을 수행하여 파이썬과 플라스크를 설치한다

```dockerfile
RUN ["echo", "안녕하세요."]

# 기본 설정이 /bin/sh을 사용하므로 위와 동일한 커맨드가 실행된다
RUN ["/bin/sh", "-c", "echo", "안녕하세요"]
```
- 위와 같이 JSON 배열 형식으로도 쉘 커맨드 실행이 가능하다

**COPY**
```dockerfile
COPY app.py /
```
- 빌드 컨텍스트의 파일을 컨테이너 파일 시스템으로 복사한다
- 예시에서는 빌드 컨테스트 (`/flask`) 에서 `app.py` 플라스크 코드를 컨테이너의 루트 경로로 복사하였다

**ENV**
```dockerfile
ENV FLASK_APP=hello
```
- 빌드 단계에서 사용할 수 있는 환경 변수를 정의한다
- 에시에서는 플라스크를 애플리케이션 수행 (`flask.run`) 을 위해 `FLASK_APP` 환경변수를 지정하였다

**EXPOSE**
```dockerfile
EXPOSE 8000
```
- 컨테이너가 실행 중일 때 특정 포트를 리스닝하고 있음을 나타낼 수 있다
- 해당 명령어가 이미지 빌드를 위해 꼭 필요한 것은 아니지다, 애플리케이션이 어떤 일을 하는지 설명하는데 유용하게 쓰일 수 있다
  - 실제를 포트를 열기 위해서는 컨테이너 수행 시 `-p`를 사용해야 함
  - `EXPOSE`는 문서화의 역할만 수행하고 실제로 포트를 열지는 않음

**CMD**
```dockerfile
CMD flask run --host 0.0.0.0 --port 8080
```
- 빌드된 이미지를 사용하여 수행한 컨테이너에서 실행할 커맨드를 정의한다

### 이미지 빌드하기
```shell
docker build -t flask-test .
```
- `docker build` 명령어로 `Dockerfile`로부터 이미지를 빌드할 수 있다
- `--file`로 도커 파일의 이름을 지정하지 않으면 기본적으로 `Dockerfile`을 사용한다
- `.`는 도커 파일을 빌드할 때 사용할 실행 컨텍스트를 지정한다. `.` 하위의 모든 파일과 폴더들이 빌드 컨텍스트에 포함된다
  - 단 `.dockerignore`에 정의된 파일들은 빌드 컨텍스트에서 제거