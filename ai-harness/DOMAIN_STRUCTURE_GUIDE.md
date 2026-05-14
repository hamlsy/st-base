# STBase Domain Structure Guide

> 이 문서는 STBase의 DDD 폴더 구조와 클래스 배치 기준을 설명합니다.

---

## 1. 기본 원칙

STBase는 도메인 중심 폴더 구조를 사용합니다.

기술별 최상위 패키지로 나누지 않습니다.

Bad:

```text
controller/
service/
repository/
entity/
dto/
```

Good:

```text
otc_bond_trade/
  application/
  domain/
  infrastructure/
  presentation/
```

---

## 2. 표준 도메인 폴더 구조

각 도메인은 아래 구조를 따릅니다.

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
    service/
  infrastructure/
    persistence/
    client/
    config/
  presentation/
    controller/
    dto/
```

필요 없는 하위 폴더는 만들지 않아도 됩니다.

---

## 3. Package Naming

Java package는 앱과 도메인을 함께 드러냅니다.

Example:

```text
com.stbase.powerbase.otcbondtrade.application.usecase
com.stbase.powerbase.otcbondtrade.application.service
com.stbase.powerbase.otcbondtrade.domain.entity
com.stbase.powerbase.otcbondtrade.domain.value
com.stbase.powerbase.otcbondtrade.domain.repository
com.stbase.powerbase.otcbondtrade.infrastructure.persistence
com.stbase.powerbase.otcbondtrade.presentation.controller
com.stbase.powerbase.otcbondtrade.presentation.dto
```

앱이 다르면 package root도 분리합니다.

Examples:

```text
com.stbase.powerbase
com.stbase.ksd
com.stbase.kofia
com.stbase.fss
com.stbase.freebond
com.stbase.externalbroker
com.stbase.admin
```

---

## 4. Application Usecase

Usecase는 하나의 명확한 업무 행위를 대표합니다.

Class naming:

```text
Verb + BusinessObject + UseCase
```

Examples:

```text
CreateOtcBondTradeUseCase
ReserveBondInventoryUseCase
SubmitKofiaOtcBondReportUseCase
RequestKsdDvpSettlementUseCase
FinalizeLedgerAfterKsdSettlementUseCase
RegisterComplaintUseCase
CollectComplaintEvidenceUseCase
RetryOutboxEventUseCase
```

Public method name:

```text
execute
```

Example:

```java
public CreateOtcBondTradeResult execute(CreateOtcBondTradeCommand command) {
    ...
}
```

Usecase should be easy to read as a business scenario.

Example flow:

```text
validate idempotency
load account
validate suitability
reserve bond inventory
create trade
create pending ledger postings
create KOFIA report outbox
create KSD settlement outbox
write audit log
return result
```

---

## 5. Application Service

Application service는 여러 usecase에서 공유되는 application-level 기능입니다.

Examples:

```text
IdempotencyService
AuditLogService
OutboxEventService
ExternalMessageLogService
TradeTimelineQueryService
ComplaintEvidenceCollector
```

Application service may depend on:

```text
domain repository interface
infrastructure adapter interface
common type
```

Application service must not:

```text
become a giant business god service
hide usecase flow
directly mutate balance
directly call external app inside trade transaction
```

---

## 6. Domain Entity

Entity는 도메인 상태와 불변식을 보호합니다.

Examples:

```text
OtcBondTrade
BondInventory
LedgerJournal
LedgerPosting
KsdSettlement
KofiaReport
Complaint
ReconciliationBreak
```

Entity method names should express business intent.

Good:

```text
reserve(...)
markSettlementRequested(...)
completeSettlement(...)
rejectReport(...)
finalizeLedger(...)
closeWithAnswer(...)
```

Bad:

```text
setStatus(...)
update(...)
process(...)
handle(...)
```

상태 전이는 Entity 또는 Domain Service가 보호합니다.

---

## 7. Domain Value Object

Value Object는 값의 의미를 명확히 합니다.

Examples:

```text
Money
Quantity
Rate
Price
Isin
AccountNumber
TradeDate
SettlementDate
CorrelationId
IdempotencyKey
```

Rules:

```text
Money/Rate/Price는 BigDecimal 기반
불변 객체로 설계
유효하지 않은 값 생성 금지
equals/hashCode 의미 보장
```

---

## 8. Domain Repository

`domain/repository`에는 interface만 둡니다.

Example:

```java
public interface OtcBondTradeRepository {
    Optional<OtcBondTrade> findById(OtcBondTradeId id);
    OtcBondTrade save(OtcBondTrade trade);
}
```

Spring Data JPA는 domain layer에 들어오면 안 됩니다.

---

## 9. Infrastructure Persistence

`infrastructure/persistence`에는 JPA 구현을 둡니다.

Typical classes:

```text
OtcBondTradeJpaEntity
OtcBondTradeJpaRepository
OtcBondTradeRepositoryAdapter
OtcBondTradeMapper
```

JPA Entity와 Domain Entity를 분리하는 것을 기본으로 합니다.

초기 속도를 위해 같은 클래스를 쓰고 싶다면, 반드시 문서화하고 나중에 분리할 수 있게 합니다.

---

## 10. Infrastructure Client

외부 앱 API client는 `infrastructure/client`에 둡니다.

Examples:

```text
KsdSettlementClient
KofiaReportClient
FssComplaintClient
PowerbaseTradeClient
```

Rules:

```text
client는 API 호출만 담당
client는 원장/거래 상태를 결정하지 않음
client 결과는 external message log로 기록
상태 변경 외부 호출은 outbox relay가 수행
```

---

## 11. Presentation

`presentation/controller`에는 REST Controller를 둡니다.

`presentation/dto`에는 request/response DTO를 둡니다.

Controller responsibility:

```text
HTTP request validation
header extraction
request DTO -> command 변환
usecase.execute 호출
result -> response DTO 변환
```

Controller forbidden:

```text
비즈니스 상태 전이
원장 posting 생성
JPA repository 직접 호출
외부기관 client 직접 호출
```

---

## 12. Common

공통은 `common`에 둡니다.

Suggested structure:

```text
common/
  api/
  error/
  money/
  quantity/
  time/
  tracing/
  idempotency/
```

Allowed:

```text
ApiResponse
ErrorCode
Money
Quantity
Rate
Price
BusinessDate
TraceId
CorrelationId
IdempotencyKey
```

Forbidden:

```text
OtcBondTrade
Account
LedgerPosting
KsdSettlement
KofiaReport
Complaint
```

Common은 특정 업무를 알면 안 됩니다.

---

## 13. Example Domain Tree

Example for `powerbase-app` OTC bond trade domain:

```text
apps/powerbase-app/src/main/java/com/stbase/powerbase/otcbondtrade/
  application/
    usecase/
      CreateOtcBondTradeUseCase.java
      FinalizeOtcBondTradeUseCase.java
      CancelOtcBondTradeUseCase.java
    service/
      OtcBondTradeIdempotencyService.java
      OtcBondTradeAuditService.java
  domain/
    entity/
      OtcBondTrade.java
    value/
      OtcBondTradeId.java
      OtcBondTradeStatus.java
      OtcBondTradeSide.java
    repository/
      OtcBondTradeRepository.java
    event/
      OtcBondTradeCreatedEvent.java
      OtcBondTradeSettlementRequestedEvent.java
  infrastructure/
    persistence/
      OtcBondTradeJpaEntity.java
      OtcBondTradeJpaRepository.java
      OtcBondTradeRepositoryAdapter.java
      OtcBondTradeMapper.java
    client/
      KofiaReportClient.java
      KsdSettlementClient.java
  presentation/
    controller/
      OtcBondTradeController.java
    dto/
      CreateOtcBondTradeRequest.java
      OtcBondTradeResponse.java
```

---

## 14. Review Rule

새 코드가 아래 냄새를 보이면 구조를 다시 봅니다.

```text
Controller가 너무 똑똑하다
Usecase가 아닌 Service가 모든 흐름을 숨긴다
Domain entity가 단순 getter/setter 덩어리다
common에 업무 클래스가 들어간다
JPA repository가 application/usecase에서 직접 사용된다
외부 API 호출이 transaction 안에서 즉시 수행된다
ledger posting이 update 된다
상태값이 String이다
테스트가 happy path만 있다
```
