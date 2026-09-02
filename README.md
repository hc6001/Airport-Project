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
- 실행 이력: **2026-08-17 (월)** — baseline 최초 실행. **2026-08-18 (화)** — 신규 9건 + 기존 2건 업데이트. **2026-08-19 (수)** — 신규 5건(DTW BHS장애, Journey Robotics, IAD 탑승교 상호충돌, 산타바바라 탑승교 붕괴, PWS 식수 안전성 연구) + 기존 1건 업데이트(버밍엄 BHX 복구 08-12). 밀라노 말펜사 수하물카트 화재는 GSE(견인카트)로 스코프 제외. **2026-08-20 (목)** — 신규 1건(AILA 산토도밍고 자동BHS 가동). **2026-08-23 (일, 08-21 금 누락분 소급)** — 신규 5건(인천공항 탑승교 사망사고 등) + 기존 1건 업데이트(브리즈번 2-8). **2026-08-24 (월)** — 신규 3건(HNL 수하물 보안검색장비 장애, Oshkosh AeroTech iOPS, ElectroAir EAC-PBB 카스텔리). 벤구리온 수하물지연(인력파업)·상하이 훙차오 재보도(중복)·인천공항 천장추락사(BHS/PBB 무관)는 스코프 제외. **2026-08-25 (화)** — 신규 3건(MCO 올랜도 터미널C 첨단BHS 기계결함, MCO $6.5억·10개년 BHS 전면개편 계획, MCO 견인차-탑승교 충돌 델타직원 사망 — 금주 최중요). 몬트리올 트뤼도 푸시백 지상직원 사망(BHS/PBB 무관)·HNL 08-24 후속보도(중복)는 스코프 제외. **2026-08-26 (수)** — 신규 항목 없음. 기존 항목 1건 업데이트(PBB 2-12 인천공항 탑승교 사망사고 — 고용노동부 중부지방고용노동청, 9월 중 인천공항공사·인천공항시설관리 검찰 송치 예정 보도). 벤구리온 컨베이어 정체(인력부족·날짜불명)·몬트리올 트뤼도 마샬링 사망(BHS/PBB 무관)·인천공항 철거작업 추락사(BHS/PBB 무관)·스포캔·피츠버그 PFAS 지하수오염(PWS 무관)·KLIA 네트워크장애(2019년 기사 오분류로 판명)는 스코프 제외. 금요일 아님 → 주간 집계 없음. **2026-08-27 (목)** — 신규 2건(PBB 장애: 인천공항 4단계 개항 직후 대한항공 B747 탑승교 오접안 날개파손 2025-01-07 — 기존 발굴 누락분 소급반영 / PBB부대설비 기술: AEME GSE S400-A 브릿지내장형 솔리드스테이트GPU+LVRTech). 기존 항목 변동 없음. 벵갈루루 KIA GPU-견인차 이탈충돌(2025년·무승객)·애틀랜타 탑승교 낙하(2023년)·산타바바라 탑승교붕괴 재보도(기존과 동일건)·자카르타 수카르노하타 수하물장애설(비공식 사이트만 보도, 현지 공식언론 미확인)·벤구리온 8월 수하물지연(인력부족·설비무관)은 스코프 제외. 금요일 아님 → 주간 집계 없음. **2026-08-30 (일, 08-28 금 누락분 소급, 주간 롤업 포함)** — 신규 2건(BHS 기술: Azalea Robotics ARC1 완전자율 이동형 수하물 분류 코봇, 2026-01-21 출시 / PBB부대설비 기술: 오스트리아 정부 GPU·PCA 친환경전환 국고보조금 300만유로 공모 — PBB부대설비 최초의 국가정책 사례). 기존 항목 변동 없음. 나이지리아 라고스 NAHCO 수하물벨트로더(이동식GSE) 항공기엔진 충돌(승객무피해)·벤구리온/스페인 파업 재보도·애틀랜타 2023 재유통·산타바바라 재유통(기존과 동일건)·자카르타 수하물장애설(미채택 유지)·NAIA 마닐라 세부퍼시픽 지연(운영사 결함부인)은 스코프 제외. 주간 집계(08-24~08-30): 금주 신규 총 10건 중 최중요는 MCO 델타직원 사망(2-13), 그 다음 인천공항 4단계 탑승교 오접안(2-14, 기존 2-12와 함께 인력·절차 구조적 패턴 시사). **2026-08-31 (월)** — 신규 2건(BHS 장애: 이스트미들랜즈공항(EMA, 영국) 08-31 새벽 정전으로 출발 수하물처리시스템 마비 / BHS 기술: 가이아나 CJIA 신규 BHS — 체크인 전 수동검색→체크인 후 컨베이어 원격스크리닝 워크플로 재설계 + 전구간 CCTV·바디캠 추적성 강화). 기존 항목 변동 없음. PBB·PBB부대설비 신규 없음. BHS 블록 재정렬(장애 1-13 삽입, 기존 기술 1-13~1-26을 1-14~1-27로 이동, 신규 기술 1-28 말미 추가). 벤구리온 수하물적체(인력요인)·BEUMER-EU반독점소송(기간초과+비기술)·상하이 비상슬라이드오작동(BHS/PBB무관)·부카라만가 탑승교충돌(기간초과)·멕시코시티 보행육교붕괴(스코프밖)·인디고 GPU분리정전(기간초과)·KLIA 4월 재유통·마이애미 유리탑승교 재유통(2024)·PBB시장전망 홍보자료·HappyOrNot 만족도조사(비특정)·DHL 화물기 제동장치화재(무관)·이지젯 말펜사 APU-ZERO 재유통(2024~2025)은 스코프 제외. 금요일 아님 → 주간 집계 없음. **2026-09-01 (화)** — 검증 전용·무변동(신규 0건). BHS/PBB/PBB부대설비 전 블록 신규 없음. 검색 재노출 항목은 전부 기존 추적 항목(1-1·1-6·2-3·2-5·2-6·2-8·2-10·2-11·2-13·3-9 등)과 동일건으로 확인. 벤구리온 8월 수하물위기(인력요인 재확인)·Beumer-Vanderlande/Siemens EU항소(기업결합 분쟁, 비기술)·PBB/PCA/GPU 시장전망 홍보자료 다수는 스코프 제외. GSE Expo Europe 2026(9/15~17 리스본) 행사 개요만 확인, 신규 전시제품 발표는 미확인(개막 후 재확인 필요). 금요일 아님 → 주간 집계 없음(다음 09-04 금). 알림 미발송. **2026-09-02 (수)** — 검증 전용·무변동(신규 0건). 18건의 개별 검색(장애·신기술·제작사 발표·업계지·한국어 인천공항 후속보도 포함)으로 전 블록 재점검. 검토 후 제외: Gerald R. Ford공항(GRR) BHS 개장(신선도 미확인, 보류)·KLIA T1 Carousel F 08-31 착공(기존 1-25와 동일건 진행상황)·AEME S28/ECOTUG 배터리GPU(이동식 카트, 스코프 밖)·브리스톨 GPU화재(이동식 발전기)·벵갈루루 GPU-견인차 이탈(2025년·이동식GSE)·인천공항 천장추락사 재검색(BHS/PBB 무관)·오스틴 탑승교인접 사망(실제 2023년 사건 오분류)·Gatwick 수하물벨트 검색결과(실제 2019년 기사)·CIMC-Tianda/AIR Security Act(신규 진전 없음). 금요일 아님 → 주간 집계 없음(다음 09-04 금). 알림 미발송(신규 없음).

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
- **2026-08-31 (월)** — 11차 실행. 08-30 대비 **실질 변동 2건(신규 tracking_lead) + 저정보 리드 1건**. 신규 confirmed_in_scope 진행중 공고는 없음. (1) **NEW** `MY-KLIA-BHS-MAHB` — MAHB(말레이시아공항) KLIA 쿠알라룸푸르 BHS 전면교체, RM500~600M/3년(최대 RM1bn 배정)≈1,700~2,000억+. 'four-horse race': Bursa 상장 4곳(MMC·Fajarbaru·Bina Puri·T7 Global)+다국적 6곳(Siemens·Beumer·Daifuku·Toyo Kanetsu·Vanderlande·Pteris) 공동입찰 후보. 이사회 승인·컨설턴트 선정중이나 tender 세부 confidential → **공개 공고문 미확인** → tracking_lead. (2) **NEW** `DE-HAM-BHS-LEONARDO` — Leonardo, Hamburg 공항 BHS 개체·확장(T1+Plaza, 크로스벨트 MBHS, 2026-2029) 낙찰, 발표 2026-04-29·~EUR90M → 창 이내 → tracking_lead(낙찰). (3) NEW 저정보 `US-GEG-BHS-SPOKANE` — Spokane RFQ #26-44-1897 BHS Upgrade(사전자격·금액미공개·포털 차단)→status_unconfirmed. 기각: BEUMER×MAG 맨체스터(2017·창초과)·Leonardo 취리히 EUR150M(2018·창초과)·TSA Gold+ USD12.9bn IDIQ(체크포인트+인력 서비스 IDIQ, hold-baggage 장비 아님·스코프외)·Omaha 2026 O&M(기존 ledger 동일건)·Nepal AMC·KLIA T2 예비부품(유지보수)·TSA R&S 월드컵(체크포인트). 무변동: AAI Bagdogra(개찰·낙찰 여전 미게시)·CPK-PBB92·Changi T3 본BHS·TSA SEDS·ACSA·HIAL·Lucknow 등 전부. 금요일 아님 → 주간 집계 없음(다음 09-04 금). **알림 발송**(MAHB KLIA 대형 형성기회 + Leonardo Hamburg 창내 낙찰). (`reports/2026-08-31_run11.md`)
- **2026-09-01 (화)** — 12차 실행. 08-31 대비 **검증 전용·무변동(실질 변동 0건)**. 신규 confirmed_in_scope 진행중 입찰공고문 없음, 추적 진행중/예정 건 낙찰·상태 변동 없음, 재분류 없음. 재확인(무변동): PL-CPK-PBB92(6응찰/5대화, 낙찰 미정 Dec-2026/2027초)·Changi T3 본BHS(H2-2026 미발주)·MAHB KLIA(공개 NIT·낙찰 없음)·TSA SEDS-CB(마감 05-15, 낙찰 미확정)·AAI Bagdogra(개찰·낙찰 여전 미게시, aai.aero 차단)·Lucknow·CPK-Vanderlande·HAM-Leonardo·ACSA 등 전부. HIAL 노트 정밀화(분류 불변): 재공고 내부참조 HIA-26-016 = find-tender 012183-2026(2026-06-04 게시)로 확인, 동일 in-scope 건. 기각: MWAA/IAD RFP-21-22745 BHS HLCS(2021·창초과)·MWAA/IAD RFP-26-23095(BHS/PBB 아님·스코프 외). 금요일 아님 → 주간 집계 없음(다음 09-04 금). 알림 미발송(무변동). (`reports/2026-09-01_run12.md`)
- **2026-09-02 (수)** — 13차 실행. 09-01 대비 **검증 전용·무변동(실질 변동 0건)**. 신규 confirmed_in_scope 진행중 입찰공고문 없음, 추적 진행중/예정 건 낙찰·상태 변동 없음, 재분류 없음. 재확인(무변동): PL-CPK-PBB92(6응찰/5대화, 낙찰 미정)·Changi T3 본BHS(H2-2026 미발주, 조기수하물보관 65%확장+T1–T3 상호수하물연결)·Changi T3 56PBB(낙찰 미확인)·TSA SEDS-CB(마감 05-15, 낙찰 미확정)·AAI Bagdogra(개찰·낙찰 여전 미게시, aai.aero 차단)·Lucknow·Kenya Moi PBB(마감 04-30, 낙찰 미확인)·CPK-Vanderlande·HAM-Leonardo·ACSA·HIAL 등 전부. **데이터 품질 경고**: MY-KLIA-BHS-MAHB(tracking_lead, 비보고) — 'four-horse race' 보도가 klia2.info /news/2021/ URL에서 재노출, RM1bn+에어로트레인+4파전 프레이밍이 2021년 기사와 일치. theedgemalaysia·klia2.info egress 차단으로 2026 현재성 재확인 불가 → 신뢰도 하향, 08-31 리드가 2021 서사를 현재로 혼동했을 가능성. 분류 불변(tracking_lead), 다음 실행 검증 필요. 기각: Tonga Fua'amotu(보조금·소규모)·Jersey HBS(창/금액 미확인, 리드 보류)·말레이시아 BHS 06-22 마감 스니펫(공항 미귀속). 금요일 아님 → 주간 집계 없음(다음 09-04 금). 알림 미발송(무변동). (`reports/2026-09-02_run13.md`)
- **2026-08-23 (일)** — 5차 실행(08-21 금·08-22 토 미실행). 08-20 대비 변동분만. **저변동 실행 — 신규 진행중 공고 없음.** 실질 변동 2건: (1) 신규 낙찰리드 Heathrow LHR — Analogic SeleCT Standard-3 HBS/EDS 공급·설치(2026-03 낙찰, 맥락 기록·입찰리스트 미포함), (2) AAI 다공항 PBB+AVDGS 공개 NIT 실체 확인(node 646356, DMSITC 풀스코프; 날짜·EMD 여전히 egress 차단 → status_unconfirmed 유지). 그 외 CPK·TSA SEDS·Changi T3·ACSA·HIAL·Bagdogra(08-24 마감 임박, 변동 없음)·Goa·Lucknow·DWC·Kuwait 전부 동일. Asheville AVL·Minneapolis MAC 후보는 창 초과/유지보수로 기각. 금요일 아님 → 주간 집계 없음. egress 차단 지속. (`reports/2026-08-23_run5.md`)

## 운영 메모
- 이 루틴이 매 실행마다 이 저장소를 자동으로 읽어 대조하려면, Claude Code 웹의 해당
  **Environment 설정에서 이 저장소를 source로 지정**해야 합니다. (세션 내부에서는 설정 불가)
