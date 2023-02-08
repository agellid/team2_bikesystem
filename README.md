## 따릉이 예약/사용 시스템

## 서비스 시나리오

...

1. 관리자는 구매한 따릉이를 시스템에 입력한다.
2. 사용자는 따릉이 예약시스템을 사용하기 위해 구매시스템에서 포인트를 구매한다
3. 사용자는 따릉이 예약시스템에서 포인트를 이용해 예약을 한다
4. 사용자는 따릉이 예약시스템에서 취소를 할 수 있다

...


## Event Storming 결과
 . MSAEz 로 모델링한 이벤트스토밍 결과:
  https://labs.msaez.io/#/storming/70d9e09c3c0a88189e6c4b72c6e2f61f

 
 . Event 도출
 
![image](https://user-images.githubusercontent.com/2344829/217235648-3a1157ed-2092-47e4-9f6a-9b391d7e2743.png)

  따릉이의 구매
  포인트의 구매
  예약 - 포인트 사용
  취소 - 포인트 복귀
  반납
  의 이벤트 


 . 완성된 모델
 
 ![image](https://user-images.githubusercontent.com/2344829/217236216-a38c0155-8026-4a97-80a3-d5ea708bb64e.png)



 . 완성된 모델에서 기능 검증
 
  1) 따릉이의 등록
  2) 포인트 결재 후 따릉이 예약

![image](https://user-images.githubusercontent.com/2344829/217236947-a2da20c1-0906-42d0-b874-e100ac646940.png)

  
## 구현

  . pub/sub를 통해 전달되는 내용을 policy에서 구현
  
  
  . 적용 후 테스트 (container 환경)
  
    각 마이크로 서비스의 기능에 대한 검증 및
    pub/sub 를 통한 서비스 확인
    gateway를 통한 서비스 확인
    CQRS 확인 을 수행 
  
    각 서비스의 Container IP와 LoadBalancer 정보는 다음과 같음
    
    
gitpod /workspace/team2_bikesystem/reservation/kubernetes (main) $ kubectl get all
NAME                               READY   STATUS    RESTARTS   AGE
pod/approval-58589868c-6dckn       2/2     Running   0          28m
pod/bikemgmt-cdcb9c5f-kpm4k        2/2     Running   0          30m
pod/dashboard-68847d6b7-nw67h      2/2     Running   0          26m
pod/gateway-6fd6dfd4f9-kptk9       2/2     Running   0          24m
pod/my-kafka-0                     1/1     Running   1          13h
pod/my-kafka-zookeeper-0           1/1     Running   0          13h
pod/point-b656f7984-kj7bp          2/2     Running   0          22m
pod/reservation-658cc95d7b-7dc7t   2/2     Running   0          21m
pod/siege                          1/1     Running   0          135m

NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)                      AGE
service/approval                      ClusterIP      10.100.181.240   <none>                                                                   8080/TCP                     28m
service/bikemgmt                      ClusterIP      10.100.146.187   <none>                                                                   8080/TCP                     30m
service/dashboard                     ClusterIP      10.100.244.203   <none>                                                                   8080/TCP                     26m
service/gateway                       LoadBalancer   10.100.18.170    ad33daaed85a44ed7a1817096c2b22e1-398369829.eu-west-2.elb.amazonaws.com   8080:31423/TCP               24m
service/my-kafka                      ClusterIP      10.100.62.5      <none>                                                                   9092/TCP                     13h
service/my-kafka-headless             ClusterIP      None             <none>                                                                   9092/TCP,9093/TCP            13h
service/my-kafka-zookeeper            ClusterIP      10.100.213.189   <none>                                                                   2181/TCP,2888/TCP,3888/TCP   13h
service/my-kafka-zookeeper-headless   ClusterIP      None             <none>                                                                   2181/TCP,2888/TCP,3888/TCP   13h
service/point                         ClusterIP      10.100.18.149    <none>                                                                   8080/TCP                     22m
service/reservation                   ClusterIP      10.100.196.39    <none>                                                                   8080/TCP                     21m

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/approval      1/1     1            1           28m
deployment.apps/bikemgmt      1/1     1            1           30m
deployment.apps/dashboard     1/1     1            1           26m
deployment.apps/gateway       1/1     1            1           24m
deployment.apps/point         1/1     1            1           22m
deployment.apps/reservation   1/1     1            1           21m

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/approval-58589868c       1         1         1       28m
replicaset.apps/bikemgmt-cdcb9c5f        1         1         1       30m
replicaset.apps/dashboard-68847d6b7      1         1         1       26m
replicaset.apps/gateway-6fd6dfd4f9       1         1         1       24m
replicaset.apps/point-b656f7984          1         1         1       22m
replicaset.apps/reservation-658cc95d7b   1         1         1       21m

NAME                                  READY   AGE
statefulset.apps/my-kafka             1/1     13h
statefulset.apps/my-kafka-zookeeper   1/1     13h

NAME                                           REFERENCE             TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/bikemgmt   Deployment/bikemgmt   <unknown>/50%   1         3         1          2m55s
gitpod /workspace/team2_bikesystem/reservation/kubernetes (main) $   




  
  ...
  
   1) 따릉이관리시스템에서 따릉이의 등록 
   
root@siege:/# http 10.100.146.187:8080/managements color="red" registeredDate="20230207" 
HTTP/1.1 201 Created
content-type: application/json
date: Wed, 08 Feb 2023 01:57:46 GMT
location: http://10.100.146.187/managements/1
server: istio-envoy
transfer-encoding: chunked
vary: Origin,Access-Control-Request-Method,Access-Control-Request-Headers
x-envoy-decorator-operation: bikemgmt.kafka.svc.cluster.local:8080/*
x-envoy-upstream-service-time: 467

{
    "_links": {
        "management": {
            "href": "http://10.100.146.187/managements/1"
        },
        "self": {
            "href": "http://10.100.146.187/managements/1"
        }
    },
    "color": "red",
    "registeredDate": "20230207"
}     
 
       
   2) point결재시스템에서 point 구매
      
      결재 (userId 10번으로 Point 3000 결재, 5000 결재 2회 실행)
      
root@siege:/# http 10.100.181.240:8080/approvals userId="10" price="3000" approveDate="20230207" 
HTTP/1.1 201 Created
content-type: application/json
date: Wed, 08 Feb 2023 01:59:27 GMT
location: http://10.100.181.240/approvals/1
server: istio-envoy
transfer-encoding: chunked
vary: Origin,Access-Control-Request-Method,Access-Control-Request-Headers
x-envoy-decorator-operation: approval.kafka.svc.cluster.local:8080/*
x-envoy-upstream-service-time: 435

{
    "_links": {
        "approval": {
            "href": "http://10.100.181.240/approvals/1"
        },
        "self": {
            "href": "http://10.100.181.240/approvals/1"
        }
    },
    "approveDate": "20230207",
    "price": 3000,
    "userId": 10
}

root@siege:/# http 10.100.181.240:8080/approvals userId="10" price="5000" approveDate="20230207" 
HTTP/1.1 201 Created
content-type: application/json
date: Wed, 08 Feb 2023 01:59:43 GMT
location: http://10.100.181.240/approvals/2
server: istio-envoy
transfer-encoding: chunked
vary: Origin,Access-Control-Request-Method,Access-Control-Request-Headers
x-envoy-decorator-operation: approval.kafka.svc.cluster.local:8080/*
x-envoy-upstream-service-time: 19

{
    "_links": {
        "approval": {
            "href": "http://10.100.181.240/approvals/2"
        },
        "self": {
            "href": "http://10.100.181.240/approvals/2"
        }
    },
    "approveDate": "20230207",
    "price": 5000,
    "userId": 10
}
      
      결재 후 확인 : point 시스템에 대해 gateway로 접근 가능여부 (userId 10번 조회)
      

 ![image](https://user-images.githubusercontent.com/2344829/217409607-07e44677-6c87-4c57-ba46-2067083f4584.png)
 
 
            
   3) 예약시스템에서 따릉이 예약

      예약 (userId 10번으로 bikeID 1번 예약)

root@siege:/# http POST 10.100.196.39:8080/reservations userId="10" bikeId="1" 
HTTP/1.1 201 Created
content-type: application/json
date: Wed, 08 Feb 2023 02:03:21 GMT
location: http://10.100.196.39/reservations/1
server: istio-envoy
transfer-encoding: chunked
vary: Origin,Access-Control-Request-Method,Access-Control-Request-Headers
x-envoy-decorator-operation: reservation.kafka.svc.cluster.local:8080/*
x-envoy-upstream-service-time: 574

{
    "_links": {
        "reservation": {
            "href": "http://10.100.196.39/reservations/1"
        },
        "self": {
            "href": "http://10.100.196.39/reservations/1"
        }
    },
    "bikeId": 1,
    "reserveDateTime": null,
    "reserveStatus": null,
    "returnDateTime": null,
    "userId": 10
}

      예약 확인
           
![image](https://user-images.githubusercontent.com/2344829/217410023-c5b80db4-03dd-4d4e-9ce2-7e19464ccafe.png)
 
 
      point 시스템에서 차감 확인
 
 ![image](https://user-images.githubusercontent.com/2344829/217411525-1c1e3984-6c65-4c9d-b16f-56c2de282aae.png)
 
      
     [CQRS] 예약DashBoard 에서 예약 현황 확인
      
![image](https://user-images.githubusercontent.com/2344829/217411732-efca23c3-02d3-4761-b2e2-fc2524ab1302.png)

 
          
  ...
  
## 
  
  

# 

## Model
www.msaez.io/#/storming/70d9e09c3c0a88189e6c4b72c6e2f61f

## Before Running Services
### Make sure there is a Kafka server running
```
cd kafka
docker-compose up
```
- Check the Kafka messages:
```
cd kafka
docker-compose exec -it kafka /bin/bash
cd /bin
./kafka-console-consumer --bootstrap-server localhost:9092 --topic
```

## Run the backend micro-services
See the README.md files inside the each microservices directory:

- approval
- reservation
- point
- bikeMgmt
- dashboard


## Run API Gateway (Spring Gateway)
```
cd gateway
mvn spring-boot:run
```

## Test by API
- approval
```
 http :8088/approvals confirmId="confirmId" userId="userId" price="price" approveDate="approveDate" 
```
- reservation
```
 http :8088/reservations reserveNo="reserveNo" userId="userId" bikeId="bikeId" reserveDateTime="reserveDateTime" returnDateTime="returnDateTime" reserveStatus="reserveStatus" 
```
- point
```
 http :8088/points userId="userId" totalPoint="totalPoint" 
```
- bikeMgmt
```
 http :8088/managements bikeId="bikeId" color="color" registeredDate="registeredDate" 
```
- dashboard
```
```


## Run the frontend
```
cd frontend
npm i
npm run serve
```

## Test by UI
Open a browser to localhost:8088

## Required Utilities

- httpie (alternative for curl / POSTMAN) and network utils
```
sudo apt-get update
sudo apt-get install net-tools
sudo apt install iputils-ping
pip install httpie
```

- kubernetes utilities (kubectl)
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

- aws cli (aws)
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

- eksctl 
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

