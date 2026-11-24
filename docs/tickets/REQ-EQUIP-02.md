# REQ-EQUIP-02: 24부위 풀 슬롯 장착 규칙 · `reqLevel` 강제 · 무지급 시작 · 펜살리르 상한 드랍풀

- **도메인:** `EQUIP` · **유형:** `[FUNC]` · **우선순위:** `P0`
- **선행 티켓:** `REQ-ARCH-03`, `REQ-EQUIP-01` · **병렬 가능:** `REQ-INVEN-01`, `REQ-SHOP-02`, `REQ-BOSS-03`과 병렬 가능

## 1. 목표 (User Story)
현대 메이플 24부위 풀 슬롯 장착 규칙을 서버에 구현하고, 신규 유저가 **장비 0개(맨손)**로 시작해 사냥터에서 Lv.10~140 펜살리르까지 직접 득템하며 착용 레벨(`reqLevel`)에 맞춰 성장하도록 전환한다.

## 2. 상세 요구사항 (FR)
1. **24부위 `SlotOrder` 확장**:
   `Weapon`, `SubWeapon`, `Emblem`, `Hat`, `Top`, `Pants`, `Shoes`, `Gloves`, `Cape`, `Shoulder`, `Ring1`, `Ring2`, `Ring3`, `Ring4`, `Pendant1`, `Pendant2`, `Face`, `Eye`, `Earring`, `Belt`, `Pocket`, `Badge`, `Medal`, `AndroidHeart`.
2. **무지급 신규 프로필 (`CreateDefaultProfile`)**:
   - `inventory = {}`, `equipped = {}`, `meso = 0`, `spellTrace = 0`, `cubes = {}`, `potions = { red_potion = 10 }`, `petCount = 1`, `magnetPetUnlocked = false`.
   - `RequestAdvanceJob`의 기존 공짜 메소/큐브/장비 지급 제거 *(전직 시 스킬/콤보만 해금, 보조무기/엠블렘은 상점 구매)*.
3. **장착 규칙 (`RequestEquipItem(string uid, string preferredSlot)`)**:
   - **착용 레벨 검증**: `profile.level < item.reqLevel`이면 거부 + 토스트 `"🔒 Lv.<reqLevel>부터 장착할 수 있는 장비입니다!"`.
   - **한벌옷 규칙**: `master.isOverall == true` 장착 시 `equipped.Pants = nil` 자동 해제; `Pants` 장착 시 현재 `equipped.Top`이 한벌옷이면 `equipped.Top = nil` 자동 해제.
   - **다중 슬롯 배치**: `master.slot == "Ring"`이면 `preferredSlot`이 유효(`Ring1`~`Ring4`)할 때 해당 칸, 아니면 빈 `Ring1`~`Ring4` 우선 → 모두 찼으면 `Ring1` 교체. (`Pendant1`~`2` 동일)
   - **동종 중복 방지**: 원작 기준 `horntail_necklace` 등 `uniqueEquip = true` 아이템은 동일 `itemId` 중복 장착 차단.
4. **사냥터 드랍풀 개편 (`GetEquipDropPoolForLevel`)**:
   - Ch.1~Ch.18(Lv.1~200) 전 구간에서 **Lv.10 카알 대검 ~ Lv.140 펜살리르 풀세트까지만** 드랍. 카루타/앱솔/아케인/보스 장신구는 일반 사냥터 드랍풀에서 완전 제거.

## 3. 예외 / 엣지 케이스
- 파괴된 장비(`isDestroyed == true`) 장착 차단 유지.
- `Weapon` 해제(또는 맨손) 시 `CombatVisualLogic:ApplyWeaponCostume`이 기본 빈 무기 상태로 안전 처리.

## 4. 수정 대상 파일 (`File Scope`)
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/EquipDataLogic.mlua` (24슬롯 & 드랍풀), `workspaces/ParallelMaple/RootDesk/MyDesk/PlayerDataLogic.mlua`

## 5. Play Mode 완료 조건 (AC)
- [ ] 신규 계정 생성 시 장비 0개, 메소 0, 큐브 0, 기본 1펫으로 시작한다.
- [ ] Lv.1 캐릭터가 Lv.30 글라디우스 장착 시도 시 차단되고 안내 토스트가 뜬다.
- [ ] 한벌옷과 하의를 번갈아 장착할 때 상호 자동 해제가 정확히 동작한다.
- [ ] 반지 4개를 순서대로 장착하면 `Ring1` → `Ring2` → `Ring3` → `Ring4`에 모두 채워진다.
- [ ] Lv.180 사냥터 사냥 시 카루타/앱솔/아케인 장비가 절대 드랍되지 않는다.
