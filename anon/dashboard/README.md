# kubernetes dashboard


## 설치
 * 최신 버전: (Dashboard Release)[https://github.com/kubernetes/dashboard/releases]

```
$  curl -sSL https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta5/aio/deploy/recommended.yaml | sed "s/amd64/arm/g" | kubectl apply -f -
```

## 로그인

 * 토큰 찾기
```
$ kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```
