
정의(조금더 추가해야 함..)
---------------------------------------------------------------------------------------------------------------
- 쿠버네티스란 컨테이너를 쉽고 빠르게 배포/확장하고 관리를 자동화해주는 오픈소스 플랫폼
	1. kubernetes 이며 줄여서 'k8s' 또는 'kube' 라고도 함
	2. 단순한 컨테이너 플랫폼이 아닌 마이크로서비스, 클라우드 플랫폼을 지향하고 컨테이너로 이루어진 것들을 손쉽게 담고 관리할 수 있는 그릇 역할을 함
	3. 서버리스, CI/CD, 머신러닝 등 다양한 기능이 쿠버네티스 플랫폼 위에서 동작 가능

- 쿠버네티스 동작방식
	1. 중앙(master)노드와 서버(worker)노드로 나뉘며 중앙노드가 서버노드를 전체적으로 관리

- 쿠버네티스 단점?
	1. 복잡하고 어렵다.
	2. yaml 설정파일이 많고 클러스터 만드는 것 또한 어려움
	3. helm 패키지 매니저를 사용하여 설정파일 관리해야 함

---------------------------------------------------------------------------------------------------------------




실습
---------------------------------------------------------------------------------------------------------------
실행환경
	master 및 worker 노드 모두 우분투 18.04(LTS)

실행환경 최소 조건
	- Master node: CPU 1 core 2 이상, 스왑메모리 항상 OFF (sudo swapoff -a)
	- Slave node: CPU 1 core 1 이상, 스왑메모리 항상 OFF (sudo swapoff -a)


1. 쿠버네티스 설치
	1.1 sudo apt install apt-transport-https
	1.2 curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
	1.3 sudo add-apt-repository "deb https://apt.kubernetes.io/ kubernetes-$(lsb_release -cs) main"
		- 설치중 다음에러 뜨는 경우 아래 직면한 문제 참고
			E: The repository 'https://apt.kubernetes.io kubernetes-bionic Release' does not have a Release file.
	1.4 sudo apt update
	1.5 sudo apt install kubelet kubeadm kubectl kubernetes-cni


2. 쿠버네티스 실행(마스터노드)
	2.0 1번을 참고하여 쿠버네티스 설치
	2.1 sudo swapoff -a
	2.2 systemctl enable docker.service
	2.3 sudo sysctl net.bridge.bridge-nf-call-iptables=1
	2.4 sudo kubeadm init --pod-network-cidr=10.244.0.0/16
		madfalcon@madfalcon:~$ sudo kubeadm init --pod-network-cidr=10.244.0.0/16
			[init] Using Kubernetes version: v1.16.1
			[preflight] Running pre-flight checks
					[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
			[preflight] Pulling images required for setting up a Kubernetes cluster
			[preflight] This might take a minute or two, depending on the speed of your internet connection
			[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
			[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
			[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
			[kubelet-start] Activating the kubelet service
			[certs] Using certificateDir folder "/etc/kubernetes/pki"
			[certs] Generating "ca" certificate and key
			[certs] Generating "apiserver" certificate and key
			[certs] apiserver serving cert is signed for DNS names [madfalcon.masterhome.com kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.211.129]
			[certs] Generating "apiserver-kubelet-client" certificate and key
			[certs] Generating "front-proxy-ca" certificate and key
			[certs] Generating "front-proxy-client" certificate and key
			[certs] Generating "etcd/ca" certificate and key
			[certs] Generating "etcd/server" certificate and key
			[certs] etcd/server serving cert is signed for DNS names [madfalcon.masterhome.com localhost] and IPs [192.168.211.129 127.0.0.1 ::1]
			[certs] Generating "etcd/peer" certificate and key
			[certs] etcd/peer serving cert is signed for DNS names [madfalcon.masterhome.com localhost] and IPs [192.168.211.129 127.0.0.1 ::1]
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
			[apiclient] All control plane components are healthy after 44.504033 seconds
			[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
			[kubelet] Creating a ConfigMap "kubelet-config-1.16" in namespace kube-system with the configuration for the kubelets in the cluster
			[upload-certs] Skipping phase. Please see --upload-certs
			[mark-control-plane] Marking the node madfalcon.masterhome.com as control-plane by adding the label "node-role.kubernetes.io/master=''"
			[mark-control-plane] Marking the node madfalcon.masterhome.com as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
			[bootstrap-token] Using token: l9hji2.dp7ftgq3v1tmv9w8
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

			kubeadm join 192.168.211.129:6443 --token l9hji2.dp7ftgq3v1tmv9w8 \
				--discovery-token-ca-cert-hash sha256:05a82ae48c94803c2309cbae1dee58243b8b0eeb205b2d2d0f9c1a7e8fedd5a5 

	2.5 위 내용에서 제일 마지막 부분에 노드를 추가하는 명령어가 있으므로 미리 복사를 해놓자
		- mkdir -p $HOME/.kube
		- sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
		- sudo chown $(id -u):$(id -g) $HOME/.kube/config
		- cat .kube/config 를 입력하여 다음과 같이 내용을 확인해보자
			madfalcon@madfalcon:~/.kube$ cat config 
				apiVersion: v1
				clusters:
				- cluster:
					certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRFNU1UQXhNekV6TURFMU1Wb1hEVEk1TVRBeE1ERXpNREUxTVZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBS3RCClpnc2NzT2JtOE8yUDRpRUZLRElrMVdIa3BKc1ZzUkxtbUg4NzgyOXVsQk0zNlpkQm9xVlBqRnVOSDQ0elBGVDQKMjVWc2JwVFdRcTJJeFAwVVY5MVVzczZVcldiUnhNYXRRWWlnckkyZVlycHJNdG1ENkFsaXNEaHg1ZmUvcXd1OApLdHlqMDNQTEk5ZlJQd3hCdnlVWWUyWTZNSmNHZm13WXZLZzBwUVFwQkJBVUhiSlpqQ3dLb2ZydDlFNXdTbEp3CjkxZHU2R1NOZVhEQ3hCMStPZEZ3VFVWVHc4ZTNtblRRVDQ5d1BZMWJzVGgydlNMZDBabXo4dWRKSVNnL3B3ZWoKUDFlQjhUT1RHYTR3eXpLMDcvYWZHNzUyQ05aM0ozQ3I2eldqVzJ6cHVrTnlKM1d1ZVBaRTNQcnpHeDY0Z0NINAo5alY0dlVkV1oxa1YraWgzLy84Q0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFFVU50SDR3NW80bEpJa2dZazE1OTN2RUtBREsKZlZZK1RmSjYxWVJrblVaZTd0WkN6Z2RBWDZUYUFTc1gzZFZDTkI3RDdlMWpUeWFaRUxJR1ppYkw1NXZmUE5kQwoyaDB2OHlkNk9XZ2pOQzhXU2lZL2FWNndkeEhpN2c3SWJDakNRZjZ5NmhkNDMyYm92MzhUaGFWOGVzblpzVVhkClRGbVlxNzJySmFYV0hXQmxJR0VySkxLVTdoak5Bdk53Q0hBU2huUlBYZ0dUZ05RYnVFZyt6eEFtazlpZFBlUk8KWGlxWDd4SjA4aWgzSTc3dFM3OHVOdXVpSyt6TmJjSDJYKzdURkxjRTg4R2ZGWFBBWXJjdXY4cUdiSXNZWlFQZQoyUjVLcFY4ak5zYncraEtvN1c1UGwyUDRkcTRycmxYQVdJR3FXYS93Tm52YWtnTnFyaUNld2gzY25qND0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
					server: https://192.168.211.129:6443
				  name: kubernetes
				contexts:
				- context:
					cluster: kubernetes
					user: kubernetes-admin
				  name: kubernetes-admin@kubernetes
				current-context: kubernetes-admin@kubernetes
				kind: Config
				preferences: {}
				users:
				- name: kubernetes-admin
				  user:
					client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM4akNDQWRxZ0F3SUJBZ0lJWHk5ZzBYMTRMOEl3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB4T1RFd01UTXhNekF4TlRGYUZ3MHlNREV3TVRJeE16QXhOVGRhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXBxaFkrWThZTVBCL3pYL1IKc0RkRDhrNDgxRUYxS25WaERaV0UwUVZvRjJEYlFWV0RNOVNDczFhbUh4Z0Y4UExKZzVmWHREbHhNbjdHVERTSgoxRjUraFhGU0xZcW9oVE02bG9lcCtGQ0htcHBJbkY3V25icENqY2NQUEpsRHFTTjYxVmorWWVDeFltUjhJUDVGCkZNeHdkU20rR0E2M0RFZGZzdHlCQ2RYYlA4RDhPQUlZQ3E2MFExL041blVGMjB4RHJYK1d2S1VvWmJRemhaaTcKeERTVVJEd0NwVVhSdjN5OVpWLy9QdDhtdUowSHdiNEdGaFdHZmhDUk9kMUZMRjV1VVNXRC9NVnBYZkVxeTg4KwpWQ1hmMVRXMXY3TTlPaGRnamtERjVYTExkekM4c0ppOEk2SldOZDNQQ1UxN0YrSFo2Q3FobG51Q3N4SGxhLzFxCnRJK2F3d0lEQVFBQm95Y3dKVEFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFIR0FHVkowTWlQRHFHVUNISitVZkdUTDliZnlhSG95eFo4Uwo1aXRuT0ZlRHdvTjRpNnlzMjVTRmV1WkhvQjlqS2pFdzh1a1BBMmM2RjE4UmswTlB2M1ArdHJjZzZPYmtlYjZECmM1NEZzWmR6TDFtdUVyY2lLZ280WU9BNkdJSy9nWUxjTTIrSFR4ZFlyTS9DZUQ0NmdUWVN6RWVFTDJtenA0bzkKay9Yb1o2bWRHUU1yUDhhODF2eE5UVWxva0pqWFdmWGFqSldrY0trczU1VW5LOXFySWg3QndrTStqTFhoSXJCOApEcTZHRGx0K0ZtS0xrdFpSWGhOdFpIYUdKaHdnS0Z3OE1JeUJLeC9STXNYR1JYTnJEL3c0MnB3SUZFL1R0UGhvCmRTb0RJOHlxM1lRQi9FYjU1MHpDdllmVWplVGQ4ZTZUQ2RjY0hiWStMOHRwenZzVEpaST0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
					client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBcHFoWStZOFlNUEIvelgvUnNEZEQ4azQ4MUVGMUtuVmhEWldFMFFWb0YyRGJRVldECk05U0NzMWFtSHhnRjhQTEpnNWZYdERseE1uN0dURFNKMUY1K2hYRlNMWXFvaFRNNmxvZXArRkNIbXBwSW5GN1cKbmJwQ2pjY1BQSmxEcVNONjFWaitZZUN4WW1SOElQNUZGTXh3ZFNtK0dBNjNERWRmc3R5QkNkWGJQOEQ4T0FJWQpDcTYwUTEvTjVuVUYyMHhEclgrV3ZLVW9aYlF6aFppN3hEU1VSRHdDcFVYUnYzeTlaVi8vUHQ4bXVKMEh3YjRHCkZoV0dmaENST2QxRkxGNXVVU1dEL01WcFhmRXF5ODgrVkNYZjFUVzF2N005T2hkZ2prREY1WExMZHpDOHNKaTgKSTZKV05kM1BDVTE3RitIWjZDcWhsbnVDc3hIbGEvMXF0SSthd3dJREFRQUJBb0lCQUdNV0VwaUdDSGNJZHFRQwp5L1ErVjRxWUU3aUxGcE5nNkl4QUNwQ1A2MXlDL0xreWsyaGNnRDBLVm9pRUt2d2dEY25NbkxZTnRReTFyVWFmCnNoYnUrOEJ4S00vazhkOElIMXhpV3A5Rm1lcEVzc2t0NWVJdlhPU1lwcWw3NG11TXFicHhTQVYvcDFkOXNRT1kKeWcvY29UdzN6a3JWYk9YREJkWDlIa2R2dkFTQnpoOEZDV2RWWHFmRU5ubG0vQWYwaTNxUW9sTXV4Nks5Y3NDMQpzaWJWRkNwU2ZqSWM5OU9OcnRWS0tNQlhlTG9uN1J4ampCV3padHRQaGhXdVdQZExrUmR0dTk0TWRjVnpKZkI3Cnd0S1hRbWRrRDNGMys3Zzh2WC9lb2xUYlV6bWNPQkk4ZDNqRkE3Z2NwSmtsd1lMSkc5WTBRUnFxSWZuUlh2LzgKWXJvdXpzRUNnWUVBd2p6WTZUK3cvZ3BFU1F5ZFlVR2J2TFlyc1BTajdtQ2ZHUTJrQWg3RmRHbWxuRHl1RUdjegpUcThFNSswTkNXSDVnS1NPUndhaGFlb28vREpHZkFnVG92MkhnclB1bStZbVlxRXZqY3NTNyt6RmtDbGZKSS9oCjNBYXBBVUV2RklsSExYb2FJWHpiNmdTYUNGOWJTZnFQcS9vZ3ZwR3UvMmlCeGVJdTdLbDY1K0VDZ1lFQTI2WnoKN1VmeWhlN0UwZFdLNVY0bnJ6YVZPdm5ENlBnTUNKY3YwUkNXMkw1L0ZoWWErbTJjRmtzZ2ZqV0Z0aTNETk9HLwovbjM0aThiZzlQMWMvZU5ZSDBxTzduYUJSb3BjQmJYSTd4ekVXbm5EYmVGSVpvZnpOd29CcGg3c0RQQUlFcFZsCkIyM3I5QzFzN1FPcDVDbkVZQUxKUEdraVpFU2ZGWk5xZmdiQ3h5TUNnWUE4RWUxVFZXczdaWmx3cmdJT0RlaEkKR0Y2eXZ6WGpodVl0TFZiSGdSUzN4K1I3eVJoYjRrNnZ5dGpOa1RZeTdLWG83dnRCWS8rUGJlZDI5MlZzL21KMApTY3dhMCtLN1BCWXE4b1p1WjV0WHIvWDVlNUg5RUxKZEJZSkc3UTNPWUJZdkxrL2VnMnJQbU5TNk9pTlZZYlFGCkQ3b0l0YTFWTjlES2pnVE5GQ3o2Z1FLQmdHVjhHc1BmSWliUGt4Q0FZWlJvVkYyWUVvc1ZLM2RRWS83MEc1dTAKMW4xK1JxbWx2UUZIODM0NVorSG9TTWRMalkyNVlFUHRZQkQwNnF0SEJOZ3BXbVhheFA5WXNaSXVDeVo0UDBaNwpQQjJ4ZEtJb0hKT0M3TlRaUXJuR1A3b2FqU0JJOWt6Z2RNeDAwSWNSMGtVaEp5SlZKelZLUGlHbHN2cjlDWThCCkFLMlJBb0dBQVJwQ0d3YVNYc1hieU1qNmN3VmtZUUtaYU1vbTVnQTJiV2ZBbXo2MWNEa25xejFyMStJYkpsRGkKaUV5TnMzV3lwL01mTnlsdGIvUnRlbUpTb0lSYW5GTXJZM0pyTEM2NElsOHlFZENWcFBYU3ZkaUJkMUhpRDNsbApUTDVCSUFTRzRoMDZ4U2VSV2htR2VyaElydVBQM2sraUxKMFdwT2R4NXNTQm9mMWYrdm89Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==

	2.6 Pod network 추가
		- Pod 간 통신을 위해 Pod network를 추가해주어야 함
		- sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
			madfalcon@madfalcon:~/.kube$ sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
				podsecuritypolicy.policy/psp.flannel.unprivileged created
				clusterrole.rbac.authorization.k8s.io/flannel created
				clusterrolebinding.rbac.authorization.k8s.io/flannel created
				serviceaccount/flannel created
				configmap/kube-flannel-cfg created
				daemonset.apps/kube-flannel-ds-amd64 created
				daemonset.apps/kube-flannel-ds-arm64 created
				daemonset.apps/kube-flannel-ds-arm created
				daemonset.apps/kube-flannel-ds-ppc64le created
				daemonset.apps/kube-flannel-ds-s390x created
				
		- "kubectl get nodes" 명령어를 입력하여 STATUS 상태가 deploy 되었는지 확인
			madfalcon@madfalcon:~/.kube$ kubectl get nodes
				NAME                       STATUS   ROLES    AGE     VERSION
				madfalcon.masterhome.com   Ready    master   7m13s   v1.16.1
		

3. 쿠버네티스 실행(워커노드)
	3.0 1번을 참고하여 쿠버네티스 설치
	3.1 sudo swapoff -a
	3.2 systemctl enable docker.service
	3.3 위 master에서 노드추가 부분 코드 입력
		- sudo kubeadm join 192.168.211.129:6443 --token l9hji2.dp7ftgq3v1tmv9w8 --discovery-token-ca-cert-hash sha256:05a82ae48c94803c2309cbae1dee58243b8b0eeb205b2d2d0f9c1a7e8fedd5a5


4. 쿠버네티스 Pod Network에 연결된 노드 상태확인
	- "kubectl get nodes"를 통해 join 되어있는 노드들의 상태 확인
		madfalcon@madfalcon:~$ kubectl get nodes
			NAME                       STATUS   ROLES    AGE    VERSION
			madfalcon.masterhome.com   Ready    master   30m    v1.16.1
			madfalcon.slavehome1.com   Ready    <none>   2m5s   v1.16.1





직면한 문제
---------------------------------------------------------------------------------------------------------------
아래 명령어 실행후 에러 발생 
	sudo add-apt-repository "deb https://apt.kubernetes.io/ kubernetes-$(lsb_release -cs) main" 
	- 우분투 18.04 버전에서 kube ~ bionic 관련 apt 업데이트가 지원되지 않는 에러내용:
		E : The repository 'https://apt.kubernetes.io kubernetes-bionic Release' does not have a Release file
	- 해결방법: 
		1. sudo vi /etc/apt/sources.list 접속 후 kubernetes-bionic 관련내용을 삭제 
			deb https://apt.kubernetes.io/ kubernetes-bionic main
			# deb-src https://apt.kubernetes.io/ kubernetes-bionic main
		2. 하기 내용을 수동으로 추가 
			- sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main" 
			- sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-yakkety main"
---------------------------------------------------------------------------------------------------------------




참고자료
---------------------------------------------------------------------------------------------------------------
1. 쿠버네티스 개념1: https://kubernetes.io/ko/docs/concepts/overview/what-is-kubernetes/
2. 쿠버네티스 개념2: https://subicura.com/2019/05/19/kubernetes-basic-1.html
2. 쿠버네티스 설치: https://hiseon.me/linux/ubuntu/ubuntu-kubernetes-install/
3. 쿠버네티스 문제: https://stackoverflow.com/questions/53068337/unable-to-add-kubernetes-bionic-main-ubuntu-18-04-to-apt-repository
---------------------------------------------------------------------------------------------------------------
