# STBase - Securities Trade Base

> **STBase(Securities Trade Base)**는 코스콤 PowerBase 구조를 모티브로 한  
> **증권사 원장·주문·결제·대외기관 전문처리 모의 시스템**입니다.  
>
> 이 프로젝트는 실제 한국거래소, 예탁결제원, 금융감독원, 금융투자협회, 코스콤 시스템과 연계하지 않습니다.  
> 대신 각 유관 시스템의 역할을 내부 멀티모듈로 구현하여, 개인·기관·증권사 거래가  
> 주문, 체결, 청산, 결제, 원장, 보고, 대사까지 어떻게 이어지는지 재현합니다.

---

## 0. 핵심 전제

STBase는 실거래 시스템이 아닙니다.

```text
실제 KRX 주문 전송                 ❌
실제 예탁결제원 결제 지시           ❌
실제 금융감독원 보고/공시 제출       ❌
실제 금융투자협회 보고               ❌
실제 코스콤 PowerBase/StockNet 연계  ❌
실제 고객 자산 처리                 ❌
실제 증권사 계좌 연동                ❌
```

대신 아래를 구현합니다.

```text
PowerBase 구조의 증권사 업무계 시뮬레이션        ✅
K-FRONT 형태의 전문 트레이딩/OMS 시뮬레이션       ✅
STP-HUB 형태의 기관 주문 허브 시뮬레이션          ✅
StockNet 형태의 전문망/주문·시세망 시뮬레이션     ✅
EXTURE/KRX 형태의 시장시스템 시뮬레이션           ✅
KSD 형태의 예탁·결제·권리관리 시뮬레이션          ✅
KOFIA FreeBond/장외채권공시 시뮬레이션            ✅
FSS 형태의 공시·감독·검사 시뮬레이션              ✅
```

이 프로젝트의 목적은 **수익을 내는 트레이딩 프로그램**이 아닙니다.  
목적은 **증권사 업무계가 거래를 어떻게 받아 원장, 결제, 보고, 대사까지 처리하는지 구현하는 것**입니다.

---

## 1. 공식 자료 반영 기준

STBase는 다음 공식 자료의 역할 구분을 참고하여 설계합니다.

- 코스콤 PowerBase: 금융투자회사 기본업무, 자산관리, 투자정보, 글로벌 트레이딩 등을 지원하는 종합 아웃소싱 서비스
- 코스콤 시장시스템/EXTURE+: KRX 운영 시장의 매매체결, 체결결과 통보, 청산결제, 지수산출, CCP 등 거래 과정을 전산 처리
- 코스콤 STP-HUB: 자산운용사·기관투자가와 증권·선물사 간 주문, 체결, 매매보고서 등 메시지 교환 허브
- 코스콤 K-FRONT: 법인영업 및 상품매매용 주문처리시스템, 저지연 FIX, 알고리즘 트레이딩, 전문 트레이더용 Front 시스템
- 코스콤 증권망/StockNet: 거래소, 증권사, 유관기관을 연결하는 주문망·시세망 성격의 통신 인프라
- 코스콤 FreeBond/장외채권공시: 장외채권 거래 전용 시스템, 트레이딩보드, 전용메신저, 호가·매매정보 공시
- 코스콤 BOND CHECK/CHECK Expert+: 장내·장외 채권 가격, 발행정보, 수익률, 단가 계산기 등 시장정보 서비스

STBase는 위 시스템들을 그대로 복제하지 않습니다.  
각 시스템의 **업무상 책임과 데이터 흐름**을 학습용으로 재구성합니다.

---

## 2. STBase의 정체성

STBase는 단일 주문 서버가 아닙니다.

STBase는 다음 시스템들의 관계를 구현하는 **모의 자본시장 백엔드**입니다.

```text
[고객/기관/딜러/직원]
        ↓
[채널/프론트 시스템]
        ↓
[PowerBase-Sim: 증권사 업무계/고객원장]
        ↓
[StockNet-Sim: 주문·체결·시세·전문망]
        ↓
[EXTURE-Sim/KRX-Sim: 거래소 시장시스템]
        ↓
[KSD-Sim: 예탁·결제·권리관리]
        ↓
[보고/대사/감사/감독]
```

그리고 장외채권은 거래소 주문 흐름이 아니라 별도 흐름을 갖습니다.

```text
[딜러/기관/증권사]
        ↓
[K-FRONT-Sim 또는 FreeBond-Sim]
        ↓
[OTC Trade Capture]
        ↓
[PowerBase-Sim]
        ↓
[KOFIA FreeBond/Disclosure-Sim]
        ↓
[KSD-Sim]
        ↓
[원장/결제/대사]
```

---

## 3. 전체 아키텍처

```text
stbase-root
├── stbase-common
├── stbase-auth
│
├── stbase-powerbase-core
├── stbase-member
├── stbase-account
├── stbase-product
├── stbase-ledger
├── stbase-settlement
├── stbase-reconciliation
├── stbase-reporting
├── stbase-risk-compliance
├── stbase-batch
│
├── stbase-retail-channel-sim
├── stbase-branch-terminal-sim
├── stbase-kfront-sim
├── stbase-stp-hub-sim
│
├── stbase-order-routing
├── stbase-sor-sim
├── stbase-stocknet-sim
│
├── stbase-exture-sim
├── stbase-krx-marketdata-sim
├── stbase-krx-clearing-sim
│
├── stbase-ksd-sim
├── stbase-kofia-freebond-sim
├── stbase-kofia-disclosure-sim
├── stbase-fss-sim
│
├── stbase-market-info-sim
├── stbase-bond-info-sim
│
├── stbase-admin
├── stbase-ops-monitoring
└── stbase-infra
```

---

## 4. 핵심 모듈 설명

---

## 4.1 stbase-powerbase-core

증권사 업무계의 중심입니다.

이 모듈은 “시장”이 아닙니다.  
이 모듈은 “거래소”도 아닙니다.  
이 모듈은 “예탁결제원”도 아닙니다.

PowerBase-Sim은 다음 역할을 담당합니다.

```text
고객 계좌 관리
고객 원장 관리
현금/증권 잔고 관리
주문 접수 결과 반영
체결 결과 반영
장외거래 입력 결과 반영
결제 예정 생성
결제 결과 반영
영업점/채널 요청 처리
대외기관 전문 송수신 기록
일마감
대사
감사로그
```

PowerBase-Sim의 핵심 책임은 다음입니다.

> “고객별 세부 원장과 증권사 내부 업무 상태를 정확하게 관리한다.”

---

## 4.2 stbase-retail-channel-sim

개인 고객 채널입니다.

예상 화면/API:

```text
로그인
계좌 조회
잔고 조회
주문 입력
체결 조회
채권 상품 조회
RP 상품 조회
거래내역 조회
```

이 모듈은 실제 HTS/MTS를 만들기 위한 모듈이 아니라, 개인 고객 요청이 PowerBase-Sim으로 들어오는 흐름을 재현합니다.

```text
개인 고객
 → Retail Channel
 → PowerBase-Sim
 → Order Routing
 → StockNet-Sim
 → EXTURE-Sim
```

---

## 4.3 stbase-branch-terminal-sim

영업점 직원/내부 직원용 단말 시뮬레이션입니다.

역할:

```text
고객 계좌 조회
고객 주문 대행 입력
고객 적합성 확인
채권 판매 처리
입출금 요청
거래내역 조회
운영성 조회
```

개인 고객이 직접 주문하는 흐름과 직원이 대행 입력하는 흐름을 구분하기 위해 존재합니다.

---

## 4.4 stbase-kfront-sim

K-FRONT-Sim은 법인영업, 상품운용, 자기매매, 전문 트레이더용 Front/OMS 시뮬레이션입니다.

역할:

```text
법인영업 주문 입력
상품운용 주문 입력
자기매매 주문 입력
바스켓 주문
알고리즘 주문 시뮬레이션
FIX 스타일 주문 메시지 생성
시장데이터 기반 주문 의사결정 시뮬레이션
```

중요한 점:

```text
K-FRONT-Sim은 고객 원장을 직접 수정하지 않는다.
K-FRONT-Sim은 주문/거래 의사를 생성한다.
원장 반영은 PowerBase-Sim이 담당한다.
```

흐름:

```text
트레이더
 → K-FRONT-Sim
 → Order Routing
 → StockNet-Sim
 → EXTURE-Sim
 → 체결 결과
 → PowerBase-Sim
```

---

## 4.5 stbase-stp-hub-sim

STP-HUB-Sim은 자산운용사, 기관투자가 등 Buy-side와 증권사 Sell-side 간 메시지 허브입니다.

역할:

```text
기관 주문 메시지 수신
증권사로 주문 라우팅
체결 결과 중계
매매보고서 중계
결제내역 확인 메시지 중계
FIX 스타일 메시지 변환
복수 증권사 연결 시뮬레이션
```

STP-HUB-Sim은 개인 고객용 채널이 아닙니다.  
기관과 증권사 사이의 주문/체결/보고 메시지 허브입니다.

```text
자산운용사
 → STP-HUB-Sim
 → 증권사 PowerBase/K-FRONT
 → StockNet-Sim
 → EXTURE-Sim
```

---

## 4.6 stbase-order-routing

증권사 내부 주문 라우팅 모듈입니다.

역할:

```text
주문 유효성 검증
시장 구분
상품 구분
주문 가능 시간 검증
가용 현금/증권 hold 요청
주문 목적지 결정
StockNet-Sim 전송 요청
주문 상태 관리
중복 주문 방지
```

장내거래 주문은 이 모듈을 통과합니다.

```text
Retail / Branch / K-FRONT / STP-HUB
 → Order Routing
 → StockNet-Sim
 → EXTURE-Sim
```

---

## 4.7 stbase-sor-sim

SOR-Sim은 복수거래시장 대응을 위한 Smart Order Routing 시뮬레이션입니다.

초기 구현에서는 KRX-Sim 하나만 사용할 수 있습니다.  
하지만 설계는 복수 거래시장에 대비합니다.

역할:

```text
거래시장별 시세 비교
최선집행 규칙 평가
주문 목적지 선택
장애 시 우선 거래시장 전송
라우팅 근거 로그 저장
최선집행 증적 생성
```

예상 확장:

```text
KRX-Sim
ATS-Sim
DarkPool-Sim
```

초기 단계에서는 다음처럼 단순화합니다.

```text
Phase 1: KRX-Sim 단일 라우팅
Phase 2: KRX-Sim + ATS-Sim 복수시장
Phase 3: SOR-Sim 최선집행 증적
```

---

## 4.8 stbase-stocknet-sim

StockNet-Sim은 증권망/주문망/시세망 성격의 전문 통신 계층입니다.

이 모듈이 존재해야 시스템이 훨씬 현실적입니다.

역할:

```text
주문 전문 전달
체결 전문 전달
시세 전문 전달
청산/결제 전문 전달
유관기관 전문 전달
전문 송수신 로그
전문 재전송
전문 지연 시뮬레이션
전문 중복 수신 시뮬레이션
순서 역전 시뮬레이션
장애 복구 시뮬레이션
```

중요한 원칙:

```text
PowerBase-Sim이 EXTURE-Sim DB를 직접 보면 안 된다.
EXTURE-Sim이 PowerBase-Sim DB를 직접 수정하면 안 된다.
모든 대외 흐름은 StockNet-Sim 또는 명시적 API를 통해 지나가야 한다.
```

---

## 4.9 stbase-exture-sim

EXTURE-Sim은 한국거래소 시장시스템을 모의 구현합니다.

역할:

```text
시장 개장/마감
호가 접수
호가장 관리
매매체결
체결결과 통보
시장별 매매규칙 적용
청산 데이터 생성
시장감시 이벤트 생성
```

지원 시장 구분:

```text
KOSPI
KOSDAQ
KONEX
DERIVATIVES
BOND_GENERAL
BOND_SMALL
BOND_GOVERNMENT
REPO
```

초기 구현 권장 순서:

```text
1. KOSPI 단순 지정가 주문
2. BOND_GENERAL 단순 주문
3. BOND_GOVERNMENT
4. REPO
5. DERIVATIVES
```

중요:

```text
EXTURE-Sim은 고객 원장을 모른다.
EXTURE-Sim은 회원/증권사 단위 주문과 체결을 처리한다.
고객별 세부 반영은 PowerBase-Sim이 담당한다.
```

---

## 4.10 stbase-krx-clearing-sim

장내거래 체결 이후 청산 데이터를 생성하는 모듈입니다.

역할:

```text
체결내역 집계
회원별 매수/매도 정산
차감 계산
결제지시 생성
KSD-Sim 결제 요청 데이터 생성
```

청산은 단순 체결과 다릅니다.

```text
체결: A가 B에게 샀다/팔았다
청산: 회원별로 최종적으로 얼마를 주고받고 어떤 증권을 넘겨야 하는지 확정
결제: 실제 증권/대금 이전
```

---

## 4.11 stbase-ksd-sim

KSD-Sim은 예탁결제원 역할을 시뮬레이션합니다.

역할:

```text
증권 전자등록
예탁계좌 관리
회원/참가기관 단위 증권잔고 관리
증권 계좌대체
DVP 결제
권리관리
채권 이자 지급
채권 만기 상환
주식 배당 지급
대사 자료 제공
```

중요한 설계 원칙:

```text
KSD-Sim은 고객별 세부 잔고를 전부 관리하지 않아도 된다.
KSD-Sim은 증권사/참가기관 단위 총량을 관리한다.
PowerBase-Sim이 고객별 내부 잔고를 관리한다.
```

예시:

```text
KSD-Sim:
증권사 A 계좌에 국고채 KR000001 100억 보유

PowerBase-Sim:
고객1 1억
고객2 3억
고객3 5천만
증권사 자기분 95.5억

대사:
KSD-Sim 증권사 A 총량 = PowerBase-Sim 고객별 합산 + 자기분
```

---

## 4.12 stbase-kofia-freebond-sim

FreeBond-Sim은 장외채권 거래 전용 시스템을 모의 구현합니다.

역할:

```text
장외채권 호가 게시
딜러 간 협의
트레이딩보드
전용메신저
거래 조건 협의
장외채권 체결 후보 생성
```

장외채권은 단순히 “거래소 주문”이 아닙니다.

```text
호가 제시
상대방 탐색
수익률/가격 협상
수량 협상
거래 조건 합의
거래 입력
보고
결제
```

---

## 4.13 stbase-kofia-disclosure-sim

장외채권 거래보고/공시 시뮬레이션입니다.

역할:

```text
장외채권 거래보고 수신
보고 지연 감지
호가/매매정보 공시
수익률 통계
거래량 통계
보고 오류 반려
```

PowerBase-Sim 또는 Trade Capture 모듈은 장외채권 거래 체결 후 이 모듈로 보고 이벤트를 보냅니다.

---

## 4.14 stbase-fss-sim

FSS-Sim은 금융감독원 역할을 시뮬레이션합니다.

중요:

```text
FSS-Sim은 주문을 체결하지 않는다.
FSS-Sim은 결제를 처리하지 않는다.
FSS-Sim은 매 거래마다 실시간으로 끼어드는 시스템이 아니다.
```

역할:

```text
발행공시 접수
증권신고서 시뮬레이션
업무보고 접수
검사 이벤트 생성
투자자보호 점검
불완전판매 의심 점검
내부통제 위반 점검
자료제출 요청
제재 이벤트 시뮬레이션
```

거래 플로우와 감독 플로우를 분리합니다.

```text
거래 플로우:
고객/기관/증권사 → PowerBase-Sim → KRX/KOFIA → KSD → 원장/대사

감독 플로우:
PowerBase-Sim/증권사 → FSS-Sim
발행회사/주관사 → FSS-Sim
FSS-Sim → 검사/자료제출 요청
```

---

## 4.15 stbase-market-info-sim

시장정보 시스템입니다.

거래 시스템과 정보 시스템은 분리합니다.

역할:

```text
시세 조회
종목 정보 조회
채권 발행정보 조회
채권 가격/수익률 정보 조회
수익률 곡선 조회
시장 통계
뉴스 더미 데이터
공시 더미 데이터
```

---

## 4.16 stbase-bond-info-sim

BOND CHECK 성격의 채권정보 시뮬레이션입니다.

역할:

```text
장내채권 가격 정보
장외채권 가격 정보
채권 발행정보
신용등급
금리 정보
채권 단가 계산
경과이자 계산
수익률 계산
FRN 정보 확장
스왑 정보 확장
```

중요:

```text
채권 가격 계산 로직을 주문 서비스 안에 흩뿌리지 않는다.
채권 계산은 Bond Info/Calculation 영역으로 격리한다.
```

---

## 5. 거래주체 구분

STBase는 개인 고객만 다루지 않습니다.

```text
INDIVIDUAL
- 개인 고객

INSTITUTION
- 자산운용사
- 보험사
- 연기금
- 은행
- 일반 법인

BROKER_DEALER
- 증권사
- 선물사

HOUSE
- 증권사 자기계정

EXCHANGE_SIM
- EXTURE/KRX-Sim

DEPOSITORY_SIM
- KSD-Sim

ASSOCIATION_SIM
- KOFIA-Sim

REGULATOR_SIM
- FSS-Sim
```

---

## 6. 상품 구분

지원 대상:

```text
주식
채권
선물
RP
ETF
장외채권
장외주식 확장 가능
```

---

## 7. 시장 구분

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

---

## 8. 핵심 업무 흐름

---

## 8.1 개인 고객 장내 주식 매수

```text
[1] 개인 고객이 Retail Channel에서 주식 매수 주문
[2] PowerBase-Sim이 계좌/현금/상품/시장 상태 검증
[3] Ledger가 매수 예정금액 hold
[4] Order Routing이 주문 목적지 결정
[5] StockNet-Sim이 주문 전문 전달
[6] EXTURE-Sim이 주문 접수
[7] EXTURE-Sim Matching Engine이 체결
[8] StockNet-Sim이 체결 전문 전달
[9] PowerBase-Sim이 고객 주문/체결 상태 반영
[10] Clearing-Sim이 청산 데이터 생성
[11] Settlement가 KSD-Sim 결제지시 생성
[12] 결제일에 KSD-Sim이 증권 계좌대체
[13] PowerBase-Sim이 고객 현금/증권 원장 확정
[14] Reconciliation이 KSD-Sim 총량과 고객별 내부 원장 대사
```

---

## 8.2 기관투자가 장내 주식 주문

```text
[1] 자산운용사가 STP-HUB-Sim으로 주문 메시지 전송
[2] STP-HUB-Sim이 증권사로 주문 라우팅
[3] 증권사 K-FRONT-Sim 또는 PowerBase-Sim이 주문 수신
[4] Order Routing이 시장/상품/한도 검증
[5] StockNet-Sim을 통해 EXTURE-Sim으로 주문 전송
[6] EXTURE-Sim에서 체결
[7] 체결 결과가 StockNet-Sim을 통해 역방향 전달
[8] STP-HUB-Sim이 기관에 체결결과/매매보고서 중계
[9] PowerBase-Sim이 원장/결제/보고 처리
```

---

## 8.3 증권사 자기계정 장내 매매

```text
[1] 트레이더가 K-FRONT-Sim에서 자기계정 주문 입력
[2] K-FRONT-Sim이 주문 메시지 생성
[3] Order Routing/SOR-Sim이 목적지 결정
[4] StockNet-Sim이 EXTURE-Sim으로 주문 전달
[5] EXTURE-Sim에서 체결
[6] 체결결과 수신
[7] PowerBase-Sim이 HOUSE 계정 원장 반영
[8] 청산/결제/대사 처리
```

---

## 8.4 개인 고객 장외채권 매수

```text
[1] 증권사가 보유 채권을 판매상품으로 등록
[2] 개인 고객이 Retail Channel에서 장외채권 매수 신청
[3] PowerBase-Sim이 투자성향/상품위험도/현금 검증
[4] 채권 재고를 조건부 차감
[5] OTC Bond Trade 생성
[6] 고객 현금 결제예정/채권 입고예정 원장 posting
[7] KOFIA Disclosure-Sim에 장외채권 거래보고
[8] 필요 시 KSD-Sim에 계좌대체/예탁잔고 반영
[9] 결제 완료 후 고객 원장 확정
[10] KSD-Sim 총량과 내부 고객별 잔고 대사
```

---

## 8.5 증권사끼리 장외채권 거래

```text
[1] 증권사 A 딜러와 증권사 B 딜러가 수익률/가격/수량 협의
[2] FreeBond-Sim의 트레이딩보드/메신저를 통해 협의 기록 생성
[3] 양쪽 증권사가 OTC Trade Capture 입력
[4] PowerBase-Sim이 거래상대방/한도/잔고/결제일 검증
[5] 거래 확정
[6] KOFIA Disclosure-Sim에 장외채권 거래보고
[7] KSD-Sim에 DVP 결제지시
[8] A 증권사 계좌의 채권 감소
[9] B 증권사 계좌의 채권 증가
[10] B의 현금 감소, A의 현금 증가
[11] 양쪽 PowerBase 원장 확정
[12] 예탁잔고/내부 원장/보고내역 대사
```

---

## 8.6 채권 이자 지급

```text
[1] Bond Info-Sim이 이자지급 스케줄 관리
[2] 기준일 보유자 스냅샷 생성
[3] KSD-Sim이 권리관리 이벤트 생성
[4] PowerBase-Sim이 고객별 보유량 기준 이자 배분
[5] 세금 계산
[6] 고객 현금 원장 입금 posting
[7] 이자 지급 결과 대사
```

---

## 8.7 채권 만기 상환

```text
[1] 만기일 도래
[2] KSD-Sim이 상환 이벤트 생성
[3] PowerBase-Sim이 고객별 보유 수량 계산
[4] 원금 상환금 계산
[5] 채권 잔고 감소 posting
[6] 현금 입금 posting
[7] 채권 상태 REDEEMED 처리
[8] 대사
```

---

## 8.8 선물 거래

```text
[1] 기관 또는 자기계정이 K-FRONT-Sim에서 선물 주문 입력
[2] Order Routing이 증거금 가능 여부 확인
[3] StockNet-Sim을 통해 EXTURE-Sim DERIVATIVES 시장으로 주문 전송
[4] 체결 발생
[5] PowerBase-Sim이 미결제약정 생성
[6] 초기증거금 hold
[7] 일일정산 배치 실행
[8] 평가손익/변동증거금 posting
[9] 증거금 부족 시 margin call
```

---

## 8.9 RP 거래

```text
[1] RP 계약 생성
[2] 기초 채권 담보 지정
[3] start leg 실행
    - 현금 이동
    - 담보 채권 이전 또는 lock
[4] repo rate 기준 이자 계산
[5] end leg 실행
    - 원금 + 이자 반환
    - 담보 채권 반환
[6] 원장 확정
[7] 대사
```

---

## 9. 원장 설계 원칙

STBase의 핵심은 주문이 아니라 원장입니다.

### 9.1 잔고 직접 수정 금지

나쁜 방식:

```sql
UPDATE account_balance
SET cash = cash - 1000000
WHERE account_id = 1;
```

좋은 방식:

```text
ledger_journal 생성
ledger_posting 생성
balance projection 갱신
audit_log 생성
```

### 9.2 원장 posting 불변

```text
posting 생성 후 수정 금지
posting 삭제 금지
오류 발생 시 reversal posting 생성
보정은 correction posting으로 처리
```

### 9.3 Pending과 Settled 분리

현금:

```text
settled_cash
available_cash
held_cash
pending_receivable
pending_payable
```

증권:

```text
settled_quantity
available_quantity
held_quantity
pending_in_quantity
pending_out_quantity
```

---

## 10. 정합성 정책

```text
모든 상태변경 API는 idempotency key 필수
모든 전문 송수신은 external_message_log 기록
모든 모듈 간 상태 전이는 correlationId 사용
외부기관 시뮬레이션 호출은 outbox 기반
분산 트랜잭션 금지
Saga 상태 명시
중복 체결 전문 수신 시 중복 반영 금지
결제 실패는 숨기지 않고 상태로 남김
대사 불일치는 자동 수정하지 않고 exception 처리
운영자 수동 수정 금지
운영자 보정은 별도 correction command로 처리
```

---

## 11. 주요 테이블

```text
members
accounts
securities
stocks
bonds
futures
rp_contracts

orders
order_routes
executions
trades
otc_bond_trades
rp_trades

ledger_journals
ledger_postings
cash_balance_snapshots
security_balance_snapshots

settlement_instructions
settlement_legs
settlement_results

external_message_logs
outbox_events
idempotency_keys
audit_logs

reconciliation_jobs
reconciliation_breaks

kofia_otc_bond_reports
fss_business_reports
fss_disclosure_documents

ksd_depository_accounts
ksd_positions
ksd_book_entry_transfers
```

---

## 12. 운영자 관점

운영자는 다음을 볼 수 있어야 합니다.

```text
오늘 주문 건수
오늘 체결 건수
미체결 주문
미결제 거래
결제 실패
전문 송수신 실패
전문 중복 수신
outbox backlog
KOFIA 보고 실패
FSS 보고 실패
KSD 결제 실패
원장 불일치
대사 exception
배치 실패
수동 재처리 대상
감사로그
```

운영자는 데이터를 직접 수정하면 안 됩니다.

```text
직접 UPDATE 금지
직접 DELETE 금지
수동 재처리 command 허용
보정 posting 허용
모든 운영자 행위 audit log 필수
```

---

## 13. 기술 스택

```text
Java 17+
Spring Boot
Spring Data JPA
MySQL
Gradle Multi Module
Spring Validation
Spring Security
Spring Batch
Flyway or Liquibase
JUnit 5
Testcontainers
OpenAPI/Swagger
Docker Compose
```

선택 확장:

```text
Redis
Kafka or RabbitMQ
Prometheus/Grafana
OpenSearch/ELK
```

---

## 14. 개발 로드맵

### Phase 1 - PowerBase Core

```text
member
account
product
ledger
idempotency
audit log
```

### Phase 2 - 장외채권 MVP

```text
bond master
house inventory
OTC bond trade
KOFIA disclosure simulation
KSD position simulation
settlement
reconciliation
```

### Phase 3 - EXTURE/KRX 장내 MVP

```text
market session
order routing
stocknet simulation
order book
matching
execution report
clearing instruction
```

### Phase 4 - 개인/기관/트레이더 채널 분리

```text
retail channel
branch terminal
K-FRONT simulation
STP-HUB simulation
```

### Phase 5 - 결제/권리관리 강화

```text
DVP
coupon payment
bond redemption
stock dividend
corporate action
```

### Phase 6 - 복수거래시장/SOR

```text
ATS-Sim
SOR-Sim
best execution rule
routing evidence
market data comparison
```

### Phase 7 - 선물/RP

```text
futures margin
daily settlement
margin call
RP start leg
RP end leg
collateral valuation
```

### Phase 8 - 운영/감독

```text
FSS-Sim
business report
inspection event
ops dashboard
reconciliation console
manual retry
```

---

## 15. 프로젝트 한 줄 요약

> **STBase는 PowerBase-Sim을 중심으로 K-FRONT-Sim, STP-HUB-Sim, StockNet-Sim, EXTURE-Sim, KSD-Sim, KOFIA FreeBond/Disclosure-Sim, FSS-Sim을 멀티모듈로 구현하여 개인·기관·증권사 거래의 주문, 체결, 청산, 결제, 원장, 보고, 대사 흐름을 재현하는 모의 증권업무 백엔드 시스템이다.**

---

## 16. 공식 자료 참고

- Koscom PowerBase  
  https://www.koscom.co.kr/portal/main/contents.do?menuNo=200307

- Koscom 시장시스템 개발운용 / EXTURE+  
  https://www.koscom.co.kr/portal/main/contents.do?menuNo=200295

- Koscom STP-HUB  
  https://www.koscom.co.kr/portal/main/contents.do?menuNo=200617

- Koscom K-FRONT  
  https://www.koscom.co.kr/portal/bbs/B0000037/view.do?menuNo=200433&nttId=729

- Koscom FreeBond  
  https://www.koscom.co.kr/portal/bbs/B0000037/view.do?menuNo=200433&nttId=709

- Koscom BOND CHECK  
  https://m.koscom.co.kr/mobile/bbs/B0000064/view.do?menuNo=400107&nttId=30036

- Koscom SOR / 복수거래시장 대응  
  https://www.koscom.co.kr/portal/bbs/B0000064/view.do?nttId=30042
