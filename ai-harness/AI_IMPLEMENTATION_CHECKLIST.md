# STBase AI Implementation Checklist

> AI가 기능을 구현하거나 수정할 때 매번 확인해야 하는 체크리스트입니다.

---

## 1. Before Coding

```text
[ ] docs/STBASE_MULTI_APP_BOND_SIMULATOR_PLAN.md를 읽었다.
[ ] ai-harness/PROJECT_SKILL.md를 읽었다.
[ ] ai-harness/DOMAIN_STRUCTURE_GUIDE.md를 읽었다.
[ ] 구현 대상 앱을 하나로 정했다.
[ ] 구현 대상 도메인을 하나로 정했다.
[ ] 기존 코드와 문서를 확인했다.
[ ] 기존 사용자의 변경사항을 되돌리지 않는다.
```

---

## 2. Scope Check

```text
[ ] 이 기능은 powerbase-app 기능인가?
[ ] 이 기능은 external-broker-app 기능인가?
[ ] 이 기능은 freebond-app 기능인가?
[ ] 이 기능은 ksd-app 기능인가?
[ ] 이 기능은 kofia-app 기능인가?
[ ] 이 기능은 fss-app 기능인가?
[ ] 이 기능은 admin-app 기능인가?
[ ] 이 기능은 common/api-contracts에만 해당하는가?
```

하나의 작업에서 여러 앱을 수정해야 한다면 API 계약부터 분리합니다.

---

## 3. Layer Check

```text
[ ] Controller는 request를 command로 변환하고 usecase.execute만 호출한다.
[ ] Usecase는 업무 흐름을 읽을 수 있게 orchestration한다.
[ ] Application service는 공통 application 기능만 담당한다.
[ ] Domain entity/value/service가 핵심 규칙을 보호한다.
[ ] Domain layer는 Spring에 의존하지 않는다.
[ ] Infrastructure layer에 JPA/API client 구현을 둔다.
[ ] Presentation layer에 DTO와 Controller를 둔다.
[ ] Common에는 특정 업무 도메인을 넣지 않는다.
```

---

## 4. Usecase Check

```text
[ ] Usecase 클래스명은 역할이 드러난다.
[ ] Usecase public method 이름은 execute다.
[ ] Usecase input은 command 또는 query object다.
[ ] Usecase output은 result 또는 response model이다.
[ ] Usecase가 너무 길면 domain service/application service로 분리했다.
[ ] 단순 CRUD가 아니라 업무 시나리오 기준으로 작성했다.
```

Example:

```text
CreateOtcBondTradeUseCase.execute(command)
FinalizeLedgerAfterKsdSettlementUseCase.execute(command)
RegisterComplaintUseCase.execute(command)
```

---

## 5. Idempotency Check

상태 변경 API라면 반드시 확인합니다.

```text
[ ] X-Idempotency-Key를 요구한다.
[ ] 같은 key + 같은 request는 같은 응답을 반환한다.
[ ] 같은 key + 다른 request는 거절한다.
[ ] 처리 중 실패 상태가 저장된다.
[ ] 중복 callback도 안전하다.
```

Required for:

```text
POST /api/powerbase/otc-bond-trades
POST /api/ksd/dvp-settlements
POST /api/kofia/otc-bond-trade-reports
POST /api/fss/complaints
POST /api/admin/outbox-events/{eventId}/retry
POST /api/admin/operations/corrections
```

---

## 6. Ledger Check

원장 영향이 있다면 반드시 확인합니다.

```text
[ ] ledger journal을 생성한다.
[ ] ledger posting을 생성한다.
[ ] posting update/delete가 없다.
[ ] balance는 projection으로 갱신한다.
[ ] reversal/correction은 새 posting으로 처리한다.
[ ] settled/pending/held를 구분한다.
[ ] KSD 결제 전 settled balance를 확정하지 않는다.
[ ] 중복 이벤트가 중복 posting을 만들지 않는다.
```

Forbidden code smell:

```java
balance.setCash(...);
balance.setQuantity(...);
ledgerPosting.setAmount(...);
ledgerPostingRepository.delete(...);
```

---

## 7. External API Check

외부 앱과 통신한다면 반드시 확인합니다.

```text
[ ] 다른 앱 DB에 직접 접근하지 않는다.
[ ] API 계약 DTO가 stbase-api-contracts에 있다.
[ ] X-Correlation-Id를 전파한다.
[ ] X-Idempotency-Key를 전파한다.
[ ] X-Source-System을 전파한다.
[ ] external_message_log를 기록한다.
[ ] 실패 응답도 기록한다.
[ ] retry 가능한 상태를 저장한다.
```

PowerBase에서 KSD/KOFIA/FSS로 나가는 상태 변경 호출은 기본적으로 outbox를 사용합니다.

```text
[ ] local transaction 안에서 aggregate 저장
[ ] ledger posting 저장
[ ] outbox event 저장
[ ] transaction commit
[ ] relay가 외부 API 호출
[ ] external message log 저장
[ ] outbox status 갱신
```

---

## 8. Failure State Check

실패를 숨기지 않습니다.

```text
[ ] KOFIA 보고 실패 상태가 있다.
[ ] KSD 결제 실패 상태가 있다.
[ ] timeout 상태가 있다.
[ ] retry required 상태가 있다.
[ ] manual review 상태가 있다.
[ ] 실패가 감사로그에 남는다.
[ ] 운영자가 상태를 추적할 수 있다.
```

---

## 9. Reconciliation Check

대사가 필요한 기능인지 확인합니다.

```text
[ ] PowerBase 고객별 잔고와 KSD 참가기관 총량을 비교할 수 있다.
[ ] 거래와 KOFIA 보고를 비교할 수 있다.
[ ] settlement instruction과 KSD result를 비교할 수 있다.
[ ] ledger posting과 balance snapshot을 비교할 수 있다.
[ ] 불일치 자동 수정은 하지 않는다.
[ ] break는 운영자 검토 대상으로 남긴다.
```

---

## 10. Complaint Check

민원과 연결될 수 있는 기능이라면 확인합니다.

```text
[ ] 거래 ID로 timeline 조회가 가능하다.
[ ] ledger journal/posting 조회가 가능하다.
[ ] KSD settlement 조회가 가능하다.
[ ] KOFIA report 조회가 가능하다.
[ ] external message log 조회가 가능하다.
[ ] outbox event 조회가 가능하다.
[ ] audit log 조회가 가능하다.
[ ] 민원 답변에는 증적이 연결된다.
```

---

## 11. Test Check

기능 추가 시 최소 테스트를 작성합니다.

```text
[ ] happy path test
[ ] idempotency duplicate test
[ ] invalid request test
[ ] failure state test
[ ] audit/reconciliation impact test
```

장외채권/원장/결제 관련이면 우선 테스트:

```text
[ ] 같은 idempotency key는 거래를 하나만 만든다.
[ ] 채권 재고는 동시 요청에서 oversell 되지 않는다.
[ ] KSD 결제 성공 전에는 settled balance가 증가하지 않는다.
[ ] KSD 결제 완료 callback 중복 수신이 중복 posting을 만들지 않는다.
[ ] KOFIA 보고 실패가 거래를 자동 취소하지 않는다.
[ ] 운영자 correction은 기존 posting을 수정하지 않는다.
```

---

## 12. Documentation Check

구조나 업무 흐름이 바뀌면 문서를 갱신합니다.

```text
[ ] docs/STBASE_MULTI_APP_BOND_SIMULATOR_PLAN.md 갱신 필요 여부 확인
[ ] ai-harness/PROJECT_SKILL.md 갱신 필요 여부 확인
[ ] ai-harness/DOMAIN_STRUCTURE_GUIDE.md 갱신 필요 여부 확인
[ ] API 계약 변경 시 문서화
[ ] 상태 모델 변경 시 문서화
[ ] 테스트 시나리오 변경 시 문서화
```

---

## 13. Final Self-Review

작업 완료 전 마지막으로 확인합니다.

```text
[ ] 실제 금융기관 연동 코드가 없다.
[ ] cross-app DB 접근이 없다.
[ ] Controller에 비즈니스 로직이 없다.
[ ] Usecase execute 구조를 지켰다.
[ ] Domain layer가 Spring에 의존하지 않는다.
[ ] JPA 구현은 infrastructure에 있다.
[ ] API DTO는 presentation 또는 api-contracts에 있다.
[ ] common에 업무 도메인을 넣지 않았다.
[ ] 테스트를 실행했다.
[ ] 실행하지 못한 테스트가 있으면 이유를 남긴다.
```
