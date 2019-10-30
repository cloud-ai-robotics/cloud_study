# nginx-ingress controller 설치하기

 * RPi3 는 arm64 이기 때문에, 아래와 같이 ingress controller 의 이미지를 arm64 로 설정해준다.
   - 다만, worker node 가 rpi2 인 경우, revision 이 1.2 미만이면 (또는 armv8 미만 ex. armv7l 이면 32 bits 임) arm 버전을 설치해야 한다.

   - 64 bits 를 지원하는 cpu 를 가진 rpi 인 경우
```
$ curl -sSL https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml | sed "s/\/nginx-ingress-controller:/\/nginx-ingress-controller-arm64:/g" | kubectl apply -f -
```

   - 32 bits 를 지원하는 cpu 를 가진 rpi 인 경우
```
$ curl -sSL https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml | sed "s/\/nginx-ingress-controller:/\/nginx-ingress-controller-arm:/g" | kubectl apply -f -
```

## TLS 를 Enable 하기 위해, self-signed certificate 생성

 * Reference: (TLS/HTTPS)[https://kubernetes.github.io/ingress-nginx/user-guide/tls]

```
$ openssl req -x509 -nodes -days 9999 -newkey rsa:2048 -keyout cluster.key -out cluster.cert -subj "/CN=cluster.nicesj.com/O=cluster.nicesj.com"
$ kubectl create secret tls cluster --key cluster.key --cert cluster.cert 
```
