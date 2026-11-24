# REQ-ARCH-03: `Profile` v3 스키마 & `v2 → v3` 무중단 마이그레이션

- **도메인:** `ARCH` · **유형:** `[FUNC]` · **우선순위:** `P0`
- **선행 티켓:** `REQ-ARCH-01` · **병렬 가능:** `REQ-EQUIP-01`, `REQ-BOSS-03`과 병렬 가능

## 1. 목표 (User Story)
AP 스탯, 펫/자석펫, 1인 사냥터 90스테이지, 오프라인 보상(`DateTime.UtcNow`), 1인 보스전 필드를 `Profile` v3 스키마에 안전하게 추가하고, 기존 `version = 2` 저장 유저가 접속해도 데이터 손실 없이 자동 마이그레이션되게 한다.

## 2. 상세 요구사항 (FR)
1. `PlayerDataLogic:MigrateProfile(decoded)` 유틸을 구현하여 `LoadProfileForUser`에서 호출한다.
2. v3 신규 필드 기본값 보장 (`docs/05-architecture.md` 2.1절 준수):
   - `version = 3`
   - `baseStats = { str = 4, dex = 4, int = 4, luk = 4 }`, `unspentAp = 5` *(기존 `level > 1` v2 유저는 `(level - 1) * 5` AP를 `baseStats.str`에 자동 반영해 전투력 급락 방지)*
   - `petCount = 1`, `magnetPetUnlocked = false`
   - `autoPotionEnabled = true`, `autoPotionThreshold = 0.50`
   - `maxUnlockedChapter = 1`, `maxUnlockedStage = 1`, `currentChapter = 1`, `currentStage = 1`, `stageKillCount = 0`, `eliteKillCounter = 0`, `treasureGauge = 0`, `treasurePendingCount = 0`
   - `lastSaveUtcMs = DateTime.UtcNow.Elapsed`, `lastHuntingMapName = "HenesysHunt1"`, `pendingOfflineReward = nil`
   - `bossCoins = { zakum = 0, horntail = 0, karuta = 0, lotus = 0, lucid = 0 }`, `bossDailyResetDay = ""`, `bossDailyClears = {}`
   - 기존 `inventory` 각 아이템에 `locked = item.locked or false` 보정
3. `SaveProfileAsync`에서 저장 직전 항상 `profile.lastSaveUtcMs = DateTime.UtcNow.Elapsed`를 갱신한다. *(기존 `_UtilLogic.ServerElapsedSeconds` 제거!)*
4. `OnReceiveSnapshot` 클라이언트 디코딩 시 v3 테이블 필드(`baseStats`, `bossCoins`, `bossDailyClears` 등)에 `or {}` 방어 코드를 추가한다.

## 3. 예외 / 엣지 케이스
- `SanitizeForJson`은 빈 테이블을 `nil`로 생략하므로, 로드 및 클라이언트 수신 시 빈 테이블 필드가 `nil`이어도 크래시 나지 않아야 한다.
- `SaveProfileAsync`에 선택적 완료 콜백 `onDone(ok)` 파라미터를 추가해 향후 `RoomService` 방 이동 전 저장 완료 대기를 지원한다.

## 4. 수정 대상 파일 (`File Scope`)
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/PlayerDataLogic.mlua`

## 5. Play Mode 완료 조건 (AC)
- [ ] 기존 `v2` 저장 계정으로 접속 시 에러 없이 `v3`으로 자동 마이그레이션되고 기존 장비/레벨/메소가 보존됨.
- [ ] 저장 후 `lastSaveUtcMs`가 13자리 UTC Epoch 밀리초 값으로 기록됨.
- [ ] 신규 계정 생성 및 재접속 시 모든 v3 필드가 클라이언트 스냅샷에 정상 전달됨.
