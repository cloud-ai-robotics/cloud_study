## 0. 라즈비안_lite 설치
- 라즈베리 파이를 원격으로 접속해서 사용할 예정이므로 이미지 용량이 작은 raspbian-buster-lite를 설치함
- 참고로 raspbian-buster-lite는 GUI 환경이 빠진 이미지임
  
## 1. SSH 설정
- 라즈비안에서는 부팅할 때 boot 폴더에 비어있는 ssh 파일이 있으면 자동으로 ssh를 활성화함
- sd카드 boot 폴더에 비어있는 ssh 파일을 만듦 (확장자 없음)

## 2. 무선랜 설정
- 라즈비안에서는 부팅할 때 boot 폴더에 있는 wpa_supplicant.conf 파일을 참고하여 무선랜을 연결함
- sd카드 boot 폴더에 wpa_supplicant.conf 파일을 만들고 아래와 같이 내용을 채움
~~~
country=GB #KR은 안되는 것 같음 (why?)
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
ssid="NETWORK_NAME"
psk="PASSWORD"
}
~~~
  
  
## 3. 라즈베리파이 IP 찾기
- 무선 공유기에 접근이 가능 할 경우에는 무선 공유기에서 라즈베리파이로 추정되는 아이피로 ssh 접속을 시도하여 확인
- 무선 공유기에 접근이 불가능 할 경우에는 nmap을 이용하여 내가 접속하고 있는 공유기를 사용하고 있는 모든 아이피를 검색하여 확인
  ex) 내 아이피가 172.30.1.xx 라면 공유기가 만들어주는 사설IP의 대역은 172.30.1.xxx임. 따라서 nmap -sn 172.30.1.0/24 명령어를 이용하면 라즈베리파이의 아이피를 검색 할 수 있음
  
## 4. SSH 접속
- 1번 설정을 통해서 SSH를 활성화 하고, 2번을 통해서 라즈베리파이를 공유기에 접속시킨 후, 3번을 통해서 IP를 알아내었다면, 이제 무선으로 접속이 가능함.
- 접속 명령어 `ssh 172.30.1.43 -l pi` 기본 비밀번호 `raspberry`

## 주의
- 라즈베리파이에 SD 카드를 넣고 한 번 부팅하면 boot 폴더의 ssh 파일과 wpa_supplicant.conf 파일은 삭제됨
- 하지만 세팅값은 저장 되는 것 같음

## 참고자료
1. https://www.raspberrypi.org/downloads/raspbian/
2. https://www.raspberrypi.org/documentation/installation/installing-images/README.md
3. https://m.blog.naver.com/PostView.nhn?blogId=specialist0&logNo=221232983680
4. https://w1.fi/cgit/hostap/plain/wpa_supplicant/wpa_supplicant.conf
