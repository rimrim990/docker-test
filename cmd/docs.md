# Docker test
## 커맨드와 엔트리포인트

### 커맨드
```dockerfile
FROM ubuntu
CMD "ls"
```
- `CMD`는 컨테이너가 실행되면 수행할 명령어를 정의한다
- 해당 도커 파일로 빌드한 이미지가 컨테이너로 실행되면 `/bin/sh -c ls` 명령어가 실행될 것이다
- 앞에 `/bin/sh -c`가 추가된 이유는 `CMD`가 기본적으로 쉘을 사용해 명령어를 실행하기 때문이다

```dockerfile
CMD ["echo", "hi"]
```
- 만약 쉘을 사용하지 않고 실행 가능한 프로그램을 수행하고 싶다면 위와 같이 JSON 배열 형식으로 작성해야 한다

### 엔트리포인트
```dockerfile
FROM ubuntu
ENTRYPOINT ["echo", "hi"]
```
- `ENTRYPOINT`도 `CMD`와 같이 컨테이너가 시작되고 수행할 명령어를 지정한다
- `CMD`와 마찬가지로 기본적으로 쉘을 사용해 실행하기 때문에, 문자열만 입력하면 앞에 `/bin/sh -c`가 추가된다

```dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
CMD ["hello", "world"]
```
- 그렇다면 `ENTRYPOINT`와 `CMD`의 차이가 뭘까?
- `ENTRYPOINT`는 `CMD`를 인자로 사용할 수 있다는 점이 다르다
- 예시로 작성된 도커 파일로부터 실행된 컨테이너는 `echo hello word`를 실행하게 된다

```dockerfile
FROM ubuntu
ENTRYPOINT ["/bin/bash", "/entrypoint.sh"]
```
- `ENTRYPOINT`와 `CMD`는 하나의 도커 파일에서 한 번씩만 정의할 수 있다
- 따라서 여러 개의 명령어 실행을 원한다면 별도로 쉘 스크립트를 정의해 실행하도록 하면 된다

```shell
# entrypoint.sh
echo $1 $2
```
- 도커 컨테이너가 실행되면 위에 작성된 `entrypoint.sh`를 실행할 것이다
- `entrypoint.sh`는 두 개의 인자를 필요로 하는데 이는 컨테이너 실행 시 설정해줄 수 있다

```shell
$ docker run -d --name entrypoint_example exec_entrypoint first second

$ docker logs entrypoint_example
// first second
```
- 예를 들어 위와 같이 first, second를 인자로 전해줄 수 있다
- 원래라면 `CMD`로서 수행되었을 것이지만, `ENTRYPOINT`가 정의되어 있기 때문에 `ENTRYPOINT`의 인자가 되었다
- 컨테이너 로그를 확인해보면 first, second 가 찍힌 것을 확인할 수 있다
