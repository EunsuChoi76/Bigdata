

=======================================================================

참고(출처):
https://bcho.tistory.com/1266?category=731548

2. Rolling Update 배포 
1) 테스트애플리케이션 2개 버전 구성(server.js)
 - 애플리케이션코드 구성(server.js)
 - 도커 빌드
 - 애플리케이션테스트(도커 run 및 호출하여 결과확인)
 - 도커 push
2) Deployment생성 yaml파일 구성(hello-deployment.yaml)
 - selector matchLabels 사용
3) Dockerfile 구성
4) Deployment yaml파일 반영
5) Service yaml 파일 구성 및 적용(hello-deployment-service.yaml)
 - 서비스 ip로 호출하여 결과 확인
6) 새로운 이미지로 변경후 서비스 호출하여 반영된 결과 확인


테스트시 고려사항
- 새로운 버전반영 방법 고려필요
  - kubectl edit 
   _. kubectl edit deployment [deployment 명]
   _. 자세한 설정 가능
  - kubectl replace
   _. kubectl replace -f [yaml 파일명]
   _. 설정파일 새로만들어서 파일단위로 업데이트 가능
  - kubectl patch
   _. kubectl patch [리소스 종류] [리소스명] --patch ‘[YAML이나 JSON 포맷으로 변경하고자 하는 설정]’
   _. 예) kubectl patch deployment hello-deployment --patch 'spec:\n template:\n  spec:\n   containers:\n   - name: hello-deployment\n     image: gcr.io/terrycho-sandbox/deployment:v2’
- 롤백방안 고려
  - undo/pause/resume 
 


### Deployment

롤링업데이트
## 애플리케이션 준비(2가지 버전으로 준비)
server.js
var os = require('os');

var http = require('http');
var handleRequest = function(request, response) {
  response.writeHead(200);
  response.end("Hello World! I'm Server version 1 .  "+os.hostname() +" \n");

  //log
  console.log("["+
		Date(Date.now()).toLocaleString()+
		"] "+os.hostname());
}
var www = http.createServer(handleRequest);
www.listen(7999);


## Deployement yaml파일(hello-deployment.yaml)
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: hello-deployment
spec:
  replicas: 3
  minReadySeconds: 5
  selector:
    matchLabels:
      app: hello-deployment
  template:
    metadata:
      name: hello-deployment-pod
      labels:
        app: hello-deployment
    spec:
      containers:
      - name: hello-deployment
        image: localhost:5000/ces/deployment:v1
        imagePullPolicy: Always
        ports:
        - containerPort: 7999

## Docker File 구성(2가지 버전으로 도커 빌드한다) 
FROM node:carbon
EXPOSE 7999
COPY server.js .
CMD node server.js > log.out

## 도커빌드 (server.js의 logging부분을 변경하여 2가지 버전으로 도커빌드함)
docker build -t localhost:5000/ces/deployment:v1 .
docker build -t localhost:5000/ces/deployment:v2 .

결과:
ubuntu@ip-172-26-5-137:~/ces/rollingUpdate$ docker images | grep deployment
localhost:5000/ces/deployment   v2                  de60b7548027        19 seconds ago       901MB
localhost:5000/ces/deployment   v1                  a5e804cc983e        About a minute ago   901MB

## 도커 push (private registry가 run 되어있어야함)
docker push localhost:5000/ces/deployment:v1
docker push localhost:5000/ces/deployment:v2
 

## Deployment yaml파일 반영
kubectl create -f hello-deployment.yaml 

결과:
ubuntu@ip-172-26-5-137:~/ces/rollingUpdate$ k get pod | grep deployment
hello-deployment-6d8d9d96cc-2bztp   1/1     Running   0          8m3s
hello-deployment-6d8d9d96cc-jfsvp   1/1     Running   0          8m3s
hello-deployment-6d8d9d96cc-fgb7m   1/1     Running   0          8m3



## Service yaml구성 및 반영(hello-deployment-service.yaml)
apiVersion: v1
kind: Service
metadata:
  name: hello-deployment-svc
spec:
  selector:
    app: hello-deployment
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 7999
  type: LoadBalancer


kubectl create -f hello-deployment-service.yaml

결과: 
ubuntu@ip-172-26-5-137:~/ces/rollingUpdate$ k get svc
NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes             ClusterIP      10.43.0.1      <none>        443/TCP        40d
hello-deployment-svc   LoadBalancer   10.43.44.245   <pending>     80:31320/TCP   24s

테스트: 
ubuntu@ip-172-26-5-137:~/ces/rollingUpdate$ curl 10.43.44.245
Hello World! I'm Server version 1 .  hello-deployment-6d8d9d96cc-2bztp




## 새로운 이미지로 변경
kubectl set image deployment hello-deployment hello-deployment=localhost:5000/ces/deployment:v2

결과:
ubuntu@ip-172-26-5-137:~/ces/rollingUpdate$ kubectl set image deployment hello-deployment hello-deployment=localhost:5000/ces/deployment:v2
deployment.apps/hello-deployment image updated
ubuntu@ip-172-26-5-137:~/ces/rollingUpdate$ k get pod | grep deployment
hello-deployment-6d8d9d96cc-fgb7m   1/1     Running             0          23m
svclb-hello-deployment-svc-tm7tl    0/1     Pending             0          10m
hello-deployment-6fbff69dcb-t6js9   1/1     Running             0          15s
hello-deployment-6d8d9d96cc-jfsvp   1/1     Terminating         0          23m
hello-deployment-6fbff69dcb-kjgzt   1/1     Running             0          7s
hello-deployment-6d8d9d96cc-2bztp   1/1     Terminating         0          23m
hello-deployment-6fbff69dcb-bm9rq   0/1     ContainerCreating   0          0s
ubuntu@ip-172-26-5-137:~/ces/rollingUpdate$ k get pod | grep deployment
svclb-hello-deployment-svc-tm7tl    0/1     Pending   0          11m
hello-deployment-6fbff69dcb-t6js9   1/1     Running   0          58s
hello-deployment-6fbff69dcb-kjgzt   1/1     Running   0          50s
hello-deployment-6fbff69dcb-bm9rq   1/1     Running   0          43s
ubuntu@ip-172-26-5-137:~/ces/rollingUpdate$ curl 10.43.44.245
Hello World! I'm Server version 2 .  hello-deployment-6fbff69dcb-bm9rq

=> 새로운 버전의 이미지의 결과가 출력됨

ubuntu@ip-172-26-5-137:~/ces/rollingUpdate$ k describe deploy hello-deployment
Name:                   hello-deployment
Namespace:              default
CreationTimestamp:      Sun, 22 Dec 2019 21:16:48 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=hello-deployment
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        5
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=hello-deployment
  Containers:
   hello-deployment:
    Image:        localhost:5000/ces/deployment:v2
    Port:         7999/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   hello-deployment-6fbff69dcb (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  44m   deployment-controller  Scaled up replica set hello-deployment-6d8d9d96cc to 3
  Normal  ScalingReplicaSet  21m   deployment-controller  Scaled up replica set hello-deployment-6fbff69dcb to 1
  Normal  ScalingReplicaSet  21m   deployment-controller  Scaled down replica set hello-deployment-6d8d9d96cc to 2
  Normal  ScalingReplicaSet  21m   deployment-controller  Scaled up replica set hello-deployment-6fbff69dcb to 2
  Normal  ScalingReplicaSet  21m   deployment-controller  Scaled down replica set hello-deployment-6d8d9d96cc to 1
  Normal  ScalingReplicaSet  21m   deployment-controller  Scaled up replica set hello-deployment-6fbff69dcb to 3
  Normal  ScalingReplicaSet  20m   deployment-controller  Scaled down replica set hello-deployment-6d8d9d96cc to 0

=> Deployment의 경우 내부적으로 Replica set으로 구성됨

## 롤백
리비전 확인 명령어
 kubectl rollout history deployment [Deployment 이름]

ubuntu@ip-172-26-5-137:~/ces/rollingUpdate$ kubectl rollout history deployment hello-deployment
deployment.apps/hello-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

롤백명령어
 kubectl rollout undo deployment [ deployment 명 ] --to-revision=[롤백할 버전명]

 kubectl rollout undo deployment hello-deployment --to-revision=1

결과:
ubuntu@ip-172-26-5-137:~/ces/rollingUpdate$ kubectl rollout undo deployment hello-deployment --to-revision=1
deployment.apps/hello-deployment rolled back
ubuntu@ip-172-26-5-137:~/ces/rollingUpdate$ curl 10.43.44.245
Hello World! I'm Server version 1 .  hello-deployment-6d8d9d96cc-pglrh

=> revision 2 -> 1 로 롤백후 서비스호출시 애플리케이션1의 응답



=======================================================================

참고(출처):
https://jeongchul.tistory.com/576

3. Blue/Green 
blue/green의 개별 yaml 파일 필요
실습 결과 차후 업데이트 예정



4. Canary