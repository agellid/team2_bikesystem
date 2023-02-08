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
  2) 결재시스템 결재 후 포인트 시스템에서 포인트 증가
  3) 예약시스템 결재 시 포인트 시스템에서 포인트 차감
  4) 얘약시스템 결재 시 Dashboard에 반영

![image](https://user-images.githubusercontent.com/2344829/217412767-34ff261d-2002-4b1c-af23-a66dc2632cfc.png)

  
## 구현

  . pub/sub를 통해 전달되는 내용을 policy에서 구현

  1. 결재시스템 결재 완료 시 userId 기준 point 시스템에 point 증가
     신규 사용자의 경우 insert 로직으로
     기존 사용자의 경우 기존 point를 update 하여 증가하는 로직

 ![image](https://user-images.githubusercontent.com/2344829/217417905-e6f28b75-70f7-425a-9cc6-1eb4b2e841f9.png)


  2. 예약시스템에서 예약 시 정해진 point 1000 이 차감되는 로직

![image](https://user-images.githubusercontent.com/2344829/217418214-be3d3f43-697b-47d5-86d0-b5adbf300659.png)


  3. 예약시스템에서 취소 시 point 1000 이 복원되는 로직

![image](https://user-images.githubusercontent.com/2344829/217418343-e9c42cf6-5414-4d08-aef8-af3d05d26afc.png)


 
  
  . 적용 후 테스트 (container 환경)
  
    각 마이크로 서비스의 기능에 대한 검증 및
    pub/sub 를 통한 서비스 확인
    gateway를 통한 서비스 확인
    CQRS 확인 을 수행 
  
    각 서비스의 Container IP와 LoadBalancer 정보는 다음과 같음
    
    



  
  ...
  
   1) 따릉이관리시스템에서 따릉이의 등록 
   
![image](https://user-images.githubusercontent.com/2344829/217415452-0f588431-a2f2-4a37-9813-c8db144b2e9f.png)

 
       
   2) point결재시스템에서 point 구매
      
      결재 (userId 10번으로 Point 3000 결재, 5000 결재 2회 실행)
      
![image](https://user-images.githubusercontent.com/2344829/217415615-af98dc8b-3bb8-4261-bcc9-d8f815478038.png)

![image](https://user-images.githubusercontent.com/2344829/217415694-6e007bd1-d9c9-4f73-9318-4ea182826a90.png)
      
      결재 후 확인 : point 시스템에 대해 gateway로 접근 가능여부 (userId 10번 조회)
      

 ![image](https://user-images.githubusercontent.com/2344829/217409607-07e44677-6c87-4c57-ba46-2067083f4584.png)
 
 
            
   3) 예약시스템에서 따릉이 예약

      예약 (userId 10번으로 bikeID 1번 예약)

![image](https://user-images.githubusercontent.com/2344829/217415755-cc6218b5-e591-4d76-a914-ecae4df0e08b.png)


      예약 확인
           
![image](https://user-images.githubusercontent.com/2344829/217410023-c5b80db4-03dd-4d4e-9ce2-7e19464ccafe.png)
 
 
      point 시스템에서 차감 확인
 
 ![image](https://user-images.githubusercontent.com/2344829/217411525-1c1e3984-6c65-4c9d-b16f-56c2de282aae.png)
 
      
     [CQRS] 예약DashBoard 에서 예약 현황 확인
      
![image](https://user-images.githubusercontent.com/2344829/217411732-efca23c3-02d3-4761-b2e2-fc2524ab1302.png)

 
          
  ...


## 운영

 1. Autoscale (HPA)
 
  . 따릉이 등록 시스템에 대해 부하 상황에 max 3개 pod 까지 증가하도록 설정 후 부하 테스트 / 결과 확인

![image](https://user-images.githubusercontent.com/2344829/217416791-df92b169-1602-4fe8-b872-44e99adb5f89.png)



![image](https://user-images.githubusercontent.com/2344829/217416702-87e0da3b-84b5-4e31-80b5-519a75a9a33c.png)



![image](https://user-images.githubusercontent.com/2344829/217416636-46657225-90f2-44fa-900f-bae53f13e969.png)


 2. Monitoring

  . 위 Autoscale 상황에서의 시스템 상황 모니터링
  
  ![image](https://user-images.githubusercontent.com/2344829/217417027-a7025029-be25-46ae-a2a4-c713a7d52120.png)



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

