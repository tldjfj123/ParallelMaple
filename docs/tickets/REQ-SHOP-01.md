# REQ-SHOP-01: NPC 상점 마스터 데이터 (잡화/강화/보조·엠블렘/펫·자석펫/보스 코인)

- **도메인:** `SHOP` · **유형:** `[DATA]` · **우선순위:** `P1`
- **선행 티켓:** `REQ-ARCH-01`, `REQ-EQUIP-01` · **병렬 가능:** `REQ-ARCH-03`, `REQ-STAT-01`, `REQ-HUNT-01`과 병렬 가능

## 1. 목표 (User Story)
NPC 상점 3탭(잡화·강화 / 펫·자석펫 / 보스 코인 교환)의 모든 품목·가격·구매 조건을 `WorldDataLogic` 단일 소스에 정의한다.

## 2. 상세 요구사항 (FR)
1. `WorldDataLogic.ShopItems` (잡화/강화/직업 전용 상점):
   - 포션 4종 ×10 (`red_potion` 500 / `white_potion` 2,500 / `blue_potion` 6,000 / `power_elixir` 30,000 메소)
   - 강화 재료: `spell_trace_100` (주흔 ×100, 50,000), `cube_suspicious_10` (수상한 큐브 ×10, 100,000), `cube_red_1` (레드 큐브 ×1, 300,000), `cube_black_1` (블랙 큐브 ×1, 600,000)
   - 특수 소비: `ap_reset_scroll` (AP 초기화 주문서 ×1, 100,000)
   - 무지급 보조무기·엠블렘 (전직 티어 조건 `minJobTier`):
     - `warrior_medallion_30` (30,000, 2차+) / `knight_medallion_60` (120,000, 3차+) / `silver_maple_emblem` (150,000, 3차+) / `adventurer_medallion` (500,000, 4차+) / `gold_maple_emblem` (600,000, 4차+)
2. `WorldDataLogic.PetUpgrades` (펫 상점):
   - `pet_slot_2` (2번 펫 퍼피, 500,000 메소, `reqPetCount = 1`)
   - `pet_slot_3` (3번 펫 키티, 2,000,000 메소, `reqPetCount = 2`)
   - `magnet_pet` (자석펫 광역 루팅 해금, 10,000,000 메소, `reqPetCount = 1`)
3. `WorldDataLogic.BossShopItems` (보스 코인 교환):
   - 자쿰 코인: `zakum_helmet` (10), `aquatic_letter_eye` (12), `condensed_power_crystal` (15), `silver_blossom_ring` (15)
   - 혼테일 코인: `crystal_ventus_badge` (15), `royal_black_metal_shoulder` (18), `horntail_necklace` (20), `dea_sidus_earring` (20), `noble_ifia_ring` (25), `deimos_warrior_shield` (25)
   - 루타비스 코인: `mechanator_pendant` (20), `golden_clover_belt` (20), `karuta_hat`/`karuta_top`/`karuta_pants` (각 30), `fafnir_sword` (50)
   - 앱솔랩스 코인: `pink_holy_cup` (30), `absolab_hat`/`top`/`gloves`/`shoes`/`cape`/`shoulder` (각 40), `absolab_saber` (60)
   - 아케인 코인: `arcane_hat`/`top`/`gloves`/`shoes`/`cape`/`shoulder` (각 50), `arcane_saber` (80)

## 3. 예외 / 엣지 케이스
- 모든 품목의 `itemId`는 `EquipDataLogic` 또는 소비 ID와 100% 일치해야 한다.

## 4. 수정 대상 파일 (`File Scope`)
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/WorldDataLogic.mlua`

## 5. Play Mode 완료 조건 (AC)
- [ ] `WorldDataLogic:EnsureInit()` 이후 3개 상점 테이블 조회 시 누락된 `itemId`나 `nil` 참조가 0건임.
