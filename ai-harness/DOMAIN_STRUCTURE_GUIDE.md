# STBase Domain Structure Guide

> ??臾몄꽌??STBase??DDD ?대뜑 援ъ“? ?대옒??諛곗튂 湲곗????ㅻ챸?⑸땲??

---

## 1. 湲곕낯 ?먯튃

STBase???꾨찓??以묒떖 ?대뜑 援ъ“瑜??ъ슜?⑸땲??

湲곗닠蹂?理쒖긽???⑦궎吏濡??섎늻吏 ?딆뒿?덈떎.

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

## 2. ?쒖? ?꾨찓???대뜑 援ъ“

媛??꾨찓?몄? ?꾨옒 援ъ“瑜??곕쫭?덈떎.

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

?꾩슂 ?녿뒗 ?섏쐞 ?대뜑??留뚮뱾吏 ?딆븘???⑸땲??

---

## 3. Package Naming

Java package???깃낵 ?꾨찓?몄쓣 ?④퍡 ?쒕윭?낅땲??

Example:

```text
com.stbase.stbaseapp.otcbondtrade.application.usecase
com.stbase.stbaseapp.otcbondtrade.application.service
com.stbase.stbaseapp.otcbondtrade.domain.entity
com.stbase.stbaseapp.otcbondtrade.domain.value
com.stbase.stbaseapp.otcbondtrade.domain.repository
com.stbase.stbaseapp.otcbondtrade.infrastructure.persistence
com.stbase.stbaseapp.otcbondtrade.presentation.controller
com.stbase.stbaseapp.otcbondtrade.presentation.dto
```

?깆씠 ?ㅻⅤ硫?package root??遺꾨━?⑸땲??

Examples:

```text
com.stbase.stbaseapp
com.stbase.ksd
com.stbase.kofia
com.stbase.fss
com.stbase.freebond
com.stbase.externalbroker
com.stbase.admin
```

---

## 4. Application Usecase

Usecase???섎굹??紐낇솗???낅Т ?됱쐞瑜???쒗빀?덈떎.

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

Application service???щ윭 usecase?먯꽌 怨듭쑀?섎뒗 application-level 湲곕뒫?낅땲??

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

Entity???꾨찓???곹깭? 遺덈??앹쓣 蹂댄샇?⑸땲??

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

?곹깭 ?꾩씠??Entity ?먮뒗 Domain Service媛 蹂댄샇?⑸땲??

---

## 7. Domain Value Object

Value Object??媛믪쓽 ?섎?瑜?紐낇솗???⑸땲??

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
Money/Rate/Price??BigDecimal 湲곕컲
遺덈? 媛앹껜濡??ㅺ퀎
?좏슚?섏? ?딆? 媛??앹꽦 湲덉?
equals/hashCode ?섎? 蹂댁옣
```

---

## 8. Domain Repository

`domain/repository`?먮뒗 interface留??〓땲??

Example:

```java
public interface OtcBondTradeRepository {
    Optional<OtcBondTrade> findById(OtcBondTradeId id);
    OtcBondTrade save(OtcBondTrade trade);
}
```

Spring Data JPA??domain layer???ㅼ뼱?ㅻ㈃ ???⑸땲??

---

## 9. Infrastructure Persistence

`infrastructure/persistence`?먮뒗 JPA 援ы쁽???〓땲??

Typical classes:

```text
OtcBondTradeJpaEntity
OtcBondTradeJpaRepository
OtcBondTradeRepositoryAdapter
OtcBondTradeMapper
```

JPA Entity? Domain Entity瑜?遺꾨━?섎뒗 寃껋쓣 湲곕낯?쇰줈 ?⑸땲??

珥덇린 ?띾룄瑜??꾪빐 媛숈? ?대옒?ㅻ? ?곌퀬 ?띕떎硫? 諛섎뱶??臾몄꽌?뷀븯怨??섏쨷??遺꾨━?????덇쾶 ?⑸땲??

---

## 10. Infrastructure Client

?몃? ??API client??`infrastructure/client`???〓땲??

Examples:

```text
KsdSettlementClient
KofiaReportClient
FssComplaintClient
PowerbaseTradeClient
```

Rules:

```text
client??API ?몄텧留??대떦
client???먯옣/嫄곕옒 ?곹깭瑜?寃곗젙?섏? ?딆쓬
client 寃곌낵??external message log濡?湲곕줉
?곹깭 蹂寃??몃? ?몄텧? outbox relay媛 ?섑뻾
```

---

## 11. Presentation

`presentation/controller`?먮뒗 REST Controller瑜??〓땲??

`presentation/dto`?먮뒗 request/response DTO瑜??〓땲??

Controller responsibility:

```text
HTTP request validation
header extraction
request DTO -> command 蹂??
usecase.execute ?몄텧
result -> response DTO 蹂??
```

Controller forbidden:

```text
鍮꾩쫰?덉뒪 ?곹깭 ?꾩씠
?먯옣 posting ?앹꽦
JPA repository 吏곸젒 ?몄텧
?몃?湲곌? client 吏곸젒 ?몄텧
```

---

## 12. Common

怨듯넻? `common`???〓땲??

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

Common? ?뱀젙 ?낅Т瑜??뚮㈃ ???⑸땲??

---

## 13. Example Domain Tree

Example for `stbase-app` OTC bond trade domain:

```text
apps/stbase-app/src/main/java/com/stbase/powerbase/otcbondtrade/
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

??肄붾뱶媛 ?꾨옒 ?꾩깉瑜?蹂댁씠硫?援ъ“瑜??ㅼ떆 遊낅땲??

```text
Controller媛 ?덈Т ?묐삊?섎떎
Usecase媛 ?꾨땶 Service媛 紐⑤뱺 ?먮쫫???④릿??
Domain entity媛 ?⑥닚 getter/setter ?⑹뼱由щ떎
common???낅Т ?대옒?ㅺ? ?ㅼ뼱媛꾨떎
JPA repository媛 application/usecase?먯꽌 吏곸젒 ?ъ슜?쒕떎
?몃? API ?몄텧??transaction ?덉뿉??利됱떆 ?섑뻾?쒕떎
ledger posting??update ?쒕떎
?곹깭媛믪씠 String?대떎
?뚯뒪?멸? happy path留??덈떎
```

