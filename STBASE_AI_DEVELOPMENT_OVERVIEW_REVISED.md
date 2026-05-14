# STBase AI Development Overview

> 이 문서는 AI 개발 도구가 STBase 프로젝트를 구현할 때 반드시 따라야 하는 개발 지시서입니다.  
> STBase는 단순 주문 API 프로젝트가 아니라, PowerBase 구조를 모티브로 한 **모의 증권업무계 + 모의 자본시장 인프라**입니다.

---

## 1. Project Identity

```text
Project Name: STBase
Full Name: Securities Trade Base
Main Stack: Java, Spring Boot, JPA, MySQL, Gradle Multi Module
Domain: Securities operations simulation
```

STBase는 다음을 구현합니다.

```text
PowerBase-Sim
K-FRONT-Sim
STP-HUB-Sim
StockNet-Sim
EXTURE/KRX-Sim
KSD-Sim
KOFIA FreeBond/Disclosure-Sim
FSS-Sim
MarketInfo/BondInfo-Sim
```

STBase는 다음을 구현하지 않습니다.

```text
실제 거래소 주문
실제 증권사 API 연동
실제 예탁결제원 전문
실제 금감원 보고
실제 금융투자협회 보고
실제 고객 자산 처리
실제 투자/매매 실행
```

---

## 2. Absolute Safety Rules

AI는 아래 규칙을 절대 위반하면 안 됩니다.

```text
1. 실제 금융기관 API 연동 코드를 만들지 않는다.
2. 실제 주문 가능성을 암시하는 broker adapter를 만들지 않는다.
3. 실제 API key, secret, certificate, account 연동 구조를 만들지 않는다.
4. 모든 기관 모듈은 내부 simulation module로만 둔다.
5. 실거래 가능 코드와 모의거래 코드를 섞지 않는다.
6. 원장 posting을 수정/삭제하는 API를 만들지 않는다.
7. balance를 직접 update하는 비즈니스 코드를 만들지 않는다.
8. 중복 요청이 중복 주문/체결/원장반영으로 이어지면 안 된다.
9. 외부기관 시뮬레이션 호출은 반드시 로그와 재처리 구조를 갖는다.
10. 정합성보다 편의성을 우선하지 않는다.
```

---

## 3. Correct Mental Model

STBase의 중심은 “주문”이 아니라 “증권사 업무계 원장”입니다.

정확한 업무 흐름:

```text
채널/프론트
→ 주문 또는 거래 입력
→ PowerBase-Sim 검증
→ 주문 라우팅 또는 장외거래 확정
→ StockNet-Sim 전문 전달
→ EXTURE-Sim 체결
→ Clearing
→ KSD-Sim 결제
→ PowerBase-Sim 원장 반영
→ KOFIA/FSS 보고
→ 대사
→ 감사
```

AI는 모든 기능을 구현할 때 아래 질문을 먼저 해야 합니다.

```text
이 기능은 어떤 거래주체에 대한 것인가?
장내거래인가 장외거래인가?
주문 기반인가 거래입력 기반인가?
PowerBase 원장에 어떤 영향을 주는가?
KSD-Sim 총량과 대사 가능한가?
KOFIA/FSS 보고 대상인가?
중복 요청에 안전한가?
실패 시 재처리 가능한가?
감사 로그가 남는가?
```

---

## 4. Module Boundaries

---

## 4.1 stbase-common

공통 타입을 둡니다.

```text
Money
Quantity
Rate
Price
BusinessDate
CorrelationId
IdempotencyKey
TraceId
DomainEvent
ErrorCode
```

규칙:

```text
Money/Rate/Price에는 double/float 금지
BigDecimal 사용
도메인 타입은 명시적으로 만든다
```

---

## 4.2 stbase-powerbase-core

증권사 업무계 중심 모듈입니다.

Responsibilities:

```text
고객 원장 중심 업무흐름 조정
계좌 상태 검증
거래 가능 여부 검증
주문 결과 반영
거래 결과 반영
결제 예정 생성
결제 결과 반영
보고 이벤트 생성
대사 이벤트 생성
```

Anti-responsibilities:

```text
호가장 관리 금지
매칭 엔진 구현 금지
KSD 잔고 직접 조작 금지
KOFIA/FSS 데이터 직접 수정 금지
```

---

## 4.3 stbase-member

거래주체 관리.

Entities:

```text
Member
IndividualProfile
InstitutionProfile
BrokerDealerProfile
SuitabilityProfile
TradingPermission
```

MemberType:

```text
INDIVIDUAL
INSTITUTION
BROKER_DEALER
HOUSE
EXCHANGE_SIM
DEPOSITORY_SIM
ASSOCIATION_SIM
REGULATOR_SIM
```

Rules:

```text
개인/기관/증권사/HOUSE 계정은 반드시 구분한다.
거래주체 유형에 따라 허용 거래가 달라진다.
```

---

## 4.4 stbase-account

계좌 관리.

Entities:

```text
Account
AccountRelation
AccountStatusHistory
```

AccountType:

```text
RETAIL_BROKERAGE
INSTITUTION
HOUSE_TRADING
SETTLEMENT
DEPOSITORY_LINKED
MARGIN
```

Rules:

```text
정지 계좌는 신규 주문 불가
폐쇄 계좌는 신규 posting 불가
위탁계좌와 자기계정은 분리
KSD 계좌와 PowerBase 고객계좌는 별도
```

---

## 4.5 stbase-product

상품마스터.

SecurityType:

```text
STOCK
BOND
FUTURE
RP
ETF
```

MarketType:

```text
KOSPI
KOSDAQ
KONEX
DERIVATIVES
BOND_GENERAL
BOND_SMALL
BOND_GOVERNMENT
REPO
OTC_BOND
OTC_RP
```

Bond fields:

```text
isin
issuer
bondType
issueDate
maturityDate
couponRate
couponFrequency
dayCountConvention
creditRating
faceValue
listed
tradableMarkets
```

---

## 4.6 stbase-retail-channel-sim

개인 고객 요청을 시뮬레이션합니다.

Responsibilities:

```text
개인 주문 요청
잔고 조회
체결 조회
상품 조회
거래내역 조회
```

Rule:

```text
Retail Channel은 원장을 직접 수정하지 않는다.
```

---

## 4.7 stbase-branch-terminal-sim

영업점/직원 단말 시뮬레이션.

Responsibilities:

```text
직원 대행 주문
고객 계좌 조회
투자성향 확인
채권 판매 입력
운영성 조회
```

Rule:

```text
직원 행위는 audit log에 operatorId와 함께 남긴다.
```

---

## 4.8 stbase-kfront-sim

전문 트레이더/법인영업/자기매매 OMS 시뮬레이션.

Responsibilities:

```text
전문 주문 입력
자기매매 주문
바스켓 주문
알고리즘 주문 더미
FIX 스타일 주문 메시지 생성
시장데이터 기반 주문 판단 더미
```

Rules:

```text
K-FRONT는 원장을 직접 수정하지 않는다.
K-FRONT는 주문/거래 의사와 메시지만 생성한다.
```

---

## 4.9 stbase-stp-hub-sim

기관투자가와 증권사 사이의 허브.

Responsibilities:

```text
Buy-side 주문 메시지 수신
Sell-side로 라우팅
체결결과 중계
매매보고서 중계
결제내역 확인 메시지 중계
FIX 스타일 메시지 변환
```

Rules:

```text
STP-HUB는 개인 고객 채널이 아니다.
STP-HUB는 기관/자산운용사 주문 허브다.
```

---

## 4.10 stbase-order-routing

장내 주문 라우팅.

Entities:

```text
Order
OrderRoute
OrderValidationResult
RouteDecision
```

OrderStatus:

```text
RECEIVED
VALIDATED
REJECTED
HELD
ROUTED
ACCEPTED_BY_EXCHANGE
PARTIALLY_FILLED
FILLED
CANCELED
EXPIRED
FAILED
```

Rules:

```text
장내 주문만 Order를 사용한다.
장외채권은 Order가 아니라 TradeCapture를 사용한다.
주문 전 현금/증권 hold가 필요하다.
상태변경 API는 idempotency key 필수다.
```

---

## 4.11 stbase-sor-sim

복수거래시장 Smart Order Routing.

Responsibilities:

```text
시장별 시세 비교
주문 목적지 결정
최선집행 규칙 평가
장애 시 우선 시장 라우팅
라우팅 증적 저장
```

Entities:

```text
Venue
VenueQuote
BestExecutionRule
RoutingDecision
RoutingEvidence
```

Initial simplification:

```text
Phase 1에서는 KRX-Sim 단일 venue만 둔다.
하지만 구조는 복수 venue를 허용한다.
```

---

## 4.12 stbase-stocknet-sim

전문망/주문망/시세망 시뮬레이션.

Responsibilities:

```text
주문 전문 전달
체결 전문 전달
시세 전문 전달
청산 전문 전달
결제 전문 전달
중복 전문 시뮬레이션
지연 전문 시뮬레이션
재전송
전문 로그
```

Entities:

```text
NetworkMessage
MessageEnvelope
MessageRoute
MessageDeliveryAttempt
ExternalMessageLog
```

Rules:

```text
Cross-module market message는 StockNet-Sim을 경유한다.
전문은 correlationId를 가진다.
전문 중복 수신에 안전해야 한다.
```

---

## 4.13 stbase-exture-sim

거래소 시장시스템.

Responsibilities:

```text
시장 세션 관리
호가 접수
호가장 관리
매칭 엔진
체결 생성
체결결과 통보
시장 규칙 적용
청산 원천 데이터 생성
```

Entities:

```text
MarketSession
ExchangeOrder
OrderBook
Quote
ExecutionReport
MarketTrade
```

MarketSessionState:

```text
CLOSED
PRE_OPEN
REGULAR
CLOSING_AUCTION
AFTER_HOURS
HALTED
```

Rules:

```text
EXTURE-Sim은 고객별 원장을 모른다.
EXTURE-Sim은 회원/증권사 단위 주문을 처리한다.
```

---

## 4.14 stbase-krx-clearing-sim

청산 시뮬레이션.

Responsibilities:

```text
체결 집계
회원별 차감 계산
매수/매도 결제금액 계산
증권 인도수량 계산
결제지시 생성
```

Entities:

```text
ClearingBatch
ClearingMemberPosition
NetSettlementObligation
ClearingInstruction
```

Rules:

```text
체결과 결제는 다르다.
청산 결과가 KSD-Sim 결제지시의 근거가 된다.
```

---

## 4.15 stbase-ksd-sim

예탁결제원 시뮬레이션.

Responsibilities:

```text
전자등록 증권 관리
예탁계좌 관리
참가기관 단위 잔고 관리
계좌대체
DVP 결제
권리관리
채권 이자 지급
채권 만기 상환
배당 지급
대사 자료 제공
```

Entities:

```text
DepositoryAccount
RegisteredSecurity
DepositoryPosition
BookEntryTransfer
DvpSettlement
CorporateAction
CouponPayment
RedemptionPayment
DividendPayment
```

Rules:

```text
KSD-Sim은 고객별 내부 원장을 직접 관리하지 않는다.
KSD-Sim은 증권사/참가기관 단위 총량을 관리한다.
PowerBase-Sim 고객별 합산과 KSD-Sim 총량이 대사되어야 한다.
```

---

## 4.16 stbase-kofia-freebond-sim

장외채권 거래 전용 시스템.

Responsibilities:

```text
호가 게시
트레이딩보드
전용메신저
거래 조건 협의
장외채권 체결 후보 생성
```

Entities:

```text
BondQuote
NegotiationRoom
DealerMessage
OtcBondDealProposal
OtcBondDealAgreement
```

Rule:

```text
FreeBond-Sim은 거래 협의와 정보 집중의 역할이다.
최종 원장 반영은 PowerBase-Sim에서 한다.
```

---

## 4.17 stbase-kofia-disclosure-sim

장외채권 보고/공시.

Responsibilities:

```text
장외채권 거래보고 접수
보고 지연 감지
호가/매매정보 공시
수익률 통계
거래량 통계
보고 오류 반려
```

Entities:

```text
OtcBondTradeReport
OtcBondDisclosure
YieldStatistic
ReportingDelayViolation
```

Rules:

```text
보고 실패는 거래 실패와 별도 상태로 관리한다.
보고 지연은 compliance event를 생성한다.
```

---

## 4.18 stbase-fss-sim

금융감독원 시뮬레이션.

Responsibilities:

```text
발행공시 접수
증권신고서 더미
업무보고 접수
검사 이벤트
자료제출 요청
투자자보호 점검
불완전판매 점검
내부통제 점검
```

Entities:

```text
DisclosureDocument
BusinessReport
InspectionEvent
ComplianceAlert
DocumentSubmissionRequest
InvestorProtectionFinding
```

Rules:

```text
FSS-Sim은 매매체결기관이 아니다.
FSS-Sim은 결제기관이 아니다.
FSS-Sim은 거래 플로우 한가운데 끼지 않는다.
```

---

## 4.19 stbase-market-info-sim

시장정보.

Responsibilities:

```text
시세 조회
종목 정보 조회
시장 통계
뉴스 더미
공시 더미
```

Rule:

```text
시장정보 조회와 주문 처리는 분리한다.
```

---

## 4.20 stbase-bond-info-sim

채권정보/계산.

Responsibilities:

```text
채권 발행정보
장내/장외 가격정보
수익률 계산
경과이자 계산
clean price / dirty price 계산
채권 단가 계산
금리정보
신용등급
```

Rules:

```text
채권 계산 로직은 이 모듈 또는 별도 calculation 모듈에 둔다.
주문 서비스/컨트롤러에 계산 로직을 흩뿌리지 않는다.
```

---

## 5. Core Business Flows

---

## 5.1 Retail Exchange Stock Buy

```text
RetailChannel
→ PowerBaseCore.validateAccountAndCash
→ Ledger.holdCash
→ OrderRouting.createOrder
→ SOR.decideVenue
→ StockNet.sendOrderMessage
→ Exture.acceptOrder
→ Exture.match
→ StockNet.sendExecutionReport
→ PowerBaseCore.applyExecution
→ Clearing.createInstruction
→ Settlement.createInstruction
→ KsdSim.dvpSettlement
→ Ledger.finalizeCashAndSecurity
→ Reconciliation.comparePowerBaseAndKsd
```

---

## 5.2 Institutional Order via STP-HUB

```text
Institution
→ StpHub.receiveFixOrder
→ StpHub.routeToBroker
→ KFrontOrPowerBase.receiveInstitutionOrder
→ OrderRouting.validate
→ StockNet.send
→ Exture.match
→ StockNet.executionReport
→ StpHub.forwardExecution
→ PowerBase.ledgerAndSettlement
```

---

## 5.3 House Trading via K-FRONT

```text
Trader
→ KFront.createHouseOrder
→ OrderRouting.validateHouseAccount
→ SOR.decide
→ StockNet.send
→ Exture.match
→ PowerBase.applyHouseExecution
→ Settlement
→ Reconciliation
```

---

## 5.4 OTC Bond Buy by Retail Client

```text
RetailClient
→ RetailChannel.requestOtcBondBuy
→ PowerBase.validateSuitabilityAndCash
→ BondInventory.reserveByConditionalUpdate
→ OtcBondTrade.create
→ Ledger.createPendingPostings
→ KofiaDisclosure.reportOtcBondTrade
→ KsdSim.bookEntryTransferIfNeeded
→ Ledger.finalize
→ Reconciliation
```

---

## 5.5 Inter-Dealer OTC Bond Trade

```text
DealerA / DealerB
→ FreeBond.quoteAndNegotiate
→ OtcTradeCapture
→ PowerBase.confirmTrade
→ KofiaDisclosure.report
→ KsdSim.dvpSettlement
→ Ledger.finalizeBothSides
→ Reconciliation
```

---

## 5.6 Bond Coupon Payment

```text
BondInfo.couponSchedule
→ KsdSim.createCouponCorporateAction
→ PowerBase.createRecordDateSnapshot
→ Ledger.postCouponCash
→ Tax.postWithholding
→ Reconciliation
```

---

## 5.7 Bond Redemption

```text
BondInfo.maturityDateArrived
→ KsdSim.redemptionEvent
→ PowerBase.calculateHolderAmounts
→ Ledger.securityDecrease
→ Ledger.cashIncrease
→ SecurityPosition.close
→ Reconciliation
```

---

## 5.8 Futures Trade and Daily Settlement

```text
KFront.createFuturesOrder
→ OrderRouting.checkMargin
→ StockNet.send
→ ExtureDerivatives.match
→ PowerBase.createOpenInterest
→ Ledger.holdInitialMargin
→ Batch.dailySettlement
→ Ledger.postVariationMargin
→ Risk.checkMarginCall
```

---

## 5.9 RP Start and End Leg

```text
RpContract.create
→ Settlement.createStartLeg
→ Ledger.cashTransfer
→ Ledger.collateralLockOrTransfer
→ Accrual.calculateRepoInterest
→ Settlement.createEndLeg
→ Ledger.returnCashPlusInterest
→ Ledger.releaseCollateral
→ Reconciliation
```

---

## 6. Ledger Rules

### 6.1 Source of Truth

```text
LedgerPosting is source of truth.
BalanceSnapshot is projection.
```

### 6.2 Immutability

```text
Posting cannot be updated.
Posting cannot be deleted.
Correction requires new posting.
Reversal requires reversal posting.
```

### 6.3 Balance Dimensions

Cash:

```text
settled_cash
available_cash
held_cash
pending_receivable
pending_payable
```

Security:

```text
settled_quantity
available_quantity
held_quantity
pending_in_quantity
pending_out_quantity
```

### 6.4 Prohibited Code

```java
// forbidden
balance.setCash(balance.getCash().subtract(amount));
accountBalanceRepository.save(balance);
```

Required approach:

```text
Create LedgerJournal
Create LedgerPosting
Update projection through ledger processor
Write audit log
```

---

## 7. Idempotency Rules

모든 상태변경 API는 idempotency key를 요구합니다.

Examples:

```text
POST /orders
POST /otc/bond-trades
POST /settlements
POST /ledger/journals
POST /ksd-sim/dvp-settlements
POST /kofia-sim/otc-bond-reports
POST /fss-sim/business-reports
```

Storage:

```text
idempotency_keys
- key
- request_hash
- response_snapshot
- status
- created_at
- expired_at
```

Rules:

```text
Same key + same request = same response
Same key + different request = reject
No key for command = reject
```

---

## 8. Outbox and Saga

### 8.1 Outbox

When local DB state changes and another module must be called:

```text
@Transactional
1. save local aggregate
2. save ledger posting if needed
3. save outbox event
```

Then relay:

```text
1. read outbox
2. call target module
3. write external_message_log
4. mark outbox sent
```

### 8.2 Saga Example: Settlement

```text
TRADE_CONFIRMED
→ SETTLEMENT_INSTRUCTION_CREATED
→ KSD_INSTRUCTION_SENT
→ KSD_ACCEPTED
→ DVP_COMPLETED
→ LEDGER_FINALIZED
→ RECONCILED
```

Failure states:

```text
KSD_REJECTED
KSD_TIMEOUT
SETTLEMENT_FAILED
RETRY_REQUIRED
MANUAL_REVIEW_REQUIRED
```

---

## 9. External Message Log

Even though all institutions are internal modules, they must be treated like external systems.

```text
external_message_logs
- id
- source_system
- target_system
- message_type
- correlation_id
- idempotency_key
- request_payload
- response_payload
- status
- retry_count
- error_code
- error_message
- created_at
- updated_at
```

---

## 10. Reconciliation

대사는 필수입니다.

Types:

```text
PowerBase customer security total vs KSD-Sim participant position
PowerBase cash ledger vs settlement result
Order/execution/trade consistency
Trade vs KOFIA report
Settlement instruction vs KSD result
FSS business report vs internal aggregate
Ledger posting vs balance snapshot
```

Result states:

```text
MATCHED
BREAK_FOUND
MANUAL_REVIEW_REQUIRED
RESOLVED
```

Rule:

```text
대사 불일치를 자동 수정하지 않는다.
대사 불일치는 exception으로 남기고 운영자 검토 대상으로 보낸다.
```

---

## 11. API Style

Command response:

```json
{
  "result": "SUCCESS",
  "data": {},
  "traceId": "TRACE-20260513-000001",
  "timestamp": "2026-05-13T12:00:00+09:00"
}
```

Error response:

```json
{
  "result": "ERROR",
  "errorCode": "INSUFFICIENT_CASH",
  "message": "Available cash is insufficient.",
  "traceId": "TRACE-20260513-000001",
  "timestamp": "2026-05-13T12:00:00+09:00"
}
```

Rules:

```text
모든 command API는 명시적 errorCode 사용
stack trace 외부 노출 금지
correlationId 전파
traceId 응답 포함
```

---

## 12. Package Structure

Example:

```text
com.stbase.order
├── api
├── application
├── domain
├── infrastructure
├── persistence
└── support
```

Layer rule:

```text
api → application → domain
infrastructure implements ports
domain must not depend on Spring
```

Recommended classes:

```text
Controller
Command
CommandHandler
QueryService
DomainService
Aggregate
ValueObject
RepositoryPort
JpaRepositoryAdapter
DomainEvent
```

---

## 13. Coding Standards

```text
Java 17+
Constructor injection only
No field injection
No business logic in Controller
No direct Entity exposure from API
Use DTOs
Use BigDecimal for money/rate/price
Use LocalDate for tradeDate/settlementDate
Use OffsetDateTime for event timestamp
Use enum for business status
Use @Version for optimistic locking
Use conditional update for scarce inventory
```

Forbidden:

```text
double for money
float for rate
String status
direct balance update
delete ledger posting
silent catch
cross-module DB access
```

---

## 14. Financial Calculation Guidance

Financial calculations must be isolated.

Bond:

```text
clean price
accrued interest
dirty price
yield
settlement amount
day count convention
coupon schedule
```

Futures:

```text
contract multiplier
entry price
settlement price
initial margin
maintenance margin
variation margin
```

RP:

```text
repo rate
principal
days
repo interest
haircut
collateral value
start leg
end leg
```

Place calculation logic in:

```text
stbase-bond-info-sim
stbase-product
or stbase-calculation if separated
```

---

## 15. Test Priority

AI must prioritize these tests.

```text
1. Same idempotency key creates one order only.
2. Duplicate execution report creates one execution only.
3. StockNet duplicate message does not duplicate ledger posting.
4. OTC bond inventory cannot be oversold under concurrency.
5. Cash cannot become negative unless margin account allows it.
6. Security cannot become negative unless short-selling simulation explicitly allows it.
7. Ledger posting cannot be updated.
8. Ledger posting cannot be deleted.
9. Reversal posting offsets original posting.
10. Settlement failure remains retryable.
11. KSD-Sim total position mismatch is detected by reconciliation.
12. KOFIA report failure does not erase trade.
13. STP-HUB routes institution order to correct broker.
14. K-FRONT house order posts to HOUSE account, not retail account.
15. FSS-Sim does not participate in execution/settlement flow.
16. Bond coupon payment uses record-date snapshot.
17. Bond redemption reduces position to zero.
18. Futures daily settlement posts variation margin.
19. RP end leg releases collateral.
20. Market closed state rejects new exchange orders.
```

---

## 16. Implementation Order

### Step 1 - Core

```text
stbase-common
stbase-member
stbase-account
stbase-product
stbase-ledger
idempotency
audit log
```

### Step 2 - OTC Bond First

```text
bond master
bond inventory
OTC bond trade capture
ledger pending/settled posting
KOFIA disclosure sim
KSD position sim
reconciliation
```

### Step 3 - StockNet and EXTURE MVP

```text
stocknet message envelope
market session
order book
matching engine
execution report
order state transition
```

### Step 4 - Channel Separation

```text
retail channel
branch terminal
K-FRONT
STP-HUB
```

### Step 5 - Settlement Hardening

```text
clearing
settlement instruction
KSD DVP
failure/retry
reconciliation breaks
```

### Step 6 - Rights and Batch

```text
coupon payment
redemption
dividend
daily close
business reports
```

### Step 7 - SOR and Multi-Venue

```text
ATS-Sim
SOR decision
best execution evidence
venue failover
```

### Step 8 - Futures/RP

```text
futures margin
daily settlement
RP start/end leg
collateral valuation
```

---

## 17. Definition of Done

A feature is done only when:

```text
Domain model exists
Command API exists
Validation exists
Idempotency exists
Ledger impact is defined
Settlement/reporting impact is defined
Audit log exists
Failure state exists
Tests exist
Reconciliation impact is considered
Docs are updated
No real external integration is introduced
```

---

## 18. AI Modification Rules

When AI modifies code:

```text
Read this document first.
Identify affected module.
Do not touch unrelated modules.
Do not merge PowerBase, K-FRONT, STP-HUB, StockNet, EXTURE responsibilities.
Never bypass ledger.
Never bypass idempotency.
Never bypass audit log.
Add tests.
Add migration if DB schema changes.
Update docs if business flow changes.
```

If uncertain:

```text
Prefer simulation.
Prefer smaller change.
Prefer explicit state.
Prefer auditability.
Prefer reconciliation.
Prefer safe failure.
```

---

## 19. One-Sentence Summary for AI

STBase is a simulated securities operations platform that separates PowerBase-style brokerage core processing, K-FRONT/STP-HUB front-office flows, StockNet-style message transport, EXTURE-style exchange matching, KSD-style settlement, KOFIA-style OTC bond disclosure, and FSS-style supervision, while preserving ledger integrity, idempotency, auditability, and reconciliation.
