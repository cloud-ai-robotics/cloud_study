# syntax=docker/dockerfile:experimental
# Leave as comments for the history
# RUN apt-get update && apt-get install -y git nodes
# RUN echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config
# RUN --mount=type=ssh git clone git@github.com:/ai-robotics-kr/cloud_study /tmp/cloud_study
# COPY --from=build /tmp/cloud_study/anon/service/webserver.js /cloud && rm -rf /tmp/cloud_study
# RUN apt remove -y git && apt autoremove -y git
FROM registry.nicesj.com/ubuntu:18.04 
MAINTAINER anon <cloud@futuremobile.net>
RUN apt-get update && apt-get install -y nodejs
EXPOSE 8087
WORKDIR /cloud
COPY service/webserver.js /cloud
CMD ["/usr/bin/node", "./webserver.js"]
