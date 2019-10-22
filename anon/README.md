# Docker

## Docker Repository 추가

 * /etc/apt/sources.list.d/docker.list 파일 생성
```
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
$ echo "deb [arch=armhf] https://download.docker.com/linux/raspbian buster stable" > /etc/apt/sources.list.d/docker.list
$ apt update
```

## Docker 패키지 설치

 * RPi 에서 Docker build 를 할 때, Unexpected EOF 에러가 나는 경우, docker 를 다시 설치한다.
```
$ apt update
$ apt install docker.io runc
$ systemctl enable docker
```

## Docker 동작 환경 설정


 * 부팅 파라미터를 수정: /boot/cmdline.txt (없으면) 아래 항목들을 추가
```
cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1
```

### Swap memory 를 모두 Disable

```
$ dphys-swapfile swapoff
$ dphys-swapfile uninstall
$ update-rc.d dphys-swapfile remove
$ vi /etc/dphys-swapfile # CONF_SWAP_SIZE=0
```

### Docker 설정 파일 수정
 * /etc/docker/damon.json 파일 수정
   - container 들이 사용할 사설망을 지정
```
{
   "exec-opts": ["native.cgroupdriver=systemd"],
   "log-driver": "json-file",
   "log-opts": {
       "max-size": "100m"
   },
   "storage-driver": "overlay2",
   "bip": "192.168.1.1/24",
   "ipv6": false
}
```

## Docker Image 만들기

## Dockerfile 작성

 * 첫 줄 syntax 커멘트는 Docker 를 빌드할 때, 아직 정식 포함되지 않은 기능을 사용할 수 있도록 Toggle 시키는 기능임
```
# syntax=docker/dockerfile:experimental
FROM ubuntu:18.04
MAINTAINER anon <cloud@futuremobile.net>
RUN apt-get update
RUN apt-get install -y git nodejs
RUN echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config
RUN --mount=type=ssh git clone git@github.com:/ai-robotics-kr/cloud_study
WORKDIR /cloud
CMD ["/usr/bin/node /cloud/cloud_study/anon/service/webserver.js"]
```
 1. ubuntu 18.04 이미지를 base 로 해서, package repository 를 갱신하고, git 과 nodejs 패키지를 설치한다.
 2. ssh client 설정에 Host key checking 옵션을 끈다. (최초 접속시 추가하겠냐고 묻지 않게 함)
 3. [--mount](https://docs.docker.com/storage/bind-mounts/) 옵션에 [type=ssh](https://docs.docker.com/develop/develop-images/build_enhancements/) 기능을 사용해서, Host 의 SSH credentials 에 접근할 수 있게 한다. (참고: Using SSH to access private data in builds)
 4. webserver 를 실행시킨다.

## 이미지 만들기
[Build Enhancements for Docker](https://docs.docker.com/develop/develop-images/build_enhancements/) 의 "DOCKER_BUILDKIT" 환경 변수에 대한 설명을 참고할 것.
```
$ DOCKER_BUILDKIT=1 docker build . -t git:0.3 --ssh default
```

## Container 실행시키기

```
$ docker run --name hello-nginx3 -d -p 8080:80 -v /root/example/data:/data hello:0.1
```

# Docker Privat Registry 만들기

## 인증을 위해 아래와 같이 htpasswd 로 사용자 계정을 생성한다.

```
$ mkdir auth
$ docker run --entrypoint htpasswd registry:2 -Bbn ${USER_ID} ${USER_PASSWD} > auth/htpasswd
$ docker stop registry
```

## https 를 위해 아래와 같이 self-signed certificate 를 만든다. (let's encrypt 를 써도 됨)
```
$ mkdir certs
$ cd certs
$ openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 9999 -out cert.pem
$ openssl x509 -text -noout -in certificate.pem
$ openssl x509 -text -noout -in cert.pem
$ openssl pkcs12 -inkey key.pem -in cert.pem -export -out cert.p12
$ openssl pkcs12 -in cert.p12 -noout -info
```

## docker registry 를 실행시킨다.

```
$ docker run -d -p 5000:5000 --restart=always --name registry -v /root/docker/registry/auth:/auth -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd -v /root/docker/registry/certs:/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/cert.pem -e REGISTRY_HTTP_TLS_KEY=/certs/key.pem -e REGISTRY_HTTP_ADDR=0.0.0.0:5000 registry:2
```

## docker login 테스트

```
$ docker login https://localhost:5000 
```

## docker 에 이미지 push 해보기
```
$ docker pull ubuntu:18.04
$ docker tag ubuntu:18.04 registry.nicesj.com/ubuntu18.04
$ docker push registry.nicesj.com/ubuntu18.04
$ docker pull registry.nicesj.com/ubuntu18.04
```

## self-signed certificate 를 신뢰하도록 추가

> Linux Ubuntu/Debian
>
> Ubuntu/Debian allows you to install extra root certificates via the /usr/local/share/ca-certificates directory. To install your own root authority certificate copy your root certificate to /usr/local/share/ca-certificates. Make sure the file has the .crt extension. so rename it when necessary.
>
> After you copied your certificate to the /usr/local/share/ca-certificates folder you need to refresh the installed certificates and hashes. Within ubuntu/debian you can perform this action via one command:
>
> sudo update-ca-certificates
>
> You will notice that the command reports it has installed one (or more) new certificate. The certificate has been added to the Operating System and signed certificates will be trusted.
>
> To remove the certificate, just remove it from /usr/local/share/ca-certificates and run
>
> sudo update-ca-certificates --fresh
>

### References

 * [Install root certificate](https://www.bounca.org/tutorials/install_root_certificate.html)

# Kubernetes

## 패키지 설치

```
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ apt-get update -q
$ apt-get install -qy kubeadm
```

## DNS 정리

 * kubernetes 를 초기화 하기 전에, 각 Node 들의 이름을 잘 정리해둔다.

```
$ cat /etc/hosts
10.0.1.24 worker0
$ hostnamectl
   Static hostname: raspberrypi
         Icon name: computer
        Machine ID: e1c2b31c295d4657a3dbab8a879a71a3
           Boot ID: 5e048c32a2494d5589de6299526e31da
  Operating System: Raspbian GNU/Linux 10 (buster)
            Kernel: Linux 4.19.66-v7+
      Architecture: arm
```

 * Raspberry Pi 의 경우, raspberry pi config tool 을 이용하여, hostname 설정


## kubernetes 초기화

  * kubeadm init 할 때 옵션들을 잘 챙겨볼것. 또는 config.yaml 파일을 만들어서 적용(이게 최신 방법임)

### Master node 초기화

```
raspberrypi:~/kubernetes# kubeadm init --pod-network-cidr=10.0.0.0/16
[init] Using Kubernetes version: v1.16.1
[preflight] Running pre-flight checks
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 19.03.2. Latest validated version: 18.09
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [raspberrypi kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.0.6]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [raspberrypi localhost] and IPs [192.168.0.6 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [raspberrypi localhost] and IPs [192.168.0.6 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.
[apiclient] All control plane components are healthy after 213.520782 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.16" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node raspberrypi as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node raspberrypi as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: uk3kdd.3ypc9mpij86wtg31
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.6:6443 --token uk3kdd.3ypc9mpij86wtg31 \
    --discovery-token-ca-cert-hash sha256:9ef15db5970a8c9987190f6f758394b9db304abaa56127c5306d67e274ac4578
raspberrypi:~/kubernetes#
```

### Addon 설치

 * Kubernetes Cluster 가 구성되면 필요한 경우, 추가 구성 요소들을 설치(Deploy) 한다.
   Reference: [Addons](https://kubernetes.io/docs/concepts/cluster-administration/addons/)

 * network addon 을 설치하지 않으면, coredns 가 pending 상태로 있게 되며,
   추가된 worker node 가 not ready 상태로 남아 있게 됨.


#### Weave net

 * 일단 AWS 를 보니, 이것 저것 얘기하는데, weave net 이 얘기가 많길레, weave net 을 설치해봄

__반드시, weavenet 을 먼저 설치하고, workernode 를 추가할것__

__RPi에서 crash 가 났음: `kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.WEAVE_NO_FASTDP=1"`__

```
raspberrypi:~/cloud_study# kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.apps/weave-net created

raspberrypi:~/cloud_study# kubectl get pods -n kube-system
NAME                       READY   STATUS              RESTARTS   AGE
coredns-5644d7b6d9-hzssx   0/1     Pending             0          80m
coredns-5644d7b6d9-kvqmz   0/1     Pending             0          80m
kube-proxy-fk6xz           1/1     Running             0          28m
weave-net-4l2xs            0/2     ContainerCreating   0          108s
```

 * 모두 설치가 끝난 후 아래와 같은 에러가 나는 경우,

```
$ journalctl -xe -f
10월 13 09:53:15 master.rpi.anon.com kubelet[4208]: E1013 09:53:15.172810    4208 dns.go:135] Nameserver limits were exceeded, some nameservers have been omitted, the applied nameserver line is: 10.0.0.1 10.0.1.1 168.126.63.1
```

 * /etc/resolve.conf 에 아래 줄을 추가한다.

```
search localdomain service.ns.svc.cluster.local
```

 * weavenet 삭제하기

```
$ kubectl -n kube-system delete -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

#### Flannel

 * Flannel 은 kubeadm init 을 할 때 pod 들이 cidr(Classless Inter-Domain Routing) 옵션을 반드시 넣어줘야 한다.

```
$ kubeadm init --pod-network-cidr=10.0.0.0/16
```

### Worker node 초기화

 * Worker node 에서 join 을 할 때, 오류가 나는 경우, -v=6 옵션을 붙여서 상세 로그를 확인할 수 있다.
```
anon@raspberrypi:~ # kubeadm join 192.168.0.6:6443 --token qp7xq6.an1s8n3x1advuln5 \
>     --discovery-token-ca-cert-hash sha256:af8d65f8c1775042adb18da87631dafec87b324a3d6685fdb1b5dbf6829a1417
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.16" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

 * 만약 토큰이 오래된 경우, master node 에서 토큰을 갱신한다.
```
$ kubeadm token create
```

 * join 명령어 찾는 방법
```
master:~/cloud_study/anon# kubeadm token create --print-join-command
kubeadm join 192.168.0.6:6443 --token u3fqr3.xesi5amthrd4mc61     --discovery-token-ca-cert-hash sha256:95155c5cd47f146405332427088f1118b430908dfda8017fb8c0144c582e84d7
```

> Worker node 가 10.0.1.x network (ethernet interface 에 구성) 에 있기 때문에, join 을 할 때, 10.0.1.1 gateway API 로 시도 했더니,
> 인증되지 않은 주소라고 해서, 이것 저것 찾아보다가,. 어짜피 worker node network 에서도 192.168.0.6 으로 접근 할 수 있기 때문에,
> 그냥 했더니.. 잘 됨. 그래도 궁금한 것 하나는 남음
> kubernetes init 을 할 때, cluster IP 인증 목록은 어떻게 변경할 수 있는거지?
> Reference: (Invalid X.509 Certificate for the K8s master)[https://stackoverflow.com/questions/46360361/invalid-x509-certificate-for-kubernetes-master]

### Master node 에서 node 목록 확인하기
```
$ kubectl get node
   NAME      STATUS  ROLES  AGE   VERSION
raspberrypi NotReady master 9m32s v1.16.1
```

## Persistent Volume (Claim)

### Persistent Volume

 * Configuration file
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: myNFS
spec:
  capacity:
    storage: 5Gi
  accessMode:
    - ReadWriteMany
  nfs:
    server: 100.100.100.100
    path: /anon
```

  * `kubectl apply`
```
$ kubectl apply -f myNFS.yaml
```

### Persistent Volume Claim
 * Configuration file
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myNFS-Claim
spec:
  accessMode:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
```

 * `kubectl apply`
```
$ kubectl apply -f myNFS-Caim.yaml
```

 StorageClass 가 없는 경우 Default 를 기준으로,
 AccessMode 와 Request Storage Size 를 이용해서, Persistent Volume 을 찾는다.

## Docker private registry 에서 image pull 하기

 * [Pull image from private registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

# 기타 (Raspberry Pi)

## DHCP Client daemon 수정 (feat. dhcpcd)
```
$ cat /etc/dhcpcd.conf
...
denyinterfaces wlan1
denyinterfaces eth0
...
```
wlan1 과 eth0 에 대해서는 dhcp 로 부터 ip 받아오는 것을 방지함

## DHCP 서버 설정 (feat. dnsmasq)

 * interface 와 listen-address 는 둘 중 하나만 설정해도 무관함`
   - interface: 서비스 접근 가능 interface 지정
   - listen-address: 서비스 접근 가능 주소 지정
   - no-resolv: /etc/resolve.conf 파일을 참고 하지 않도록 함

```
$ cat /etc/dnsmasq.conf
interface=wlan1
interface=eth0
listen-address=10.0.0.1
listen-address=10.0.1.1
bind-interfaces
bogus-priv
dhcp-range=10.0.0.3,10.0.0.250,255.255.255.0,24h
dhcp-range=10.0.1.3,10.0.1.250,255.255.255.0,24h
server=8.8.8.8
server=8.8.4.4
no-resolv
```

 * wlan1 네트워크 대역은 10.0.0.1
   eth0 네트워크 대역은 10.0.1.1

> 현재 eth0 은 rpi2 와 연결 시켰고,
> wlan1 은 AP 로 사용하고 있음

```
$ systemctl enable dnsmasq
```

## eth0 NAT table 에 추가하기

 * ethernet 으로 연결되는 망 (rpi2) 에 대한 masquerading 을 위해 NAT table 을 아래와 같이 수정(추가)

```
-A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -i eth0 -o wlan0 -j ACCEPT
```

## local DNS 서버 설정 (feat. dnsmasq)

 * 마지막 줄에 domain name 과 ip 추가

```
address=/master.rpi.anon.com/192.168.0.6
address=/master.rpi.anon.com/10.0.0.1
address=/master.rpi.anon.com/10.0.1.1
address=/worker0.rpi.anon.com/10.0.1.24
```

 * /etc/default/dnsmasq 에 port 번호 옵션 추가 (Optional)
```
# This file has five functions: 
# 1) to completely disable starting dnsmasq, 
# 2) to set DOMAIN_SUFFIX by running `dnsdomainname` 
# 3) to select an alternative config file
#    by setting DNSMASQ_OPTS to --conf-file=<file>
# 4) to tell dnsmasq to read the files in /etc/dnsmasq.d for
#    more configuration variables.
# 5) to stop the resolvconf package from controlling dnsmasq's
#    idea of which upstream nameservers to use.
# For upgraders from very old versions, all the shell variables set 
# here in previous versions are still honored by the init script
# so if you just keep your old version of this file nothing will break.

#DOMAIN_SUFFIX=`dnsdomainname`
#DNSMASQ_OPTS="--conf-file=/etc/dnsmasq.alt"
DNSMASQ_OPTS="--port=53"

# Whether or not to run the dnsmasq daemon; set to 0 to disable.
ENABLED=1

# By default search this drop directory for configuration options.
# Libvirt leaves a file here to make the system dnsmasq play nice.
# Comment out this line if you don't want this. The dpkg-* are file
# endings which cause dnsmasq to skip that file. This avoids pulling
# in backups made by dpkg.
CONFIG_DIR=/etc/dnsmasq.d,.dpkg-dist,.dpkg-old,.dpkg-new

# If the resolvconf package is installed, dnsmasq will use its output 
# rather than the contents of /etc/resolv.conf to find upstream 
# nameservers. Uncommenting this line inhibits this behaviour.
# Note that including a "resolv-file=<filename>" line in 
# /etc/dnsmasq.conf is not enough to override resolvconf if it is
# installed: the line below must be uncommented.
#IGNORE_RESOLVCONF=yes
```

# References

 * [CoreDNS](https://coredns.io/): Kubernetes cluster 안의 Pod 들이 참고하는 DNS Service
 * [Kube Proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)
 * [Custom domains with dnsmasq](https://github.com/RMerl/asuswrt-merlin/wiki/Custom-domains-with-dnsmasq)
