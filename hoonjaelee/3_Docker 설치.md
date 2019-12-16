# 라즈베리파이에 Docker 설치

## Docker 설치 스크립트

Production 환경에서는 추천하지 않는다고 하지만, 테스트용으로 간편하게 설치 할 수 있는 스크립트를 이용함.

### 1. 설치 스크립트를 이용하여 설치하기

~~~ shell
curl -fsSL https://get.docker.com -o get-docker.sh
sudo bash get-docker.sh
~~~

### 2. 유저 `pi`에게 docker 실행 권한 주기

~~~ shell
sudo usermod -aG docker pi
~~~

### 3. Docker 실행 테스트

~~~ shell
docker run hello-world
~~~

## 참고자료

1. <https://docs.docker.com/install/linux/docker-ce/debian/>
