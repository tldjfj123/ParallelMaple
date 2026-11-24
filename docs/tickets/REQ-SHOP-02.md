# REQ-SHOP-02: NPC 상점 · 펫(1→3) 확장 & 자석펫 해금 서버 로직 및 펫 연동

- **도메인:** `SHOP` · **유형:** `[FUNC]` · **우선순위:** `P1`
- **선행 티켓:** `REQ-ARCH-03`, `REQ-SHOP-01` · **병렬 가능:** `REQ-EQUIP-02`, `REQ-INVEN-01`, `REQ-SURV-01`과 병렬 가능

## 1. 목표 (User Story)
서버 권위 `ShopLogic`에서 메소 구매·보스 코인 교환·펫 슬롯 확장(1→2→3마리)·자석펫 영구 해금을 처리하고, `CombatVisualLogic`이 유저의 실제 보유 펫 수(`petCount`)와 자석펫 해금 여부(`magnetPetUnlocked`)를 실시간 반영한다.

## 2. 상세 요구사항 (FR)
1. 신규 `ShopLogic` (`.mlua` + `.codeblock`) 서버 RPC 3종 구현:
   - `@ExecSpace("Server") RequestBuyShopItem(string shopItemId, integer qty)`:
     - `1 <= qty <= 99` 검증 (장비류는 `qty == 1` 고정).
     - `minJobTier` 충족 여부, 총 메소 보유량, 장비인 경우 `#profile.inventory < 60` 검증 후 차감 & 지급 & 즉시 저장.
   - `@ExecSpace("Server") RequestBuyPetUpgrade(string upgradeId)`:
     - `pet_slot_2`: `petCount == 1` & 50만 메소 → `petCount = 2`.
     - `pet_slot_3`: `petCount == 2` & 200만 메소 → `petCount = 3`.
     - `magnet_pet`: `not magnetPetUnlocked` & 1,000만 메소 → `magnetPetUnlocked = true`.
   - `@ExecSpace("Server") RequestExchangeBossItem(string bossShopItemId)`:
     - 해당 보스 코인 보유량 및 `#profile.inventory < 60` 검증 후 코인 차감 → `CreateEquipInstance(itemId)` 지급 & 도감 등록 & 저장.
2. `CombatVisualLogic` 연동:
   - `EnsureMultiPets(lp)`가 `ClientSnapshot.profile.petCount`(기본 1, 최대 3)만큼만 펫 엔티티를 스폰/유지하도록 수정 *(기존 고정 3마리 스폰 변경!)*.
   - `MagnetPetEnabled` 초기값은 `false`로 변경하고, `ToggleMagnetPet()`(`P` 키) 호출 시 `not profile.magnetPetUnlocked`이면 활성화 거부 + 토스트 `"🔒 자석펫은 상점(B) [펫 상점]에서 1,000만 메소로 해금할 수 있습니다!"`.
   - 자석펫 해금 구매 완료 즉시 `MagnetPetEnabled = true` 자동 활성화!

## 3. 예외 / 엣지 케이스
- 이미 구매한 펫 업그레이드 중복 구매 차단 (메소 이중 차감 방지).
- `qty <= 0` 또는 `qty > 99` 조작 요청 서버에서 즉시 차단.

## 4. 수정 대상 파일 (`File Scope`)
- 생성: `workspaces/ParallelMaple/RootDesk/MyDesk/ShopLogic.mlua` + `.codeblock`
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/CombatVisualLogic.mlua`

## 5. Play Mode 완료 조건 (AC)
- [ ] 신규 계정 접속 시 필드에 기본 펫 **1마리(루나)**만 따라다니고 `P` 키 입력 시 잠금 안내 토스트가 뜬다.
- [ ] 2번/3번 펫 구매 시 즉시 필드에 펫이 2마리 → 3마리로 늘어난다.
- [ ] 자석펫 구매 후 `P` 토글로 광역 자석 루팅 ON/OFF가 정상 동작한다.
- [ ] 보스 코인/메소 부족 및 인벤 60칸 초과 시 구매가 안전하게 거부된다.
