

===================================
1. Replication Controller를 사용한 기본 배포

1) node js로 구성된 애플리케이션 구성 
 - nodejs 설치 및 애플리케이션 코드

2) 이미지생성을 위한 Dockerfile 구성
3) Dockerfile registry 확인
4) 기본 배포 yaml 파일 구성(hello-node-rc.yaml) 및 yaml파일 적용   

테스트시 고려사항
- 애플리케이션 컨테이너의 로깅 및 테스트
- private docker registry 구성
- 서비스port와 containerport
- STG에서 테스트시 docker pull 권한 확인필요(개인환경에서 진행)

기타 배포 고려사항
- 배포정책: taint-노드에 배포되지 않도록  affinity-특정 노드에 pod 배포
- KSONNET: json기반 배포OSS template 엔진
- Skaffold: 코드컴파일+도커패키징+태깅+yaml변경+테스트 수행 배포자동화툴 
- Helm: chart 배포 및 repository 구성 고려


## nodejs 설치

sudo apt-get update
sudo apt-get install build-essential libssl-dev

curl -sL https://raw.githubusercontent.com/creationix/nvm/v0.31.0/install.sh -o install_nvm.sh
bash install_nvm.sh
source ~/.profile
nvm ls-remote
=> 13.5.0 설치
nvm install 13.5.0
nvm use 13.5.0
node -v

## server.js
var os = require('os');
 
var http = require('http');
var handleRequest = function(request, response) {
  response.writeHead(200);
  response.end("Hello World! I'm "+os.hostname());

  //log1.
  console.log("["+
                Date(Date.now()).toLocaleString()+
                "] "+os.hostname());
}

var www = http.createServer(handleRequest);
www.listen(7999);


## vi Dockerfile (Docker File)
FROM node:carbon
EXPOSE 7999
COPY server.js .
CMD node server.js > log.out


## 도커 private registry 설정
# registry 이미지를 가져오기
$ docker pull registry

# registry를 실행하기
$ docker run -dit --name docker-registry -p 5000:5000 registry

## 도커빌드
docker build -t localhost:5000/ces/hello-node:v1 .

## 도커 구동
docker images
docker run -d -p 7999:7999 localhost:5000/ces/hello-node:v1

## 도커접속
docker exec -it [컨테이너id] /bin/bash

## 도커로그인
sudo docker login
UserName: eunsuchoi76
Password: saint0724!

saint0724!

## 도커 push
docker push localhost:5000/ces/hello-node:v1

-----------------
#### 기본배포해보기(ReplicationController)

## rc생성(hello-node-rc.yaml)
apiVersion:v1
kind: ReplicationController
metadata:
  nmae: hello-node-rc
spec:
  replicas: 3
  selector:
    app: hello-node
  template:
    metadata:
      name: hello-node-pod
      labels:
        app: hello-node
    spec:
      containers:
      - name: hello-node
        image: localhost:5000/ces/hello-node:v1
        #imagePullPolicy: Always
        ports:
        - containerPort: 7999  

=> port는 알아서 적당히 정할것

## rc 적용
kubectl create -f hello-node-rc.yaml 

결과:
ubuntu@ip-XXX-XX-X-XXX:~/ces$ k get pod | grep hello
hello-node-rc-jclx4       1/1     Running   0          63s
hello-node-rc-z9j2w       1/1     Running   0          63s
hello-node-rc-zd254       1/1     Running   0          63s


## Service 등록
hello-node-svc.yaml

apiVersion: v1
kind: Service
metadata:
  name: hello-node-svc
spec:
  selector:
    app: hello-node
  ports:
    - port: 80
      protocol: TCP
      targetPort: 7999
  type: LoadBalancer

## Service 적용
kubectl create -f hello-node-svc.yaml

결과:
ubuntu@ip-XXX-XX-X-XXX:~/ces$ k get svc
NAME             TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP      10.4X.0.1       <none>        443/TCP        39d
hello-node-svc   LoadBalancer   10.4X.1XX.199   <pending>     80:31337/TCP   18s


ubuntu@ip-XXX-XX-X-XXX:~/ces$ curl 10.4X.1XX.199:80
Hello World! I'm hello-node-rc-jclx4

  