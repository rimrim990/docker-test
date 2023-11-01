# syntax=docker/dockerfile:1
FROM ubuntu
# echo hello world
ENTRYPOINT ["echo"]
CMD ["hello", "world"]

FROM ubuntu
RUN apt-get update
RUN apt-get install apache2 -y
ADD entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
# 쉘 스크립트 실행
ENTRYPOINT ["/bin/bash", "/entrypoint.sh"]
