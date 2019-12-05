# Dockerhub 를 이용한 Private registry 이용방법

### 1. Dockerhub(https://hub.docker.com) 에서 회원가입을 진행

### 2. CLI에서 'docker login' 입력 후 회원가입한 ID와 비밀번호 입력
```bash
- EX) ID가 chadool116일 경우
	madfalcon@test_madfalcon_server:~$ docker login 
		Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
		
		Username: chadool116 <- dockerhub에서 가입한 아이디
		Password:            <- 비밀번호, 입력해도 화면에 보이지 않는다.
		
		WARNING! Your password will be stored unencrypted in /home/madfalcon/.docker/config.json.
		Configure a credential helper to remove this warning. See
		https://docs.docker.com/engine/reference/commandline/login/#credentials-store

		Login Succeeded
	madfalcon@test_madfalcon_server:~$
```

### 3. tag 명령어를 이용하여 생성한 이미지를 다음과 같이 복사
 - 'madfalcon/node-web-app'를 다음의 명령어로 태깅할 수 있음
 - "docker tag [기존이미지] [회원가입ID/생성한이미지파일]"
 - 예시) "docker tag madfalcon/node-web-app:latest chadool116/node-web-app"
```bash
	madfalcon@test_madfalcon_server:~$ docker images
		REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
		madfalcon/node-web-app   latest              77f343714c04        8 days ago          109MB
		node                     alpine3.10          fac3d6a8e034        9 days ago          106MB
		hello-world              latest              fce289e99eb9        11 months ago       1.84kB
		
	madfalcon@test_madfalcon_server:~$ docker tag madfalcon/node-web-app:latest chadool116/node-web-app

	madfalcon@test_madfalcon_server:~$ docker images
		REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
		madfalcon/node-web-app    latest              77f343714c04        8 days ago          109MB
		chadool116/node-web-app   latest              77f343714c04        8 days ago          109MB
		node                      alpine3.10          fac3d6a8e034        9 days ago          106MB
		hello-world               latest              fce289e99eb9        11 months ago       1.84kB
```

### 4. push 명령어를 이용해 Dockerhub의 private registry에 업로드
 - "docker push [회원가입ID/생성한이미지파일]"
 - 예시) "docker push chadool116/node-web-app"
 - push 후 dockerhub에서 이미지가 업로드 되었음을 확인할 수 있음
```bash
	madfalcon@test_madfalcon_server:~$ docker push chadool116/node-web-app
		The push refers to repository [docker.io/chadool116/node-web-app]
		83a088502ffd: Pushed 
		4c66dc201393: Pushed 
		f845fd53498a: Pushed 
		f7eb0a63edef: Pushed 
		20d6ac69de87: Mounted from library/node 
		15b7d94033f5: Mounted from library/node 
		4322be3f308a: Mounted from library/node 
		77cae8ab23bf: Mounted from library/node 
		latest: digest: sha256:c591c96e1864d2685a6690daa8e053012c3958197ac70cc4abdbd1637447aa18 size: 1990
	madfalcon@test_madfalcon_server:~$
```

### 5. pull 명령어를 이용해 Dockerhub에 업로드한 이미지 가져오기
 - "docker pull [업로드한 이미지]
 - 예시) "docker pull chadool116/node-web-app"
```bash
	madfalcon@test_madfalcon_server:~$ docker images
		REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
		madfalcon/node-web-app   latest              77f343714c04        8 days ago          109MB
		node                     alpine3.10          fac3d6a8e034        9 days ago          106MB
		hello-world              latest              fce289e99eb9        11 months ago       1.84kB

	madfalcon@test_madfalcon_server:~$ docker pull chadool116/node-web-app
		Using default tag: latest
		latest: Pulling from chadool116/node-web-app
		Digest: sha256:c591c96e1864d2685a6690daa8e053012c3958197ac70cc4abdbd1637447aa18
		Status: Downloaded newer image for chadool116/node-web-app:latest
		docker.io/chadool116/node-web-app:latest

	madfalcon@test_madfalcon_server:~$ docker images
		REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
		chadool116/node-web-app   latest              77f343714c04        8 days ago          109MB
		madfalcon/node-web-app    latest              77f343714c04        8 days ago          109MB
		node                      alpine3.10          fac3d6a8e034        9 days ago          106MB
		hello-world               latest              fce289e99eb9        11 months ago       1.84kB
```
