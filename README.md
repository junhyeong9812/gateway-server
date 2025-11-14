# Spring Cloud Gateway 정리

## 1. Spring Cloud Gateway란?

Spring Cloud Gateway는 **Spring Cloud 생태계에서 API Gateway 역할을 수행**하는 서버로, 마이크로서비스 아키텍처(MSA)에서 들어오는 요청을 라우팅하고, 공통 기능(필터링, 인증/인가, 로깅, CORS 등)을 제공합니다. 기존에 사용되던 Zuul 1.x의 차세대 버전으로, **Netty 기반 WebFlux(논블로킹)**을 사용하여 고성능을 제공합니다.

---

## 2. Spring Cloud Gateway가 필요한 이유

1. **서비스 진입점 통합(Entry Point)**

   * 클라이언트가 여러 서비스의 도메인을 직접 알 필요 없음
   * 게이트웨이가 요청을 받아 적절한 서비스로 라우팅

2. **공통 기능을 중앙 집중화**

   * 인증/인가
   * 로깅/트래픽 모니터링
   * Rate Limiting
   * 공통 헤더 추가
   * CORS 처리

3. **MSA 환경의 복잡도 감소**

   * 클라이언트 → Gateway → Eureka(Service Discovery) → 내부 서비스 구조로 단순화됨

---

## 3. Spring Cloud Gateway 내부 동작 구조

### (1) 요청 흐름

```
Client
  ↓
Spring Cloud Gateway
  ↓  (Route + Predicate + Filter 조합)
Service Discovery(Eureka)
  ↓
Target Service (hub-service, product-service 등)
```

### (2) 핵심 구성 요소

#### ✔ Route

* 요청을 어떤 서비스로 보낼지 결정
* **uri**, **predicates**, **filters**로 구성됨

#### ✔ Predicates

* 요청 조건을 정의하는 영역
* 예: Path, Method, Header, Query, RemoteAddr

#### ✔ Filters

* 요청/응답을 변환하는 기능을 담당
* 예: 토큰 추출, 인증, 로깅, Path 재작성, Rate Limit 등 수행 가능

---

## 4. 주요 설정 예시

```yaml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
          lower-case-service-id: true

      routes:
        - id: hub-service
          uri: lb://hub-service
          predicates:
            - Path=/v1/hub/**
          filters:
            - RewritePath=/v1/hub/(?<path>.*), /${path}

        - id: product-service
          uri: lb://product-service
          predicates:
            - Path=/v1/product/**
          filters:
            - RewritePath=/v1/product/(?<path>.*), /${path}
```

---

## 5. Load Balancing 동작 방식

* `uri: lb://service-name` 사용 시 Spring Cloud LoadBalancer가 적용됨
* Eureka에서 서비스 인스턴스 목록을 조회
* 라운드로빈 방식으로 분산 처리

---

## 6. 필터 종류

### ✔ Pre Filter

요청이 서비스에 도달하기 전에 수행

* 인증 / 인가
* Header 추가
* IP 검증

### ✔ Post Filter

응답이 클라이언트에게 가기 전에 수행

* 응답 Body 변환
* 공통 응답 Wrapper 적용
* 응답 로깅

---

## 7. Gateway에서 자주 사용하는 기능

1. **JWT 인증 필터**
2. **Path 재작성(RewritePath)**
3. **CORS 처리**
4. **Rate Limiting (Redis 기반)**
5. **Circuit Breaker(Resilience4j)**
6. **Retry 정책 적용**

---

## 8. Spring Cloud Gateway의 장점

1. **Netty 기반 WebFlux로 고성능 처리 가능**
2. Spring Cloud와 자연스럽게 통합(Eureka, Config Server 등)
3. Kotlin DSL, Java 코드 기반 커스텀 필터 생성 가능
4. 선언적 라우팅 → 유지보수 용이

---

## 9. Spring Cloud Gateway vs Nginx 비교

| 항목         | Spring Cloud Gateway | Nginx                 |
| ---------- | -------------------- | --------------------- |
| 목적         | API Gateway(동적 라우팅)  | Reverse Proxy(정적 라우팅) |
| 프로그래밍      | Java 기반 커스텀 로직 가능    | Lua 등 별도 스크립팅 필요      |
| 서비스 디스커버리  | O (Eureka 연동)        | X                     |
| Rate Limit | O                    | O                     |
| 인증/인가      | 커스텀 필터로 용이           | 직접 구현 필요              |

---

## 10. Spring Cloud Gateway를 쓸 때 주의사항

* WebFlux 기반이므로 Servlet 기반(Spring MVC) 필터와 섞으면 안 됨
* 고성능이지만 비즈니스 로직은 절대 Gateway에 넣으면 안 됨

  * **Gateway = Request 가공 + 라우팅만 담당**
* Config Server와 Eureka 의존성이 높아짐

---

## 11. 전체 아키텍처 예시 (Eureka + Gateway)

```
          [ Client ]
               ↓
     [ Spring Cloud Gateway ]
               ↓
         ┌───────────┬───────────┐
         ↓           ↓           ↓
 [ hub-service ] [ product-service ] [ auth-service ]
         ↑           ↑           ↑
      (Eureka Service Discovery)
```

---


