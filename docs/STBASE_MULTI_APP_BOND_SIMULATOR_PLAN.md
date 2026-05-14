# STBase Multi-App Bond Simulator Plan

> 이 문서는 STBase를 AI 바이브코딩으로 구현할 때 따라야 할 기획/개발 기준서입니다.
>
> 목표는 단순 주문 API가 아니라, **외부 증권사와 유관기관이 API로 통신하며 장내/장외 채권거래, 원장, 결제, 보고, 민원, 운영 재처리까지 경험할 수 있는 모의 증권업무 시뮬레이터**를 만드는 것입니다.

---

## 1. 개발 방향 결론

STBase는 초기부터 단일 모놀리스가 아니라, **여러 Spring Boot 앱이 별도 서버로 실행되는 API형 시뮬레이터**로 개발합니다.

단, 운영용 마이크로서비스처럼 복잡하게 시작하지 않습니다.

```text
권장 방식:
Gradle multi-project
+ 여러 Spring Boot application
+ 공통 library module
+ Docker Compose 로컬 실행
+ HTTP REST API 통신
+ Outbox 기반 재처리
+ External Message Log 기반 증적
```

초기에는 Kafka/RabbitMQ를 사용하지 않습니다.

```text
초기 통신 방식:
HTTP REST
+ idempotency key
+ correlation id
+ external message log
+ outbox relay
+ retry status
```

나중에 필요하면 outbox relay의 전송 방식을 HTTP에서 Kafka/RabbitMQ로 바꿀 수 있게 설계합니다.

---

## 2. 이 방식이 필요한 이유

이 프로젝트의 핵심 학습 대상은 단순 매수/매도 로직이 아닙니다.

핵심은 다음입니다.

```text
기관 간 경계
API 호출 실패
응답 지연
중복 전문
결제 실패
보고 실패
대사 불일치
운영자 재처리
고객 민원 추적
감사 증적
```

같은 JVM 내부 메서드 호출로 만들면 위 상황을 제대로 재현하기 어렵습니다.

별도 서버 API형 시뮬레이터로 만들면 다음 상황을 현실적으로 구현할 수 있습니다.

```text
거래는 확정됐지만 KOFIA 보고가 실패함
보고는 성공했지만 KSD 결제가 실패함
KSD 결제는 완료됐지만 PowerBase 내부 원장과 총량이 안 맞음
외부 증권사가 동일 거래 요청을 중복 전송함
운영자가 재처리했지만 중복 posting은 발생하면 안 됨
고객 민원이 들어와서 거래, 보고, 결제, 원장, 감사로그를 추적해야 함
FSS-Sim이 민원 관련 자료제출을 요청함
```

따라서 STBase는 **학습용 분산 증권업무 시뮬레이터**로 간주합니다.

---

## 3. 우선 구현 범위

처음부터 주식, 선물, RP, 복수거래시장까지 구현하지 않습니다.

우선순위는 다음입니다.

```text
1. 장외채권 거래
2. 채권 재고/상품/가격/수익률 계산
3. PowerBase 원장 처리
4. KOFIA 장외채권 거래보고
5. KSD 계좌대체/DVP 결제
6. 대사
7. 고객 민원 접수/처리
8. 운영자 재처리/감사로그
9. 장내채권 거래소 시뮬레이션
```

주식은 초기 범위에서 제외합니다.

```text
초기 제외:
주식 주문
주식 호가장
주식 체결
선물
RP
ATS/SOR
실제 금융기관 API 연동
실제 계좌/자산 처리
```

---

## 4. 전체 앱 구성

초기 권장 앱 구성은 다음입니다.

```text
apps/
  powerbase-app
  external-broker-app
  freebond-app
  ksd-app
  kofia-app
  fss-app
  admin-app
  bond-exchange-app

libs/
  stbase-common
  stbase-api-contracts
  stbase-test-fixtures

infra/
  docker-compose.yml
```

### 4.1 powerbase-app

증권사 업무계 중심 앱입니다.

Responsibilities:

```text
고객 관리
계좌 관리
채권 상품 관리
고객별 채권 잔고 관리
현금 원장 관리
채권 원장 관리
장외채권 매수/매도 거래 확정
외부 증권사 거래 요청 수신
KOFIA 보고 요청 생성
KSD 결제 요청 생성
대사 작업 생성
민원 접수 및 처리
운영자 재처리 command
감사로그 기록
```

Forbidden:

```text
KSD-Sim DB 직접 조회/수정 금지
KOFIA-Sim DB 직접 조회/수정 금지
FSS-Sim DB 직접 조회/수정 금지
ledger posting 수정/삭제 금지
balance 직접 update 금지
실제 금융기관 API 연동 금지
```

### 4.2 external-broker-app

외부 증권사 시뮬레이터입니다.

실제 외부 증권사가 STBase로 채권 거래를 요청하는 상황을 재현합니다.

Responsibilities:

```text
외부 증권사 계정 시뮬레이션
장외채권 매수/매도 요청 생성
거래 상태 조회
결제 상태 조회
보고 상태 조회
민원 또는 정정 요청 생성
중복 요청/지연 요청/오류 요청 시뮬레이션
```

Rule:

```text
external-broker-app은 powerbase-app의 DB를 절대 직접 보지 않는다.
모든 요청은 공개 API로만 수행한다.
```

### 4.3 freebond-app

장외채권 호가, 협의, 딜 후보 생성을 담당합니다.

Responsibilities:

```text
장외채권 호가 게시
딜러 간 협의
메신저 스타일 협의 로그
거래 조건 합의
OtcBondDealAgreement 생성
PowerBase 거래입력 요청
```

Rule:

```text
freebond-app은 원장을 수정하지 않는다.
freebond-app은 최종 결제를 처리하지 않는다.
```

### 4.4 ksd-app

예탁결제원 시뮬레이터입니다.

Responsibilities:

```text
참가기관 계좌 관리
증권사 단위 채권 총량 관리
계좌대체
DVP 결제
결제 승인/거절/지연 시뮬레이션
권리관리 이벤트
대사 자료 제공
```

Rule:

```text
ksd-app은 고객별 내부 원장을 관리하지 않는다.
ksd-app은 증권사/참가기관 단위 총량만 관리한다.
```

### 4.5 kofia-app

금융투자협회 장외채권 보고/공시 시뮬레이터입니다.

Responsibilities:

```text
장외채권 거래보고 수신
보고 승인/반려
보고 지연 감지
호가/매매정보 공시
수익률 통계
거래량 통계
보고 오류 이벤트 생성
```

Rule:

```text
보고 실패는 거래 실패와 별도 상태로 관리한다.
보고 반려가 발생해도 원장 posting을 자동 취소하지 않는다.
```

### 4.6 fss-app

금융감독원/감독/민원 시뮬레이터입니다.

Responsibilities:

```text
민원 접수
민원 이관
자료제출 요청
검사 이벤트 생성
투자자보호 점검
불완전판매 의심 점검
내부통제 위반 점검
제재 이벤트 시뮬레이션
```

Rule:

```text
fss-app은 주문을 체결하지 않는다.
fss-app은 결제를 처리하지 않는다.
fss-app은 거래 플로우 중간에 직접 개입하지 않는다.
```

### 4.7 admin-app

운영자 콘솔 API입니다.

Responsibilities:

```text
거래 상태 조회
결제 상태 조회
보고 상태 조회
대사 break 조회
민원 처리
outbox 재처리
external message 재전송
운영자 correction command 실행
감사로그 조회
```

Forbidden:

```text
DB 직접 수정 API 금지
ledger posting 직접 수정/삭제 API 금지
잔고 직접 보정 API 금지
```

### 4.8 bond-exchange-app

장내채권 거래소 시뮬레이터입니다.

초기 MVP 이후 구현합니다.

Responsibilities:

```text
장내채권 시장 세션
채권 호가 접수
단순 호가장
단순 매칭
체결 결과 통보
청산 원천 데이터 생성
```

Rule:

```text
초기 장외채권 MVP 완료 전까지 구현하지 않는다.
```

---

## 5. 공통 라이브러리 구성

### 5.1 stbase-common

모든 앱이 공유할 수 있는 순수 공통 타입입니다.

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
```

Forbidden:

```text
특정 앱의 Entity
특정 앱의 Repository
특정 앱의 Service
Spring Web Controller
JPA Entity
```

### 5.2 stbase-api-contracts

앱 간 API 요청/응답 DTO를 둡니다.

Examples:

```text
KsdDvpSettlementRequest
KsdDvpSettlementResponse
KofiaOtcBondReportRequest
KofiaOtcBondReportResponse
FssComplaintSubmissionRequest
PowerbaseOtcBondTradeRequest
ExternalBrokerTradeStatusResponse
```

Rule:

```text
api-contracts에는 비즈니스 로직을 넣지 않는다.
DTO, enum, error code, API schema만 둔다.
```

### 5.3 stbase-test-fixtures

통합 테스트용 fixture를 둡니다.

Examples:

```text
SampleMembers
SampleAccounts
SampleBonds
SampleOtcTrades
SampleKsdPositions
SampleComplaints
```

---

## 6. 통신 원칙

모든 앱 간 상태 변경 API는 다음 헤더를 요구합니다.

```http
X-Correlation-Id: CORR-20260514-000001
X-Idempotency-Key: IDEMP-20260514-000001
X-Source-System: POWERBASE
```

모든 command API는 다음을 만족해야 합니다.

```text
idempotency key 필수
correlation id 필수
external message log 기록
실패 응답도 저장
재처리 가능 상태 저장
중복 요청 안전
```

API 응답 형식:

```json
{
  "result": "SUCCESS",
  "data": {},
  "traceId": "TRACE-20260514-000001",
  "timestamp": "2026-05-14T12:00:00+09:00"
}
```

오류 응답 형식:

```json
{
  "result": "ERROR",
  "errorCode": "KSD_SETTLEMENT_REJECTED",
  "message": "KSD settlement was rejected.",
  "traceId": "TRACE-20260514-000001",
  "timestamp": "2026-05-14T12:00:00+09:00"
}
```

---

## 7. 핵심 데이터 원칙

### 7.1 Ledger가 원천이다

```text
LedgerPosting is source of truth.
BalanceSnapshot is projection.
```

금지:

```java
balance.setCash(balance.getCash().subtract(amount));
accountBalanceRepository.save(balance);
```

필수:

```text
LedgerJournal 생성
LedgerPosting 생성
BalanceProjection 갱신
AuditLog 기록
```

### 7.2 Posting은 불변이다

```text
posting update 금지
posting delete 금지
오류는 reversal posting
보정은 correction posting
```

### 7.3 외부기관은 DB로 연결하지 않는다

금지:

```text
powerbase-app이 ksd-app DB 조회
powerbase-app이 kofia-app DB 조회
admin-app이 각 앱 DB를 직접 수정
```

허용:

```text
공개 API 호출
read-only status API 호출
external message log 조회 API
대사 전용 export API
```

---

## 8. 장외채권 MVP 흐름

첫 번째 MVP는 아래 흐름 하나를 끝까지 완성합니다.

```text
[1] external-broker-app 또는 retail 요청자가 장외채권 매수 요청
[2] powerbase-app이 고객/계좌/적합성/현금/상품위험도 검증
[3] powerbase-app이 채권 재고를 조건부 예약
[4] powerbase-app이 OTC Bond Trade 생성
[5] powerbase-app이 pending ledger posting 생성
[6] powerbase-app이 KOFIA 보고 outbox event 생성
[7] powerbase-app이 KSD 결제 outbox event 생성
[8] outbox relay가 kofia-app에 거래보고 API 호출
[9] outbox relay가 ksd-app에 DVP 또는 계좌대체 API 호출
[10] kofia-app은 보고 승인/반려/지연 중 하나를 응답
[11] ksd-app은 결제 완료/거절/지연 중 하나를 응답
[12] powerbase-app이 결과에 따라 원장 확정 또는 실패 상태 보존
[13] reconciliation job이 PowerBase 잔고와 KSD 총량 대사
[14] 고객 민원이 들어오면 complaint workflow 시작
[15] 운영자는 admin-app에서 거래/보고/결제/원장/대사/감사로그를 추적
```

MVP 성공 기준:

```text
거래 생성 가능
pending posting 생성 가능
KOFIA 보고 성공/실패 시뮬레이션 가능
KSD 결제 성공/실패 시뮬레이션 가능
결제 성공 시 원장 확정 가능
결제 실패 시 재처리 가능
중복 요청 시 중복 posting 없음
대사 break 생성 가능
민원 접수 후 거래 추적 가능
운영자 재처리 audit log 기록 가능
```

---

## 9. 민원/감독 시뮬레이션

민원은 단순 게시판이 아닙니다.

민원은 거래, 원장, 보고, 결제, 대사, 운영행위 추적과 연결되어야 합니다.

### 9.1 민원 예시

```text
채권 매수했는데 잔고에 안 보인다
결제 완료라고 했는데 현금이 이상하다
장외채권 수익률 안내와 실제 체결 조건이 다르다
매수 취소 요청이 반영되지 않았다
KOFIA 보고가 반려됐는데 고객에게 안내되지 않았다
이자 지급일이 지났는데 입금이 안 됐다
```

### 9.2 complaint workflow

```text
RECEIVED
ASSIGNED
INVESTIGATING
EVIDENCE_COLLECTING
WAITING_EXTERNAL_RESPONSE
ANSWER_DRAFTED
ANSWER_SENT
CLOSED
ESCALATED_TO_FSS
```

### 9.3 민원 처리 시 조회해야 할 증적

```text
trade
ledger_journal
ledger_posting
balance_snapshot
ksd_settlement
kofia_report
external_message_log
outbox_event
reconciliation_break
operator_audit_log
```

### 9.4 FSS-Sim 연계

FSS-Sim은 다음 API를 제공할 수 있습니다.

```text
POST /api/fss/complaints
POST /api/fss/document-requests
POST /api/fss/inspection-events
GET  /api/fss/complaints/{complaintId}
```

FSS-Sim은 거래를 직접 취소하거나 원장을 수정하지 않습니다.

---

## 10. 권장 API 목록

### 10.1 powerbase-app

```text
POST /api/powerbase/otc-bond-trades
GET  /api/powerbase/otc-bond-trades/{tradeId}
GET  /api/powerbase/accounts/{accountId}/balances
GET  /api/powerbase/accounts/{accountId}/ledger-postings
POST /api/powerbase/complaints
GET  /api/powerbase/complaints/{complaintId}
POST /api/powerbase/operations/retry-outbox/{outboxEventId}
POST /api/powerbase/operations/corrections
```

### 10.2 external-broker-app

```text
POST /api/external-broker/otc-bond-orders
GET  /api/external-broker/otc-bond-orders/{orderId}
POST /api/external-broker/complaints
```

### 10.3 freebond-app

```text
POST /api/freebond/quotes
GET  /api/freebond/quotes
POST /api/freebond/negotiations
POST /api/freebond/deal-agreements
GET  /api/freebond/deal-agreements/{agreementId}
```

### 10.4 ksd-app

```text
POST /api/ksd/dvp-settlements
POST /api/ksd/book-entry-transfers
GET  /api/ksd/participant-positions
GET  /api/ksd/settlements/{settlementId}
POST /api/ksd/test-scenarios/settlement-delay
POST /api/ksd/test-scenarios/settlement-reject
```

### 10.5 kofia-app

```text
POST /api/kofia/otc-bond-trade-reports
GET  /api/kofia/otc-bond-trade-reports/{reportId}
GET  /api/kofia/disclosures/otc-bonds
POST /api/kofia/test-scenarios/report-delay
POST /api/kofia/test-scenarios/report-reject
```

### 10.6 fss-app

```text
POST /api/fss/complaints
GET  /api/fss/complaints/{complaintId}
POST /api/fss/document-requests
POST /api/fss/inspection-events
```

### 10.7 admin-app

```text
GET  /api/admin/trades/{tradeId}/timeline
GET  /api/admin/outbox-events
POST /api/admin/outbox-events/{eventId}/retry
GET  /api/admin/external-message-logs
GET  /api/admin/reconciliation-breaks
POST /api/admin/reconciliation-breaks/{breakId}/resolve
GET  /api/admin/audit-logs
```

---

## 11. 상태 모델

### 11.1 OtcBondTradeStatus

```text
REQUESTED
VALIDATED
INVENTORY_RESERVED
PENDING_LEDGER_POSTED
REPORT_REQUESTED
SETTLEMENT_REQUESTED
REPORT_ACCEPTED
REPORT_REJECTED
SETTLEMENT_COMPLETED
SETTLEMENT_REJECTED
LEDGER_FINALIZED
FAILED
MANUAL_REVIEW_REQUIRED
CANCELED
```

### 11.2 KofiaReportStatus

```text
RECEIVED
ACCEPTED
REJECTED
DELAYED
CORRECTION_REQUIRED
```

### 11.3 KsdSettlementStatus

```text
REQUESTED
ACCEPTED
COMPLETED
REJECTED
DELAYED
TIMEOUT
RETRY_REQUIRED
```

### 11.4 ReconciliationStatus

```text
MATCHED
BREAK_FOUND
MANUAL_REVIEW_REQUIRED
RESOLVED
```

---

## 12. 테스트 우선순위

AI는 다음 테스트를 우선 구현합니다.

```text
1. 같은 idempotency key로 장외채권 거래를 두 번 요청해도 거래는 하나만 생성된다.
2. 같은 idempotency key와 다른 payload가 오면 거절된다.
3. 장외채권 재고는 동시 요청에서도 oversell 되지 않는다.
4. pending ledger posting 없이 KSD 결제를 요청할 수 없다.
5. KSD 결제 성공 전에는 settled balance가 증가하지 않는다.
6. KSD 결제 성공 후 ledger finalize가 한 번만 수행된다.
7. KOFIA 보고 실패는 거래 자체를 자동 취소하지 않는다.
8. KSD 결제 실패는 retryable 상태로 남는다.
9. 중복 KSD 완료 응답은 중복 ledger posting을 만들지 않는다.
10. PowerBase 고객별 채권 합산과 KSD 참가기관 총량 불일치가 대사 break로 생성된다.
11. 민원 조회 화면에서 trade, ledger, KSD, KOFIA, external message log를 연결해서 볼 수 있다.
12. 운영자 재처리는 audit log를 남긴다.
13. 운영자 correction은 기존 posting을 수정하지 않고 correction posting을 만든다.
14. FSS-Sim은 거래/결제 상태를 직접 변경할 수 없다.
15. external-broker-app은 powerbase DB에 접근하지 않고 API로만 거래한다.
```

---

## 13. AI 구현 규칙

AI가 코드를 수정할 때는 다음 순서를 따릅니다.

```text
1. 이 문서를 먼저 읽는다.
2. 구현 대상 앱을 하나로 정한다.
3. 해당 앱의 책임과 금지사항을 확인한다.
4. API 계약이 필요한 경우 stbase-api-contracts에 DTO를 먼저 추가한다.
5. 상태 변경 API에는 idempotency key를 적용한다.
6. 외부 앱 호출은 outbox event로 기록한다.
7. 외부 호출 결과는 external message log로 기록한다.
8. 원장 영향이 있으면 ledger journal/posting으로 처리한다.
9. 직접 balance update를 작성하지 않는다.
10. 실패 상태와 재처리 경로를 추가한다.
11. 테스트를 추가한다.
12. 문서를 갱신한다.
```

AI가 절대 하면 안 되는 것:

```text
실제 금융기관 API 연동 코드 작성
실제 API key/secret/cert 구조 작성
cross-app DB 접근
ledger posting update/delete
balance 직접 update
보고 실패를 조용히 무시
결제 실패를 성공처럼 처리
민원 처리에서 증적 없이 상태만 종료
```

---

## 14. 구현 순서

### Phase 1 - Foundation

```text
Gradle multi-project 생성
stbase-common 생성
stbase-api-contracts 생성
powerbase-app 생성
기본 ApiResponse/ErrorCode/TraceId 구성
idempotency 저장소
audit log 저장소
ledger journal/posting 기본 모델
```

### Phase 2 - OTC Bond Trade MVP

```text
member/account/bond/product 모델
채권 재고 모델
장외채권 매수 command
pending ledger posting
idempotency test
inventory concurrency test
```

### Phase 3 - KOFIA/KSD API Simulators

```text
kofia-app 생성
ksd-app 생성
KOFIA 보고 API
KSD 계좌대체/DVP API
outbox relay
external message log
보고 실패/결제 실패 시나리오
```

### Phase 4 - Ledger Finalization and Reconciliation

```text
KSD 결제 완료 반영
ledger finalize
balance projection
PowerBase vs KSD 대사
reconciliation break
운영자 재처리
```

### Phase 5 - External Broker and Complaint

```text
external-broker-app 생성
외부 증권사 거래 요청 API
complaint workflow
FSS-Sim 민원/자료제출 API
admin timeline API
```

### Phase 6 - FreeBond

```text
freebond-app 생성
호가 게시
협의 로그
deal agreement
PowerBase 거래입력 연계
```

### Phase 7 - Bond Exchange

```text
bond-exchange-app 생성
장내채권 시장 세션
단순 호가 접수
단순 매칭
체결 결과 통보
청산/결제 연계
```

---

## 15. 첫 번째 완성 시나리오

AI는 프로젝트 초기 구현에서 아래 시나리오를 최우선으로 완성합니다.

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

이 시나리오가 완성되면 STBase는 단순 CRUD가 아니라 실제 증권업무 흐름을 학습할 수 있는 시뮬레이터가 됩니다.

---

## 16. 한 줄 요약

STBase는 장외채권 거래를 시작점으로, PowerBase 업무계와 KSD/KOFIA/FSS/FreeBond/외부증권사 시뮬레이터를 별도 API 서버로 실행하여 원장, 결제, 보고, 대사, 민원, 운영 재처리까지 재현하는 학습용 분산 증권업무 시뮬레이터입니다.
