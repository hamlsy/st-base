# STBase Project Skill

> ??臾몄꽌??STBase ?꾨줈?앺듃?먯꽌 AI媛 ?곕씪????湲곗닠 ?ㅽ궗怨?援ы쁽 痍⑦뼢??怨좎젙?⑸땲??

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

STBase??DDD 援ъ“瑜??ъ슜?⑸땲??

?꾨찓?몄? 湲곗닠 ?대뜑媛 ?꾨땲???낅Т 寃쎄퀎 湲곗??쇰줈 ?섎닏?덈떎.

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

媛??꾨찓?몄? ?꾨옒 援ъ“瑜??곕쫭?덈떎.

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

`application/usecase`?먮뒗 ?섎굹???낅Т 紐⑹쟻????쒗븯??usecase ?대옒?ㅻ? ?〓땲??

Usecase ?대옒?ㅻ챸? ??븷???쒕윭?섏빞 ?⑸땲??

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

Usecase public method ?대쫫? `execute`濡??듭씪?⑸땲??

```java
public OtcBondTradeResult execute(CreateOtcBondTradeCommand command) {
    ...
}
```

Usecase??orchestration???대떦?⑸땲??

Allowed:

```text
command validation ?몄텧
domain service ?몄텧
repository port ?몄텧
outbox event ?앹꽦
audit log 湲곕줉 ?붿껌
transaction boundary
```

Forbidden:

```text
Controller??鍮꾩쫰?덉뒪 濡쒖쭅 ?묒꽦
JPA Entity瑜?API ?묐떟?쇰줈 吏곸젒 ?몄텧
?몃? ??API瑜?transaction ?덉뿉??吏곸젒 ?몄텧
balance 吏곸젒 update
ledger posting ?섏젙/??젣
```

---

## 4. Application Service Rule

`application/service`?먮뒗 usecase瑜?蹂댁“?섎뒗 application service瑜??〓땲??

Examples:

```text
IdempotencyService
AuditLogService
OutboxEventService
ExternalMessageLogService
ComplaintEvidenceCollectService
```

Application service???뱀젙 usecase ?섎굹蹂대떎 ?ъ궗?⑹꽦???덉뼱???⑸땲??

---

## 5. Domain Layer Rule

`domain`? Spring???섏〈?섏? ?딆뒿?덈떎.

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

?꾨찓??媛앹껜???곹깭 蹂寃?洹쒖튃???ㅼ뒪濡?蹂댄샇?⑸땲??

Examples:

```text
LedgerPosting cannot be updated.
OtcBondTrade cannot be finalized before KSD settlement is completed.
Complaint cannot be closed without answer evidence.
BondInventory cannot be reserved below zero.
```

---

## 6. Infrastructure Layer Rule

`infrastructure`???몃? 湲곗닠 援ы쁽???대떦?⑸땲??

Examples:

```text
JPA Entity
Spring Data JpaRepository
Repository adapter
HTTP client adapter
MySQL mapping
Configuration
```

Repository 援ы쁽? domain repository interface瑜?援ы쁽?⑸땲??

```text
domain/repository/OtcBondTradeRepository
infrastructure/persistence/JpaOtcBondTradeRepository
infrastructure/persistence/OtcBondTradeRepositoryAdapter
```

?몃? ??API ?몄텧? `infrastructure/client`???〓땲??

```text
KsdClient
KofiaClient
FssClient
StbaseClient
```

?? ?곹깭 蹂寃??몃? ?몄텧? usecase?먯꽌 吏곸젒 ?몄텧?섏? ?딄퀬 outbox relay瑜??듯빐 ?섑뻾?섎뒗 寃껋쓣 湲곕낯?쇰줈 ?⑸땲??

---

## 7. Presentation Layer Rule

`presentation`? API ?낆텧?μ쓣 ?대떦?⑸땲??

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
鍮꾩쫰?덉뒪 洹쒖튃
?먯옣 泥섎━
?몃?湲곌? ?몄텧 濡쒖쭅
JPA Entity 吏곸젒 諛섑솚
```

Controller??request瑜?command濡?蹂?섑븯怨?usecase??`execute`瑜??몄텧?⑸땲??

---

## 8. Common Rule

怨듯넻 ??낆? `common`???〓땲??

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
?뱀젙 ?깆쓽 鍮꾩쫰?덉뒪 Entity
?뱀젙 ?꾨찓?몄쓽 Repository
?뱀젙 API Client
?뱀젙 ?붾㈃/Controller DTO
```

`common`??鍮꾨??댁?硫??꾨줈?앺듃 ?꾩껜媛 萸됯컻吏묐땲??

---

## 9. Persistence Rule

Spring Data JPA? MySQL???ъ슜?⑸땲??

Rules:

```text
Money/Rate/Price??BigDecimal ?ъ슜
double/float 湲덉?
?곹깭媛믪? enum ?ъ슜
String status 湲덉?
LocalDate???낅Т??嫄곕옒??寃곗젣?쇱뿉 ?ъ슜
OffsetDateTime? ?대깽??諛쒖깮 ?쒓컖???ъ슜
?숈떆??蹂댄샇媛 ?꾩슂??aggregate??@Version ?ъ슜
?ш퀬 李④컧? 議곌굔遺 update ?먮뒗 ?숆??????ъ슜
```

Ledger 愿??湲덉?:

```text
ledger_postings update 湲덉?
ledger_postings delete 湲덉?
?붽퀬 吏곸젒 蹂댁젙 湲덉?
蹂댁젙? correction posting
痍⑥냼??reversal posting
```

---

## 10. API Communication Rule

??媛??듭떊? 怨듦컻 API濡쒕쭔 ?⑸땲??

Required headers:

```http
X-Correlation-Id
X-Idempotency-Key
X-Source-System
```

紐⑤뱺 ?곹깭 蹂寃?API??idempotency key媛 ?꾩슂?⑸땲??

?몃? ???몄텧 湲곕줉? `external_message_log`???④퉩?덈떎.

PowerBase?먯꽌 KSD/KOFIA/FSS濡??섍????몄텧? 湲곕낯?곸쑝濡?outbox event瑜?癒쇱? ??ν븳 ??relay媛 ?꾩넚?⑸땲??

---

## 11. Test Rule

?뚯뒪?몃뒗 happy path蹂대떎 ?뺥빀??源⑥쭚??癒쇱? ?≪뒿?덈떎.

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

