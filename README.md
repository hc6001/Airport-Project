# Airport-Project — BHS / PBB / Hold-Baggage-Screening Tender Tracker

전세계 공항 건설·개선 사업의 **수하물처리시설(BHS)**, **탑승교(PBB)**, 그리고 BHS 연계
**보안검색장비(EDS · CBIS · Hold Baggage Screening)** 입찰공고를 주기적으로 추적하는 저장소.

이 저장소는 Claude 정기 실행 루틴들의 **상태 저장소(state store)** 역할을 합니다.
매 실행마다 원장 파일을 읽어 이전 결과와 대조하고, **변동분만** 사용자에게 보고합니다.

이 저장소는 두 개의 독립적인 정기 루틴을 저장합니다.

## ⚠️ 필수: 매 실행 마지막 단계 — `main`으로 병합
각 정기 실행은 **매번 새 세션 + 새 브랜치(무작위 이름)** 로 시작되며, 그 브랜치는 그 시점의
`main`에서 새로 분기합니다. 즉 **`main`에 반영되지 않은 커밋은 다음 실행에서 보이지 않습니다**
(2026-08-17 최초 실행 시 이 문제로 동향 추적 루틴이 매번 "baseline"으로 재시작되는 버그가 있었음 — 원인:
그날 커밋이 작업 브랜치에만 있고 `main`에 병합되지 않았기 때문. 2026-08-18에 수정.)

따라서 두 루틴 모두, **작업(원장 파일 갱신 + 보고서 커밋)을 마친 직후, 세션 종료 전에 반드시**:
1. `git fetch origin main`으로 최신 `main`을 확인
2. 현재 작업 브랜치가 `main`의 조상(ancestor)인지 확인(`git merge-base --is-ancestor origin/main HEAD`)
3. 문제없으면 `git push origin <현재브랜치>:main` 으로 fast-forward 반영
   (충돌 등으로 fast-forward가 안 되면 `main`을 먼저 병합/리베이스해서 해결 후 반영)

이 단계를 건너뛰면 다음 실행이 이전 상태를 전혀 못 보고 매번 baseline으로 재시작됩니다.

## 루틴 1) 입찰 추적 (주간)
- `ledger.json` — 추적 중인 모든 입찰의 기계판독용 원장(상태·날짜·금액·링크·첨부·first_seen/last_updated)
- `reports/` — 각 실행일자별 스냅샷 보고서 (사람이 읽는 마크다운)

## 루틴 2) BHS·PBB 장애사례·신기술 동향 추적 (평일 데일리, 금요일 주간 집계)
- `trends_ledger.json` — 추적 중인 모든 사고사례/신기술 동향의 기계판독용 원장(넘버링·날짜·링크·first_seen/last_updated)
- `reports/trends/` — 각 실행일자별 스냅샷 보고서 (사람이 읽는 마크다운)
- 넘버링 규칙: `1-x`=BHS, `2-x`=PBB, `3-x`=PBB 부대설비(PC-Air/PWS/GPU). 각 블록 내 장애사례를 먼저, 신기술 동향을 뒤에 배치.
- 실행 이력: **2026-08-17 (월)** — baseline 최초 실행. **2026-08-18 (화)** — 신규 9건 + 기존 2건 업데이트. **2026-08-19 (수)** — 신규 5건(DTW BHS장애, Journey Robotics, IAD 탑승교 상호충돌, 산타바바라 탑승교 붕괴, PWS 식수 안전성 연구) + 기존 1건 업데이트(버밍엄 BHX 복구 08-12). 밀라노 말펜사 수하물카트 화재는 GSE(견인카트)로 스코프 제외. **2026-08-20 (목)** — 신규 1건(AILA 산토도밍고 자동BHS 가동). **2026-08-23 (일, 08-21 금 누락분 소급)** — 신규 5건(인천공항 탑승교 사망사고 등) + 기존 1건 업데이트(브리즈번 2-8). **2026-08-24 (월)** — 신규 3건(HNL 수하물 보안검색장비 장애, Oshkosh AeroTech iOPS, ElectroAir EAC-PBB 카스텔리). 벤구리온 수하물지연(인력파업)·상하이 훙차오 재보도(중복)·인천공항 천장추락사(BHS/PBB 무관)는 스코프 제외. **2026-08-25 (화)** — 신규 3건(MCO 올랜도 터미널C 첨단BHS 기계결함, MCO $6.5억·10개년 BHS 전면개편 계획, MCO 견인차-탑승교 충돌 델타직원 사망 — 금주 최중요). 몬트리올 트뤼도 푸시백 지상직원 사망(BHS/PBB 무관)·HNL 08-24 후속보도(중복)는 스코프 제외. **2026-08-26 (수)** — 신규 항목 없음. 기존 항목 1건 업데이트(PBB 2-12 인천공항 탑승교 사망사고 — 고용노동부 중부지방고용노동청, 9월 중 인천공항공사·인천공항시설관리 검찰 송치 예정 보도). 벤구리온 컨베이어 정체(인력부족·날짜불명)·몬트리올 트뤼도 마샬링 사망(BHS/PBB 무관)·인천공항 철거작업 추락사(BHS/PBB 무관)·스포캔·피츠버그 PFAS 지하수오염(PWS 무관)·KLIA 네트워크장애(2019년 기사 오분류로 판명)는 스코프 제외. 금요일 아님 → 주간 집계 없음.

## 스크리닝 규칙
- 대상: 실제 **입찰공고문**이 공개 확인된 건만 `confirmed_in_scope`. 뉴스·발표만 있는 건은 `tracking_lead`.
- 금액 임계값(KRW): **BHS 50억 미만 제외**, **PBB 10억 미만 제외** → `excluded_low_amount` (사업명·링크만)
- 마감 건은 **공고일 1년 이내**만 유효 → 초과 시 `excluded_out_of_window`
- 보고 순서: 1) 진행중 → 2) 예정 → 3) 마감 → 4) 금액 미달 제외

## 카테고리(category)
| 값 | 의미 |
|---|---|
| `confirmed_in_scope` | 공고문 확인 + 스코프/금액 조건 충족 (진행중·예정·마감) |
| `excluded_low_amount` | 임계값 미달 (사업명·링크만 공유) |
| `excluded_out_of_window` | 공고일 1년 초과 마감 건 |
| `tracking_lead` | 실제 공고문 미공개, 다음 루틴 추적 대상 |

## 실행 이력
- **2026-08-03 (월)** — baseline 최초 실행. 이전 상태 없어 전량 기록. (주의: 실행 환경 egress 정책으로 조달포털 첨부파일 직접 다운로드는 불가 → 첨부 접근성은 추정치)
- **2026-08-18 (화)** — 2차 실행. baseline 대비 변동분만 보고. 신규 확정공고(AAI Agra/Leh/PBB-AVDGS, 케냐 모이 PBB, 창이 T3 56-PBB, PHL D/E make-up, 캘거리 YYC, 위니펙 PBB) + 주요 변동(HIAL 재공고 OPEN, ACSA 전국 HBS ZAR31.5억·낙찰 법원동결, AAI Bagdogra OPEN 마감08-24, CPK 5개사 대화진출). egress 차단 지속 → 첨부 접근성 여전히 추정. (`reports/2026-08-18_run2.md`)
- **2026-08-19 (수)** — 3차 실행. 08-18 대비 변동분만. 신규 확정 BHS 공고 2건(AAI Goa Dabolim 추가BHS+Vanderlande 통합 / AAI Lucknow BHS, EMD ₹1.4cr→임계값 상회). 그 외 CPK·Changi T3·ACSA·Bagdogra·HIAL 전부 변동 없음. Billings(2021)·Albany(2024) PBB는 창 초과로 제외. 금요일 아님 → 주간 집계 없음. egress 차단 지속. (`reports/2026-08-19_run3.md`)
- **2026-08-20 (목)** — 4차 실행. 08-19 대비 변동분만. **US-TSA-SEDS-CB** 실체 확인 → 재공고 `70T04026R7672N004`(2026-04-03 공고 / 2026-05-15 마감) → status_unconfirmed → closed. 신규 소액제외 2건(Changi T3 HT장비 부품, Wilmington ILM PBB+BHS 정비). 신규 추적리드 3건(AAI Solapur BHS, MIA BHS 설계자문 PSA, IND EDS/BHS 개조안). 그 외 CPK·Changi T3 본 BHS·ACSA·HIAL·Bagdogra·Goa·Lucknow 모두 변동 없음. 금요일 아님 → 주간 집계 없음. egress 차단 지속. (`reports/2026-08-20_run4.md`)
- **2026-08-24 (월)** — 6차 실행. 08-23 대비 **변동 0건(무변동 실행)**. 신규 스코프 내 공고 없음, 추적항목 상태변동 없음. AAI Bagdogra 제출 마감 오늘(08-24) 도달했으나 개찰·낙찰 공고 미게시(aai.aero egress 차단). TED 237844/259958/238520-2026(핸들링서비스·자동문)·Albany PBB(2024·창초과)·CVG·AVL 전부 스코프 외/기배제로 기각. 금요일 아님 → 주간 집계 없음. egress 차단(aai.aero·ted·etenders.gov.za·highergov·bidnetdirect) 지속. 사용자 알림 미발송(신규 없음). (`reports/2026-08-24_run6.md`)
- **2026-08-25 (화)** — 7차 실행. 08-24 대비 **실질 변동 2건(둘 다 하향 재분류)**. 신규 스코프 내 진행중 공고 없음. (1) AAI Agra BHS — 실제 추정가 ₹21.66cr(≈34.6억) 확정 → 50억 미만 → `excluded_low_amount` 재분류. (2) AAI Goa Dabolim 추가 BHS — EMD ₹24.83lakh + 입찰능력 ₹8.28cr ⇒ 추정 ~₹12cr(≈20억) → 50억 미만 → `excluded_low_amount` 재분류. Atlanta ATL FC-8676(2016·창초과)·PVD Inline EDS BHS(값 미확인)·MIA PBB Phase-2(공고 미공개)·TED 신규(핸들링/스코프외) 전부 기각/보류. Bagdogra 개찰·낙찰 공고 여전히 미게시(aai.aero 차단), Lucknow(~₹70cr) 스코프 유지. 금요일 아님 → 주간 집계 없음. 알림 미발송(신규 유효 기회 없음). (`reports/2026-08-25_run7.md`)
- **2026-08-26 (수)** — 8차 실행. 08-25 대비 **검증 전용·무변동**. 신규 스코프 내 공고·낙찰·분류 변경 없음. (1) AAI Goa(Dabolim) 추가 BHS 금액을 NIT_3405 직접 조회로 ₹8.28cr(≈13억) 확정 → `excluded_low_amount` 유지(분류 불변). (2) Patna·Goa·Bagdogra 검색에 반복된 "₹114.93cr/22개월" 스니펫은 검색엔진 캐시 아티팩트로 판정 → 재분류 근거로 미채택. (3) AAI Bagdogra 개찰·낙찰 공고 여전히 미게시(aai.aero 차단). CPK(6응찰/5대화, Dec-2026)·Lucknow(~₹70cr)·TSA SEDS·Changi T3·ACSA·HIAL 전부 변동 없음. 금요일 아님 → 주간 집계 없음. 알림 미발송. (`reports/2026-08-26_run8.md`)
- **2026-08-27 (목)** — 9차 실행. 08-26 대비 **실질 변동 2건**. (1) **NEW backfill** `PL-CPK-BHS-VANDERLANDE` — CPK 바르샤바 신공항 여객터미널 BHS 낙찰: **Vanderlande 컨소시엄** 선정, 발표 2025-10-30, EUR 115M(공급) + PLN 97M(5년 O&M) + PLN 93M(옵션 5년) ⇒ 합계 약 2,150억+원. ICS 토트 방식, ~80,000㎡, >16km. 창(12개월) 이내(2026-10-30 만료) → `tracking_lead`(awarded). baseline 당시 CPK 검색이 PBB92 위주여서 누락되었던 항목의 backfill. (2) **NEW 저금액** `US-TPA-BHS-REXEL` — Tampa TPA와 Rexel USA(Rockwell 독점대리점) 간 BHS 하드웨어/소프트웨어 지원 sole-source 계약(2026-03~2031-02), 경쟁 입찰 아님·소액 → `excluded_low_amount`(명·링크만). (3) AAI Bagdogra 개찰·낙찰 여전히 미게시(aai.aero 차단). CPK-PBB92·TSA SEDS-CB·Changi T3·ACSA·HIAL·AAI Lucknow/PBB-AVDGS/Patna/Varanasi/Leh·CIAL Kochi CT-HBS 전부 변동 없음. 인도 62개 공항 국가 지침(2026-07)은 개별 NIT 없음 → 스코프 외. 금요일 아님 → 주간 집계 없음(내일 08-28 금요일 예정). **알림 발송**(CPK BHS Vanderlande backfill의 창 내 낙찰). (`reports/2026-08-27_run9.md`)
- **2026-08-23 (일)** — 5차 실행(08-21 금·08-22 토 미실행). 08-20 대비 변동분만. **저변동 실행 — 신규 진행중 공고 없음.** 실질 변동 2건: (1) 신규 낙찰리드 Heathrow LHR — Analogic SeleCT Standard-3 HBS/EDS 공급·설치(2026-03 낙찰, 맥락 기록·입찰리스트 미포함), (2) AAI 다공항 PBB+AVDGS 공개 NIT 실체 확인(node 646356, DMSITC 풀스코프; 날짜·EMD 여전히 egress 차단 → status_unconfirmed 유지). 그 외 CPK·TSA SEDS·Changi T3·ACSA·HIAL·Bagdogra(08-24 마감 임박, 변동 없음)·Goa·Lucknow·DWC·Kuwait 전부 동일. Asheville AVL·Minneapolis MAC 후보는 창 초과/유지보수로 기각. 금요일 아님 → 주간 집계 없음. egress 차단 지속. (`reports/2026-08-23_run5.md`)

## 운영 메모
- 이 루틴이 매 실행마다 이 저장소를 자동으로 읽어 대조하려면, Claude Code 웹의 해당
  **Environment 설정에서 이 저장소를 source로 지정**해야 합니다. (세션 내부에서는 설정 불가)
