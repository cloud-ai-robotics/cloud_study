Docker와 node.js를 사용하여 웹 서버 배포해보기


참고주소: https://riptutorial.com/ko/docker/example/27294/%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EC%97%90%EC%84%9C-basic-node-js-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%EC%8B%A4%ED%96%89%ED%95%98%EA%B8%B0


1. 웹 서버 패키지(package.json, server.js) 및 Dockerfile 파일 생성 및 빌드

	1.1 자신만의 개인 디렉토리 생성(예시: mkdir /var/tmp/docker_node_web)
	
	
	1.2 웹 서버 생성을 위한 패키지 파일과 서버파일 생성
	 - node.js는 웹 서버 구축 시 응용프로그램과의 종속성을 설명하는 패키지(package.json) 파일과 웹 응용프로그램을 정의하는 서버(server.js) 파일을 만들어야 함


		 sudo vi /var/tmp/docker_node_web/package.json
		
			 {
			   "name": "madfalcon_docker_web_app",
			   "version": "1.0.0",
			   "description": "Node.js on Docker",
			   "author": "madfalcon <kmyong92@gmail.com>",
			   "main": "server.js",
			   "scripts": {
				 "start": "node server.js"
			   },
			   "dependencies": {
				 "express": "^4.13.3"
			   }
			 }
		
		


		sudo vi /var/tmp/docker_node_web/server.js
		
			 var express = require('express');
			 var PORT = 8080;
			 var app = express();
			 app.get('/', function (req, res) {
			   res.send('Hello world\n');
			 });

			 app.listen(PORT);
			 console.log('Running on http://localhost:' + PORT);
		
		
		
		
	1.3	touch 명령어를 통해 Dockerfile 생성(반드시 똑같이 생성할 것, 만약 이름이 'Dockerfile'이 아니라면 -f 옵션을 통해 해당파일이름을 지정해주어야 함)
		
	- vi 편집기를 통해 다음과 같이 넣어준다.
		
		sudo vi /var/tmp/docker_node_web/Dockerfile
		
			---------------------------------------------------------

			FROM node:alpine3.10
			RUN mkdir -p /usr/src/madfalcon_first_app
			WORKDIR /usr/src/madfalcon_first_app
			COPY package.json /usr/src/madfalcon_first_app
			RUN npm install
			COPY . /usr/src/madfalcon_first_app
			EXPOSE 8080
			CMD [ "npm", "start" ]

			---------------------------------------------------------
			TIP: 아래는 Dockerfile 생성 시 옵션정보
				FROM	베이스 이미지 지정	
				ADD	파일 및 디렉터리 추가
				MAINTAINER	Dockerfile 작성자 저장	
				COPY	파일 복사
				RUN	커맨드 실행	
				VOLUME	볼륨 마운트
				CMD	데몬 실행	
				ENTRYPOINT	데몬 실행
				LABEL	라벨 설정	
				USER	사용자 설정
				EXPOSE	개방할 포트 export	
				WORKDIR	작업 디렉터리 지정
				ENV	환경변수 설정	
				ONBUILD	build 완료 후 실행될 명령어



	1.4 이미지 build
	
		- sudo docker build -t [생성할 이미지명]:[버전명] [Dockerfile을 생성한 폴더 경로]
		TIP: 이미지명은 무조건 소문자로 지정한다(버전생략시 latest로 생성)
		ex) sudo docker build -t madfalcon/node-web-app /var/tmp/docker_node_web/
			madfalcon@test_madfalcon_server:/var/tmp/docker_node_web$ docker build -t madfalcon/node-web-app /var/tmp/docker_node_web/
				Sending build context to Docker daemon  4.096kB
				Step 1/8 : FROM node:alpine3.10
				alpine3.10: Pulling from library/node
				89d9c30c1d48: Pull complete 
				5320ee7fe9ff: Pull complete 
				0a42696890fc: Pull complete 
				12c581b27455: Pull complete 
				Digest: sha256:bdf054f006078036f72de45553f3b11176c1c00d5451d8fc2af206636eb54d70
				Status: Downloaded newer image for node:alpine3.10
				 ---> fac3d6a8e034
				Step 2/8 : RUN mkdir -p /usr/src/my_first_app
				 ---> Running in 0914f8d2387b
				Removing intermediate container 0914f8d2387b
				 ---> 4d4b766482cc
				Step 3/8 : WORKDIR /usr/src/my_first_app
				 ---> Running in ea8e9a2efdab
				Removing intermediate container ea8e9a2efdab
				 ---> c9f5beffc270
				Step 4/8 : COPY package.json /usr/src/my_first_app
				 ---> 6db62d9467ea
				Step 5/8 : RUN npm install
				 ---> Running in 371227ffdc4f
				npm notice created a lockfile as package-lock.json. You should commit this file.
				npm WARN madfalcon_docker_web_app@1.0.0 No repository field.
				npm WARN madfalcon_docker_web_app@1.0.0 No license field.

				added 50 packages from 37 contributors and audited 126 packages in 3.051s
				found 0 vulnerabilities

				Removing intermediate container 371227ffdc4f
				 ---> 5aa55d423d6f
				Step 6/8 : COPY . /usr/src/my_first_app
				 ---> 60e7629ab6a6
				Step 7/8 : EXPOSE 8080
				 ---> Running in 1db73cb2c47e
				Removing intermediate container 1db73cb2c47e
				 ---> 5492498f99a3
				Step 8/8 : CMD [ "npm", "start" ]
				 ---> Running in 0ac469592957
				Removing intermediate container 0ac469592957
				 ---> 77f343714c04
				Successfully built 77f343714c04
				Successfully tagged madfalcon/node-web-app:latest
			
			
			
	1.5 빌드 후 이미지 리스트 확인
	
		- sudo docker images
		
			madfalcon@test_madfalcon_server:/var/tmp/docker_node_web$ docker images
				REPOSITORY               TAG                 IMAGE ID            CREATED              SIZE
				madfalcon/node-web-app   latest              77f343714c04        About a minute ago   109MB
				node                     alpine3.10          fac3d6a8e034        12 hours ago         106MB
				hello-world              latest              fce289e99eb9        10 months ago        1.84kB
			



2. build한 image 실행 및 접속 테스트

	2.1 image 실행
	
		- sudo docker run -p 35000:8080 -d madfalcon/node-web-app
		
			madfalcon@test_madfalcon_server:~$ sudo docker run -p 35000:8080 -d madfalcon/node-web-app
				95c5fc4d5ed5161035d7b491421ae2a0aa689af3cd66eec73eb8875b61f206ed
			
			
			
	2.2 웹 서버 실행상태 확인
	
		- sudo docker ps
		
			madfalcon@test_madfalcon_server:~$ docker ps
				CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                     NAMES
				95c5fc4d5ed5        madfalcon/node-web-app   "docker-entrypoint.s   21 seconds ago      Up 19 seconds       0.0.0.0:35000->8080/tcp   awesome_bose
		
		
		
	2.2 로컬 접속 테스트
	
		- curl 명령어를 통해 35000 포트로 웹 서버 호출이 가능하다.
		
			madfalcon@test_madfalcon_server:~$ curl -i localhost:35000
				HTTP/1.1 200 OK
				X-Powered-By: Express
				Content-Type: text/html; charset=utf-8
				Content-Length: 12
				ETag: W/"c-M6tWOb/Y57lesdjQuHeB1P/qTV0"
				Date: Tue, 26 Nov 2019 13:51:58 GMT
				Connection: keep-alive
				
				Hello world
