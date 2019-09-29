## 0. 라즈비안_lite 설치
- 라즈베리 파이를 원격으로 접속해서 사용할 예정이므로 이미지 용량이 작은 raspbian-buster-lite를 설치함
- 참고로 raspbian-buster-lite는 GUI 환경이 빠진 이미지임
- 참고자료
  1. https://www.raspberrypi.org/downloads/raspbian/
  2. https://www.raspberrypi.org/documentation/installation/installing-images/README.md
  
## 1. SSH 설정
- 라즈비안에서는 부팅할 때 boot 폴더에 비어있는 ssh 파일이 있으면 자동으로 ssh를 활성화함
- sd카드 boot 폴더에 비어있는 ssh 파일을 만듦 (확장자 없음)

## 2. 무선랜 설정
- 라즈비안에서는 부팅할 때 boot 폴더에 있는 wpa_supplicant.conf 파일을 참고하여 무선랜을 연결함
- sd카드 boot 폴더에 wpa_supplicant.conf 파일을 만들고 아래와 같이 내용을 채움
~~~
country=KR
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
ssid="NETWORK_NAME"
psk="PASSWORD"
key_mgmt=WPA-PSK
}
~~~
- 참고자료
  1. https://m.blog.naver.com/PostView.nhn?blogId=specialist0&logNo=221232983680
  2. https://w1.fi/cgit/hostap/plain/wpa_supplicant/wpa_supplicant.conf
  
