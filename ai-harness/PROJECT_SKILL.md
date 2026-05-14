# STBase Project Skill

> 이 문서는 STBase 프로젝트에서 AI가 따라야 할 기술 스킬과 구현 취향을 고정합니다.

---

## 1. Main Stack

```text
Language: Java 17+
Framework: Spring Boot
Persistence: Spring Data JPA
Database: MySQL
Build: Gradle multi-project
Architecture: DDD-based modular multi-app
Communication: HTTP REST first
Reliability: Idempotency + Outbox + External Message Log
Testing: JUnit 5 + Testcontainers
```

---

## 2. Architectural Style

STBase는 DDD 구조를 사용합니다.

도메인은 기술 폴더가 아니라 업무 경계 기준으로 나눕니다.

Examples:

```text
member
account
bond
ledger
otc-bond-trade
settlement
reconciliation
complaint
outbox
external-message
audit
```

각 도메인은 아래 구조를 따릅니다.

```text
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

---

## 3. Application Layer Rule

`application/usecase`에는 하나의 업무 목적을 대표하는 usecase 클래스를 둡니다.

Usecase 클래스명은 역할이 드러나야 합니다.

```text
CreateOtcBondTradeUseCase
ReserveBondInventoryUseCase
SubmitKofiaReportUseCase
RequestKsdSettlementUseCase
FinalizeLedgerAfterSettlementUseCase
RegisterComplaintUseCase
ResolveComplaintUseCase
RetryOutboxEventUseCase
```

Usecase public method 이름은 `execute`로 통일합니다.

```java
public OtcBondTradeResult execute(CreateOtcBondTradeCommand command) {
    ...
}
```

Usecase는 orchestration을 담당합니다.

Allowed:

```text
command validation 호출
domain service 호출
repository port 호출
outbox event 생성
audit log 기록 요청
transaction boundary
```

Forbidden:

```text
Controller에 비즈니스 로직 작성
JPA Entity를 API 응답으로 직접 노출
외부 앱 API를 transaction 안에서 직접 호출
balance 직접 update
ledger posting 수정/삭제
```

---

## 4. Application Service Rule

`application/service`에는 usecase를 보조하는 application service를 둡니다.

Examples:

```text
IdempotencyService
AuditLogService
OutboxEventService
ExternalMessageLogService
ComplaintEvidenceCollectService
```

Application service는 특정 usecase 하나보다 재사용성이 있어야 합니다.

---

## 5. Domain Layer Rule

`domain`은 Spring에 의존하지 않습니다.

Allowed:

```text
Entity
Value Object
Domain Service
Repository interface
Domain Event
Enum
Business rule
```

Forbidden:

```text
@Service
@Component
@Repository
@RestController
JpaRepository
WebClient
RestTemplate
Spring Transaction
```

도메인 객체는 상태 변경 규칙을 스스로 보호합니다.

Examples:

```text
LedgerPosting cannot be updated.
OtcBondTrade cannot be finalized before KSD settlement is completed.
Complaint cannot be closed without answer evidence.
BondInventory cannot be reserved below zero.
```

---

## 6. Infrastructure Layer Rule

`infrastructure`는 외부 기술 구현을 담당합니다.

Examples:

```text
JPA Entity
Spring Data JpaRepository
Repository adapter
HTTP client adapter
MySQL mapping
Configuration
```

Repository 구현은 domain repository interface를 구현합니다.

```text
domain/repository/OtcBondTradeRepository
infrastructure/persistence/JpaOtcBondTradeRepository
infrastructure/persistence/OtcBondTradeRepositoryAdapter
```

외부 앱 API 호출은 `infrastructure/client`에 둡니다.

```text
KsdClient
KofiaClient
FssClient
PowerbaseClient
```

단, 상태 변경 외부 호출은 usecase에서 직접 호출하지 않고 outbox relay를 통해 수행하는 것을 기본으로 합니다.

---

## 7. Presentation Layer Rule

`presentation`은 API 입출력을 담당합니다.

Allowed:

```text
Controller
Request DTO
Response DTO
Exception handler
API validation annotation
```

Forbidden:

```text
비즈니스 규칙
원장 처리
외부기관 호출 로직
JPA Entity 직접 반환
```

Controller는 request를 command로 변환하고 usecase의 `execute`를 호출합니다.

---

## 8. Common Rule

공통 타입은 `common`에 둡니다.

Allowed:

```text
Money
Quantity
Rate
Price
BusinessDate
CorrelationId
IdempotencyKey
TraceId
ErrorCode
ApiResponse
PageResponse
ClockProvider
```

Forbidden:

```text
특정 앱의 비즈니스 Entity
특정 도메인의 Repository
특정 API Client
특정 화면/Controller DTO
```

`common`이 비대해지면 프로젝트 전체가 뭉개집니다.

---

## 9. Persistence Rule

Spring Data JPA와 MySQL을 사용합니다.

Rules:

```text
Money/Rate/Price는 BigDecimal 사용
double/float 금지
상태값은 enum 사용
String status 금지
LocalDate는 업무일/거래일/결제일에 사용
OffsetDateTime은 이벤트 발생 시각에 사용
동시성 보호가 필요한 aggregate는 @Version 사용
재고 차감은 조건부 update 또는 낙관적 락 사용
```

Ledger 관련 금지:

```text
ledger_postings update 금지
ledger_postings delete 금지
잔고 직접 보정 금지
보정은 correction posting
취소는 reversal posting
```

---

## 10. API Communication Rule

앱 간 통신은 공개 API로만 합니다.

Required headers:

```http
X-Correlation-Id
X-Idempotency-Key
X-Source-System
```

모든 상태 변경 API는 idempotency key가 필요합니다.

외부 앱 호출 기록은 `external_message_log`에 남깁니다.

PowerBase에서 KSD/KOFIA/FSS로 나가는 호출은 기본적으로 outbox event를 먼저 저장한 뒤 relay가 전송합니다.

---

## 11. Test Rule

테스트는 happy path보다 정합성 깨짐을 먼저 잡습니다.

Priority:

```text
idempotency
duplicate message
inventory concurrency
ledger immutability
settlement failure
report failure
reconciliation break
complaint evidence
operator audit log
cross-app API boundary
```

Test names should explain business behavior.

Example:

```text
same_idempotency_key_creates_only_one_otc_bond_trade
ksd_completion_callback_does_not_create_duplicate_ledger_posting
operator_correction_creates_new_posting_instead_of_updating_existing_posting
```
