# STBase AI Development Overview

> ??臾몄꽌??AI 媛쒕컻 ?꾧뎄媛 STBase ?꾨줈?앺듃瑜?援ы쁽????諛섎뱶???곕씪???섎뒗 媛쒕컻 吏?쒖꽌?낅땲??  
> STBase???⑥닚 二쇰Ц API ?꾨줈?앺듃媛 ?꾨땲?? PowerBase 援ъ“瑜?紐⑦떚釉뚮줈 ??**紐⑥쓽 利앷텒?낅Т怨?+ 紐⑥쓽 ?먮낯?쒖옣 ?명봽??*?낅땲??

---

## 1. Project Identity

```text
Project Name: STBase
Full Name: Securities Trade Base
Main Stack: Java, Spring Boot, JPA, MySQL, Gradle Multi Module
Domain: Securities operations simulation
```

STBase???ㅼ쓬??援ы쁽?⑸땲??

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

STBase???ㅼ쓬??援ы쁽?섏? ?딆뒿?덈떎.

```text
?ㅼ젣 嫄곕옒??二쇰Ц
?ㅼ젣 利앷텒??API ?곕룞
?ㅼ젣 ?덊긽寃곗젣???꾨Ц
?ㅼ젣 湲덇컧??蹂닿퀬
?ㅼ젣 湲덉쑖?ъ옄?묓쉶 蹂닿퀬
?ㅼ젣 怨좉컼 ?먯궛 泥섎━
?ㅼ젣 ?ъ옄/留ㅻℓ ?ㅽ뻾
```

---

## 2. Absolute Safety Rules

AI???꾨옒 洹쒖튃???덈? ?꾨컲?섎㈃ ???⑸땲??

```text
1. ?ㅼ젣 湲덉쑖湲곌? API ?곕룞 肄붾뱶瑜?留뚮뱾吏 ?딅뒗??
2. ?ㅼ젣 二쇰Ц 媛?μ꽦???붿떆?섎뒗 broker adapter瑜?留뚮뱾吏 ?딅뒗??
3. ?ㅼ젣 API key, secret, certificate, account ?곕룞 援ъ“瑜?留뚮뱾吏 ?딅뒗??
4. 紐⑤뱺 湲곌? 紐⑤뱢? ?대? simulation module濡쒕쭔 ?붾떎.
5. ?ㅺ굅??媛??肄붾뱶? 紐⑥쓽嫄곕옒 肄붾뱶瑜??욎? ?딅뒗??
6. ?먯옣 posting???섏젙/??젣?섎뒗 API瑜?留뚮뱾吏 ?딅뒗??
7. balance瑜?吏곸젒 update?섎뒗 鍮꾩쫰?덉뒪 肄붾뱶瑜?留뚮뱾吏 ?딅뒗??
8. 以묐났 ?붿껌??以묐났 二쇰Ц/泥닿껐/?먯옣諛섏쁺?쇰줈 ?댁뼱吏硫????쒕떎.
9. ?몃?湲곌? ?쒕??덉씠???몄텧? 諛섎뱶??濡쒓렇? ?ъ쿂由?援ъ“瑜?媛뽯뒗??
10. ?뺥빀?깅낫???몄쓽?깆쓣 ?곗꽑?섏? ?딅뒗??
```

---

## 3. Correct Mental Model

STBase??以묒떖? ?쒖＜臾멤앹씠 ?꾨땲???쒖쬆沅뚯궗 ?낅Т怨??먯옣?앹엯?덈떎.

?뺥솗???낅Т ?먮쫫:

```text
梨꾨꼸/?꾨줎??
??二쇰Ц ?먮뒗 嫄곕옒 ?낅젰
??PowerBase-Sim 寃利?
??二쇰Ц ?쇱슦???먮뒗 ?μ쇅嫄곕옒 ?뺤젙
??StockNet-Sim ?꾨Ц ?꾨떖
??EXTURE-Sim 泥닿껐
??Clearing
??KSD-Sim 寃곗젣
??PowerBase-Sim ?먯옣 諛섏쁺
??KOFIA/FSS 蹂닿퀬
?????
??媛먯궗
```

AI??紐⑤뱺 湲곕뒫??援ы쁽?????꾨옒 吏덈Ц??癒쇱? ?댁빞 ?⑸땲??

```text
??湲곕뒫? ?대뼡 嫄곕옒二쇱껜?????寃껋씤媛?
?λ궡嫄곕옒?멸? ?μ쇅嫄곕옒?멸??
二쇰Ц 湲곕컲?멸? 嫄곕옒?낅젰 湲곕컲?멸??
PowerBase ?먯옣???대뼡 ?곹뼢??二쇰뒗媛?
KSD-Sim 珥앸웾怨????媛?ν븳媛?
KOFIA/FSS 蹂닿퀬 ??곸씤媛?
以묐났 ?붿껌???덉쟾?쒓??
?ㅽ뙣 ???ъ쿂由?媛?ν븳媛?
媛먯궗 濡쒓렇媛 ?⑤뒗媛?
```

---

## 4. Module Boundaries

---

## 4.1 stbase-common

怨듯넻 ??낆쓣 ?〓땲??

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

洹쒖튃:

```text
Money/Rate/Price?먮뒗 double/float 湲덉?
BigDecimal ?ъ슜
?꾨찓????낆? 紐낆떆?곸쑝濡?留뚮뱺??
```

---

## 4.2 stbase-core

利앷텒???낅Т怨?以묒떖 紐⑤뱢?낅땲??

Responsibilities:

```text
怨좉컼 ?먯옣 以묒떖 ?낅Т?먮쫫 議곗젙
怨꾩쥖 ?곹깭 寃利?
嫄곕옒 媛???щ? 寃利?
二쇰Ц 寃곌낵 諛섏쁺
嫄곕옒 寃곌낵 諛섏쁺
寃곗젣 ?덉젙 ?앹꽦
寃곗젣 寃곌낵 諛섏쁺
蹂닿퀬 ?대깽???앹꽦
????대깽???앹꽦
```

Anti-responsibilities:

```text
?멸???愿由?湲덉?
留ㅼ묶 ?붿쭊 援ы쁽 湲덉?
KSD ?붽퀬 吏곸젒 議곗옉 湲덉?
KOFIA/FSS ?곗씠??吏곸젒 ?섏젙 湲덉?
```

---

## 4.3 stbase-member

嫄곕옒二쇱껜 愿由?

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
媛쒖씤/湲곌?/利앷텒??HOUSE 怨꾩젙? 諛섎뱶??援щ텇?쒕떎.
嫄곕옒二쇱껜 ?좏삎???곕씪 ?덉슜 嫄곕옒媛 ?щ씪吏꾨떎.
```

---

## 4.4 stbase-account

怨꾩쥖 愿由?

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
?뺤? 怨꾩쥖???좉퇋 二쇰Ц 遺덇?
?먯뇙 怨꾩쥖???좉퇋 posting 遺덇?
?꾪긽怨꾩쥖? ?먭린怨꾩젙? 遺꾨━
KSD 怨꾩쥖? PowerBase 怨좉컼怨꾩쥖??蹂꾨룄
```

---

## 4.5 stbase-product

?곹뭹留덉뒪??

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

媛쒖씤 怨좉컼 ?붿껌???쒕??덉씠?섑빀?덈떎.

Responsibilities:

```text
媛쒖씤 二쇰Ц ?붿껌
?붽퀬 議고쉶
泥닿껐 議고쉶
?곹뭹 議고쉶
嫄곕옒?댁뿭 議고쉶
```

Rule:

```text
Retail Channel? ?먯옣??吏곸젒 ?섏젙?섏? ?딅뒗??
```

---

## 4.7 stbase-branch-terminal-sim

?곸뾽??吏곸썝 ?⑤쭚 ?쒕??덉씠??

Responsibilities:

```text
吏곸썝 ???二쇰Ц
怨좉컼 怨꾩쥖 議고쉶
?ъ옄?깊뼢 ?뺤씤
梨꾧텒 ?먮ℓ ?낅젰
?댁쁺??議고쉶
```

Rule:

```text
吏곸썝 ?됱쐞??audit log??operatorId? ?④퍡 ?④릿??
```

---

## 4.8 stbase-kfront-sim

?꾨Ц ?몃젅?대뜑/踰뺤씤?곸뾽/?먭린留ㅻℓ OMS ?쒕??덉씠??

Responsibilities:

```text
?꾨Ц 二쇰Ц ?낅젰
?먭린留ㅻℓ 二쇰Ц
諛붿뒪耳?二쇰Ц
?뚭퀬由ъ쬁 二쇰Ц ?붾?
FIX ?ㅽ???二쇰Ц 硫붿떆吏 ?앹꽦
?쒖옣?곗씠??湲곕컲 二쇰Ц ?먮떒 ?붾?
```

Rules:

```text
K-FRONT???먯옣??吏곸젒 ?섏젙?섏? ?딅뒗??
K-FRONT??二쇰Ц/嫄곕옒 ?섏궗? 硫붿떆吏留??앹꽦?쒕떎.
```

---

## 4.9 stbase-stp-hub-sim

湲곌??ъ옄媛? 利앷텒???ъ씠???덈툕.

Responsibilities:

```text
Buy-side 二쇰Ц 硫붿떆吏 ?섏떊
Sell-side濡??쇱슦??
泥닿껐寃곌낵 以묎퀎
留ㅻℓ蹂닿퀬??以묎퀎
寃곗젣?댁뿭 ?뺤씤 硫붿떆吏 以묎퀎
FIX ?ㅽ???硫붿떆吏 蹂??
```

Rules:

```text
STP-HUB??媛쒖씤 怨좉컼 梨꾨꼸???꾨땲??
STP-HUB??湲곌?/?먯궛?댁슜??二쇰Ц ?덈툕??
```

---

## 4.10 stbase-order-routing

?λ궡 二쇰Ц ?쇱슦??

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
?λ궡 二쇰Ц留?Order瑜??ъ슜?쒕떎.
?μ쇅梨꾧텒? Order媛 ?꾨땲??TradeCapture瑜??ъ슜?쒕떎.
二쇰Ц ???꾧툑/利앷텒 hold媛 ?꾩슂?섎떎.
?곹깭蹂寃?API??idempotency key ?꾩닔??
```

---

## 4.11 stbase-sor-sim

蹂듭닔嫄곕옒?쒖옣 Smart Order Routing.

Responsibilities:

```text
?쒖옣蹂??쒖꽭 鍮꾧탳
二쇰Ц 紐⑹쟻吏 寃곗젙
理쒖꽑吏묓뻾 洹쒖튃 ?됯?
?μ븷 ???곗꽑 ?쒖옣 ?쇱슦??
?쇱슦??利앹쟻 ???
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
Phase 1?먯꽌??KRX-Sim ?⑥씪 venue留??붾떎.
?섏?留?援ъ“??蹂듭닔 venue瑜??덉슜?쒕떎.
```

---

## 4.12 stbase-stocknet-sim

?꾨Ц留?二쇰Ц留??쒖꽭留??쒕??덉씠??

Responsibilities:

```text
二쇰Ц ?꾨Ц ?꾨떖
泥닿껐 ?꾨Ц ?꾨떖
?쒖꽭 ?꾨Ц ?꾨떖
泥?궛 ?꾨Ц ?꾨떖
寃곗젣 ?꾨Ц ?꾨떖
以묐났 ?꾨Ц ?쒕??덉씠??
吏???꾨Ц ?쒕??덉씠??
?ъ쟾??
?꾨Ц 濡쒓렇
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
Cross-module market message??StockNet-Sim??寃쎌쑀?쒕떎.
?꾨Ц? correlationId瑜?媛吏꾨떎.
?꾨Ц 以묐났 ?섏떊???덉쟾?댁빞 ?쒕떎.
```

---

## 4.13 stbase-exture-sim

嫄곕옒???쒖옣?쒖뒪??

Responsibilities:

```text
?쒖옣 ?몄뀡 愿由?
?멸? ?묒닔
?멸???愿由?
留ㅼ묶 ?붿쭊
泥닿껐 ?앹꽦
泥닿껐寃곌낵 ?듬낫
?쒖옣 洹쒖튃 ?곸슜
泥?궛 ?먯쿇 ?곗씠???앹꽦
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
EXTURE-Sim? 怨좉컼蹂??먯옣??紐⑤Ⅸ??
EXTURE-Sim? ?뚯썝/利앷텒???⑥쐞 二쇰Ц??泥섎━?쒕떎.
```

---

## 4.14 stbase-krx-clearing-sim

泥?궛 ?쒕??덉씠??

Responsibilities:

```text
泥닿껐 吏묎퀎
?뚯썝蹂?李④컧 怨꾩궛
留ㅼ닔/留ㅻ룄 寃곗젣湲덉븸 怨꾩궛
利앷텒 ?몃룄?섎웾 怨꾩궛
寃곗젣吏???앹꽦
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
泥닿껐怨?寃곗젣???ㅻⅤ??
泥?궛 寃곌낵媛 KSD-Sim 寃곗젣吏?쒖쓽 洹쇨굅媛 ?쒕떎.
```

---

## 4.15 stbase-ksd-sim

?덊긽寃곗젣???쒕??덉씠??

Responsibilities:

```text
?꾩옄?깅줉 利앷텒 愿由?
?덊긽怨꾩쥖 愿由?
李멸?湲곌? ?⑥쐞 ?붽퀬 愿由?
怨꾩쥖?泥?
DVP 寃곗젣
沅뚮━愿由?
梨꾧텒 ?댁옄 吏湲?
梨꾧텒 留뚭린 ?곹솚
諛곕떦 吏湲?
????먮즺 ?쒓났
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
KSD-Sim? 怨좉컼蹂??대? ?먯옣??吏곸젒 愿由ы븯吏 ?딅뒗??
KSD-Sim? 利앷텒??李멸?湲곌? ?⑥쐞 珥앸웾??愿由ы븳??
PowerBase-Sim 怨좉컼蹂??⑹궛怨?KSD-Sim 珥앸웾????щ릺?댁빞 ?쒕떎.
```

---

## 4.16 stbase-kofia-freebond-sim

?μ쇅梨꾧텒 嫄곕옒 ?꾩슜 ?쒖뒪??

Responsibilities:

```text
?멸? 寃뚯떆
?몃젅?대뵫蹂대뱶
?꾩슜硫붿떊?
嫄곕옒 議곌굔 ?묒쓽
?μ쇅梨꾧텒 泥닿껐 ?꾨낫 ?앹꽦
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
FreeBond-Sim? 嫄곕옒 ?묒쓽? ?뺣낫 吏묒쨷????븷?대떎.
理쒖쥌 ?먯옣 諛섏쁺? PowerBase-Sim?먯꽌 ?쒕떎.
```

---

## 4.17 stbase-kofia-disclosure-sim

?μ쇅梨꾧텒 蹂닿퀬/怨듭떆.

Responsibilities:

```text
?μ쇅梨꾧텒 嫄곕옒蹂닿퀬 ?묒닔
蹂닿퀬 吏??媛먯?
?멸?/留ㅻℓ?뺣낫 怨듭떆
?섏씡瑜??듦퀎
嫄곕옒???듦퀎
蹂닿퀬 ?ㅻ쪟 諛섎젮
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
蹂닿퀬 ?ㅽ뙣??嫄곕옒 ?ㅽ뙣? 蹂꾨룄 ?곹깭濡?愿由ы븳??
蹂닿퀬 吏?곗? compliance event瑜??앹꽦?쒕떎.
```

---

## 4.18 stbase-fss-sim

湲덉쑖媛먮룆???쒕??덉씠??

Responsibilities:

```text
諛쒗뻾怨듭떆 ?묒닔
利앷텒?좉퀬???붾?
?낅Т蹂닿퀬 ?묒닔
寃???대깽??
?먮즺?쒖텧 ?붿껌
?ъ옄?먮낫???먭?
遺덉셿?꾪뙋留??먭?
?대??듭젣 ?먭?
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
FSS-Sim? 留ㅻℓ泥닿껐湲곌????꾨땲??
FSS-Sim? 寃곗젣湲곌????꾨땲??
FSS-Sim? 嫄곕옒 ?뚮줈???쒓??대뜲 ?쇱? ?딅뒗??
```

---

## 4.19 stbase-market-info-sim

?쒖옣?뺣낫.

Responsibilities:

```text
?쒖꽭 議고쉶
醫낅ぉ ?뺣낫 議고쉶
?쒖옣 ?듦퀎
?댁뒪 ?붾?
怨듭떆 ?붾?
```

Rule:

```text
?쒖옣?뺣낫 議고쉶? 二쇰Ц 泥섎━??遺꾨━?쒕떎.
```

---

## 4.20 stbase-bond-info-sim

梨꾧텒?뺣낫/怨꾩궛.

Responsibilities:

```text
梨꾧텒 諛쒗뻾?뺣낫
?λ궡/?μ쇅 媛寃⑹젙蹂?
?섏씡瑜?怨꾩궛
寃쎄낵?댁옄 怨꾩궛
clean price / dirty price 怨꾩궛
梨꾧텒 ?④? 怨꾩궛
湲덈━?뺣낫
?좎슜?깃툒
```

Rules:

```text
梨꾧텒 怨꾩궛 濡쒖쭅? ??紐⑤뱢 ?먮뒗 蹂꾨룄 calculation 紐⑤뱢???붾떎.
二쇰Ц ?쒕퉬??而⑦듃濡ㅻ윭??怨꾩궛 濡쒖쭅???⑸퓣由ъ? ?딅뒗??
```

---

## 5. Core Business Flows

---

## 5.1 Retail Exchange Stock Buy

```text
RetailChannel
??PowerBaseCore.validateAccountAndCash
??Ledger.holdCash
??OrderRouting.createOrder
??SOR.decideVenue
??StockNet.sendOrderMessage
??Exture.acceptOrder
??Exture.match
??StockNet.sendExecutionReport
??PowerBaseCore.applyExecution
??Clearing.createInstruction
??Settlement.createInstruction
??KsdSim.dvpSettlement
??Ledger.finalizeCashAndSecurity
??Reconciliation.comparePowerBaseAndKsd
```

---

## 5.2 Institutional Order via STP-HUB

```text
Institution
??StpHub.receiveFixOrder
??StpHub.routeToBroker
??KFrontOrPowerBase.receiveInstitutionOrder
??OrderRouting.validate
??StockNet.send
??Exture.match
??StockNet.executionReport
??StpHub.forwardExecution
??PowerBase.ledgerAndSettlement
```

---

## 5.3 House Trading via K-FRONT

```text
Trader
??KFront.createHouseOrder
??OrderRouting.validateHouseAccount
??SOR.decide
??StockNet.send
??Exture.match
??PowerBase.applyHouseExecution
??Settlement
??Reconciliation
```

---

## 5.4 OTC Bond Buy by Retail Client

```text
RetailClient
??RetailChannel.requestOtcBondBuy
??PowerBase.validateSuitabilityAndCash
??BondInventory.reserveByConditionalUpdate
??OtcBondTrade.create
??Ledger.createPendingPostings
??KofiaDisclosure.reportOtcBondTrade
??KsdSim.bookEntryTransferIfNeeded
??Ledger.finalize
??Reconciliation
```

---

## 5.5 Inter-Dealer OTC Bond Trade

```text
DealerA / DealerB
??FreeBond.quoteAndNegotiate
??OtcTradeCapture
??PowerBase.confirmTrade
??KofiaDisclosure.report
??KsdSim.dvpSettlement
??Ledger.finalizeBothSides
??Reconciliation
```

---

## 5.6 Bond Coupon Payment

```text
BondInfo.couponSchedule
??KsdSim.createCouponCorporateAction
??PowerBase.createRecordDateSnapshot
??Ledger.postCouponCash
??Tax.postWithholding
??Reconciliation
```

---

## 5.7 Bond Redemption

```text
BondInfo.maturityDateArrived
??KsdSim.redemptionEvent
??PowerBase.calculateHolderAmounts
??Ledger.securityDecrease
??Ledger.cashIncrease
??SecurityPosition.close
??Reconciliation
```

---

## 5.8 Futures Trade and Daily Settlement

```text
KFront.createFuturesOrder
??OrderRouting.checkMargin
??StockNet.send
??ExtureDerivatives.match
??PowerBase.createOpenInterest
??Ledger.holdInitialMargin
??Batch.dailySettlement
??Ledger.postVariationMargin
??Risk.checkMarginCall
```

---

## 5.9 RP Start and End Leg

```text
RpContract.create
??Settlement.createStartLeg
??Ledger.cashTransfer
??Ledger.collateralLockOrTransfer
??Accrual.calculateRepoInterest
??Settlement.createEndLeg
??Ledger.returnCashPlusInterest
??Ledger.releaseCollateral
??Reconciliation
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

紐⑤뱺 ?곹깭蹂寃?API??idempotency key瑜??붽뎄?⑸땲??

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
??SETTLEMENT_INSTRUCTION_CREATED
??KSD_INSTRUCTION_SENT
??KSD_ACCEPTED
??DVP_COMPLETED
??LEDGER_FINALIZED
??RECONCILED
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

??щ뒗 ?꾩닔?낅땲??

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
???遺덉씪移섎? ?먮룞 ?섏젙?섏? ?딅뒗??
???遺덉씪移섎뒗 exception?쇰줈 ?④린怨??댁쁺??寃????곸쑝濡?蹂대궦??
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
紐⑤뱺 command API??紐낆떆??errorCode ?ъ슜
stack trace ?몃? ?몄텧 湲덉?
correlationId ?꾪뙆
traceId ?묐떟 ?ы븿
```

---

## 12. Package Structure

Example:

```text
com.stbase.order
?쒋?? api
?쒋?? application
?쒋?? domain
?쒋?? infrastructure
?쒋?? persistence
?붴?? support
```

Layer rule:

```text
api ??application ??domain
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

