# ParallelMaple v1.0 요구사항 명세 인덱스 (SRS & Ticket Index)

> **목적:** 2인 풀 바이브코딩 시 각 AI 에이전트가 **티켓 파일 1개만 읽고도 충돌 없이 원샷 구현·검증**할 수 있도록 모든 요구사항을 **도메인(9개) × 작업 유형(5개)** 으로 원자화했습니다.
>
> **AI 작업 지시 예시:** `"AGENTS.md와 docs/tickets/REQ-STAT-01.md를 읽고 구현해줘."`

---

## 1. 작업 유형 분류 (`Type`) — 충돌 방지 기준

| 태그 | 영역 | 주로 수정하는 파일 | 협업 규칙 |
| :---: | :--- | :--- | :--- |
| **`[FUNC]`** | 기능 · 서버 로직 | `RootDesk/MyDesk/*Logic.mlua`, `*Component.mlua` | 순수 mLua 로직, RPC/보안 검증 필수 |
| **`[UI]`** | UI 레이아웃 & 컨트롤러 | `ui/*.ui`, `UI*.mlua` | `.ui` 파일 단독 소유 필수 |
| **`[DATA]`** | 마스터 데이터 · RUID · 밸런스 | `*DataLogic.mlua`, `docs/03-data-reference.md` | `msw-mcp`로 RUID 시각 검증 필수 |
| **`[MAP/FX]`** | 맵 · 스폰 · 이펙트/사운드 | `map/*.map`, `*VisualLogic.mlua` | `.map` 파일 절대 동시 수정 금지 |
| **`[QA]`** | 통합 검증 · 마이그레이션 · 릴리즈 | 테스트 체크리스트 & 밸런스 튜닝 | Play Mode 콘솔 에러 0건 확인 |

---

## 2. 전체 요구사항 티켓 목록 (총 26개 — 도메인별)

| 티켓 ID | 유형 | 우선순위 | 제목 | 선행 티켓 | 주요 수정 파일 (Lock) | 상태 |
| :--- | :---: | :---: | :--- | :--- | :--- | :---: |
| [REQ-ARCH-01](tickets/REQ-ARCH-01.md) | `[DATA]` | P0 | `GameDataLogic` 3모듈 분리 (파사드 유지) | 없음 | `GameDataLogic`, `EquipDataLogic`, `EnhanceDataLogic`, `WorldDataLogic` | `[ ]` |
| [REQ-ARCH-02](tickets/REQ-ARCH-02.md) | `[UI]` | P0 | `UIIdleManager` 4모듈 분리 (라우터+3컨트롤러) | 없음 | `UIIdleManager`, `UICharacterWindows`, `UIEnhanceController`, `UINavigationWindows` | `[ ]` |
| [REQ-ARCH-03](tickets/REQ-ARCH-03.md) | `[FUNC]` | P0 | `Profile` v3 스키마 & `v2→v3` 마이그레이션 (`DateTime.UtcNow`) | `ARCH-01` | `PlayerDataLogic` | `[ ]` |
| [REQ-STAT-01](tickets/REQ-STAT-01.md) | `[FUNC]` | P0 | 수동 AP 배분(`STR/DEX/INT/LUK`) & `(STR×4+DEX)`·맨손 공식 | `ARCH-03` | `PlayerDataLogic`, `PlayerAttack` | `[ ]` |
| [REQ-STAT-02](tickets/REQ-STAT-02.md) | `[UI]` | P1 | 스탯창(`S`) AP 배분 UI(`[+]`/`[자동]`) · 세트 · 자동포션 표시 | `ARCH-02`, `STAT-01` | `UICharacterWindows`, `IdleHUDGroup.ui` | `[ ]` |
| [REQ-EQUIP-01](tickets/REQ-EQUIP-01.md) | `[DATA]` | P0 | KMS Lv.10~200 전사 전체 장비 DB & `msw-mcp` RUID 검증 | `ARCH-01` | `EquipDataLogic`, `docs/03-data-reference.md` | `[ ]` |
| [REQ-EQUIP-02](tickets/REQ-EQUIP-02.md) | `[FUNC]` | P0 | 24부위 풀 슬롯 장착 규칙 · `reqLevel` 강제 · 무지급 · 펜살리르 상한 드랍 | `ARCH-03`, `EQUIP-01` | `PlayerDataLogic`, `EquipDataLogic` | `[ ]` |
| [REQ-EQUIP-03](tickets/REQ-EQUIP-03.md) | `[FUNC]` | P1 | 장비 세트 효과 서버 합산 (`보장/펜살/카루타/앱솔/아케인`) | `EQUIP-02` | `EquipDataLogic`, `PlayerDataLogic` | `[ ]` |
| [REQ-EQUIP-04](tickets/REQ-EQUIP-04.md) | `[UI]` | P1 | 장비창(`E`) 24슬롯 격자 개편 + 세트 배지 + 강화창 24슬롯 연동 | `ARCH-02`, `EQUIP-02` | `UICharacterWindows`, `UIEnhanceController`, `IdleHUDGroup.ui` | `[ ]` |
| [REQ-INVEN-01](tickets/REQ-INVEN-01.md) | `[FUNC]` | P1 | 인벤 60칸 · 잠금(`locked`) · 개별 판매/분해 · 안전 일괄 분해 | `ARCH-03` | `InventoryLogic` *(신규)*, `EnhanceLogic` | `[ ]` |
| [REQ-INVEN-02](tickets/REQ-INVEN-02.md) | `[UI]` | P1 | 인벤창(`I`) 60칸 · 잠금/판매/분해 버튼 · 일괄 분해 확인 모달 | `ARCH-02`, `INVEN-01` | `UICharacterWindows` | `[ ]` |
| [REQ-SHOP-01](tickets/REQ-SHOP-01.md) | `[DATA]` | P1 | NPC 상점 마스터 테이블 (잡화/강화/보조·엠블렘/펫·자석펫/보스코인) | `ARCH-01`, `EQUIP-01` | `WorldDataLogic` | `[ ]` |
| [REQ-SHOP-02](tickets/REQ-SHOP-02.md) | `[FUNC]` | P1 | NPC 상점 · 펫 1→3 확장 & 자석펫 해금 서버/펫 연동 | `ARCH-03`, `SHOP-01` | `ShopLogic` *(신규)*, `CombatVisualLogic` | `[ ]` |
| [REQ-SHOP-03](tickets/REQ-SHOP-03.md) | `[UI]` | P1 | NPC 상점창(`B`) 3탭 UI (`ShopUIGroup.ui`) | `ARCH-02`, `SHOP-02` | `UIShopController` *(신규)*, `ShopUIGroup.ui` *(신규)*, `AutoHuntComponent` | `[ ]` |
| [REQ-HUNT-01](tickets/REQ-HUNT-01.md) | `[MAP/FX]` | P0 | 사냥터 18개 `InstanceMap` 설정 & 1인 개인 사냥터 `RoomService` 전이 | `ARCH-03` | `HuntingInstanceLogic` *(신규)*, `map/HenesysHunt1.map`, `map/Hunt01~17.map` | `[ ]` |
| [REQ-HUNT-02](tickets/REQ-HUNT-02.md) | `[FUNC]` | P1 | 18챕터×5스테이지(90스테이지) 스케일링 · 3,000마리 · 챕터 보스(60초) | `HUNT-01` | `HuntingInstanceLogic`, `FieldSpawnerLogic`, `WorldDataLogic` | `[ ]` |
| [REQ-HUNT-03](tickets/REQ-HUNT-03.md) | `[FUNC]` | P1 | 사냥 도파민 이벤트 4종 (엘몹 100마리 · 룬 2배 · 30분 버닝 · 도파민 상자) | `HUNT-02` | `HuntingEventLogic` *(신규)*, `Monster` | `[ ]` |
| [REQ-HUNT-04](tickets/REQ-HUNT-04.md) | `[UI]` | P1 | 길라잡이(`M`) 챕터/버닝 개편 + 상단 진행바(`X/3,000`) & 도파민 상자 팝업 | `ARCH-02`, `HUNT-02`, `HUNT-03` | `UINavigationWindows`, `IdleHUDGroup.ui` | `[ ]` |
| [REQ-SURV-01](tickets/REQ-SURV-01.md) | `[FUNC]` | P1 | HP 50% 자동 포션(1초 쿨, 25% 파엘 우선) & 무패널티 헤네시스 부활 | `ARCH-03` | `SurvivalLogic` *(신규)*, `PlayerHit`, `PlayerDataLogic` | `[ ]` |
| [REQ-SURV-02](tickets/REQ-SURV-02.md) | `[FUNC]` | P1 | `DateTime.UtcNow` 기반 최대 12시간 오프라인 방치 정산 & 중복 방지 | `ARCH-03` | `OfflineRewardLogic` *(신규)*, `PlayerDataLogic` | `[ ]` |
| [REQ-SURV-03](tickets/REQ-SURV-03.md) | `[UI]` | P1 | 오프라인 방치 정산 팝업 (`OfflinePopupGroup.ui`) | `ARCH-02`, `SURV-02` | `UIOfflinePopup` *(신규)*, `OfflinePopupGroup.ui` *(신규)* | `[ ]` |
| [REQ-BOSS-01](tickets/REQ-BOSS-01.md) | `[FUNC]` | P1 | 1인 레이드 보스 5단계 코어 (`BossLogic`: KST 00:00, 10분, 데카 5) & 방무/보공 | `ARCH-03`, `EQUIP-01` | `BossLogic` *(신규)*, `WorldDataLogic`, `PlayerAttack` | `[ ]` |
| [REQ-BOSS-02](tickets/REQ-BOSS-02.md) | `[FUNC]` | P1 | 보스 몬스터 컴포넌트 & 5종 시그니처 패턴 AI (자쿰~루시드) | `BOSS-01` | `BossMonster` *(신규)* | `[ ]` |
| [REQ-BOSS-03](tickets/REQ-BOSS-03.md) | `[MAP/FX]` | P1 | 보스 아레나 맵 5종(`Boss01~05.map`) & 패턴 1.2초 바닥 경고 연출 | 없음 *(병렬 가능)* | `map/Boss01~05.map` *(신규)*, `BossPatternVisualLogic` *(신규)* | `[ ]` |
| [REQ-BOSS-04](tickets/REQ-BOSS-04.md) | `[UI]` | P1 | 보스 입장창(`T`) · 보스 상단 HUD · 클리어/실패 결과 팝업 (`BossUIGroup.ui`) | `ARCH-02`, `BOSS-01` | `UIBossController` *(신규)*, `BossUIGroup.ui` *(신규)*, `AutoHuntComponent` | `[ ]` |
| [REQ-QA-01](tickets/REQ-QA-01.md) | `[QA]` | P2 | v1.0 E2E 플레이테스트(Lv.1~200), `v2→v3` 마이그레이션, 멀티 격리 & Private 출시 | 전체 완료 후 | 전체 검증 (코드 버그 핫픽스) | `[ ]` |

---

## 3. 작업 유형별 빠른 분류 (역할/충돌 방지 기준)

- **`[DATA]` (3개)**: `REQ-ARCH-01`, `REQ-EQUIP-01`, `REQ-SHOP-01`
- **`[FUNC]` (12개)**: `REQ-ARCH-03`, `REQ-STAT-01`, `REQ-EQUIP-02`, `REQ-EQUIP-03`, `REQ-INVEN-01`, `REQ-SHOP-02`, `REQ-HUNT-02`, `REQ-HUNT-03`, `REQ-SURV-01`, `REQ-SURV-02`, `REQ-BOSS-01`, `REQ-BOSS-02`
- **`[UI]` (8개)**: `REQ-ARCH-02`, `REQ-STAT-02`, `REQ-EQUIP-04`, `REQ-INVEN-02`, `REQ-SHOP-03`, `REQ-HUNT-04`, `REQ-SURV-03`, `REQ-BOSS-04`
- **`[MAP/FX]` (2개)**: `REQ-HUNT-01`, `REQ-BOSS-03`
- **`[QA]` (1개)**: `REQ-QA-01`

---

## 4. 2인 병렬 실행 추천 페어링 (충돌 0% 조합)

| 라운드 | 작업자 A | 작업자 B | 충돌 여부 |
| :---: | :--- | :--- | :---: |
| **Round 1** | `REQ-ARCH-01` (`GameData` 분리) | `REQ-ARCH-02` (`UIIdleManager` 분리) | **0%** |
| **Round 2** | `REQ-ARCH-03` (`Profile v3`) → `REQ-STAT-01` | `REQ-EQUIP-01` (KMS DB & RUID) → `REQ-SHOP-01` | **0%** |
| **Round 3** | `REQ-EQUIP-02` + `REQ-EQUIP-03` (24슬롯/세트) | `REQ-INVEN-01` + `REQ-SHOP-02` (신규 Logic) | **0%** |
| **Round 4** | `REQ-SURV-01` + `REQ-SURV-02` (생존/오프라인) | `REQ-SHOP-03` + `REQ-SURV-03` (별도 `.ui` 파일!) | **0%** |
| **Round 5** | `REQ-HUNT-01` → `02` → `03` (1인 사냥터) | `REQ-BOSS-03` (보스 맵 5종) → `REQ-BOSS-01` | **0%** |
| **Round 6** | `REQ-HUNT-04` + `REQ-EQUIP-04`/`INVEN-02` | `REQ-BOSS-02` + `REQ-BOSS-04` (`BossUIGroup.ui`) | **0%** |
| **Round 7** | `REQ-QA-01` 공동 E2E 검증 & Private 출시 | | |
