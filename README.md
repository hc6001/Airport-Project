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
- 실행 이력: **2026-08-17 (월)** — baseline 최초 실행. **2026-08-18 (화)** — 신규 9건 + 기존 2건 업데이트.

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

## 운영 메모
- 이 루틴이 매 실행마다 이 저장소를 자동으로 읽어 대조하려면, Claude Code 웹의 해당
  **Environment 설정에서 이 저장소를 source로 지정**해야 합니다. (세션 내부에서는 설정 불가)
