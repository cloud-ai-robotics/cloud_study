# Overview
아래 문서는 RPi 에서 AP 및 k8s 환경 구축을 위한 가이드입니다.
일부 권한이 필요한 명령의 경우, sudo 를 사용해야 하지만, 문서에서는 생략하고 있습니다.

# Docker

## Docker Package Repository 추가 - Raspian Buster

 * /etc/apt/sources.list.d/docker.list 파일 생성
```
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
$ echo "deb [arch=armhf] https://download.docker.com/linux/raspbian buster stable" > /etc/apt/sources.list.d/docker.list
$ apt update
```

## Docker Package 설치

 * RPi 에서 Docker build 를 할 때, Unexpected EOF 에러가 나는 경우, docker 를 다시 설치한다. (18.09 버전이 설치되어야 Crash 가 나지 않음)
```
$ apt update
$ apt install docker.io runc
$ systemctl enable docker
```

## Docker 환경 설정

### 커널 부트 파라미터 수정

 * /boot/cmdline.txt (없으면) 아래 항목들을 줄의 끝에 추가
```
cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1
```

### Swap memory 를 모두 Disable

```
$ dphys-swapfile swapoff
$ dphys-swapfile uninstall
$ update-rc.d dphys-swapfile remove
$ vi /etc/dphys-swapfile # CONF_SWAP_SIZE=0 로 수정
```

### Docker 설정 파일 수정
 * /etc/docker/daemon.json 파일 수정
   - container 들이 사용할 IP 주소 대역을 지정
     추 후 K8s cluster init 을 할 때, pod 들이 사용할 IP address 대역을 지정합니다.
     이 IP 주소들은 docker command 로 k8s cluster 를 거치지 않고 직접 container 를 동작시킬 때 사용하게 되는 ip 영역으로 cluster 내에서 각 container 가 사용하게 될 주소와는 다를 수 있습니다.
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

## Docker Image 만들어 보기

실제 Docker image 를 만들어 보고, 실행시켜서 container 를 생성해 보겠습니다.

### Dockerfile 작성

 * 첫 줄 syntax 커멘트는 Docker 를 빌드할 때, 아직 정식 포함되지 않은 기능을 사용할 수 있도록 Toggle 시키는 기능임
   docker/dockerfile 이라는 image 에서 experimental version 이 붙은 것을 dockerhub.io 에서 받아 사용하게 됩니다.
```
# syntax=docker/dockerfile:experimental
FROM ubuntu:18.04
MAINTAINER anon <cloud@futuremobile.net>
RUN apt-get update
RUN apt-get install -y git nodejs
RUN echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config
RUN --mount=type=ssh git clone git@github.com:/ai-robotics-kr/cloud_study
WORKDIR /cloud
CMD ["/usr/bin/node", "/cloud/cloud_study/anon/service/webserver.js"]
```

 1. ubuntu 18.04 이미지를 base 로 해서, package repository 를 갱신하고, git 과 nodejs 패키지를 설치한다.
 2. ssh client 설정에 Host key checking 옵션을 끈다. (최초 접속시 추가하겠냐고 묻지 않게 함)
 3. [--mount](https://docs.docker.com/storage/bind-mounts/) 옵션에 [type=ssh](https://docs.docker.com/develop/develop-images/build_enhancements/) 기능을 사용해서, Host 의 SSH credentials 에 접근할 수 있게 한다. (참고: Using SSH to access private data in builds)
 4. webserver 를 실행시킨다.

### 이미지 build 하기

 * ssh-agent 를 실행시킵니다.
 * ssh-add 로 현재 사용중인 ssh credential 을 ssh-agent daemon 에 추가시킵니다.
 * [Build Enhancements for Docker](https://docs.docker.com/develop/develop-images/build_enhancements/) 의 "DOCKER_BUILDKIT" 환경 변수에 대한 설명을 참고할 것.

```
$ eval `ssh-agent`
$ ssh-add
$ DOCKER_BUILDKIT=1 docker build . -t git:0.3 --ssh default
```

### Container 실행시키기

```
$ docker run --name hello-nginx3 -d -p 8080:80 -v /root/example/data:/data hello:0.1
```

# Docker Privat Registry 만들기

## Docker Registry 사용자 인증을 위해 아래와 같이 htpasswd 로 계정을 생성한다.

```
$ mkdir auth
$ docker run --entrypoint htpasswd registry:2 -Bbn ${USER_ID} ${USER_PASSWD} > auth/htpasswd
$ docker stop registry
```

## https (TLS enabling) 를 위해 아래와 같이 self-signed certificate 를 만든다. (let's encrypt 를 써도 됨)
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

 * REGISTRY_STORAGE_DELETE_ENABLED 는 기본적으로 false 상태로 동작합니다. 이 환경 변수는 registry 에 등록된 이미지에 대한 삭제 기능 활성화 여부를 설정합니다.

```
$ docker run -d -p 5000:5000 --restart=always --name registry -v /root/docker/registry/auth:/auth -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd -v /root/docker/registry/certs:/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/cert.pem -e REGISTRY_HTTP_TLS_KEY=/certs/key.pem -e REGISTRY_HTTP_ADDR=0.0.0.0:5000 -e REGISTRY_STORAGE_DELETE_ENABLED=true registry:2
```

## docker login 테스트

```
$ docker login https://localhost:5000 
```

## docker 에 이미지 push 해보기
```
$ docker pull ubuntu:18.04
$ docker tag ubuntu:18.04 registry.nicesj.com/ubuntu:18.04
$ docker push registry.nicesj.com/ubuntu:18.04
$ docker pull registry.nicesj.com/ubuntu:18.04
#
# Dockerfile 빌드를 위해 dockerfile:experimental image 도 push (굳이 안해도 되지만, 완전히 폐쇄된 cluster 환경을 구축하려고 한다면야...)
#
$ docker pull docker.io/docker/dockerfile:experimental
$ docker tag docker/dockerfile:experimental registry.nicesj.com/docker/dockerfile:experimental
$ docker push registry.nicesj.com/dockerfile:experimental
```

## 기존 Dockerfile 을 빌드해서 private registry 에 넣어보기
```
#
# 빌드하기, tag 를 그을때 registry.nicesj.com/hello-world:0.1 이라고 하면 태그를 다시 긋지 않아도 됨
#
$ DOCKER_BUILDKIT=1 docker build . -t hello-world:0.1 --ssh default 
[+] Building 71.8s (14/14) FINISHED
 => [internal] load .dockerignore                                                                                                                                                1.5s
 => => transferring context: 2B                                                                                                                                                  0.0s
 => [internal] load build definition from Dockerfile                                                                                                                             0.5s
 => => transferring dockerfile: 474B                                                                                                                                             0.0s
 => resolve image config for docker.io/docker/dockerfile:experimental                                                                                                            0.0s
 => CACHED docker-image://docker.io/docker/dockerfile:experimental                                                                                                               0.0s
 => [internal] load build definition from Dockerfile                                                                                                                             1.0s
 => => transferring dockerfile: 474B                                                                                                                                             0.0s
 => [internal] load .dockerignore                                                                                                                                                0.0s
 => [internal] load metadata for registry.nicesj.com/ubuntu:18.04                                                                                                                0.0s
 => [1/6] FROM registry.nicesj.com/ubuntu:18.04                                                                                                                                  0.0s
 => CACHED [2/6] RUN apt-get update                                                                                                                                              0.0s
 => CACHED [3/6] RUN apt-get install -y git nodejs                                                                                                                               0.0s
 => CACHED [4/6] RUN echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config                                                                                                      0.0s
 => CACHED [5/6] RUN --mount=type=ssh git clone git@github.com:/ai-robotics-kr/cloud_study && mv cloud_study/anon/service/webserver.js /cloud                                    0.0s
 => [6/6] RUN apt remove -y git                                                                                                                                                 25.8s
 => exporting to image                                                                                                                                                          38.4s
 => => exporting layers                                                                                                                                                         37.9s
 => => writing image sha256:0472e3d1d059424f7d4ab5abb8b1654f86d5c527cbe70aa08c94cb4ccd34b4a9                                                                                     0.0s
 => => naming to docker.io/library/hello-world:0.1                                                                                                                               0.0s
#
# 태그 긋기 (빌드할 때, tag 를 registry.nicesj.com/hello-world:0.1 로 했다면 바로 push 하면 됨)
#
$ docker tag hello-world:0.1 registry.nicesj.com/hello-world:0.1
#
# push 하기
#
$ docker push registry.nicesj.com/hello-world:0.1
The push refers to repository [registry.nicesj.com/hello-world]
0e1202019357: Pushed
03936511f26e: Pushed
abb8b2ee0c22: Pushed
d79f655bec49: Pushed
fc8d18b74b84: Pushed
b4337dcf41dd: Mounted from ubuntu
bb5cec76f6eb: Mounted from ubuntu
660db4a2f7fb: Mounted from ubuntu
4a1049d3ae45: Mounted from ubuntu
0.1: digest: sha256:dcc2bf8583bac15535c16137c8527aa156c4288d94fbe065349a41946aa21405 size: 2203
```

## self-signed certificate 를 신뢰하도록 추가

직접 만든 인증서의 경우, 기본적으로 추가 되어 있는 "신뢰할 수 있는 인증기관" 에 내 인증서에 대한 정보가 없으므로, 신뢰할 수 있는 인증기관에 내 인증서를 추가한다.

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

## docker registry 내용 확인하기

```
#
# registry 가 동작중인 container Id 를 찾는다.
#
$ docker ps -a
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                    NAMES
44656edcb0c3        ef3b5d63729b           "/opt/bin/flanneld -…"   9 hours ago         Up 9 hours                                   k8s_kube-flannel_kube-flannel-ds-arm-5xw4x_kube-system_047eacc5-b887-4e08-b7e5-a0f96c980d3a_6
9b966978f24e        56b69fd5e66b           "/usr/local/bin/kube…"   9 hours ago         Up 9 hours                                   k8s_kube-proxy_kube-proxy-vcjrv_kube-system_874a86b2-9536-40ca-a857-df428d7b8db9_5
235167eafa7a        k8s.gcr.io/pause:3.1   "/pause"                 9 hours ago         Up 9 hours                                   k8s_POD_kube-flannel-ds-arm-5xw4x_kube-system_047eacc5-b887-4e08-b7e5-a0f96c980d3a_5
8d638c4fd8b8        k8s.gcr.io/pause:3.1   "/pause"                 9 hours ago         Up 9 hours                                   k8s_POD_kube-proxy-vcjrv_kube-system_874a86b2-9536-40ca-a857-df428d7b8db9_5
a50753985be4        85ba11732b25           "kube-controller-man…"   9 hours ago         Up 9 hours                                   k8s_kube-controller-manager_kube-controller-manager-master.rpi.nicesj.com_kube-system_42cb3c7ba289276039d9a6a8ec36bf63_6
b2d7c1afd7e2        ad0038846086           "etcd --advertise-cl…"   9 hours ago         Up 9 hours                                   k8s_etcd_etcd-master.rpi.nicesj.com_kube-system_a55e1b299c7417851a7bc7f1a1825a8a_6
9658c9988c57        dc9dd64ec734           "kube-scheduler --au…"   9 hours ago         Up 9 hours                                   k8s_kube-scheduler_kube-scheduler-master.rpi.nicesj.com_kube-system_74dea8da17aa6241e5e4f7b2ba4e1d8e_7
603d05140317        2b20336a96ec           "kube-apiserver --ad…"   9 hours ago         Up 9 hours                                   k8s_kube-apiserver_kube-apiserver-master.rpi.nicesj.com_kube-system_d4db64777301cacf8f286ff6c4393ea1_10
840e5e57f217        k8s.gcr.io/pause:3.1   "/pause"                 9 hours ago         Up 9 hours                                   k8s_POD_kube-controller-manager-master.rpi.nicesj.com_kube-system_42cb3c7ba289276039d9a6a8ec36bf63_6
52914d751666        k8s.gcr.io/pause:3.1   "/pause"                 9 hours ago         Up 9 hours                                   k8s_POD_etcd-master.rpi.nicesj.com_kube-system_a55e1b299c7417851a7bc7f1a1825a8a_6
6ac76ea5e118        k8s.gcr.io/pause:3.1   "/pause"                 9 hours ago         Up 9 hours                                   k8s_POD_kube-scheduler-master.rpi.nicesj.com_kube-system_74dea8da17aa6241e5e4f7b2ba4e1d8e_6
00c290eb25d4        k8s.gcr.io/pause:3.1   "/pause"                 9 hours ago         Up 9 hours                                   k8s_POD_kube-apiserver-master.rpi.nicesj.com_kube-system_d4db64777301cacf8f286ff6c4393ea1_6
cf219e47e27a        registry:2             "/entrypoint.sh /etc…"   11 hours ago        Up 9 hours          0.0.0.0:5000->5000/tcp   registry
#
# registry container 의 shell 을 연다.
#
$ docker exec -it cf219e47e27a /bin/sh
#
# registry container shell 로 진입
#
/ # ls /var/lib/registry/docker/registry/v2/
blobs/         repositories/
/ # ls /var/lib/registry/docker/registry/v2/repositories/ubuntu18.04
_layers/     _manifests/  _uploads/
```

# Kubernetes

## 패키지 설치

 * Package repository 추가

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

 RPi3 에서 할 때, 초기화 중 에러가 발생하는 경우가 다수 발생했었음.
 중간에 실패하면, kubeadm 을 reset 하고 다시... 해보거나, 각 단계별로 직접 실행해줘야 함.
 특히, k8s API server 가 초기화 되는데, 많은 시간이 걸리면서, 다른 작업이 k8s API server 가 동작하지 않는다고, 실패하는 경우임

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
 * Weave net 은 ARM 버전 지원이 명확하지 않음.

__RPi에서 crash 가 났음: `kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.WEAVE_NO_FASTDP=1"` 해도 똑같음 <== ISA 가 다르기 때문임..__

 * 설치하기

```
raspberrypi:~/cloud_study# kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.apps/weave-net created
```

 * 삭제하기

```
$ kubectl -n kube-system delete -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

 * 설치 확인
```
raspberrypi:~/cloud_study# kubectl get pods -n kube-system
NAME                       READY   STATUS              RESTARTS   AGE
coredns-5644d7b6d9-hzssx   0/1     Pending             0          80m
coredns-5644d7b6d9-kvqmz   0/1     Pending             0          80m
kube-proxy-fk6xz           1/1     Running             0          28m
weave-net-4l2xs            0/2     ContainerCreating   0          108s
```

   - 모두 설치가 끝난 후 아래와 같은 에러가 나는 경우,
```
$ journalctl -xe -f
10월 13 09:53:15 master.rpi.anon.com kubelet[4208]: E1013 09:53:15.172810    4208 dns.go:135] Nameserver limits were exceeded, some nameservers have been omitted, the applied nameserver line is: 10.0.0.1 10.0.1.1 168.126.63.1
```

   - /etc/resolve.conf 에 아래 줄을 추가한다. (ISA 가 다른 경우에는 이걸 추가해도 의미 없음)
```
search localdomain service.ns.svc.cluster.local
```

#### Flannel

flannel 은 kubeadm init 을 할 때 pod 들이 cidr(Classless Inter-Domain Routing) 옵션을 반드시 넣어줘야 한다.

```
$ kubeadm init --pod-network-cidr=10.1.0.0/16
```

 * 설치하기
   RPi (ARM64) 에 설치하는 경우, kube-flannel.yml 파일의 amd64 를 arm64 로 변경해줘야 한다.
```
$ curl -sSL https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml | sed "s/amd64/arm64/g"  | kubectl apply -f -
```

 * 삭제하기
```
$ curl -sSL https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml | sed "s/amd64/arm64/g"  | kubectl delete -f -
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

### node 목록 확인하기
```
$ kubectl get node
   NAME      STATUS  ROLES  AGE   VERSION
raspberrypi NotReady master 9m32s v1.16.1
```

### cluster 에 hello-world (from private registry) deploy 시키기


  * deploy.yaml 파일 만들기
```
apiVersion: v1
kind: Pod
metadata:
        name: hello-world
spec:
        containers:
                - name: hello-world-container
                  image: registry.nicesj.com/hello-world:0.1
        imagePullSecrets:
                - name: regcred
```

  * kubectl 로 deploy 시키기
```
$ kubectl apply -f deploy.yaml 
$ kubectl get pod hello-world
$ kubectl get pods
NAME          READY   STATUS              RESTARTS   AGE
hello-world   0/1     ContainerCreating   0          3m56s
```


## 기타 - 추 후 정리 필요
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

 Reference: (Custom domains with dnsmasq)[https://github.com/RMerl/asuswrt-merlin/wiki/Custom-domains-with-dnsmasq]

 * systemd-resolved 설치

  /etc/resolv.conf 파일에 127.0.0.x 와 같이 localhost 가 지정되지 않도록 수정 필요 (localhost 로 지정되면, CoreDNS 에서 loop detection 으로 에러가 발생함)

  /etc/systemd/resovled.conf
```
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
#
# Entries in this file show the compile time defaults.
# You can change settings by editing this file.
# Defaults can be restored by simply deleting this file.
#
# See resolved.conf(5) for details

[Resolve]
DNS=10.0.1.1
#FallbackDNS=
#Domains=
#LLMNR=yes
#MulticastDNS=yes
#DNSSEC=allow-downgrade
#DNSOverTLS=no
#Cache=yes
#DNSStubListener=yes
#ReadEtcHosts=yes
```

# References

 * [Invalid X.509 Certificate for the K8s master](https://stackoverflow.com/questions/46360361/invalid-x509-certificate-for-kubernetes-master)
 * [Install root certificate](https://www.bounca.org/tutorials/install_root_certificate.html)
 * [CoreDNS](https://coredns.io/): Kubernetes cluster 안의 Pod 들이 참고하는 DNS Service
 * [Kube Proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)
 * [Custom domains with dnsmasq](https://github.com/RMerl/asuswrt-merlin/wiki/Custom-domains-with-dnsmasq)
