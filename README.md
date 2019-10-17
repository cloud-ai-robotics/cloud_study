# AI Robotics KR - Cloud Computing 스터디 Repository

![image_link](https://github.com/ai-robotics-kr/cloud_study/blob/master/images/docker-logo.png?raw=true | width=100)
![image_link](https://github.com/ai-robotics-kr/cloud_study/blob/master/images/kubernetes.png?raw=true | width=100)
![image_link](https://github.com/ai-robotics-kr/cloud_study/blob/master/images/terraform.png?raw=true | width=100)

## 스터디 소개:

- 목적: Cloud & Fog Computing 이해하기
- [Hands-on Cloud & Fog Computing 도서](https://github.com/ai-robotics-kr/cloud_hands_on) 출판 
- [해커톤](https://github.com/ai-robotics-kr/cloud_hackathon) 개최
- 기간: 2019년 7월 ~ 
- 그룹: [AI Robotics KR](https://www.facebook.com/groups/airoboticskr/)


## 가지고 있으면 스터디에 도움이 되는 것
 - Cloud Computing 에 대한 관심
 - Git 사용법
 - Linux (system, kernel, etc) 에 대한 이해
 - Network 에 대한 이해

 - 아래 장비는 없어도 스터디에 큰 지장은 없습니다.
   - Raspberry Pi 2 이상
   - 대용량 SD 카드
   - 무선 Lan USB (AP 구성을 할 예정이라면 필요. 또는 RPi2 이하인 경우 필요)

## 스터디 진도표

|                 스터디 내용                   |   날짜와 시간   | 스터디 대상 |
| --------------------------------------------- |:---------------:|------------:|
| Raspberry Pi 를 이용한 Access Point 구성<br/> * Network 기본에 대한 이해<br/> * NAT 에 대한 이해<br/> * Bridged Network 에 대한 이해 | 2019-9-18, 2019-9-25 | 신규멤버 |
| 개발 환경에 Docker 설치<br/> * Docker 가 무엇인지 기본에 대한 이해 <br/> * Docker 기본 명령어 실습<br/>Docker Image 만들어 보기<br/> * Dockerfile 형식에 대한 이해<br/> * Docker 이미지 생성시 필요한 사용자 인증(user credentials) 정보 처리 방법<br/>Kurbernetes 에 대한 이해 (Microservice architecture introduction) | 2019-10-2 | 신규멤버, 기존멤버 통합 |
| Docker Private Registry 만들어보기<br/> * Docker image 를 관리하기 위한 Private Registry 를 구성해본다.<br/>개발 환경에 Kubernetes 설치<br/> * Kubernetes cluster 이해하기<br/> * kubectl 명령어 사용해보기<br/> * Kubernetes 에서 deploy 할 때 Docker Private Registry 로 부터 Docker 이미지를 받아 올 수 있도록 만들기<br/> * [K8s Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)| 2019-10-8 | 통합멤버 |
| RPi3 여러대를 Kubernetes 로 Cluster 구성 하기<br> * [RPi3 K8s](https://medium.com/nycdev/k8s-on-pi-9cc14843d43) | 2019-10-15 | 통합멤버 |
| Terraform/Ansible 미리보기 | 2019-10-22 | 통합멤버 |

 * 2019-9-18/25 상세 일정
   장소: 하비웍스 (사당)

   19:30 - 20:00  개인적으로 공부하던 내용에 대해 궁금한 것 공유 / Github 사용 방법에 대한 공유 (모르는 분들을 위해서)
   20:00 - 21:10  신규 멤버는 RPi 를 이용해 AP 만드는 것을 진행 해봄 - systemd 구성 이해, iptables, dhcpd, hostapd 등과 같은 network 구성 데몬에 대한 이해
   21:10 - 21:30  각 조별로 실습 진행 한 내용에 대해 공유 및 질의 응답 (Yunkyu)
   - Docker Image 는 간단한 Service 를 만들지, Hello-world 를 쓸지, 진행 상태를 보고 결정

 * 2019-10-2 상세 일정
   장소: 스페이스클라우드 (강남 - 22천원)

   19:00 - 21:00  Docker 설치 및 Dockerfile 작성. 각자 편한 언어로 간단한 web service 코드 작성
   21:00 - 21:30  Dockerfile 및 user credentials 처리 부분에 대한 공유 및 Microservice overview 공유 (anon)

 * 2019-10-8 상세 일정
   장소: 하비웍스 (사당)

   19:30 - 21:30 Dockerfile 작성 - docker build 시 RPi 에서 docker daemon 이 segmentation fault 로 종료되는 문제 발견 (docker version 19.x.x)

 * 2019-10-10-15 상세 일정 (30분 초과)
   장소: 하비웍스 (사당)

   19:30 - 21:00 Kubernetes 설치 - flannel, weave-net 이 crash back off 상태가 되면서 coredens 가 pending 상태로 남는 문제 발견
   21:00 - 10:00 @madfalcon - k8s 환경 셋업 과정 발표, @HoonjaeLee Kubernetes network 동작 원리 발표 (overview)

### 스터디 소주제 (스터디를 하면서 만들어 보고 싶은 것)

 1. Tensorflow 및 PyTorch 등과 같은 머신러닝 관련 서비스를 Docker 이미지로 만들어 deploy 시켜서 작은 서비스 개발 (Docker, K8s, Terraform 모두 적용)
 2. AI Chatbot Service Docker 이미지를 만들어 서비스 개발 (Docker, K8s, Terraform 모두 적용)
 3. 여러대의 Rapsberry Pi 들을 Node 로 구성하여 Kubernetes 로 관리
 4. Kubernetes & Docker 활용 Service 배포 및 Enabling 자동화
 
## 클라우드 컴퓨팅 스터디 안내

### 시간과 장소

- 시간 : 매주 화요일 19:30 ~ 21:30 (2019.07 ~ )
- 장소 : 사당역 하비웍스[사당역 범주 빌딩]  (변경 시 사전 공유)

### 연락처

  * email: cloud@futuremobile.net
  * 슬랙 채널 [#study_클라우드_컴퓨팅](https://airoboticskr.slack.com/messages/CKUJC3XM2), [#ch_클라우드_컴퓨팅](https://app.slack.com/client/THA6JGK6G/CKTDU9SAY)
  * AI robotics 카카오톡 단톡방

### 스터디 형식: 실습 중심, 발표 자료 준비 필요 없음, 발표자 없음.

 * 조 별 실습

   참여 인원에 따라, 조를 편성<br>
   2인 1조 형식으로 진행, 스터디 정규 멤버가 아닌 경우, 임의의 조에 편성되어 같이 실습합니다.

 * 시간표

   7시 30분 ~ 8시 00분 : 기초 이론에 대하여 궁금한것에 대해 서로 묻고 답하는 시간 
                         기초 이론은 Network, Operating System(Linux) 이며,
                         업무하다가, 공부하다가 궁금한것이 있었다면, 처음 30분 동안 서로 질문하고 답하는 시간입니다.

   8시 00분 ~ 9시 00분 : 주별 지정된 주제에 대해 실습을 진행합니다. 

   9시 00분 ~ 9시 ??분 : 조별 실습 결과 공유 한 후 종료: 어려웠던 점, 배운 점, 궁금한 점등 (궁금한 것은 다음 주에 이어서 논의)
                         종료 시간은 장소를 제공해주시는 대표님 재량에 따라 조금씩 늘어날 수도 있습니다. (필요한 경우라면)

### 이론 공부 자료: 굳이 아래 자료가 아니어도 상관없습니다. 지금 가지고 있는 책으로 공부하셔도 무방합니다.

 * (완벽한 인프라 구축을 위한 Docker)[https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=166082298&start=slayer]
 * (Infrastructure as Code)[http://shop.oreilly.com/product/0636920039297.do]
 * (Beginning Linux Programming)[https://doc.lagout.org/operating%20system%20/linux/Beginning%20Linux%20Programming%2C%204%20Ed.pdf]
 * (모두의 도커 자료) http://www.pyrasis.com/private/2014/11/30/publish-docker-for-the-really-impatient-book
 * (도커 한글문서/ 영상 모음) http://documents.docker.co.kr/
 * `...`

### 질의 응답: 

  - 1주일동안 업무를 하다가 또는 공부를 하다가 궁금한 것이 있으면, 메모해 두셨다가 가져오시면 됩니다.
  - 질문과 답을 모두 준비하셔도 되고, 질문만 준비하셔도 됩니다.
    * 질문만 던지면, 아는 분이 답을 하거나, 모두 같이 답을 찾을 것이고,
    * 답도 준비하면, 같이 공유 하며 서로 배웁니다.
  - 질문 할 것이 없어도, 답을 할 수 없어도, 상관 없습니다.

### 기타 알림 사항

  - 마음껏 질문과 의견을 나누어주세요.
  - 이 스터디는 같이 배우는 것이 주요 목적이지만, 친목 도모도 중요하다고 생각합니다. 너무 부담 갖지 마시고 오세요.
  - ___스터디 정규 멤버가 아니어도, 참여할 수 있습니다.___ 스터디 시간에 지정 장소로 오면 됩니다.

## Deposit

 * 스터디의 원할한 운영을 위하여 아래와 같이 Deposit 제도를 도입(이번 스터디 전체 Deposit)

   - 스터디 2일 전(전 주 일요일)에 Slack을 통해 참여여부 확인
   - 이 후 발생 되는 예외에 대해서는 아래 Rule 적용(가차없음..)

|        구분        |     직장인      |  학생  |
| ------------------ |:---------------:|-------:|
|       Deposit      |      5만원      |  3만원 |
|      지각 10분     |      2천원      |  1천원 |
|      지각 30분     |      3천원      |  2천원 |
|        결석        |      5천원      |  4천원 |

 * 해당 주에 실습 결과 공유를 못하면, 다음주에 간식 사오기!!!
 * 스터디가 끝나거나, 참여를 못하게 되면, ___남은 금액___ 을 다시 돌려드립니다. (아니면 종강 PARTY!!)

## 유용한 링크 모음

### Docker
  - [Docker for IT Pros and System Administrators](https://training.play-with-docker.com/ops-stage1/)

### Terraform
  - [Providers](https://www.terraform.io/docs/providers/index.html)

### Kubernetes
  - [What is Kubernetes](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)
  - [Docker Kubernetes Service](https://www.docker.com/products/kubernetes)
  - [Kubernetes 기초 모듈](https://kubernetes.io/ko/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/)
  - [Kubernetes](https://www.level-up.one/kubernetes-networking-pods-levelup/)

### Understanding Container Technology
  - [LXC vs Docker](https://pasztor.at/blog/lxc-vs-docker)
  - [Under the hood of Docker](https://pasztor.at/blog/under-the-hood-of-docker)
  - [What is a container](https://www.docker.com/resources/what-container)
  - [Docker Swarm](https://searchitoperations.techtarget.com/definition/Docker-Swarm)
  - [Docker Overview](https://docs.docker.com/engine/docker-overview/)
  - [Docker: get started](https://docs.docker.com/get-started/)

### Understanding Infrastructure as a code
  - [Infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_code)
  - [Terraform(Golang)](https://en.wikipedia.org/wiki/Terraform_(software))
  - [Ansible(Python)](https://en.wikipedia.org/wiki/Ansible_(software))
  - [Book(Hard Copy): Infrastructure as code (O'Reilly)](http://shop.oreilly.com/product/0636920039297.do)


## Git 사용법

Cloud study 는 github 를 통해 communication 합니다. 아래 사용법을 숙지해주세요.

### github 에 계정 생성 하기

### SSH key 등록
```
$ ssh-keygn

# ~/.ssh/id_rsa.pub 내용을 복사해서 github account setting 에 SSH & GPG 항목에 추가
$ cat ~/.ssh/id_rsa.pub
```

### git 환경 설정

  사용자 이름, 이메일 주소등의 설정 작업을 해주세요.

### Fork the repository

 git@github.com:/ai-robotics-kr/cloud_study repository 를 개인 계정으로 fork 합니다. (github 우측 상단 Fork 버튼 클릭)

### Clone and setup the forked repository
```
$ git clone git@github.com:/{아이디}/cloud_study
$ git remote add upstream git@github.com:/ai-robotics-kr/cloud_study
```

### 변경 사항 만들어서 작성 후 PR 만들기

  변경 사항을 만들때는 현재 master branch 에서 새로운 branch 를 만들어 수정 사항을 반영한 후 PR 을 생성합니다.

```
$ git checkout -b {브랜치이름:패치 내용을 한 단어로 요약}
# ... 수정하기 ...
$ git add {수정한 파일들}
# Commit message 작성 및 sign off
$ git comit -s   
$ git push origin {브랜치이름}
```

  github 에 가면 PR 을 만들꺼냐고 물어보는 버튼이 생성되어 있습니다. 그걸 눌러서 PR 을 만들어 주면 됩니다.

### 기존 PR 에 수정사항 overrding 하기

  하나의 PR 에 여러개의 commit 을 만들어도 무방합니다만, 하나의 PR 에 하나의 commit 을 유지하는 것을 권장합니다.
  여기서는 하나의 PR 에 하나의 commit 만 남기는 방식으로 설명합니다.

```
# 기존에 작업하던 branch 로 이동
$ git checkout {브랜치이름}
# ... 수정하기 ...
$ git add {수정한 파일들}
# Commit message 수정 및 sign off
$ git commit -s --amend
$ git push origin {브랜치이름} -f
```

  push 까지 하면, 기존 PR 이 자동으로 갱신됩니다.
