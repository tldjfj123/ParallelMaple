# ParallelMaple v1.0 개발 로드맵 & 독립 티켓 보드

> **사용법:** 각 티켓은 **2명이 동시에 풀 바이브코딩해도 Git 충돌이 나지 않도록** 수정 대상 파일을 최대한 분리해 두었습니다.  
> **Phase 0(T0.1, T0.2)** 만 먼저 끝내면, 이후 **Phase 1 ~ Phase 3의 티켓들은 서로 다른 파일로 자유롭게 병렬 진행**할 수 있습니다!

---

## 진행 상태 범례
- `[ ]` 미착수 · `[-]` 진행 중 · `[x]` 완료

---

## Phase 0. 협업 기반 리팩토링 *(최우선 — 충돌 방지 핵심)*

> `T0.1`과 `T0.2`는 건드리는 파일이 완전히 달라 **2명이 동시에 병렬 진행 가능**합니다!

### `[ ] T0.1` `GameDataLogic` 도메인별 데이터 모듈 분리
- **목표:** 1,337줄짜리 `GameDataLogic.mlua`를 3개 데이터 모듈로 분리하고, 기존 `_GameDataLogic`은 하위 호환 파사드로 유지.
- **수정/생성 파일:** `GameDataLogic.mlua` + 신규 `EquipDataLogic.mlua`, `EnhanceDataLogic.mlua`, `WorldDataLogic.mlua` (각 `.codeblock` 포함)
- **선행 조건:** 없음
- **완료 조건 (AC):** 기존 게임 Play Mode에서 사냥·드랍·강화·전직·사냥터 이동이 에러 없이 100% 동일하게 동작.

### `[ ] T0.2` `UIIdleManager` 기능별 UI 컨트롤러 분리
- **목표:** 1,454줄짜리 `UIIdleManager.mlua`를 라우터+HUD(`UIIdleManager`) / 캐릭터창(`UICharacterWindows`) / 강화창(`UIEnhanceController`) / 사냥터창(`UINavigationWindows`)으로 분리.
- **수정/생성 파일:** `UIIdleManager.mlua` + 신규 `UICharacterWindows.mlua`, `UIEnhanceController.mlua`, `UINavigationWindows.mlua` (각 `.codeblock` 포함)
- **선행 조건:** 없음
- **완료 조건 (AC):** 단축키 `E/I/S/K/M/U/C/ESC` 및 모든 버튼이 기존과 동일하게 작동, 콘솔 에러 0건.

---

## Phase 1. 정통 점진 성장 · 세트 효과 · 인벤토리 · NPC 상점

> `T0.1`, `T0.2` 완료 후 아래 4개 티켓 중 파일이 안 겹치는 조합으로 병렬 진행 가능!

### `[ ] T1.1` 정통 점진 성장 전환 (Lv.1 스타터 + 초반 장비 + `reqLevel` + 드랍풀 정리)
- **목표:** Lv.1~100 초반 장비 13종(`wooden_sword`, `bronze_*`, `iron_*`, `knight_*`) 및 보조무기/엠블렘(30/60/100) RUID 조회·등록, 신규 프로필을 Lv.1 스타터로 변경, 장착 시 `reqLevel` 강제, 사냥터 드랍풀을 Lv.140 펜살리르 상한으로 개편, 2·3·4차 전직 시 본메 고증 보조무기(30/60/100)·엠블렘(60/100) 자동 지급.
- **수정 파일:** `EquipDataLogic.mlua`, `PlayerDataLogic.mlua`, `docs/03-data-reference.md` (확정 RUID 갱신)
- **선행 조건:** `T0.1`
- **완료 조건 (AC):** 신규 계정 Lv.1 초보자 장비 시작, Lv.30 장비 Lv.1 장착 거부, Lv.180 사냥터에서 카루타/앱솔/아케인 미드랍.

### `[ ] T1.2` 장비 세트 효과 (2/3/4/5세트) 서버 합산 & UI 표시
- **목표:** 펜살리르/카루타/앱솔랩스/아케인셰이드 `setId` 매핑, `RecalculateStats`에서 장착 세트별 보너스 합산 및 `stats.activeSets` 전달, 장비창(`E`)/스탯창(`S`)에 활성 세트 표시.
- **수정 파일:** `EquipDataLogic.mlua` (`setId` 및 `SetEffects`), `PlayerDataLogic.mlua` (`RecalculateStats`), `UICharacterWindows.mlua`
- **선행 조건:** `T0.1`, `T0.2`
- **완료 조건 (AC):** 카루타 3세트 착용 시 공격력+55·보공+30% 등이 전투력/스탯창에 즉시 반영되고, 장비 파괴 시 세트 수에서 제외됨.

### `[ ] T1.3` 인벤토리 60칸 확장 · 잠금 · 판매 · 분해 · 일괄 분해
- **목표:** 장비 잠금(`locked`), 개별 판매(메소), 개별 분해(주흔+에픽이상 큐브 보너스), 안전한 하위 장비 일괄 분해 구현. (`EnhanceLogic:RequestRestoreTrace`도 잠금 장비 스페어 소모 방지 추가.)
- **수정/생성 파일:** 신규 `InventoryLogic.mlua`, `EnhanceLogic.mlua` (스페어 잠금 체크 1줄), `UICharacterWindows.mlua`
- **선행 조건:** `T0.2`
- **완료 조건 (AC):** 잠금/장착/1성+/에픽+/블랙큐브 대기 장비는 일괄 분해에서 절대 삭제되지 않음.

### `[ ] T1.4` NPC 상점 (잡화/강화 + 보스 코인 교환) 서버 & UI
- **목표:** 포션/주흔/큐브 메소 상점 + 보스 코인 5종 장비 교환 서버 로직(`ShopLogic`) 및 전용 UI(`UIShopController` + `ShopUIGroup.ui`, 단축키 `B`).
- **수정/생성 파일:** `WorldDataLogic.mlua` (상점 테이블), 신규 `ShopLogic.mlua`, `UIShopController.mlua`, `ui/ShopUIGroup.ui`, `AutoHuntComponent.mlua` (`B` 단축키)
- **선행 조건:** `T0.1`, `T0.2`
- **완료 조건 (AC):** 메소/코인 부족 및 인벤 60칸 초과 시 구매 거부, 정상 구매 시 재화 차감 및 아이템 지급·저장.

---

## Phase 2. 생존(자동포션/사망) & 오프라인 12시간 방치 보상

> `T2.1`과 `T2.2`는 파일이 겹치지 않아 **완전 병렬 진행 가능**!

### `[ ] T2.1` 자동 포션 (HP 50%) & 일반 필드 사망 마을 부활
- **목표:** `RecalculateStats` 임시 자동 풀회복 제거, 피격 시 HP 50% 이하 자동 포션(1초 쿨, 위급 시 파엘 우선), 일반 필드 HP 0 시 경험치 무손실 헤네시스 귀환 & 자동사냥 OFF.
- **수정/생성 파일:** 신규 `SurvivalLogic.mlua`, `PlayerHit.mlua`, `PlayerDataLogic.mlua` (임시 회복 3줄 제거)
- **선행 조건:** 없음
- **완료 조건 (AC):** 피격 시 HP 50% 이하에서 포션 자동 소모, 포션 0개일 때 사망 시 헤네시스 부활 및 경험치 보존.

### `[ ] T2.2` 오프라인 방치 보상 (`DateTime.UtcNow`, 최대 12시간) & 정산 팝업
- **목표:** `lastSaveUtcMs`(UTC ms) + `lastHuntingMapName` 저장, 재접속 시 60초~12시간 구간 80% 효율 정산, 수령 팝업 UI 및 중복 수령 방지.
- **수정/생성 파일:** 신규 `OfflineRewardLogic.mlua`, `UIOfflinePopup.mlua`, `ui/OfflinePopupGroup.ui`, `PlayerDataLogic.mlua` (저장/로드 훅)
- **선행 조건:** `T0.2`
- **완료 조건 (AC):** 60초 미만 미표시, 12시간 초과 캡 적용, 서버 재시작 후에도 정상 정산, 수령 버튼 연타 시 1회만 지급.

---

## Phase 3. 1인 인스턴스 보스전 5단계 (v1.0 하이라이트)

### `[ ] T3.1` 보스전 코어 서버 로직 (`BossLogic`) & 방무/보공 데미지 연동
- **목표:** 5단계 보스(`zakum`, `horntail`, `karuta`, `lotus`, `lucid`) 마스터 테이블, 1인 인스턴스 입장/퇴장, KST 00:00 일일 클리어 횟수 초기화, 제한시간 10분, 데스카운트 5회, 클리어 시 코인/큐브/장비 보상 지급, `PlayerAttack:CalcDamage`에 보공·방무(PDR) 공식 연동.
- **수정/생성 파일:** `WorldDataLogic.mlua`, 신규 `BossLogic.mlua`, `PlayerAttack.mlua`
- **선행 조건:** `T0.1`
- **완료 조건 (AC):** 레벨·횟수 검증, 클리어 시에만 횟수 차감, 실패/시간초과 시 횟수 미차감 마을 복귀.

### `[ ] T3.2` 보스 몬스터 컴포넌트 & 5종 패턴 AI (`BossMonster`)
- **목표:** 보스별 시그니처 패턴 2~3종(1.2초 바닥 경고 텔레그래프 → `MaxHp` 비율 피해), 피격/사망 이벤트 `BossLogic` 연동.
- **수정/생성 파일:** 신규 `BossMonster.mlua`, `BossPatternVisualLogic.mlua`
- **선행 조건:** `T3.1`
- **완료 조건 (AC):** 5종 보스 각각 고유 패턴과 경고 연출이 정상 동작하고 사망 시 클리어 트리거 호출.

### `[ ] T3.3` 보스 전용 아레나 맵 5종 세팅
- **목표:** 자쿰/혼테일/벨룸/스우/루시드 보스 맵 `.map` 제작(또는 공식 맵 임포트) 및 스폰 위치·BGM·보스 스프라이트 배치.
- **수정/생성 파일:** `map/Boss01_Zakum.map` ~ `map/Boss05_Lucid.map` *(단독 작업 필수 — 동시 편집 금지)*
- **선행 조건:** 없음 (병렬 가능)
- **완료 조건 (AC):** 5개 보스 맵 입장 시 발판/카메라/스폰 정상, 낙사 버그 없음.

### `[ ] T3.4` 보스 입장창(`T`) · 보스 HUD · 결과 팝업 UI
- **목표:** 보스 5종 선택/횟수 확인 팝업, 보스전 상단 HP바·타이머·데스카운트 HUD, 클리어/실패 결과 모달 구현.
- **수정/생성 파일:** 신규 `UIBossController.mlua`, `ui/BossUIGroup.ui`, `AutoHuntComponent.mlua` (`T` 단축키)
- **선행 조건:** `T0.2`, `T3.1`
- **완료 조건 (AC):** 입장→전투 HUD 실시간 갱신→클리어/실패 팝업→마을 귀환 전 과정 무에러 완주.

---

## Phase 4. v1.0 통합 QA & 밸런스 검증

### `[ ] T4.1` Lv.1 → Lv.200 E2E 플레이테스트 & 밸런스 튜닝
- **체크리스트:**
  - [ ] Lv.1 신규 생성 → 전직(10/30/60/100) → 사냥터 18개 순차 해금 → 보스 5종 입장·클리어·코인 교환
  - [ ] 3대 강화(주문서/스타포스 파괴→스페어 12성 복구/블랙큐브 선택) & 세트효과 & 인벤 일괄분해
  - [ ] 오프라인 12시간 보상 정산 & 기존 `version = 2` 저장 데이터 마이그레이션 정상 동작
  - [ ] 2인 동시 접속 공용 사냥터 타격 기여 보상 & 개인 보스 인스턴스 격리 검증

---

## v1.1+ 확장 백로그 (v1.0 이후)
- `[ ] B0` **도파민 특수 강화 확장**: 놀라운 긍정의 혼돈의 주문서(놀긍혼 + 리턴) & 놀라운 장비 강화 주문서(놀장강 파란별 12성)
- `[ ] B1` 전투력 / 레벨 실시간 랭킹 보드 (`SortableDataStorage`)
- `[ ] B2` 장비 도감 전용 UI 창 (세트별 수집률 & 보너스 시각화)
- `[ ] B3` 원작 풀 슬롯 확장 (하의/망토/견장/벨트/귀걸이/펜던트/반지 2~4)
- `[ ] B4` 에디셔널 잠재능력 & 에디셔널 큐브
- `[ ] B5` 타 직업군 확장 (마법사/궁수/도적) & 5차 전직
- `[ ] B6` 수익화(BM) 기획 및 MSW 월드 상품 연동
