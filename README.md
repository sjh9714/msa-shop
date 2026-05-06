# MSA Shop

MSA Shop은 쇼핑몰 도메인을 User, Product, Order, Payment, Settlement, API Gateway로 나누어 구현한 Spring Boot 멀티 모듈 프로젝트입니다. 서비스 경계, REST 호출, RabbitMQ 이벤트, SAGA/Outbox 보상 흐름, Gateway 인증과 rate limit, 관측성 구성을 함께 연습합니다.

## 문제 의식

모놀리식 CRUD를 넘어 서비스별 책임과 장애 경계를 분리하려면 주문 흐름이 특히 중요합니다. 이 저장소는 재고 예약, 결제, 주문 저장, 정산 집계를 서비스별로 나누고, 실패 시 보상 트랜잭션과 Outbox 처리로 흐름을 복구하는 구조를 실험합니다.

## 서비스 구성

| 서비스 | 포트 | 역할 |
| --- | --- | --- |
| `api-gateway` | 8080 | 단일 진입점, 라우팅, JWT 검증, rate limit |
| `user-service` | 8081 | 회원가입, 로그인, 내 정보 조회 |
| `product-service` | 8082 | 상품, 검색, 카테고리, 재고 예약/복구, 캐시 |
| `order-service` | 8083 | 주문, 취소, 장바구니, SAGA/Outbox 보상 |
| `payment-service` | 8084 | 가짜 PG 결제/취소, 결제 완료 이벤트 발행 |
| `settlement-service` | 8085 | 결제 완료 이벤트 구독, 일/월 매출 집계 |

## 주요 기능

- Spring Cloud Gateway 기반 API Gateway
- JWT 인증과 서비스별 보안 설정
- 상품 검색, 카테고리, 재고 예약/복구
- 장바구니 CRUD와 주문 생성/취소
- 주문 실패 시 재고 보상, 결제 성공 후 주문 저장 실패 시 Outbox 보상
- RabbitMQ 기반 결제 완료 이벤트 발행/구독
- Settlement 서비스의 일/월 매출 집계
- Prometheus, Grafana, Zipkin 기반 관측성 구성
- Docker Compose, Kubernetes YAML, Helm chart
- E2E 시나리오 스크립트와 Gradle 테스트

## 기술 스택

| 영역 | 기술 |
| --- | --- |
| Backend | Java 21, Spring Boot 3.5, Spring Data JPA |
| Gateway | Spring Cloud Gateway |
| Data / Messaging | MySQL 8, Redis 7, RabbitMQ |
| Resilience | Resilience4j Retry, CircuitBreaker |
| Observability | Prometheus, Grafana, Zipkin |
| Infra | Docker Compose, Kubernetes manifests, Helm |
| Test | JUnit 5, 통합 테스트, E2E shell scripts |

## 구조

```text
.
├── api-gateway/
├── user-service/
├── product-service/
├── order-service/
├── payment-service/
├── settlement-service/
├── docker/                 # MySQL init, Prometheus/Grafana 설정
├── docs/                   # 아키텍처, API, 실행, 장애 시나리오 문서
├── helm/msa-shop/          # Helm chart
├── k8s/                    # Kubernetes manifest
└── scripts/                # E2E와 Helm 배포 스크립트
```

## 실행 방법

전체 테스트는 Gradle로 실행합니다.

```bash
./gradlew test
```

로컬에서 서비스별로 실행할 수 있습니다.

```bash
./gradlew :user-service:bootRun
./gradlew :product-service:bootRun
./gradlew :order-service:bootRun
./gradlew :payment-service:bootRun
./gradlew :settlement-service:bootRun
```

RabbitMQ가 필요한 흐름은 별도 컨테이너를 실행한 뒤 E2E 스크립트를 사용합니다.

```bash
docker run -d -p 5672:5672 -p 15672:15672 rabbitmq:3-management
./scripts/e2e-flow.sh
```

Docker Compose로 Gateway, MySQL, RabbitMQ, Redis, 관측성 도구, 6개 앱을 함께 실행할 수 있습니다.

```bash
docker-compose up --build -d
GATEWAY_URL=http://localhost:8080 ./scripts/e2e-flow.sh
```

자세한 실행 순서와 트러블슈팅은 `docs/RUN-LOCAL.md`를 참고합니다.

## 문서

- `docs/ARCHITECTURE.md`: 아키텍처와 서비스 간 흐름
- `docs/API-SPEC.md`: API 명세
- `docs/IMPLEMENTED-SUMMARY.md`: 구현 기능 요약
- `docs/FAILURE-SCENARIOS.md`: 실패/보상 시나리오
- `docs/PROFILES-AND-SECRETS.md`: 프로필과 시크릿 운영
- `helm/README.md`: Helm 배포
- `docs/README.md`: 문서 전체 목차
