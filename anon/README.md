# Docker 설치

## /etc/apt/sources.list.d/docker.list 
```
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
$ echo "deb [arch=armhf] https://download.docker.com/linux/raspbian buster stable" > /etc/apt/sources.list.d/docker.list
$ apt update
```

## Install packages
If you failed to build a docker image with "Unexpected EOF" error, install docker packages again.
```
$ apt update
$ apt install docker.io runc
$ systemctl enable docker
```

## /boot/cmdline.txt
 * (없으면) 아래 항목들을 추가
```
cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1
```

## Swap off
```
$ dphys-swapfile swapoff
$ dphys-swapfile uninstall
```
Disable swap permanently
```
$ update-rc.d dphys-swapfile remove
$ vi /etc/dphys-swapfile # CONF_SWAP_SIZE=0
```

# Docker
 * /etc/docker/damon.json 파일 수정
   - container 들이 사용할 사설망을 지정

## Dockerfile

## `docker build`
```
$ DOCKER_BUILDKIT=1 docker build . -t git:0.3 --ssh default
```

## `docker run`
```
$ docker run --name hello-nginx3 -d -p 8080:80 -v /root/example/data:/data hello:0.1
```

# Kubernetes

## install
```
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ apt-get update -q
$ apt-get install -qy kubeadm
```

## kubeadm init
```
raspberrypi:~/kubernetes# kubeadm init
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

kubeadm join 을 하기전에 worker node 의 hostname 을 알맞게 수정해줘야 한다.
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

Worker node 에서 join 을 할 때, 오류가 나는 경우, -v=6 옵션을 붙여서 상세 로그를 확인할 수 있다.
```
$ kubeadm join 192.168.0.6:6443 --token uk3kdd.3ypc9mpij86wtg31 \
    --discovery-token-ca-cert-hash sha256:9ef15db5970a8c9987190f6f758394b9db304abaa56127c5306d67e274ac4578 -v=6
```

만약 토큰이 오래된 경우, master node 에서 토큰을 갱신한다.
```
$ kubeadm token create
```

join 명령어 찾는 방법
```
master:~/cloud_study/anon# kubeadm token create --print-join-command
kubeadm join 192.168.0.6:6443 --token u3fqr3.xesi5amthrd4mc61     --discovery-token-ca-cert-hash sha256:95155c5cd47f146405332427088f1118b430908dfda8017fb8c0144c582e84d7
```

### Worker node initialization
```
nicesj.park@raspberrypi:~ # kubeadm join 192.168.0.6:6443 --token qp7xq6.an1s8n3x1advuln5 \
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

Worker node 가 10.0.1.x network (ethernet interface 에 구성) 에 있기 때문에, join 을 할 때, 10.0.1.1 gateway API 로 시도 했더니,
인증되지 않은 주소라고 해서, 이것 저것 찾아보다가,. 어짜피 worker node network 에서도 192.168.0.6 으로 접근 할 수 있기 때문에,
그냥 했더니.. 잘 됨. 그래도 궁금한 것 하나는 남음

> kubernetes init 을 할 때, cluster IP 인증 목록은 어떻게 변경할 수 있는거지?
> Reference: (Invalid X.509 Certificate for the K8s master)[https://stackoverflow.com/questions/46360361/invalid-x509-certificate-for-kubernetes-master]

### master node 에서 node 목록 확인하기
```
$ kubectl get node
   NAME      STATUS  ROLES  AGE   VERSION
raspberrypi NotReady master 9m32s v1.16.1
```

### Addon 설치하기

Kubernetes Cluster 가 구성되면 필요한 경우, 추가 구성 요소들을 설치(Deploy) 한다.

Reference: (Addons)[https://kubernetes.io/docs/concepts/cluster-administration/addons/]

network addon 을 설치하지 않으면, coredns 가 pending 상태로 있게 되며,
추가된 worker node 가 not ready 상태로 남아 있게 됨.


### Weave net
일단 AWS 를 보니, 이것 저것 얘기하는데, weave net 이 얘기가 많길레, weave net 을 설치해봄
"""반드시, weavnet 을 먼저 설치하고, workernode 를 추가할것"""
"""RPi에서 crash 가 났음: kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.WEAVE_NO_FASTDP=1""""
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

모두 설치가 끝난 후
```
$ journalctl -xe -f
10월 13 09:53:15 master.rpi.nicesj.com kubelet[4208]: E1013 09:53:15.172810    4208 dns.go:135] Nameserver limits were exceeded, some nameservers have been omitted, the applied nameserver line is: 10.0.0.1 10.0.1.1 168.126.63.1
```
이런 에러가 나는 경우,
/etc/resolve.conf 에 아래 줄을 추가한다.
```
search localdomain service.ns.svc.cluster.local
```

 * weavenet 삭제하기
```
$ kubectl -n kube-system delete -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

### Flannel
Flannel 은 kubeadm init 을 할 때 pod 들이 cidr(Classless Inter-Domain Routing) 옵션을 반드시 넣어줘야 한다.
```
$ kubeadm init --pod-network-cidr=10.0.0.0/16
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
wlan1 네트워크 대역은 10.0.0.1
eth0 네트워크 대역은 10.0.1.1

현재 eth0 은 rpi2 와 연결 시켰고,
wlan1 은 AP 로 사용하고 있음

```
$ systemctl enable dnsmasq
```

## eth0 NAT table 에 추가하기
ethernet 으로 연결되는 망 (rpi2) 에 대한 masquerading 을 위해 NAT table 을 아래와 같이 수정(추가)
```
-A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -i eth0 -o wlan0 -j ACCEPT
```

## local DNS 서버 설정 (feat. dnsmasq)

마지막 줄에 domain name 과 ip 추가
```
address=/master.rpi.nicesj.com/192.168.0.6
address=/master.rpi.nicesj.com/10.0.0.1
address=/master.rpi.nicesj.com/10.0.1.1
address=/worker0.rpi.nicesj.com/10.0.1.24
```

/etc/default/dnsmasq 에 port 번호 옵션 추가
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
