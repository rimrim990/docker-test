# Docker test
## Docker Compose

### 도커 컴포즈란?
**어디에 쓰는걸까?**
```shell
$ docker run -p 3306:3306 --name mysql -d mysqltest
$ docker run -d -p 80:80 --link mysql:db --name web webtest
```
- 위와 같이 두 개의 컨테이너가 하나의 웹 애플리케이션을 구성하는 상황을 가정해보자
- 웹 애플리케이션을 실행하고 종료하려면 두 개의 컨테이너에 `docker` 명령어를 각각 수행해줘야 한다
- 만약 하나의 애플리케이션을 이루는 여러 개의 컨테이너를 묶어서 관리하는 기능이 있다면 편리할 것이다

**도커 컴포즈**
- 도커 컴포즈는 여러 개의 컨테이너를 묶어서 관리하는 기능을 제공한다
- 도커 컴포즈는 여러 컨테이너의 옵션과 환경을 정의한 파일을 읽어 순차적으로 컨테이너를 생성한다
- 별도의 설정을 하지 않았다면 도커 컴포즈는 `docker-compose.yml` 파일을 읽어들인다

### docker-compose.yml 예시
```yml
services:
  web:
    build: .
    ports:
      - "8000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    depends_on:
      - redis

  redis:
    image: redis

volumes:
  logvolume01: {}
```
- 위와 같이 `docker-compose.yml` 파일을 작성할 수 있다
- 해당 프로젝트는 `web`과 `redis`라는 이름의 컨테이너 두 개를 관리한다

```shell
$ docker compose up
```
- 위의 명령어 수행으로 현재 경로에 있는 `docker-compose.yml`을 읽어 순차적으로 컨테이너들을 실행한다

```shell
$ docker ps
CONTAINER ID   IMAGE         COMMAND                   CREATED         STATUS         PORTS                    NAMES
8ba5d43a5eca   compose-web   "flask run"               5 seconds ago   Up 4 seconds   0.0.0.0:8000->5000/tcp   compose-web-1
a208adc111c1   redis         "docker-entrypoint.s…"   5 seconds ago   Up 4 seconds   6379/tcp                 compose-redis-1
```
- `docker-compose.yml`에 정의한 대로 컨테이너들이 정상적으로 시작됐음을 확인할 수 있다
- 컨테이너 이름 앞에 `compose`가 붙은 것은 도커 컴포즈로 생성된 프로젝트의 이름이 `compose`이기 때문이다
- 별도로 프로젝트르 이름을 지정하지 않았다면 현재 디렉터리 이름이 프로젝트 이름으로 사용된다

```shell
$ docker compose down
```
- 또한 위의 명령어로 실행된 컨네이들을 순차적으로 종료시킬 수 있다

### docker-compose.yml 살펴보기
**services**
```yml
services:
  web:
    build: .
    ports:
      - "8000:5000"
    environment:
      FLASK_DEBUG: "true"
    volumes:
      - .:/code
      - logvolume01:/var/log
    depends_on:
      - redis

  redis:
    image: redis
```
- `services`는 도커 컴포즈로 생성할 컨테이너 (서비스) 를 정의할 수 있다
- `services`에 사용된 `web`과 `redis`는 각각 컨테이너로 구현되며, 도커 컴포즈는 이 둘을 하나의 프로젝트로 관리한다

**build**
```yml
web:
  build: .
```
- 컨테이너 이미지 빌드에 사용할 `Dockerfile`을 설정한다
- 현재 디렉터리에 `Dockerfile`이 존재하기 때문에 이를 토대로 이미지를 생성해 컨테이너를 실행한다

```shell
$ docker images

REPOSITORY                                                TAG           
compose-web                                               latest
```
- 실제로 `docker compose up` 실행 이후 이미지 목록을 출력해보니, `compose-web`이라는 이름의 이미지가 생성됨을 확인할 수 있었다

**ports**
```yml
web:
  ports:
    - "8000:5000"
```
- 컨테이너 포트 포워딩을 설정할 수 있다
- 호스트 IP를 기입하지 않으면 호스트의 모든 네트워크 인터페이스에 포트 포워딩이 적용된다

```shell
$ docker ps

CONTAINER ID   IMAGE         COMMAND                   CREATED         STATUS         PORTS                    NAMES
8ba5d43a5eca   compose-web   "flask run"               5 seconds ago   Up 4 seconds   0.0.0.0:8000->5000/tcp   compose-web-1
```
- 컨테이너 정보를 출력해보니 호스트의 8000번 포트가 컨테이너의 5000번 포트로 정상적으로 포트포워딩 되었다

**volumes**
```yml
volumes:
  logvolume01:
```
- 서비스들 간에 데이터 영구 저장을 위해 사용 가능한 도커 볼륨을 정의할 수 있다
- `logvolume01`이라는 이름의 볼륨을 생성하였고, 별도의 값이 없으므로 기본 설정에 따라 볼륨을 생성한다

```shell
$ docker volume ls 

DRIVER    VOLUME NAME
local     compose_logvolume01
```
- 볼륨 목록을 출력해보니 `logvolume01`이 정상적으로 생성되었다

```yml
web:
  volumes:
    - .:/code
    - logvolume01:/var/log
```
- `services` 키 하위에도 `volumes`를 정의하여 컨테이너 볼륨을 설정할 수 있다
- `.:/code`는 지정된 호스트 디렉터리를 컨테이너 내부로 마운트하였다
- `logvolume01:/var/log`는 앞서 생성한 `logvolume01` 도커 볼륨을 컨테이너 내부로 마운트하였다

**environment**
```yml
web:
  environment:
    FLASK_DEBUG: "true"
```
- 컨테이너 내부에서 사용 가능한 환경변수를 설정할 수 있다
- 이는 `docker run -e FLASK_DEBUG="true"`와 동일하게 동작한다

**depends_on**
```yml
web:
  depends_on:
    redis
```
- 컨테이너 시작과 종료 시의 의존 관계를 설정할 수 있다
- `web` 컨테이너는 `redis`를 의존하므로 `reids`보다 늦게 생성되고, `redis`보다 먼저 종료될 것이다

**images**
```yml
redis:
  image: redis
```
- 컨테이너가 사용할 베이스 이미지를 설정한다