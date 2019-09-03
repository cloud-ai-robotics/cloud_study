# AI Robotics KR - Cloud Computing 스터디 Repository

![image_link](https://github.com/ai-robotics-kr/cloud_study/blob/master/images/docker-logo.png?raw=true)
![image_link](https://github.com/ai-robotics-kr/cloud_study/blob/master/images/kubernetes.png?raw=true)
![image_link](https://github.com/ai-robotics-kr/cloud_study/blob/master/images/terraform.png?raw=true)

## 스터디 소개:
- 목적: Cloud & Fog Computing 이해하기
- [Hands-on Cloud & Fog Computing 도서](https://github.com/ai-robotics-kr/cloud_hands_on) 출판 
- [해커톤](https://github.com/ai-robotics-kr/cloud_hackathon) 개최
- 기간: 2019년 7월 ~ 
- 기획 그룹: [AI Robotics KR](https://www.facebook.com/groups/airoboticskr/)

## 스터디 진행 방법 / Repository 사용법:
- 라즈베리파이로 AP 만들기 + 다양한 센서 사용해보기
- Docker, K8, Terraform/Ansible 사용해보기
- Cloud/Fog computing API 구현
- IoT/Edge Computing Service Client 구현
- 직접 구성한 Cloud Computing이 AWS, Azure, Google Cloud 등과 같은 상용 서비스에는 어떻게 서비스가 되고 있는지 비교

## 스터디 진도표
|          스터디 내용         |   날짜와 시간     | 발표자 
| -------------------------- |:---------------:|-----:|
| Raspberry Pi 를 이용한 Access Point 구성<br> * Network 기본에 대한 이해<br> * NAT 에 대한 이해<br> * Bridged Network 에 대한 이해 | -- | -- |
| 개발 환경에 Docker 설치<br> * Docker 가 무엇인지 기본에 대한 이해 | -- | -- |
| Docker Image 만들어 보기<br> * Dockerfile 형식에 대한 이해<br> * Docker 이미지 생성시 필요한 사용자 인증(user credentials) 정보 처리 방법 | -- | -- |
| Docker Private Registry 만들어보기<br> * Docker image 를 관리하기 위한 Private Registry 를 구성해본다.| -- | -- |
| 개발 환경에 Kubernetes (minikube) 설치<br> * Kurbernetes 에 대한 이해 (기초 모듈 파악하기) | -- | -- |
| Kubernetes cluster 이해하기<br> * kubectl 명령어 사용해보기| -- | -- |
| Terraform 이해하기<br>...| -- | -- |

### 스터디 소주제 (스터디를 하면서 만들어 보고 싶은 것)
 1. Tensorflow 및 PyTorch 등과 같은 머신러닝 관련 서비스를 Docker 이미지로 만들어 deploy 시켜서 작은 서비스를 만들어 보자. (Docker, K8s, Terraform 모두 적용)
 2. AI Chatbot Service Docker 이미지를 만들어 서비스 해보자. (Docker, K8s, Terraform 모두 적용)
 3. 여러대의 Rapsberry Pi 들을 Node 로 구성하여 Kubernetes 로 관리해보자.

## 클라우드 컴퓨팅 스터디 안내
- 시간과 장소

  오프라인으로 매주 화요일 19시 00분~ 21시 00분에 진행됩니다. (2019.07 ~ )
  장소는 이변이 없는 한 사당 하비웍스 입니다. (변경되면 사전 공유 합니다)

- 연락처

  * email: cloud@futuremobile.net
  * 슬랙 채널 [#study_클라우드_컴퓨팅](https://airoboticskr.slack.com/messages/CKUJC3XM2), [#ch_클라우드_컴퓨팅](https://app.slack.com/client/THA6JGK6G/CKTDU9SAY)
  * AI robotics 카카오톡 단톡방

- 스터디 형식: 실습 중심, 발표 자료 준비 필요 없음, 발표자 없음.

 * 조 별 실습

   참여 인원에 따라, 조를 편성합니다.
   한 조당 기본 최대 3인이며, 스터디 정규 멤버가 아닌 경우, 임의의 조에 편성되어 같이 실습합니다.

 * 시간표
   7시 00분 ~ 7시 30분 : 기초 이론에 대하여 궁금한것에 대해 서로 묻고 답하는 시간 
                         기초 이론은 Network, Operating System(Linux) 이며,
                         업무하다가, 공부하다가 궁금한것이 있었다면, 처음 30분 동안 서로 질문하고 답하는 시간입니다.

   7시 30분 ~ 9시 00분 : 주별 지정된 주제에 대해 실습을 진행합니다. 

- 이론 공부 자료: 굳이 아래 자료가 아니어도 상관없습니다. 지금 가지고 있는 책으로 공부하셔도 무방합니다.

 * (완벽한 인프라 구축을 위한 Docker)[https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=166082298&start=slayer]
 * (Infrastructure as Code)[http://shop.oreilly.com/product/0636920039297.do]
 * (Beginning Linux Programming)[https://doc.lagout.org/operating%20system%20/linux/Beginning%20Linux%20Programming%2C%204%20Ed.pdf]
 * `...`

- 질의 응답: 

  - 1주일동안 업무를 하다가 또는 공부를 하다가 궁금한 것이 있으면, 메모해 두셨다가 가져오시면 됩니다.
  - 질문과 답을 모두 준비하셔도 되고, 질문만 준비하셔도 됩니다.
    * 질문만 던지면, 아는 분이 답을 하거나, 모두 같이 답을 찾을 것이고,
    * 답도 준비하면, 같이 공유 하며 서로 배웁니다.
  - 질문 할 것이 없어도, 답을 할 수 없어도, 상관 없습니다.

- 기타 알림 사항

  - 마음껏 질문과 의견을 나누어주세요.
  - 이 스터디는 같이 배우는 것이 주요 목적이지만, 친목 도모도 중요하다고 생각합니다. 너무 부담 갖지 마시고 오세요.
  - 스터디 정규 멤버가 아니어도, 참여할 수 있습니다. 스터디 시간에 지정 장소로 오면 됩니다.

## Deposit
  - 스터디의 원할한 운영을 위하여 아래와 같이 Deposit 제도를 도입하기로 하였습니다.

스터디 2일전에 참여 여부를 확정해주시고, 그 이 후에는 아래 rule 이 적용됩니다. 가차없이...

|     구분           |  직장인     | 학생 |
| ----------------- |:---------------:|-----:|
| Deposit | 5만원 | 3만원 |
| 지각10분 | 2천원 | 1천원 |
| 지각30분 | 3천원 | 2천원 |
| 결석 | 5천원 | 4천원 |

 * 스터디가 끝나거나, 참여를 못하게 되면, 남은 금액을 다시 돌려드립니다. (아니면 종강 PARTY!!)

## 유용한 링크 모음

- Kubernetes
  - [What is Kubernetes](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)
  - [Docker Kubernetes Service](https://www.docker.com/products/kubernetes)
  - [Kubernetes 기초 모듈](https://kubernetes.io/ko/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/)

- Understanding Container Technology
  - [LXC vs Docker](https://pasztor.at/blog/lxc-vs-docker)
  - [Under the hood of Docker](https://pasztor.at/blog/under-the-hood-of-docker)
  - [What is a container](https://www.docker.com/resources/what-container)
  - [Docker Swarm](https://searchitoperations.techtarget.com/definition/Docker-Swarm)
  - [Docker Overview](https://docs.docker.com/engine/docker-overview/)
  - [Docker: get started](https://docs.docker.com/get-started/)

- Understanding Infrastructure as a code
  - [Infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_code)
  - [Terraform(Golang)](https://en.wikipedia.org/wiki/Terraform_(software))
  - [Ansible(Python)](https://en.wikipedia.org/wiki/Ansible_(software))
  - [Book(Hard Copy): Infrastructure as code (O'Reilly)](http://shop.oreilly.com/product/0636920039297.do)
