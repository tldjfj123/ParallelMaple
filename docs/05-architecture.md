# ParallelMaple 시스템 아키텍처 & 코드 계약 (v2.0)

> 2인 풀 바이브코딩 환경에서 **Git 충돌을 최소화하고, 서버/클라이언트 계약을 명확히 유지**하기 위한 기술 설계서입니다.

---

## 1. 현재 스크립트 맵 (As-Is) → 목표 모듈 구조 (To-Be)

### 1.1 현재 구조의 병목
- `UIIdleManager.mlua` (**1,454줄**): HUD, 장비, 인벤, 스탯, 스킬, 사냥터, 3대 강화 6개 창 이벤트 바인딩과 렌더링이 한 파일에 집중.
- `GameDataLogic.mlua` (**1,337줄**): 직업/스킬, 사냥터, 장비, 포션/전리품, 강화/큐브/잠재 테이블이 한 파일에 집중.

### 1.2 목표 모듈 구조 (Phase 0 리팩토링 후)

| 레이어 | 스크립트 (Logic / Component) | 책임 (단일 소유권) | 상태 |
| :--- | :--- | :--- | :---: |
| **Data** | `GameDataLogic.mlua` | **파사드(Facade)** — 기존 호출부(`_GameDataLogic:*`) 100% 하위 호환 유지, 하위 데이터 모듈 위임 | 리팩토링 |
| **Data** | `EquipDataLogic.mlua` *(신규)* | 장비 마스터, 세트 효과 테이블, 포션/전리품, 사냥터 드랍풀 | T0.1 |
| **Data** | `EnhanceDataLogic.mlua` *(신규)* | 주문서, 스타포스 확률/비용/보너스, 큐브, 잠재능력 옵션 풀 | T0.1 |
| **Data** | `WorldDataLogic.mlua` *(신규)* | 직업/스킬, 사냥터 18개, 경험치 테이블, **보스 5단계 & 상점 테이블** | T0.1 |
| **Server Core** | `PlayerDataLogic.mlua` | 프로필 로드/저장(`idle_save_v1`), 스탯/세트효과 합산, 드랍/보상, 전직, 텔레포트, 스냅샷 푸시 | 유지/확장 |
| **Server** | `EnhanceLogic.mlua` | 주문서/스타포스/흔적복구/큐브/블랙큐브 서버 권위 처리 | 완료 |
| **Server** | `InventoryLogic.mlua` *(신규)* | 장비 장착(`reqLevel` 검증), 잠금, 개별 판매/분해, 일괄 분해 | T1.3 |
| **Server** | `ShopLogic.mlua` *(신규)* | 잡화/강화 상점(메소), 보스 코인 상점 교환 | T1.4 |
| **Server** | `SurvivalLogic.mlua` *(신규)* | HP 50% 이하 자동 포션 발동, 일반 필드 사망→마을 부활 처리 | T2.1 |
| **Server** | `OfflineRewardLogic.mlua` *(신규)* | `DateTime.UtcNow` 기반 최대 12시간 오프라인 보상 계산 및 수령 | T2.2 |
| **Server** | `BossLogic.mlua` *(신규)* | 1인 보스 인스턴스 생성/입장/데스카운트/제한시간/클리어 보상/KST 00:00 리셋 | T3.1 |
| **Server/Field** | `FieldSpawnerLogic.mlua`, `Monster.mlua`, `MonsterAttack.mlua`, `BossMonster.mlua` *(신규)* | 사냥터 밀집 스폰, 일반/보스 몬스터 AI & 패턴 | 일부 신규 |
| **Combat** | `PlayerAttack.mlua`, `PlayerHit.mlua`, `AutoHuntComponent.mlua`, `CombatVisualLogic.mlua` | 스킬 타격/데미지(보스 방무·보공 추가), 자동사냥, 3펫/드랍/이펙트 | 유지/확장 |
| **Client UI** | `UIIdleManager.mlua` | **창 토글 라우터 & 메인 HUD + 획득 로그**만 담당 (기존 1,454줄 → ~400줄 축소) | T0.2 |
| **Client UI** | `UICharacterWindows.mlua` *(신규)* | 장비창(`E`), 인벤토리창(`I`), 스탯창(`S`), 스킬창(`K`) | T0.2 |
| **Client UI** | `UIEnhanceController.mlua` *(신규)* | 강화·큐브 통합창(`U`/`C`) | T0.2 |
| **Client UI** | `UINavigationWindows.mlua` *(신규)* | 사냥터 이동창(`M`) | T0.2 |
| **Client UI** | `UIShopController.mlua` *(신규)* | NPC 잡화/보스 코인 상점창(`B`) + `ShopUIGroup.ui` | T1.4 |
| **Client UI** | `UIBossController.mlua` *(신규)* | 보스 입장창(`T`), 보스 HUD, 클리어/실패 결과 팝업 + `BossUIGroup.ui` | T3.4 |
| **Client UI** | `UIOfflinePopup.mlua` *(신규)* | 오프라인 12시간 정산 팝업 + `OfflinePopupGroup.ui` | T2.2 |

---

## 2. 데이터 스키마 계약 (`DataStorageService` 키: `"idle_save_v1"`)

### 2.1 `Profile` 스키마 (v3 — 하위 호환 마이그레이션 필수)

```lua
local Profile = {
    version = 3,                     -- v2 -> v3 자동 마이그레이션
    jobId = "warrior",
    jobTier = 0,                     -- 0~4
    level = 1,                       -- 1~200
    exp = 0,
    meso = 5000,
    spellTrace = 50,
    cubes = { suspicious = 3, master = 0, meister = 0, red = 0, black = 0 },
    potions = { red_potion = 20, white_potion = 0, blue_potion = 0, power_elixir = 0 },
    etcLoot = {},                    -- { [etcId] = count }
    equipped = {},                   -- { [slotId] = equipUid }
    inventory = {},                  -- EquipInstance[] (최대 60칸)
    codex = {},                      -- { [itemId] = true }

    -- [v3 신규 필드]
    autoPotionEnabled = true,
    autoPotionThreshold = 0.50,      -- HP 50% 이하 자동 사용
    lastSaveUtcMs = 0,               -- DateTime.UtcNow.Elapsed (UTC ms)
    lastHuntingMapName = "HenesysHunt1",
    pendingOfflineReward = nil,      -- { elapsedSec, mapName, kills, exp, meso, trace, suspiciousCube } | nil

    -- [v3 보스전 필드]
    bossCoins = {
        zakum = 0,                   -- 하급 보스 코인
        horntail = 0,                -- 중급 보스 코인
        karuta = 0,                  -- 루타비스 코인
        lotus = 0,                   -- 앱솔랩스 코인
        lucid = 0,                   -- 아케인 코인
    },
    bossDailyResetDay = "",          -- "YYYY-MM-DD" (KST 기준)
    bossDailyClears = {},            -- { [bossId] = count }
}
```

### 2.2 `EquipInstance` 스키마 (v3 추가 필드: `locked`)

```lua
local EquipInstance = {
    uid = "eq_<guid>",
    itemId = "pensalir_sword",
    slot = "Weapon",
    reqLevel = 140,
    locked = false,                  -- [v3 신규] true면 판매/분해/스페어 소모 차단

    -- 주문서
    upgradeMax = 8,
    upgradeRemaining = 0,
    hammerUsed = false,
    scrollSuccess = 8,
    scrollFail = 0,
    scrollBonus = { mainStat = 16, att = 40 },

    -- 스타포스
    star = 17,
    isDestroyed = false,
    consecutiveFailDrop = 0,

    -- 잠재능력
    potentialGrade = "Legendary",    -- Rare | Epic | Unique | Legendary
    cubePityCount = 0,
    potentials = { { stat = "attPct", value = 12, label = "공격력 +12%", grade = "Legendary" }, ... },
    pendingPotential = nil,          -- 블랙 큐브 선택 대기
    additionalPotentialGrade = "None",
}
```

### 2.3 스냅샷 페이로드 (`PushSnapshotToClient` → `OnReceiveSnapshot`)
```lua
local SnapshotPayload = {
    profile = sanitizedProfile,
    stats = sanitizedStats,          -- activeSets = { { setId, name, count, bonusText } } 추가
    expNeeded = number,
    currentMapName = string,
    bossState = activeBossStateOrNil -- 보스전 진행 중일 때만 포함
}
```

---

## 3. 서버 ↔ 클라이언트 RPC 계약

### 3.1 기존 RPC (유지)
- `PlayerDataLogic`: `RequestSyncState()`, `RequestPickupDrop(dropId)`, `RequestUsePotion(potionId)`, `RequestSellEtcLoot(etcId)`, `RequestAdvanceJob()`, `RequestEquipItem(uid)`, `RequestTeleportMap(mapName)`
- `EnhanceLogic`: `RequestScroll(uid, scrollId, repeatAll)`, `RequestStarforce(uid, useSafeguard, starCatch)`, `RequestRestoreTrace(traceUid)`, `RequestCube(uid, cubeId)`, `RequestConfirmBlackCube(uid, applyNew)`
- `PlayerAttack`: `RequestCastSkill(skillId, lookDir)`, `RequestAutoAttack(lookDir)`

### 3.2 신규 RPC (v1.0 구현 대상)
- **`InventoryLogic` (T1.3)**
  - `@ExecSpace("Server") RequestToggleLockItem(string uid)`
  - `@ExecSpace("Server") RequestSellEquip(string uid)`
  - `@ExecSpace("Server") RequestDismantleEquip(string uid)`
  - `@ExecSpace("Server") RequestBulkDismantle(integer maxReqLevel)`
- **`ShopLogic` (T1.4)**
  - `@ExecSpace("Server") RequestBuyShopItem(string shopItemId, integer qty)`
  - `@ExecSpace("Server") RequestExchangeBossItem(string bossShopItemId)`
- **`SurvivalLogic` (T2.1)**
  - `@ExecSpace("Server") RequestSetAutoPotion(boolean enabled, number threshold)`
- **`OfflineRewardLogic` (T2.2)**
  - `@ExecSpace("Server") RequestClaimOfflineReward()`
  - `@ExecSpace("Client") OnShowOfflineRewardPopup(string rewardJson)`
- **`BossLogic` (T3.1~T3.4)**
  - `@ExecSpace("Server") RequestEnterBoss(string bossId)`
  - `@ExecSpace("Server") RequestLeaveBoss()`
  - `@ExecSpace("Client") OnBossStateUpdate(string bossStateJson)` *(HP, 남은시간, 데스카운트)*
  - `@ExecSpace("Client") OnBossResult(string resultJson)` *(클리어/실패 + 보상 내역)*

---

## 4. 1인 보스 인스턴스 구현 전략 (MSW)

MSW에서 1인 개인 보스 인스턴스를 구현하는 권장 방식은 **`RoomService:CreateInstanceRoom`** (또는 인스턴스 맵 `InstanceMap` 스폰)입니다:
1. `BossLogic:RequestEnterBoss(bossId)`에서 레벨·일일 클리어 횟수 검증 후, 해당 유저 전용 **1인 인스턴스 맵**(`BossArena_<bossId>`)으로 텔레포트.
2. 인스턴스 진입 시 서버가 보스 엔티티(`BossMonster` 컴포넌트 부착) 스폰 + 600초 타이머 & 데스카운트 5 초기화.
3. 보스 HP 변경/1초 틱마다 해당 유저에게 `OnBossStateUpdate` 전송.
4. 클리어 / 실패 / 퇴장 / 유저 접속 종료 시 타이머 정리 및 보스 인스턴스 정리 후 `TileMapTemplate`(헤네시스)로 복귀.
5. **KST 일일 초기화 유틸**: `kstDateStr` = `DateTime(DateTime.UtcNow.Elapsed + 9 * 3600 * 1000)`의 `YYYY-MM-DD` 문자열을 비교하여 날짜가 바뀌었으면 `profile.bossDailyClears = {}` 리셋.
