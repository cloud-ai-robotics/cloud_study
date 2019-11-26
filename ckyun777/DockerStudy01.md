## Docker

### nVidia Docker install

 nVidia Docker를 사용하면 nVidia GPU 환경을 독립된 독커 상에서 구현하여 여러가지 nVidia GPU 환경을 하나의 PC에서 동시에 사용 가능 하다. 

* https://github.com/NVIDIA/nvidia-docker

* Permission 문제 해결 :

  sudo usermod -a -G docker $USER 


* docker.sock 문제 있는 경우 :

  sudo setfacl -m user:$USER:rw /var/run/docker.sock

* Lazydocker 설치 : 독커의 편한 사용을 위해 

  https://github.com/jesseduffield/lazydocker

* -gpu 옵션 에러 해결 : 독커 재시작 

  sudo systemctl restart docker

  