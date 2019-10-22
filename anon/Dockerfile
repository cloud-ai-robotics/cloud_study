# syntax=docker/dockerfile:experimental
FROM registry.nicesj.com/ubuntu:18.04
MAINTAINER anon <cloud@futuremobile.net>
RUN apt-get update
RUN apt-get install -y git nodejs
RUN echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config
RUN --mount=type=ssh git clone git@github.com:/ai-robotics-kr/cloud_study && mv cloud_study/anon/service/webserver.js /cloud
RUN apt remove -y git
WORKDIR /cloud
CMD ["/usr/bin/node /cloud/webserver.js"]
