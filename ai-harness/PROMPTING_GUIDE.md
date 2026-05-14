# STBase Prompting Guide

> 이 문서는 STBase를 AI 바이브코딩으로 개발할 때 Claude/Codex 같은 AI 코딩 도구에 어떻게 지시할지 정리한 프롬프트 가이드입니다.
>
> 핵심 원칙은 **문서에는 정답 코드가 있고, AI는 구조와 반복 코드를 만들며, 사람은 핵심 usecase/service를 직접 따라 치며 익힌다**입니다.

---

## 1. Prompting Principles

AI에게는 다음을 항상 명확히 줍니다.

```text
역할
프로젝트 맥락
참조할 문서
작업 범위
생성할 파일
생성하지 말아야 할 것
출력 형식
검증 기준
```

AI에게 "구현해줘"라고만 말하지 않습니다.

좋은 지시:

```text
문서의 정답 코드를 기준으로 entity/repository/adapter/dto skeleton만 생성하라.
usecase.execute 내부 핵심 비즈니스 로직은 내가 직접 타이핑할 것이므로 TODO로 비워둬라.
```

나쁜 지시:

```text
장외채권 거래 기능 만들어줘.
```

---

## 2. Why This Matters

STBase는 일반 CRUD 프로젝트가 아닙니다.

특히 아래 영역은 AI가 그럴듯하게 틀리기 쉽습니다.

```text
원장 posting 의미
pending/settled/held 잔고 구분
KSD 결제 성공/실패 반영
KOFIA 보고 실패 처리
대사 break 판단
민원 처리 증적
운영자 correction 정책
```

따라서 AI는 반복적이고 구조적인 코드를 만들고, 사람은 업무 판단이 들어가는 핵심 코드를 직접 작성합니다.

```text
AI 역할:
패키지/폴더 생성
DTO 생성
Enum 생성
JPA entity 생성
Repository interface 생성
Repository adapter 생성
Mapper 초안 생성
Controller skeleton 생성
Test skeleton 생성
문서 업데이트

사람 역할:
거래 상태 전이 작성
원장 posting 순서 작성
잔고 pending/settled 처리 작성
KSD/KOFIA 실패 정책 작성
대사 판단 작성
민원 처리 판단 작성
운영자 correction 정책 작성
```

---

## 3. Token Saving Workflow

긴 문서를 매번 전부 붙여넣지 않습니다.

AI에게는 필요한 문서 경로를 지정합니다.

```text
참조:
- docs/STBASE_MULTI_APP_BOND_SIMULATOR_PLAN.md
- ai-harness/PROJECT_SKILL.md
- ai-harness/DOMAIN_STRUCTURE_GUIDE.md
- ai-harness/AI_IMPLEMENTATION_CHECKLIST.md
- docs/implementation-guides/.../대상-정답-코드.md
```

작업 단위는 작게 유지합니다.

```text
한 세션 = 한 앱 + 한 도메인 + 한 usecase
```

권장 운영:

```text
새 도메인으로 넘어가면 새 세션을 시작한다.
긴 대화가 누적되면 요약 후 새로 시작한다.
큰 문서는 경로만 제공하고 필요한 부분만 읽게 한다.
테스트 실패 출력은 전체 로그가 아니라 실패 부분만 제공한다.
```

---

## 4. Base Prompt for Skeleton Generation

아래 프롬프트는 AI에게 구조 코드 생성을 맡길 때 사용합니다.

```text
<role>
너는 STBase 프로젝트의 Spring Boot / Spring Data JPA / MySQL / DDD 코드 생성 보조자다.
너는 증권 업무 로직을 임의로 창작하지 않는다.
문서에 있는 정답 코드와 설계만 따른다.
</role>

<context>
이 프로젝트는 장외채권 거래를 시작점으로 하는 증권업무 시뮬레이터다.
나는 학습 목적으로 핵심 usecase/service 코드를 직접 타이핑할 것이다.
따라서 너는 반복적인 구조 코드와 skeleton 생성을 담당한다.
</context>

<documents_to_read>
1. docs/STBASE_MULTI_APP_BOND_SIMULATOR_PLAN.md
2. ai-harness/README.md
3. ai-harness/PROJECT_SKILL.md
4. ai-harness/DOMAIN_STRUCTURE_GUIDE.md
5. ai-harness/AI_IMPLEMENTATION_CHECKLIST.md
6. 내가 추가로 지정하는 정답 코드 문서
</documents_to_read>

<task>
대상 앱: powerbase-app
대상 도메인: otc-bond-trade

다음을 생성하라.
1. domain/entity
2. domain/value
3. domain/repository interface
4. infrastructure/persistence JPA entity
5. infrastructure/persistence Spring Data repository
6. infrastructure/persistence repository adapter
7. infrastructure/persistence mapper
8. presentation/dto request/response
9. application/usecase 클래스 skeleton
10. application/service 클래스 skeleton
</task>

<constraints>
Usecase public method 이름은 execute로 통일한다.
Usecase와 service 내부 핵심 비즈니스 로직은 작성하지 않는다.
문서에 정답 코드가 있는 경우에는 정답 코드를 그대로 반영한다.
문서에 정답 코드가 없는 경우에는 TODO 주석만 남긴다.
Controller에는 비즈니스 로직을 넣지 않는다.
Domain layer는 Spring에 의존하지 않는다.
JPA 구현은 infrastructure에 둔다.
공통 타입은 common을 사용한다.
balance 직접 update 코드는 만들지 않는다.
ledger posting update/delete 코드는 만들지 않는다.
외부기관 API 직접 호출 로직은 usecase 안에 넣지 않는다.
</constraints>

<output>
파일을 직접 생성/수정하라.
작업 후 변경한 파일 목록과 아직 내가 직접 타이핑해야 할 TODO 목록만 요약하라.
</output>
```

---

## 5. Prompt for Answer-Code Document Creation

아래 프롬프트는 사람이 따라 칠 정답 코드 문서를 만들 때 사용합니다.

```text
<role>
너는 STBase의 교육용 설계 문서 작성자다.
목표는 AI가 코드를 생성하기 위한 문서가 아니라, 내가 직접 따라 치며 익힐 수 있는 정답 코드 문서를 만드는 것이다.
</role>

<task>
다음 도메인에 대한 구현 학습 문서를 작성하라.

앱: powerbase-app
도메인: otc-bond-trade
주제: CreateOtcBondTradeUseCase

문서에는 다음을 포함하라.
1. 업무 목적
2. 선행 조건
3. 상태 전이
4. 필요한 Entity 목록
5. 필요한 Repository 목록
6. 필요한 Service 목록
7. execute 메서드의 정답 코드
8. 각 코드 줄이 왜 필요한지 설명
9. 실패 케이스
10. 테스트 시나리오
</task>

<style>
코드는 Java 17, Spring Boot, Spring Data JPA, MySQL 기준으로 작성한다.
DDD 구조를 따른다.
application/usecase의 public method는 execute로 통일한다.
설명은 따라 치며 이해할 수 있게 단계별로 작성한다.
</style>

<constraints>
실제 금융기관 API 연동 코드는 작성하지 않는다.
balance 직접 update 코드는 작성하지 않는다.
ledger posting은 불변으로 다룬다.
KOFIA/KSD 호출은 outbox event 생성까지만 usecase에서 처리한다.
</constraints>
```

---

## 6. Prompt for Generating Skeleton from Answer-Code Document

정답 코드 문서를 먼저 만든 뒤, 그 문서를 기준으로 skeleton을 생성시킵니다.

```text
docs/implementation-guides/powerbase/otc-bond-trade/create-otc-bond-trade-usecase.md의 정답 코드 문서를 기준으로 skeleton을 생성하라.

참조:
- ai-harness/PROJECT_SKILL.md
- ai-harness/DOMAIN_STRUCTURE_GUIDE.md
- ai-harness/AI_IMPLEMENTATION_CHECKLIST.md

대상 앱:
powerbase-app

대상 도메인:
otc-bond-trade

생성:
- entity
- value object
- repository interface
- JPA entity
- Spring Data repository
- repository adapter
- mapper
- request/response DTO
- controller skeleton
- usecase skeleton
- service skeleton

중요:
CreateOtcBondTradeUseCase.execute 내부는 내가 직접 타이핑할 것이다.
따라서 execute 내부에는 정답 코드의 단계별 TODO 주석만 넣고 실제 비즈니스 로직은 작성하지 마라.

금지:
- 문서에 없는 업무 로직 창작
- balance 직접 update
- ledger posting update/delete
- 외부 API 직접 호출
- Controller에 비즈니스 로직 작성
```

---

## 7. Prompt for Review

AI에게 내가 직접 작성한 usecase/service 코드를 검토시키는 프롬프트입니다.

```text
<role>
너는 STBase 프로젝트의 코드 리뷰어다.
리뷰 목적은 코드 스타일이 아니라 증권업무 정합성, DDD 경계, 원장 안정성, 재처리 안정성 검증이다.
</role>

<documents_to_read>
1. docs/STBASE_MULTI_APP_BOND_SIMULATOR_PLAN.md
2. ai-harness/PROJECT_SKILL.md
3. ai-harness/DOMAIN_STRUCTURE_GUIDE.md
4. ai-harness/AI_IMPLEMENTATION_CHECKLIST.md
5. 해당 usecase의 정답 코드 문서
</documents_to_read>

<review_target>
내가 직접 작성한 CreateOtcBondTradeUseCase와 관련 service 코드를 리뷰하라.
</review_target>

<review_focus>
1. idempotency 누락
2. ledger posting 불변성 위반
3. balance 직접 update
4. KSD 결제 전 settled 반영
5. KOFIA 보고 실패 처리 누락
6. outbox/external message log 누락
7. audit log 누락
8. 대사/민원 추적 불가능
9. domain/application/infrastructure 경계 위반
</review_focus>

<output>
문제점이 있으면 심각도 순으로 파일/라인과 함께 제시하라.
문제점이 없으면 "발견된 중대 이슈 없음"이라고 말하고, 남은 리스크를 짧게 적어라.
코드를 임의로 수정하지 말고 먼저 리뷰만 하라.
</output>
```

---

## 8. Prompt for Test Skeleton

테스트 skeleton을 만들 때 사용합니다.

```text
다음 정답 코드 문서를 기준으로 테스트 skeleton을 생성하라.

문서:
docs/implementation-guides/powerbase/otc-bond-trade/create-otc-bond-trade-usecase.md

대상:
CreateOtcBondTradeUseCase

생성할 테스트:
1. 같은 idempotency key로 두 번 요청해도 거래는 하나만 생성된다.
2. 같은 idempotency key와 다른 payload는 거절된다.
3. 채권 재고는 동시 요청에서 oversell 되지 않는다.
4. pending ledger posting이 생성된다.
5. KOFIA 보고 outbox가 생성된다.
6. KSD 결제 outbox가 생성된다.
7. audit log가 생성된다.

조건:
- 테스트명은 업무 의미가 드러나게 작성한다.
- 테스트 skeleton과 fixture만 만든다.
- 아직 내가 직접 구현할 assertion에는 TODO를 남긴다.
```

---

## 9. Recommended Document Layout

정답 코드 문서는 아래 위치에 둡니다.

```text
docs/implementation-guides/
  powerbase/
    otc-bond-trade/
      create-otc-bond-trade-usecase.md
      finalize-otc-bond-trade-usecase.md
    ledger/
      create-ledger-posting-usecase.md
    complaint/
      register-complaint-usecase.md
  ksd/
    settlement/
      process-dvp-settlement-usecase.md
  kofia/
    report/
      receive-otc-bond-report-usecase.md
```

각 정답 코드 문서는 아래 형식을 따릅니다.

```text
1. 목적
2. 업무 배경
3. 입력/출력
4. 상태 전이
5. 필요한 클래스
6. 정답 코드
7. 코드 설명
8. 실패 케이스
9. 테스트 시나리오
10. AI skeleton 생성 지시문
```

---

## 10. One-Sentence Rule

```text
AI는 구조와 반복 코드를 만든다.
문서는 정답 코드를 제공한다.
사람은 핵심 usecase/service를 직접 따라 치며 익힌다.
AI는 execute 내부 업무 로직을 임의로 창작하지 않는다.
```
