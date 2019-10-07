# Docker 설치

## /etc/apt/sources.list.d/docker.list 
```
deb [arch=armhf] https://download.docker.com/linux/raspbian buster stable
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
