# Airport-Project — BHS / PBB / Hold-Baggage-Screening Tender Tracker

전세계 공항 건설·개선 사업의 **수하물처리시설(BHS)**, **탑승교(PBB)**, 그리고 BHS 연계
**보안검색장비(EDS · CBIS · Hold Baggage Screening)** 입찰공고를 주기적으로 추적하는 저장소.

이 저장소는 Claude 정기 실행(주간 루틴)의 **상태 저장소(state store)** 역할을 합니다.
매 실행마다 `ledger.json`을 읽어 이전 결과와 대조하고, **변동분만** 사용자에게 보고합니다.

## 파일 구조
- `ledger.json` — 추적 중인 모든 입찰의 기계판독용 원장(상태·날짜·금액·링크·첨부·first_seen/last_updated)
- `reports/` — 각 실행일자별 스냅샷 보고서 (사람이 읽는 마크다운)

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
- **2026-08-17 (월)** — 2번째 실행(주중 누적). 4개 지역 병렬 조사. 신규 자본 PBB 3건(Calgary YYC·Detroit DTW P5·Cape Town A3/A4), CPK 대화참여사 5→2 압축, Patna 기간초과 재분류. 원장 26→39건. **금요일 전용 규칙에 따라 사용자 알림 미발송(상태 저장만).** egress 차단 지속.

## 운영 메모
- 이 루틴이 매 실행마다 이 저장소를 자동으로 읽어 대조하려면, Claude Code 웹의 해당
  **Environment 설정에서 이 저장소를 source로 지정**해야 합니다. (세션 내부에서는 설정 불가)
