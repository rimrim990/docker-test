# syntax=docker/dockerfile:1

# stage 1
FROM alpine as git
RUN apk add git

# stage 2
FROM git as fetch
WORKDIR /repo
RUN git clone https://github.com/rimrim990/docker-test.git .

# stage 3
FROM nginx as site
COPY --from=fetch /repo/flask/app.py /usr/share/nginx