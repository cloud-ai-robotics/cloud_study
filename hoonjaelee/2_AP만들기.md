## 라즈베리파이로 AP 만들기
이번 실습에서는 무선 랜카드 두 개로 AP를 만드는 것을 했음
1. AP 만들기
2. DHCP 설정
3. 내부 네트워크와 외부 네트워크 연결

## 0. 무선랜 드라이버 설치
왠만한 무선랜은 바로 인식한다고 알고 있었는데, TL-WN823Nv2 (8192eu) 동글은 드라이버를 따로 잡아줘야 함.
(무선랜 자동설치 스크립트 https://www.raspberrypi.org/forums/viewtopic.php?t=198015)

## 1. AP 만들기
  #### 1. 무선랜카드 정보 확인
   - 무선랜 관리 패키지 설치 `sudo apt-get install iw` 
   - `iw list` 실행 후 Supported interface modes 에 AP가 있는지 확인. 없으면 AP모드를 설정 할 수 없음
   - `ifconfig` 실행 후, AP로 사용 할 무선랜카드의 이름 확인 (ex. wlan1)
   
  #### 2. AP 모드 설정
   - AP 패키지 설치 `sudo apt-get install hostapd`
   - `sudo vim /etc/hostapd/hostapd.conf` 으로 hostapd.conf 파일을 만들고 아래와 같이 작성
  ~~~
  interface=wlan1
  ssid=raspberryAP
  ignore_broadcast_ssid=0
  hw_mode=g
  channel=11
  wpa=2
  wpa_passphrase=berryberry
  wpa_key_mgmt=WPA-PSK
  wpa_pairwise=TKIP
  rsn_pairwise=CCMP
  wpa_ptk_rekey=600
  macaddr_acl=0
  ~~~

  - `sudo vim /etc/default/hostapd` 를 이용해 hostapd에 위에서 작성한 파일을 추가 해 줌
  ~~~
  DAEMON_CONF="/etc/hostapd/hostapd.conf"
  ~~~
  
  - `sudo hostapd -d /etc/hostapd/hostapd.conf` 명령어로 AP 테스트
  
  AP는 설정이 되었으나, IP를 할당 해 주지 못하여 SSID는 보이지만 접속이 불가능 함.
  
  - 백그라운드 모드로 실행하는 명령어
  ~~~
  sudo systemctl unmask hostapd
  sudo systemctl enable hostapd
  sudo systemctl start hostapd
  ~~~
  
  
  #### 3. DHCP 설정
  - AP로 사용할 무선랜카드를 고정 아이피로 변경 `sudo vim /etc/dhcpcd.conf` 아래 내용 추가
  ~~~
  interface wlan0
    static ip_address=192.168.4.1/24
    nohook wpa_supplicant
  ~~~
  - 아래 명령어로 dhcpcd 재시작 후 ip 다시 받아오기
  ~~~
  sudo service dhcpcd restart
  sudo ifconfig wlan1 down
  sudo ifconfig wlan1 up
  ~~~
  - DHCP 패키지 설치 `sudo apt-get install dnsmasq`
  - dnsmasq.conf 설정 `sudo vim /etc/dnsmasq.conf`
  ~~~
  interface=wlan1
  dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
  ~~~
  
  - 여기까지 완료하면 만든 AP에 연결 가능
  
  
  ### 4. 내부 네트워크와 외부 네트워크 연결하기 <span style="color:blue">(*잘 안됨*)</span>
  내부 네트워크와 외부 네트워크를 연결시켜서 AP에 접속하여 인터넷까지 되게 해보자
  - 무선 네트워크끼리의 연결은 masqerading 방식만 지원 한다고 함
  ~~~
  sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
  sudo iptables -A FORWARD -i wlan0 -o wlan1 -m state --state RELATED,ESTABLISHED -j ACCEPT
  sudo iptables -A FORWARD -i wlan1 -o wlan0 -j ACCEPT
  sudo iptables-save > iptables.ipv4.nat
  ~~~
  
  ### 5. 여러개의 무선랜이 있을 때, 특정랜에 정보 설정하기
  - `sudo vim /etc/wpa_supplicant/wpa_supplicant-wlan0.conf`
  - `systemctl enable wpa_supplicant@wlan0`
  

##참고자료
1. https://dejavuqa.tistory.com/284
2. https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md
3. https://github.com/ai-robotics-kr/cloud_study/wiki/Build-up-the-Access-Point-on-the-RPi3
  
  
