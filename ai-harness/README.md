# STBase AI Harness

> 이 폴더는 AI 바이브코딩 도구가 STBase를 구현할 때 반드시 먼저 읽어야 하는 작업 하네스입니다.
>
> STBase는 Spring Boot, Spring Data JPA, MySQL 기반의 DDD 구조 프로젝트이며, 장외채권 거래를 시작점으로 유관기관 API 시뮬레이터와 통신하는 학습용 분산 증권업무 시스템입니다.

---

## 1. AI 작업 시작 순서

AI는 코드 작업 전에 아래 문서를 순서대로 읽습니다.

```text
1. docs/STBASE_MULTI_APP_BOND_SIMULATOR_PLAN.md
2. ai-harness/PROJECT_SKILL.md
3. ai-harness/DOMAIN_STRUCTURE_GUIDE.md
4. ai-harness/PROMPTING_GUIDE.md
5. ai-harness/AI_IMPLEMENTATION_CHECKLIST.md
```

기존 개요 문서도 참고할 수 있습니다.

```text
STBASE_README_REVISED.md
STBASE_AI_DEVELOPMENT_OVERVIEW_REVISED.md
```

---

## 2. 프로젝트 핵심 방향

STBase는 단일 모놀리스가 아니라 여러 Spring Boot 앱이 API로 통신하는 구조를 목표로 합니다.

```text
powerbase-app
external-broker-app
freebond-app
ksd-app
kofia-app
fss-app
admin-app
bond-exchange-app
```

초기 구현은 장외채권 MVP를 먼저 완성합니다.

```text
외부 증권사 장외채권 거래 요청
-> PowerBase 검증
-> pending ledger posting
-> KOFIA 보고
-> KSD 결제
-> ledger finalize
-> 대사
-> 민원 접수/처리
-> 운영자 재처리/감사로그
```

---

## 3. 절대 원칙

```text
실제 금융기관 API 연동 금지
실제 주문 가능 broker adapter 금지
실제 API key/secret/certificate 구조 금지
cross-app DB 접근 금지
ledger posting update/delete 금지
balance 직접 update 금지
보고 실패 무시 금지
결제 실패 은폐 금지
민원 처리 시 증적 누락 금지
```

---

## 4. 기술 스택

```text
Java 17+
Spring Boot
Spring Data JPA
MySQL
Gradle multi-project
JUnit 5
Testcontainers
Docker Compose
```

선택 확장은 나중에 도입합니다.

```text
Redis
Kafka or RabbitMQ
Prometheus/Grafana
OpenSearch/ELK
```

초기에는 HTTP REST + Outbox + External Message Log로 충분합니다.

---

## 5. 기본 코드 구조

각 앱의 도메인 폴더는 다음 구조를 따릅니다.

```text
domain-name/
  application/
    usecase/
    service/
  domain/
    entity/
    value/
    repository/
    event/
  infrastructure/
    persistence/
    client/
    config/
  presentation/
    controller/
    dto/
```

공통 타입과 공통 유틸은 `common`에 둡니다.

---

## 6. AI가 기능을 구현할 때의 기준

기능 하나를 구현할 때는 다음을 명확히 해야 합니다.

```text
어느 앱의 기능인가?
어느 도메인 폴더에 들어가는가?
상태 변경 command인가?
idempotency key가 필요한가?
ledger 영향이 있는가?
외부 앱 API 호출이 있는가?
outbox event가 필요한가?
external message log가 필요한가?
실패 상태와 재처리 경로가 있는가?
민원/대사/감사와 연결되는가?
```

---

## 7. 네이밍 핵심

Usecase 클래스는 역할이 드러나는 이름으로 만들고, 실행 메서드는 `execute`로 통일합니다.

Examples:

```text
CreateOtcBondTradeUseCase.execute(...)
FinalizeKsdSettlementUseCase.execute(...)
SubmitKofiaOtcBondReportUseCase.execute(...)
RegisterComplaintUseCase.execute(...)
RetryOutboxEventUseCase.execute(...)
```

Service는 도메인 규칙 또는 application orchestration을 보조합니다.

```text
BondInventoryReservationService
LedgerPostingService
ComplaintEvidenceCollector
SettlementRetryService
```

---

## 8. 첫 구현 목표

첫 번째 완성 시나리오는 다음입니다.

```text
외부 증권사가 장외채권 매수 요청을 보낸다.
PowerBase가 계좌와 재고를 검증한다.
PowerBase가 pending ledger posting을 만든다.
PowerBase가 KOFIA 보고와 KSD 결제 outbox를 만든다.
KOFIA 보고는 성공한다.
KSD 결제는 성공한다.
PowerBase가 ledger를 finalize 한다.
대사가 MATCHED 된다.
고객이 민원을 넣는다.
운영자가 admin timeline에서 거래, 원장, 보고, 결제, 대사, 감사로그를 확인한다.
운영자가 답변을 작성하고 민원을 종료한다.
```

이 시나리오가 통과하기 전까지 주식, 선물, RP, ATS/SOR 구현을 시작하지 않습니다.

---

## 9. Prompting Rule

AI에게 작업을 맡길 때는 [PROMPTING_GUIDE.md](PROMPTING_GUIDE.md)의 템플릿을 사용합니다.

핵심 원칙:

```text
AI는 구조와 반복 코드를 만든다.
문서는 정답 코드를 제공한다.
사람은 핵심 usecase/service를 직접 따라 치며 익힌다.
AI는 execute 내부 업무 로직을 임의로 창작하지 않는다.
```
