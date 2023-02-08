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
  3) 예약시스템 예약 시 포인트 시스템에서 포인트 차감
  4) 얘약시스템 예약 시 Dashboard에 반영

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
    
    
![image](https://user-images.githubusercontent.com/2344829/217419668-00880f88-b3c9-4be5-b6b0-3d7513b96d51.png)



  
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
  
  1) HPA 설정
  
![image](https://user-images.githubusercontent.com/2344829/217442626-6a7757ab-53a0-4c60-9937-d9f49bb34581.png)

  2) 설정 내용 확인 : replica
  
![image](https://user-images.githubusercontent.com/2344829/217416791-df92b169-1602-4fe8-b872-44e99adb5f89.png)

  3) siege 에서 부하 시작

![image](https://user-images.githubusercontent.com/2344829/217416702-87e0da3b-84b5-4e31-80b5-519a75a9a33c.png)

  
![image](https://user-images.githubusercontent.com/2344829/217442804-ca8dce6a-f635-496e-ab69-350a1d37a5e8.png)

  4) replica 변경 부분의 확인

![image](https://user-images.githubusercontent.com/2344829/217416636-46657225-90f2-44fa-900f-bae53f13e969.png)



 2. liveness, readiness probe 관련 무중단 deploy

  . 모델에서 generate 된 설정을 그대로 사용
  
  ![image](https://user-images.githubusercontent.com/2344829/217438610-d6dea3a3-b5df-43ed-aeea-951c8fd9d682.png)

  . liveness probe 조회 화면 (gateway)
  
  ![image](https://user-images.githubusercontent.com/2344829/217439095-f458c52d-dd27-4d54-8c10-9a5d4d35363b.png)


  . 따릉이 등록 시스템 대상 readiness probe 설정으로 무중단 배포 테스트
  
    1) image version up / push
    
![image](https://user-images.githubusercontent.com/2344829/217439532-5352bccb-b7ae-45dd-b991-f5cb2fbfb1b9.png)
  
    2) 등록 시스템에 siege 부하를 통해 pod를 3개까지 증가 시킴
    
![image](https://user-images.githubusercontent.com/2344829/217440001-ca1aa679-4d7d-41e9-9dee-2db53d0c18d3.png)

![image](https://user-images.githubusercontent.com/2344829/217442176-d6d9a1e9-d134-4bdf-9a59-0f83855b7c1e.png)


    3) siege 에서 부하를 주면서 새로운 image의 deploy 실행
    
![image](https://user-images.githubusercontent.com/2344829/217440228-e4cf13cc-1765-4de9-b7da-72cf6ef843e8.png)

![image](https://user-images.githubusercontent.com/2344829/217440302-fbcdf499-6b92-4a86-a472-9ccf49768ca8.png)

    4) deploy 기간 중 pod가 교체 되는 부분 확인 및 siege 의 요청이 모두 available 했음을 확인
    
![image](https://user-images.githubusercontent.com/2344829/217440493-964b4a67-7700-4562-9c49-78da580e9128.png)

![image](https://user-images.githubusercontent.com/2344829/217440730-27702040-a7e2-49c8-9152-98add34c9e85.png)




 3. Monitoring

  . 위 Autoscale / 무중단 deploy 에서 시스템 상황 모니터링
  
  ![image](https://user-images.githubusercontent.com/2344829/217417027-a7025029-be25-46ae-a2a4-c713a7d52120.png)


![image](https://user-images.githubusercontent.com/2344829/217442358-7ca7d132-6c4a-4439-8415-c30622b125b8.png)


![image](https://user-images.githubusercontent.com/2344829/217442482-d09e678e-090a-417e-b85f-49fb28ac3b77.png)


 4. ConfigMap 사용
  . 개발/운영 환경을 구분하기 위한 landscape 라는 환경 변수를 임의로 가정하고 진행
  

  . configmap 생성 후 deply

![image](https://user-images.githubusercontent.com/2344829/217449339-277ee824-e66b-4e1d-b200-e076cb0a06ed.png)


    kubectl apply -f configmap.yaml
    
![image](https://user-images.githubusercontent.com/2344829/217449560-8a6f7830-c60c-4e8c-a3f5-dd40be44505a.png)

  . deployment 에 configmap의 landscape 항목 반영
  
![image](https://user-images.githubusercontent.com/2344829/217449757-6ad77a01-8b6d-4fdf-8c64-ba41850b2c32.png)

  . pod 반영 후 정상적으로 수행 됨 확인
  
![image](https://user-images.githubusercontent.com/2344829/217450151-46d0154e-7a56-4dfc-bd7e-1170ffbf6900.png)


![image](https://user-images.githubusercontent.com/2344829/217450272-d0d2cc28-8a24-404d-b3bd-c8fe4b19a34f.png)


5. Persistence Volume / Secret 구현

  . mysql pod를 생성한다.
  . persistence volume을 사용하게 설정하고, 접속 정보는 Secret를 사용하게 구현한다
  
![image](https://user-images.githubusercontent.com/2344829/217456572-7064c083-e3a3-40dc-a1ed-d36623af517a.png)


![image](https://user-images.githubusercontent.com/2344829/217456667-c63e06b9-5fc2-4b18-b29a-9911b8b106bc.png)


![image](https://user-images.githubusercontent.com/2344829/217456916-decdbe66-394a-4a89-9f68-680fac56eaab.png)


![image](https://user-images.githubusercontent.com/2344829/217457025-daef72ce-d9a0-4795-ad43-ba8e38b0dc57.png)
  
  
 ![image](https://user-images.githubusercontent.com/2344829/217457660-2bd7c543-20f1-46b8-a682-c578b4b67a77.png)
  
  . database를 생성해서 사용할 수 있도록 한다
  
  . 따릉이 관리 시스템에서 따릉이 등록이 정상적으로 수행되는 것을 확인한다.
    1번 bike가 자동증가로 등록이 된 것을 확인한다.
  
![image](https://user-images.githubusercontent.com/2344829/217457894-ce231a2a-bcc1-44cd-aab0-39061aa00df7.png)

  . pod 와 mysql 서비스를 재가동 후 동일하게 수행 시 2번 bike 부터 입력이 되는 것을 확인한다
  
![image](https://user-images.githubusercontent.com/2344829/217458387-db1e0c74-28ba-496c-8bdb-3154f1fdf0c7.png)


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

