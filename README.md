# 과일 주문 서비스
![image](https://user-images.githubusercontent.com/27180840/131888372-c52390ac-5f22-4da3-b8c6-44c80ddb16c7.png)

## 서비스 시나리오

### 기능적 요구 사항
1. 고객이 과일을 주문한다.
2. 고객이 결제한다.
3. 결제가 되면 배송이 시작된다.
4. 고객이 주문현황을 조회한다.
5. 결제가 성공시 배송이 가능하다.
6. 고객이 주문을 취소할 수 있다.
7. 고객이 주문을 취소하면 결제도 취소된다.

### 비기능적 요구 사항
1. 트랜잭션 
   - 고객은 결제가 완료되어야 배송 서비스 이용이 가능하다. (Sync)

2. 장애격리 
   - 과일 주문 서비스는 24시간 이용이 가능하다 (Async 호출-event-driven)
   - 결제 서비스가 과중되면 잠시 후에 하도록 유도한다. (Circuit breaker, fallback)

3. 성능
   - 고객이 주문한 내역의 진행현황을 확인할 수 있다. (CQRS)
   - 주문/걸제상태가 바뀔 때 마다 MyPage에서 상태를 확인할 수 있다.(Event Driven)
   
*****

## 분석/설계

### AS-IS 조직 (Horizontally-Aligned)

![image](https://user-images.githubusercontent.com/27180840/131845338-b5e0c548-1070-419e-b902-9f8ca8de1fd8.png)

### TO-BE 조직 (Vertically-Aligned)

![image](https://user-images.githubusercontent.com/27180840/131845668-3921794c-0b8d-4bae-84b9-ab0bd7b2fb5a.png)


### Event Storming 결과
MSAEz 로 모델링한 이벤트스토밍 결과:
http://www.msaez.io/#/storming/3CCWjZexX3Y7Ypm85RPzPTQIPLg1/113c0155f10101dd4e75418ff9cf25fe

### 이벤트스토밍 - Event

![image](https://user-images.githubusercontent.com/27180840/131846689-ad71f683-d921-42e5-9e38-67f7e15f09df.png)

### 이벤트스토밍 – 비적격 이벤트 제거

![image](https://user-images.githubusercontent.com/27180840/131846981-c706b08f-c2ac-4c3d-80ce-2180ccb60fbe.png)

### 이벤트스토밍 - Actor, Command

![image](https://user-images.githubusercontent.com/27180840/131900057-fefc78ac-f83f-4e62-82e1-d4896fee3144.png)

### 이벤트스토밍 - Aggregate

![image](https://user-images.githubusercontent.com/27180840/131900208-d83160fc-ee80-4569-b9a9-1b678f703355.png)

### 이벤트스토밍 - Bounded Context

![image](https://user-images.githubusercontent.com/27180840/131900317-867fcd9c-3414-418a-9454-dcc0c94a7361.png)

### 이벤트스토밍 - Policy

![image](https://user-images.githubusercontent.com/27180840/131900425-1479b2d8-d306-4eb2-adf9-4b550b4c33aa.png)

### 이벤트스토밍 - Context Mapping

![image](https://user-images.githubusercontent.com/27180840/131900493-94ca6dc6-f5eb-48fa-8e06-ca8752e00c20.png)

### 이벤트스토밍 - 완성된 모형

![image](https://user-images.githubusercontent.com/27180840/131900964-1a339a63-0b18-426b-81cd-cbefaca5fa35.png)

### 이벤트스토밍 - 기능 요구사항 Coverage Check

![image](https://user-images.githubusercontent.com/27180840/131900771-d970985c-dbec-484f-b9a5-f3880fa5570a.png)

### 이벤트스토밍 - 비기능 요구사항 Coverage Check

![image](https://user-images.githubusercontent.com/27180840/131901431-b7a38c42-6ad9-471a-818f-937e7f3ab372.png)

   1. 고객은 결제가 완료되어야 배송 서비스 이용이 가능하다. (Sync)
   2. 과일 주문 서비스는 24시간 이용이 가능하다 (Async 호출-event-driven)
   3. 결제 서비스가 과중되면 잠시 후에 하도록 유도한다. (Circuit breaker, fallback)
   4. 고객이 주문한 내역의 진행현황을 확인할 수 있다. (CQRS)
   5. 주문/결제상태가 바뀔 때 마다 MyPage에서 상태를 확인할 수 있다.(Event Driven)

### 헥사고날 아키텍처

![image](https://user-images.githubusercontent.com/27180840/131903383-a74d7f5d-58c9-4071-9a1b-5373c04a5094.png)


## 구현

분석/설계 단계에서 도출된 헥사고날 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 스프링부트로 구현하였다. 
구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다 (서비스 포트는 8086, 8082, 8087, 8084, 8088 이다)

```
cd order
mvn spring-boot:run

cd payment
mvn spring-boot:run

cd delivery
mvn spring-boot:run

cd gateway
mvn spring-boot:run

cd mypage
mvn spring-boot:run

```
*****

### DDD의 적용

1. 각 서비스내에 도출된 핵심 Aggregate Root 객체를 Entity 로 선언하였다. 
(예시는 order 마이크로 서비스 )

#### order.java

```
package fruitsorenew;

import javax.persistence.*;
import org.springframework.beans.BeanUtils;
import java.util.List;
import java.util.Date;

@Entity
@Table(name="Order_table")
public class Order {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long orderId;
    private Long userId;
    private String prodNm;
    private Integer qty;
    private Integer price;
    private String address;
    private String orderStatus;

    @PostPersist
    public void onPostPersist(){
        OrderPlaced orderPlaced = new OrderPlaced();
        BeanUtils.copyProperties(this, orderPlaced);
        orderPlaced.publishAfterCommit();

        fruitsorenew.external.Payment payment = new fruitsorenew.external.Payment();
        payment.setOrderId(this.getOrderId());
        payment.setUserId(this.getUserId());
        payment.setPrice(this.getPrice());
        payment.setPayStatus("결제요청");
        OrderApplication.applicationContext.getBean(fruitsorenew.external.PaymentService.class)
            .payment(payment);
    }

    @PostRemove
    public void onPostRemove(){
        OrderCanceled orderCanceled = new OrderCanceled();
        BeanUtils.copyProperties(this, orderCanceled);
        orderCanceled.publishAfterCommit();
    }

    public Long getOrderId() {
        return orderId;
    }

    public void setOrderId(Long orderId) {
        this.orderId = orderId;
    }
    public Long getUserId() {
        return userId;
    }

    public void setUserId(Long userId) {
        this.userId = userId;
    }
    public String getProdNm() {
        return prodNm;
    }
    ...
    
```

2. Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 다양한 
데이터소스 유형 (RDB or NoSQL) 에 대한 별도의 처리가 없도록 데이터 접근 어댑터를 
자동 생성하기 위하여 Spring Data REST 의 RestRepository 를 적용하였다

#### OrderRepository.java

```
package fruitsorenew;

import org.springframework.data.repository.PagingAndSortingRepository;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;

@RepositoryRestResource(collectionResourceRel="orders", path="orders")
public interface OrderRepository extends PagingAndSortingRepository<Order, Long>{
}

```

3. 적용 후 REST API 의 테스트

#### request 서비스의 요청처리

```
http order:8080/orders orderId="2" userId="7181" prodNm="사과" qty=2 price=5000  address="경기도 성남시" orderStatus="주문신청"  

```

#### Gateway 적용
API GateWay를 통하여 마이크로 서비스들의 진입점을 통일할 수 있다. 다음과 같이 GateWay를 적용하였다.

```
server:
  port: 8089

---

spring:
  profiles: default
  cloud:
    gateway:
      routes:
        - id: order
          uri: http://localhost:8086
          predicates:
            - Path=/orders/** 
        - id: mypage
          uri: http://localhost:8082
          predicates:
            - Path= /myPages/**
        - id: payment
          uri: http://localhost:8087
          predicates:
            - Path=/payments/** 
        - id: delivery
          uri: http://localhost:8084
          predicates:
            - Path=/deliveries/** 
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins:
              - "*"
            allowedMethods:
              - "*"
            allowedHeaders:
              - "*"
            allowCredentials: true


---

spring:
  profiles: docker
  cloud:
    gateway:
      routes:
        - id: order
          uri: http://order:8080
          predicates:
            - Path=/orders/** 
        - id: mypage
          uri: http://mypage:8080
          predicates:
            - Path= /myPages/**
        - id: payment
          uri: http://payment:8080
          predicates:
            - Path=/payments/** 
        - id: delivery
          uri: http://delivery:8080
          predicates:
            - Path=/deliveries/** 
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins:
              - "*"
            allowedMethods:
              - "*"
            allowedHeaders:
              - "*"
            allowCredentials: true

server:
  port: 8080
  
  ```

#### 요청상태 확인

```
http http://order:8080/orders/107

@ root@siege:/# http order:8080/orders/107
HTTP/1.1 201 
Content-Type: application/json;charset=UTF-8
Date: Fri, 03 Sep 2021 02:01:31 GMT
Location: http://order:8080/orders/107
Transfer-Encoding: chunked

{
    "_links": {
        "order": {
            "href": "http://order:8080/orders/107"
        },
        "self": {
            "href": "http://order:8080/orders/107"
        }
    },
    "address": "경기도 성남시",
    "orderStatus": "주문신청",
    "price": 50000,
    "prodId": 200,
    "qty": 5,
    "userId": 7181
}

@ root@siege:/# http payment:8080/payments/17
HTTP/1.1 200 
Content-Type: application/hal+json;charset=UTF-8
Date: Fri, 03 Sep 2021 02:23:22 GMT
Transfer-Encoding: chunked

{
    "_links": {
        "payment": {
            "href": "http://payment:8080/payments/17"
        },
        "self": {
            "href": "http://payment:8080/payments/17"
        }
    },
    "orderId": 5,
    "orderStatus": null,
    "payStatus": "결제요청",
    "price": 50000,
    "prodId": null,
    "qty": null,
    "userId": 7181
}

@http://delivery:8080/deliveries/38

HTTP/1.1 201 
Content-Type: application/json;charset=UTF-8
Date: Fri, 03 Sep 2021 02:30:45 GMT
Location: http://delivery:8080/deliveries/38
Transfer-Encoding: chunked

{
    "_links": {
        "delivery": {
            "href": "http://delivery:8080/deliveries/38"
        },
        "self": {
            "href": "http://delivery:8080/deliveries/38"
        }
    },
    "address": "경기도 성남시",
    "deliveryStatus": "배송시작",
    "orderId": 5,
    "orderStatus": null,
    "payStatus": null,
    "prodId": null,
    "qty": 5,
    "userId": 7181
}
```
*****

### CQRS
과일 주문 내역 및 Status 에 대하여 고객(Customer)이 조회 할 수 있도록 CQRS 로 구현하였다.
order, payment, delivery 의 개별 Aggregate Status 를 통합 조회하여 성능 Issue 를 사전에 예방할 수 있다.
비동기식으로 처리되어 발행된 이벤트 기반 Kafka 를 통해 수신/처리 되어 별도 Table 에 관리한다

```

@Service
public class MyPageViewHandler {


    @Autowired
    private MyPageRepository myPageRepository;

    //주문시 신규 생성
    @StreamListener(KafkaProcessor.INPUT)
    public void whenOrderPlaced_then_CREATE_1 (@Payload OrderPlaced orderPlaced) {
        try {

            if (!orderPlaced.validate()) return;

            // view 객체 생성
            MyPage myPage = new MyPage();
            // view 객체에 이벤트의 Value 를 set 함
            myPage.setOrderId(orderPlaced.getOrderId());
            myPage.setUserId(orderPlaced.getUserId());
            myPage.setProdNm(orderPlaced.getProdNm());
            myPage.setQty(orderPlaced.getQty());
            myPage.setPrice(orderPlaced.getPrice());
            myPage.setOrderStatus(orderPlaced.getOrderStatus());
            // view 레파지 토리에 save
            myPageRepository.save(myPage);

        }catch (Exception e){
            e.printStackTrace();
        }
    }
    //결제승인시 결제 정보 업데이트
    @StreamListener(KafkaProcessor.INPUT)
    public void whenPaymentApproved_then_CREATE_2 (@Payload PaymentApproved paymentApproved) {
        try {

            if (!paymentApproved.validate()) return;

            // view 객체 생성
            MyPage myPage = new MyPage();
            // view 객체에 이벤트의 Value 를 set 함
            myPage.setOrderId(paymentApproved.getOrderId());
            myPage.setPayId(paymentApproved.getPayId());
            myPage.setPayStatus(paymentApproved.getPayStatus());
            // view 레파지 토리에 save
            myPageRepository.save(myPage);

        }catch (Exception e){
            e.printStackTrace();
        }
    }

    //결제 취소때 결제상태 업데이트
    @StreamListener(KafkaProcessor.INPUT)
    public void whenPaymentCanceled_then_UPDATE_1(@Payload PaymentCanceled paymentCanceled) {
        try {
            if (!paymentCanceled.validate()) return;
                // view 객체 조회
            Optional<MyPage> myPageOptional = myPageRepository.findByOrderId(paymentCanceled.getOrderId());

            if( myPageOptional.isPresent()) {
                 MyPage myPage = myPageOptional.get();
            // view 객체에 이벤트의 eventDirectValue 를 set 함
                 myPage.setPayStatus(paymentCanceled.getPayStatus());
                // view 레파지 토리에 save
                 myPageRepository.save(myPage);
                }


        }catch (Exception e){
            e.printStackTrace();
        }
    }
    // 주문 취소때 주문상태 업데이트
    @StreamListener(KafkaProcessor.INPUT)
    public void whenOrderCanceled_then_UPDATE_2(@Payload OrderCanceled orderCanceled) {
        try {
            if (!orderCanceled.validate()) return;
                // view 객체 조회
            Optional<MyPage> myPageOptional = myPageRepository.findByOrderId(orderCanceled.getOrderId());

            if( myPageOptional.isPresent()) {
                 MyPage myPage = myPageOptional.get();
            // view 객체에 이벤트의 eventDirectValue 를 set 함
                 myPage.setOrderStatus(orderCanceled.getOrderStatus());
                // view 레파지 토리에 save
                 myPageRepository.save(myPage);
                }


        }catch (Exception e){
            e.printStackTrace();
        }
    }

}

```


### 동기식 호출과 Fallback 처리

분석단계에서의 조건 중 하나로 주문(order)-> 결제(payment) 간의 호출은 동기식 일관성을 유지하는 트랜잭션으로 처리한다. 
호출 프로토콜은 이미 앞서 Rest Repository 에 의해 노출되어있는 REST 서비스를 FeignClient 를 이용하여 호출한다.

1. 결제서비스를 호출하기 위하여 FeignClient 를 이용하여 Service 대행 인터페이스 구현

#### PaymentService.java

```
@FeignClient(name="payment", url="http://localhost:8087", fallback = PaymentServiceFallback.class)
public interface PaymentService {
    @RequestMapping(method= RequestMethod.GET, path="/payments")
    public void payment(@RequestBody Payment payment);
}
```

2. 주문 생성 직후(@PostPersist) 결제를 요청하도록 처리 Order.java Entity Class 내 추가 (Correlation)

#### Order.java

```
    @PostPersist
    public void onPostPersist(){
        OrderPlaced orderPlaced = new OrderPlaced();
        BeanUtils.copyProperties(this, orderPlaced);
        orderPlaced.publishAfterCommit();

        fruitsorenew.external.Payment payment = new fruitsorenew.external.Payment();
        payment.setOrderId(this.getOrderId());
        payment.setUserId(this.getUserId());
        payment.setPrice(this.getPrice());
        payment.setPayStatus("결제요청");
        OrderApplication.applicationContext.getBean(fruitsorenew.external.PaymentService.class)
            .payment(payment);
    }

    @PostRemove
    public void onPostRemove(){
        OrderCanceled orderCanceled = new OrderCanceled();
        BeanUtils.copyProperties(this, orderCanceled);
        orderCanceled.publishAfterCommit();
    }

    public Long getOrderId() {
        return orderId;
    }

    public void setOrderId(Long orderId) {
        this.orderId = orderId;
    }
    public Long getUserId() {
        return userId;
    }

    public void setUserId(Long userId) {
        this.userId = userId;
    }
    public String getProdNm() {
        return prodNm;
    }

```
*****



배송 서비스에서는 결제승인 이벤트에 대해서 이를 수신하여 자신의 정책을 처리하도록 PolicyHandler 를 구현한다

#### PolicyHandler.java

```
package fruitsorenew;

import fruitsorenew.config.kafka.KafkaProcessor;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.stereotype.Service;

@Service
public class PolicyHandler{
    @Autowired DeliveryRepository deliveryRepository;

    @StreamListener(KafkaProcessor.INPUT)
    public void wheneverPaymentApproved_Acceptdelivery(@Payload PaymentApproved paymentApproved){

        if(!paymentApproved.validate()) return;

        System.out.println("\n\n##### listener Acceptdelivery : " + paymentApproved.toJson() + "\n\n");

    }
    @StreamListener(KafkaProcessor.INPUT)
    public void wheneverPaymentCanceled_CancleDelivery(@Payload PaymentCanceled paymentCanceled){

        if(!paymentCanceled.validate()) return;

        System.out.println("\n\n##### listener CancleDelivery : " + paymentCanceled.toJson() + "\n\n");


    }


    @StreamListener(KafkaProcessor.INPUT)
    public void whatever(@Payload String eventString){}


}
```
*****

## 운영
*****

### 서킷 브레이킹

1. Spring FeignClient + Hystrix 옵션을 사용하여 구현

2. 주문-걸제시 Request/Response 로 연동하여 구현이 되어있으며 요청이 과도할 경우 CB를 통하여 장애격리 

3. Hystrix 를 설정: 요청처리 쓰레드에서 처리시간이 610 밀리가 넘어서기 시작하여 어느정도 유지되면 

CB 회로가 닫히도록 (요청을 빠르게 실패처리, 차단) 설정

#### application.yml

```
hystrix:
  command:
    default:
      execution.isolation.thread.timeoutInMilliseconds: 610
        
```

4. 결제 서비스의 임의 부하 처리 

#### Payment.java (Entity)

```
    @PostPersist
    public void onPostPersist(){
     ...
		try {
		    Thread.currentThread().sleep((long) (400 + Math.random() * 220));
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
    }
    
```

5. 부하테스터 seige 툴을 통한 서킷 브레이커 동작 확인
( 동시사용자 100명, 90초간 진행 )

root@siege:/# siege -v -c100 -t90S -r10 --content-type "application/json" 'http://order:8080/orders POST {"orderId"="001", "prodNm"="수박"}'

```

```
운영시스템은 죽지 않고 지속적으로 CB 에 의하여 적절히 회로가 열림과 닫힘이 벌어지면서 자원을 보호하고 있음을 보여줌. 
동적 Scale out (replica의 자동적 추가,HPA) 을 통하여 시스템을 확장 해주는 후속처리가 필요.
*****

### 오토스케일 아웃

#### 앞서 CB 는 시스템을 안정되게 운영할 수 있게 해줬지만 사용자의 요청을 100% 받아들여주지 못했기 때문에 이에 대한 보완책으로 자동화된 확장 기능을 적용하고자 한다.
결제서비스에 대한 replica 를 동적으로 늘려주도록 HPA 를 설정한다. 설정은 CPU 사용량이 50프로를 넘어서면 replica 를 10개까지 늘려준다

```

root@labs--1339476173:/home/project/fruitstorenew/order# kubectl autoscale deployment order --cpu-percent=50 --min=1 --max=10
horizontalpodautoscaler.autoscaling/order autoscaled

root@labs--1339476173:/home/project/fruitstorenew/order# kubectl autoscale deployment payment --cpu-percent=50 --min=1 --max=10
Error from server (AlreadyExists): horizontalpodautoscalers.autoscaling "payment" already exists

root@labs--1339476173:/home/project/fruitstorenew/order# kubectl get horizontalpodautoscalers
NAME      REFERENCE            TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
order     Deployment/order     <unknown>/50%   1         10        1          118s
payment   Deployment/payment   <unknown>/50%   1         10        1          15m



root@labs--1339476173:/home/project/fruitstorenew# cd payment
root@labs--1339476173:/home/project/fruitstorenew/payment# cd kubernetes

root@labs--1339476173:/home/project/fruitstorenew/payment/kubernetes# kubectl apply -f deployment.yml
error: error parsing deployment.yml: error converting YAML to JSON: yaml: line 47: did not find expected key

root@labs--1339476173:/home/project/fruitstorenew/payment/kubernetes# kubectl get pod
NAME                         READY   STATUS             RESTARTS   AGE
delivery-64f989599d-wsnwj    1/1     Running            0          24h
gateway-749574fc88-jf59x     1/1     Running            0          24h
homepage-69685b8f76-99f9x    1/1     Running            0          23h
mypage-f95b4876c-fv5dw       1/1     Running            0          6h32m
order-5578d978b5-knl4g       1/1     Running            0          21h
payment-789689f8dd-x9f8r     1/1     Running            0          24h
payment-797cf7dc88-xtv56     0/2     CrashLoopBackOff   2          15s
php-apache-d4cf67d68-nlddz   1/1     Running            0          10m
siege                        1/1     Running            0          24h

root@labs--1339476173:/home/project/fruitstorenew/payment/kubernetes# kubectl get pod
NAME                         READY   STATUS             RESTARTS   AGE
delivery-64f989599d-wsnwj    1/1     Running            0          24h
gateway-749574fc88-jf59x     1/1     Running            0          24h
homepage-69685b8f76-99f9x    1/1     Running            0          23h
mypage-f95b4876c-fv5dw       1/1     Running            0          6h42m
order-849df6cfcf-mkt52       1/1     Running            0          96s
payment-65b8847957-nr6sb     0/1     CrashLoopBackOff   5          5m21s
php-apache-d4cf67d68-nlddz   1/1     Running            0          20m
siege                        1/1     Running            0          24h

root@labs--1339476173:/home/project/fruitstorenew/payment/kubernetes# kubectl get hpa
NAME         REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
order        Deployment/order        94%/50%         1         10        2          136m
payment      Deployment/payment      <unknown>/50%   1         10        1          149m
php-apache   Deployment/php-apache   0%/50%          1         10        1          26m

```

#### 부하 테스트 진행


```
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "500Mi"
              cpu: "500m"
```

root@siege:/# siege -v -c100 -t90S -r10 --content-type "application/json" 'http://order:8080/orders POST {"orderId":"005", "userId":"07181", "qty":5, "price":50000, "address":"경기도 성남시", "orderStatus":"주문신청됨"}'
( 동시사용자 100명, 90초간 진행 )

```
root@siege:/# siege -v -c100 -t30S -r10 --content-type "application/json" 'http://order:8080/orders POST {"orderId":"005", "userId":"07181", "qty":5, "price":50000, "address":"경기도 성남시", "orderStatus":"주문신청됨"}'
[error] CONFIG conflict: selected time and repetition based testing
defaulting to time-based testing: 30 seconds
** SIEGE 4.0.4
** Preparing 100 concurrent users for battle.
The server is now under siege...
HTTP/1.1 500     5.05 secs:     248 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     5.83 secs:     248 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     6.39 secs:     248 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     7.47 secs:     248 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     8.51 secs:     248 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.02 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.01 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.07 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.07 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.01 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.00 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.01 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.01 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.00 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.01 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.08 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.07 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.01 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.01 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.00 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.00 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.01 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.01 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.00 secs:     820 bytes ==> POST http://order:8080/orders
HTTP/1.1 500     0.00 secs:     820 bytes ==> POST http://order:8080/orders

Lifting the server siege...
Transactions:                      0 hits
Availability:                   0.00 %
Elapsed time:                  29.55 secs
Data transferred:               0.08 MB
Response time:                  0.00 secs
Transaction rate:               0.00 trans/sec
Throughput:                     0.00 MB/sec
Concurrency:                   22.84
Successful transactions:           0
Failed transactions:             123
Longest transaction:           28.87
Shortest transaction:           0.00
 

```

#### Terminal 을 추가하여 오토스케일링 현황을 모니터링 한다. ( watch kubectl get pod )

#### 부하 테스트 진행전 

```

```

#### 부하 테스트 진행 후 

```

```

#### 부하테스트 결과 Availability 는 100% 를 보이며 성공하였고, 늘어난 pod 개수를 통하여

오토 스케일링이 정상적으로 수행되었음을 확인할 수 있다. 
*****



*****

### Deploy
1. 서비스별로 deploy.yaml 파일을 생성했다. 

```
@ delivery서비스 내 deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: delivery
  labels:
    app: delivery
spec:
  replicas: 1
  selector:
    matchLabels:
      app: delivery
  template:
    metadata:
      labels:
        app: delivery
    spec:
      containers:
        - name: delivery
          image: 052937454741.dkr.ecr.ap-northeast-1.amazonaws.com/u8-delivery:latest
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: '/actuator/health'
              port: 8080
            initialDelaySeconds: 10
            timeoutSeconds: 2
            periodSeconds: 5
            failureThreshold: 10
          livenessProbe:
            httpGet:
              path: '/actuator/health'
              port: 8080
            initialDelaySeconds: 120
            timeoutSeconds: 2
            periodSeconds: 5
            failureThreshold: 5
```
@ delivery서비스 내 service.yaml
apiVersion: v1
kind: Service
metadata:
  name: delivery
  labels:
    app: delivery
spec:
  ports:
    - port: 8080
      targetPort: 8080
  selector:
    app: delivery
    ```
2. 명령어로 실행

```
root@labs--1339476173:/home/project/fruitstorenew/delivery# export ECR=052937454741.dkr.ecr.ap-northeast-1.amazonaws.com

#package
root@labs--1339476173:/home/project/fruitstorenew/delivery# mvn package -Dmaven.test.skip=true
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by com.google.inject.internal.cglib.core.$ReflectUtils$1 (file:/usr/share/maven/lib/guice.jar) to method java.lang.ClassLoader.defineClass(java.lang.String,byte[],int,int,java.security.ProtectionDomain)
WARNING: Please consider reporting this to the maintainers of com.google.inject.internal.cglib.core.$ReflectUtils$1
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
[INFO] Scanning for projects...
[INFO] 
[INFO] -----------------------< fruitsorenew:delivery >------------------------
[INFO] Building delivery 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:3.1.0:resources (default-resources) @ delivery ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ delivery ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 11 source files to /home/project/fruitstorenew/delivery/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:3.1.0:testResources (default-testResources) @ delivery ---
[INFO] Not copying test resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.8.1:testCompile (default-testCompile) @ delivery ---
[INFO] Not compiling test sources
[INFO] 
[INFO] --- maven-surefire-plugin:2.22.2:test (default-test) @ delivery ---
[INFO] Tests are skipped.
[INFO] 
[INFO] --- maven-jar-plugin:3.1.2:jar (default-jar) @ delivery ---
[INFO] Building jar: /home/project/fruitstorenew/delivery/target/delivery-0.0.1-SNAPSHOT.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.1.9.RELEASE:repackage (repackage) @ delivery ---
[INFO] Replacing main artifact with repackaged archive
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  3.951 s
[INFO] Finished at: 2021-09-03T05:31:26Z
[INFO] ------------------------------------------------------------------------

#도커 빌드
root@labs--1339476173:/home/project/fruitstorenew/delivery# docker build -t ${ECR}/u8-delivery:latest .
Sending build context to Docker daemon 59.79 MB
Step 1/4 : FROM openjdk:8u212-jdk-alpine
 ---> a3562aa0b991
Step 2/4 : COPY target/*SNAPSHOT.jar app.jar
 ---> c62bdbf2fbc8
Step 3/4 : EXPOSE 8080
 ---> Running in 55f7b5f911c0
Removing intermediate container 55f7b5f911c0
 ---> 6993fba06c53
Step 4/4 : ENTRYPOINT ["java","-Xmx400M","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar","--spring.profiles.active=docker"]
 ---> Running in 9c901ba326ae
Removing intermediate container 9c901ba326ae
 ---> b695794a3cb1
Successfully built b695794a3cb1
Successfully tagged 052937454741.dkr.ecr.ap-northeast-1.amazonaws.com/u8-delivery:latest

#
root@labs--1339476173:/home/project/fruitstorenew/delivery# docker push ${ECR}/u8-delivery:latest
The push refers to repository [052937454741.dkr.ecr.ap-northeast-1.amazonaws.com/u8-delivery]
7c8a2cd7547e: Pushed 
ceaf9e1ebef5: Layer already exists 
9b9b7f3d56a0: Layer already exists 
f1b5933fe4b5: Layer already exists 
latest: digest: sha256:9d4843bc776d1554171e412cc893b6aeb8678c8c8d7ec0d8951d0dd07d05c2b3 size: 1159
root@labs--1339476173:/home/project/fruitstorenew/delivery# 

```
    
