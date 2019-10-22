# syntax=docker/dockerfile:experimental
FROM python:3.6-slim-stretch
MAINTAINER inwon <bigdu154@gmail.com>
RUN apt-get update
RUN pip install Flask
EXPOSE 8080
COPY ./ /cloud/
WORKDIR /cloud
CMD ["python3", "app.py"]
