# syntax=docker/dockerfile:experimental
FROM ubuntu:18.04
MAINTAINER anon <cloud@futuremobile.net>
RUN apt-get update
RUN apt-get install -y git nodejs
RUN echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config
RUN --mount=type=ssh git clone git@github.com:/ai-robotics-kr/cloud_study
WORKDIR /cloud
CMD ["/usr/bin/node /cloud/cloud_study/anon/service/webserver.js"]
