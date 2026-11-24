# REQ-ARCH-01: `GameDataLogic` 도메인별 3모듈 분리

- **도메인:** `ARCH` · **유형:** `[DATA]` · **우선순위:** `P0`
- **선행 티켓:** 없음 · **병렬 가능:** `REQ-ARCH-02`와 완전 병렬 가능

## 1. 목표 (User Story)
2인 협업 시 1,337줄짜리 단일 `GameDataLogic.mlua`에서 발생하는 Git 머지 충돌을 없애기 위해, 데이터 테이블을 3개 도메인 모듈로 분리하되 기존 `_GameDataLogic:*` 호출 코드는 100% 그대로 동작하도록 파사드 패턴을 유지한다.

## 2. 상세 요구사항 (FR)
1. 아래 3개의 신규 Logic 스크립트(`.mlua` + `.codeblock` 쌍)를 생성한다:
   - `EquipDataLogic`: `SlotOrder`, `SlotLabels`, `Items`, `Potions`, `EtcLoots` 및 `GetItemMaster`, `GetEquipDropPoolForLevel`, `GetPotionIdForLevel`, `GetEtcLootIdForLevel`
   - `EnhanceDataLogic`: `Scrolls`, `StarforceTable`, `Cubes`, `PotentialGrades`, `PotentialGradeIndex`, `PotentialPools` 및 `GetMaxStar`, `GetStarforceMesoCost`, `GetStarforceBonus`, `GetPoolKeyForSlot`, `GetGradeKorean`
   - `WorldDataLogic`: `JobTiers`, `Skills`, `HuntingMaps`, `MaxLevel` 및 `GetJobTierInfo`, `GetNextAdvanceTier`, `GetSkillInfo`, `GetHuntingMapInfo`, `GetRecommendedHuntingMap`, `GetNextLockedHuntingMap`, `GetExpToNextLevel`, `FormatNumber`, `FormatComma`
2. `GameDataLogic:EnsureInit()`은 3개 모듈의 `EnsureInit()`을 순서대로 호출하고, 기존 프로퍼티 참조(`_GameDataLogic.Items`, `_GameDataLogic.HuntingMaps` 등)가 깨지지 않도록 동일한 테이블 레퍼런스를 연결한다.
3. 기존 `GameDataLogic`의 모든 public 메서드는 하위 모듈로 위임(delegate)한다.

## 3. 예외 / 엣지 케이스
- 클라이언트와 서버 어느 쪽에서 먼저 `_GameDataLogic:EnsureInit()`을 호출해도 중복 초기화 없이 1회만 실행되어야 한다.
- 기존 `PlayerDataLogic`, `EnhanceLogic`, `FieldSpawnerLogic`, `UIIdleManager`, `AutoHuntComponent`, `CombatVisualLogic`을 수정하지 않고도 무결하게 동작해야 한다.

## 4. 수정 대상 파일 (`File Scope`)
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/GameDataLogic.mlua`
- 생성: `EquipDataLogic.mlua` + `.codeblock`, `EnhanceDataLogic.mlua` + `.codeblock`, `WorldDataLogic.mlua` + `.codeblock`

## 5. Play Mode 완료 조건 (AC)
- [ ] MSW Maker `Reimport All` 후 콘솔 에러 0건.
- [ ] 게임 시작 시 사냥터 18개 스폰, 자동사냥, 드랍 줍기, 주문서/스타포스/큐브 강화, 전직, 사냥터 이동이 리팩토링 전과 100% 동일하게 동작.
- [ ] `GameDataLogic.mlua` 파일 크기가 250줄 이하로 축소됨.
