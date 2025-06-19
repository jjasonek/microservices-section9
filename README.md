# Udemy Course Microservices with Spring Boot, Docker, Kubernetes Section9
https://www.udemy.com/course/master-microservices-with-spring-docker-kubernetes/
## spring version: 3.4.5
## gatewayservice version: 3.5.0
## Java 21


## Documentation
After adding the library, the swagger page is accessible through address 
http://localhost:8080/swagger-ui/index.html,
http://localhost:8090/swagger-ui/index.html,
http://localhost:9000/swagger-ui/index.html.


## Note: 
### We start on section 8


## Gateway Server Testing
After start there is message in the log:
"
The use of configuration keys that have been renamed was found in the environment:

Property source 'Config resource 'class path resource [application.yml]' via location 'optional:classpath:/'':
Key: spring.cloud.gateway.discovery.locator.enabled
Line: 10
**Replacement: spring.cloud.gateway.server.webflux.discovery.locator.enabled**
"
So I did the recommended change. 

### Link from Eureka Server to Gateway Server
http://172.24.64.1:8072/actuator/info
{
    "app": {
        "name": "gatewayserver",
        "description": "Eazy Bank Gateway Server Application",
        "version": "1.0.0"
    }
}

### Further links
http://localhost:8072/actuator
http://localhost:8072/actuator/gateway
http://localhost:8072/actuator/gateway/routes

### Example invoking an service:
POST http://localhost:8072/ACCOUNTS/api/create

### Example invoking an service:
POST http://localhost:8072/eazybank/accounts/api/create


## Add correlation ID logging

### after implementation and creating DB data we call:
GET http://localhost:8072/eazybank/accounts/api/fetchCustomerDetails?mobileNumber=4354437687
Response Headers
transfer-encoding: chunked
Content-Type: application/json
Date: Wed, 18 Jun 2025 12:17:13 GMT
**eazybank-correlation-id: 92d6acec-8833-4e38-a381-2c51fa710786**
X-Response-Time: 2025-06-18T14:14:15.409015600

### and in the gatewayserver log:
2025-06-18T14:16:13.739+02:00 DEBUG 16952 --- [gatewayserver] [ctor-http-nio-3] c.e.g.filters.RequestTraceFilter         : eazyBank-correlation-id generated in RequestTraceFilter : 42990758-204b-424b-af9b-1c508fd8e3c8
2025-06-18T14:16:14.242+02:00 DEBUG 16952 --- [gatewayserver] [ctor-http-nio-3] c.e.g.filters.ResponseTraceFilter        : Updated the correlation id to the outbound headers: 42990758-204b-424b-af9b-1c508fd8e3c8
2025-06-18T14:16:51.958+02:00 DEBUG 16952 --- [gatewayserver] [ctor-http-nio-3] c.e.g.filters.RequestTraceFilter         : eazyBank-correlation-id generated in RequestTraceFilter : 14bbb1f3-9ac5-498b-887e-85b7506e72da
2025-06-18T14:16:52.286+02:00 DEBUG 16952 --- [gatewayserver] [ctor-http-nio-3] c.e.g.filters.ResponseTraceFilter        : Updated the correlation id to the outbound headers: 14bbb1f3-9ac5-498b-887e-85b7506e72da
2025-06-18T14:16:59.618+02:00 DEBUG 16952 --- [gatewayserver] [ctor-http-nio-3] c.e.g.filters.RequestTraceFilter         : eazyBank-correlation-id generated in RequestTraceFilter : cb09bdb5-908d-409a-9822-f1ea4fa5591f
2025-06-18T14:16:59.938+02:00 DEBUG 16952 --- [gatewayserver] [ctor-http-nio-3] c.e.g.filters.ResponseTraceFilter        : Updated the correlation id to the outbound headers: cb09bdb5-908d-409a-9822-f1ea4fa5591f
2025-06-18T14:17:13.574+02:00 DEBUG 16952 --- [gatewayserver] [ctor-http-nio-3] c.e.g.filters.RequestTraceFilter         : eazyBank-correlation-id generated in RequestTraceFilter : 92d6acec-8833-4e38-a381-2c51fa710786
2025-06-18T14:17:13.730+02:00 DEBUG 16952 --- [gatewayserver] [ctor-http-nio-3] c.e.g.filters.ResponseTraceFilter        : Updated the correlation id to the outbound headers: 92d6acec-8833-4e38-a381-2c51fa710786

### and microservices logs:
2025-06-18T14:17:13.588+02:00 DEBUG 8376 --- [accounts] [nio-8080-exec-2] c.e.a.controller.CustomerController      : eazybank-correlation-id found: 92d6acec-8833-4e38-a381-2c51fa710786
Hibernate: select c1_0.customer_id,c1_0.created_at,c1_0.created_by,c1_0.email,c1_0.mobile_number,c1_0.name,c1_0.updated_at,c1_0.updated_by from customer c1_0 where c1_0.mobile_number=?
Hibernate: select a1_0.account_number,a1_0.account_type,a1_0.branch_address,a1_0.created_at,a1_0.created_by,a1_0.customer_id,a1_0.updated_at,a1_0.updated_by from accounts a1_0 where a1_0.customer_id=?
2025-06-18T14:21:54.661+02:00  INFO 8376 --- [accounts] [rap-executor-%d] c.n.d.s.r.aws.ConfigClusterResolver      : Resolving eureka endpoints via configuration

2025-06-18T14:17:13.678+02:00 DEBUG 46152 --- [loans] [nio-8090-exec-2] c.e.loans.controller.LoansController     : eazybank-correlation-id found: 92d6acec-8833-4e38-a381-2c51fa710786
Hibernate: select l1_0.loan_id,l1_0.amount_paid,l1_0.created_at,l1_0.created_by,l1_0.loan_number,l1_0.loan_type,l1_0.mobile_number,l1_0.outstanding_amount,l1_0.total_loan,l1_0.updated_at,l1_0.updated_by from loans l1_0 where l1_0.mobile_number=?
2025-06-18T14:21:50.835+02:00  INFO 46152 --- [loans] [rap-executor-%d] c.n.d.s.r.aws.ConfigClusterResolver      : Resolving eureka endpoints via configuration

2025-06-18T14:17:13.712+02:00 DEBUG 24664 --- [cards] [nio-9000-exec-2] c.e.cards.controller.CardsController     : eazybank-correlation-id found: 92d6acec-8833-4e38-a381-2c51fa710786
Hibernate: select c1_0.card_id,c1_0.amount_used,c1_0.available_amount,c1_0.card_number,c1_0.card_type,c1_0.created_at,c1_0.created_by,c1_0.mobile_number,c1_0.total_limit,c1_0.updated_at,c1_0.updated_by from cards c1_0 where c1_0.mobile_number=?
2025-06-18T14:21:50.835+02:00  INFO 24664 --- [cards] [rap-executor-%d] c.n.d.s.r.aws.ConfigClusterResolver      : Resolving eureka endpoints via configuration


## Docker Images and Docker Compose

### for all microservices we call following to generate docker images:
mvn compile jib:dockerBuild

docker images | grep s9
jjasonek/eurekaserver                      s9        f59a03a7eb30   55 years ago    344MB
jjasonek/gatewayserver                     s9        52fa96c13f6c   55 years ago    338MB
jjasonek/loans                             s9        57bdb8d2f3d9   55 years ago    374MB
jjasonek/cards                             s9        af786fbc14c8   55 years ago    374MB
jjasonek/configserver                      s9        755f84749b36   55 years ago    331MB
jjasonek/accounts                          s9        5cb2437feeda   55 years ago    375MB

docker image ls --filter=reference="jjasonek/*:s9"
REPOSITORY               TAG       IMAGE ID       CREATED        SIZE
jjasonek/loans           s9        57bdb8d2f3d9   55 years ago   374MB
jjasonek/configserver    s9        755f84749b36   55 years ago   331MB
jjasonek/eurekaserver    s9        f59a03a7eb30   55 years ago   344MB
jjasonek/gatewayserver   s9        52fa96c13f6c   55 years ago   338MB
jjasonek/accounts        s9        5cb2437feeda   55 years ago   375MB
jjasonek/cards           s9        af786fbc14c8   55 years ago   374MB

### push images to docker hub:
docker image push docker.io/jjasonek/accounts:s9
docker image push docker.io/jjasonek/loans:s9
docker image push docker.io/jjasonek/cards:s9
docker image push docker.io/jjasonek/configserver:s9
docker image push docker.io/jjasonek/eurekaserver:s9
docker image push docker.io/jjasonek/gatewayserver:s9

### run docker compose
docker compose up -d
docker compose down
