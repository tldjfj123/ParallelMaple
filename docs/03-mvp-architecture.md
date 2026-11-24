# MVP 1차 구현 설계서 (MSW 아키텍처 & 작업 계획)

> **목표:** 헤네시스(마을) ↔ 헤네시스 사냥터 1(공용 사냥터)을 오가며 **자동사냥으로 메소·주문의 흔적·큐브·장비를 파밍하고, 3대 강화(주문서/스타포스/큐브)로 캐릭터를 강하게 만드는 풀 코어 루프**를 플레이 가능한 수준으로 완성합니다.

---

## 1. 맵 & 이동 구조 (MVP 1차)

| 맵 용도 | MSW 공식 맵 ID | 워크스페이스 맵 이름 | 역할 |
| :--- | :---: | :--- | :--- |
| **마을 (시작 맵)** | `100000000` (Henesys) | `TileMapTemplate` *(이미 임포트 완료)* | 안전지대, 장비/강화 정비, 사냥터 포탈 |
| **공용 사냥터 1** | `921170004` (Henesys Hunting Ground 1) | `RectTileMapTemplate` *(또는 신규 맵에 임포트)* | 달팽이·주황버섯·초록버섯 빠른 리젠, 자동사냥/수동사냥 |

- **이동 방법:** 마을 포탈 이동 + HUD 상단 **[사냥터 이동 / 마을 귀환]** 원클릭 버튼 병행 (방치형 편의성)

---

## 2. 스크립트 모듈 구조 (`workspaces/Jerry/RootDesk/MyDesk/`)

### 2.1 데이터 테이블 (공용 Logic / Script)
- `GameDataLogic.mlua`
  - 장비 8부위 마스터 데이터 (초보자 Lv.10 / 모험가 Lv.60 / 펜살리르 Lv.140 / 카루타 Lv.150)
  - 주문의 흔적 확률/보너스 테이블
  - 스타포스 0~25성 확률/비용/스탯 테이블
  - 큐브 5종 등업률/천장/부위별 잠재 옵션 풀 테이블
  - 경험치 요구량 & 전직(1차 Lv.10) 스킬 테이블

### 2.2 서버 핵심 로직 (`[server only]` 권한 검증)
- `PlayerDataLogic.mlua`
  - 플레이어 접속 시 `DataStorageService:GetUserDataStorage(userId)` 로드 (신규 유저면 기본 8부위 장비 + 초기 재화 지급)
  - 레벨/경험치/메소/주문의 흔적/큐브 보유량/장비 인벤토리/장착 슬롯 관리
  - 스탯 합산 (`RecalculateStats`) → 공격력, 주스탯, 보공, 방무, 크뎀, 메획, 드랍률, 전투력 계산 후 클라이언트로 동기화
  - 주기적 자동 저장(60초) + 강화/장착/퇴장 시 즉시 저장
- `EnhanceLogic.mlua`
  - `RequestScroll(uid, scrollType, repeatAll)` — 주문의 흔적 100/70/30/15%, 순백 10%, 이노센트 50%, 황금 망치
  - `RequestStarforce(uid, useSafeguard, starCatchSuccess)` — 0~25성 판정, 찬스타임, 파괴 시 장비의 흔적 전환
  - `RequestRestoreTrace(traceUid, spareUid)` — 동일 `itemId` 스페어 소모 → 12성 복구
  - `RequestCube(uid, cubeType)` — 윗잠재 등급업(천장 포함) 및 3줄 리롤, 블랙 큐브 시 `pendingPotential` 대기
  - `RequestConfirmBlackCube(uid, applyNew)` — 블랙 큐브 Before/After 확정
- `DropRewardLogic.mlua`
  - 몬스터 사망 시 기여자(`hitContributors`)별 개인 경험치·메소·주문의 흔적·수상한 큐브·장비 드랍 지급 + 클라이언트 플로팅 토스트/로그 알림

### 2.3 필드 전투 컴포넌트
- `AutoHuntComponent.mlua` (플레이어 부착)
  - 토글 ON 시 현재 맵 몬스터 탐색 → 좌우 이동/점프 추적 → 사거리 내 스킬 자동 시전
  - 방향키/공격키 입력 감지 시 2초간 수동 우선권 부여
- `FieldMonsterComponent.mlua` + `FieldSpawnerComponent.mlua`
  - 사냥터 발판 위 빠른 리젠(2.5초 주기, 최대 15~20마리)
  - 타격 시 `hitContributors[userId] = true` 기록 및 데미지 스킨 출력

### 2.4 클라이언트 UI (`workspaces/Jerry/ui/`)
- **상단/하단 HUD (`DefaultGroup.ui`):**
  - 레벨 / 경험치 바 / 전투력 / 메소 / 주문의 흔적 / 보유 큐브 표시
  - **[자동사냥 ON/OFF]** 토글, **[사냥터 이동 / 헤네시스 귀환]** 버튼, **[장비 & 강화]** 열기 버튼
- **장비 & 3대 강화 통합 창 (`UIEnhancePanel`):**
  - 좌측: 장착 8부위 슬롯 + 인벤토리 장비 목록(장비의 흔적 표시 포함)
  - 우측 상단: 선택 장비 툴팁 (이름, `+업횟`, `★성수`, `잠재 등급 테두리`, 옵션 3줄, 스탯 증가치)
  - 우측 하단 탭 3개:
    1. **[주문서]** — 100/70/30/15% + 순백/이노센트/황망 + [1회 / 연속 강화]
    2. **[스타포스]** — 현재 성수→다음 성수, 성공/유지/하락/파괴 확률, 비용, 파괴방지 체크, 스타캐치 토글, 파괴 시 [스페어 전승 복구] 버튼
    3. **[큐브]** — 큐브 선택(수상/장인/명장/레드/블랙), 현재 천장 게이지(`N / Max`), 블랙 큐브 시 `BEFORE vs AFTER` 비교 선택 버튼

---

## 3. MVP 1차 구현 순서 (단계별 검증)

1. **Step 1 — 사냥터 맵 세팅:** `Henesys Hunting Ground 1 (921170004)` 임포트 + 헤네시스 ↔ 사냥터 이동 연결
2. **Step 2 — 데이터 & 저장 코어:** `GameDataLogic` + `PlayerDataLogic` 작성, 플레이어 접속 시 기본 8부위 장비/재화 지급 및 HUD 동기화
3. **Step 3 — 자동사냥 & 파밍 루프:** `FieldSpawnerComponent` + `FieldMonsterComponent` + `AutoHuntComponent` 구현 및 실제 사냥/드랍/레벨업 Play 테스트
4. **Step 4 — 3대 강화 서버 로직 & UI:** `EnhanceLogic` + `UIEnhancePanel` (주문서 / 스타포스 / 큐브) 구현 및 Play 테스트 (빌드 에러/런타임 로그 0건 검증)
