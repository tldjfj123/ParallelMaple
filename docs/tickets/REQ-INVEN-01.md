# REQ-INVEN-01: 인벤 60칸 · 장비 잠금 · 개별 판매/분해 · 안전 일괄 분해 서버 로직

- **도메인:** `INVEN` · **유형:** `[FUNC]` · **우선순위:** `P1`
- **선행 티켓:** `REQ-ARCH-03` · **병렬 가능:** `REQ-EQUIP-02`, `REQ-SHOP-02`, `REQ-HUNT-01`과 병렬 가능

## 1. 목표 (User Story)
인벤토리 상한을 60칸으로 늘리고, 플레이어가 소중한 장비는 잠금(🔒)으로 보호하면서 불필요한 장비는 판매(메소) 또는 분해(주문의 흔적)로 깔끔하게 정리할 수 있게 한다.

## 2. 상세 요구사항 (FR)
1. 인벤토리 최대 장비 수를 `40 → 60`으로 확장 (`PlayerDataLogic` 드랍/줍기 상한 체크 포함).
2. 신규 `InventoryLogic` (`.mlua` + `.codeblock`)에 서버 RPC 4종 구현:
   - `@ExecSpace("Server") RequestToggleLockItem(string uid)`: `item.locked = not (item.locked or false)`, 스냅샷 푸시 & 저장.
   - `@ExecSpace("Server") RequestSellEquip(string uid)`:
     - 미장착 · 미잠금 · `pendingPotential == nil` 검증.
     - 판매가 `math.max(100, (item.reqLevel or 1) * 150)` 메소 지급 → 인벤토리 제거 → 스냅샷 푸시 & 저장.
   - `@ExecSpace("Server") RequestDismantleEquip(string uid)`:
     - 미장착 · 미잠금 · `pendingPotential == nil` 검증.
     - 환급: 주문의 흔적 `math.max(10, math.floor((item.reqLevel or 1) * 0.4))` + 잠재가 `Epic`이면 수상한 큐브 `+1`, `Unique`/`Legendary`이면 수상한 큐브 `+2`.
   - `@ExecSpace("Server") RequestBulkDismantle(integer maxReqLevel)`:
     - 아래 6개 조건을 **모두** 만족하는 장비만 일괄 분해:
       `미장착`, `not item.locked`, `not item.isDestroyed`, `(item.star or 0) == 0`, `(item.potentialGrade or "Rare") == "Rare"`, `item.pendingPotential == nil`, `(item.reqLevel or 1) <= maxReqLevel`.
     - 총 분해 개수와 총 획득 주문의 흔적을 토스트/획득 로그로 안내.
3. `EnhanceLogic:RequestRestoreTrace`의 스페어 탐색 조건에 `and not cand.locked` 1줄을 추가해 잠금된 스페어가 실수로 소모되지 않게 보호.

## 3. 예외 / 엣지 케이스
- 장착 중인 장비, 잠금 장비, 블랙 큐브 선택 대기(`pendingPotential ~= nil`) 장비는 개별 판매/분해 요청 시 서버에서 차단 + 안내 토스트.
- 일괄 분해 대상이 0개일 때 에러 없이 `"분해할 수 있는 하위 장비가 없습니다."` 토스트 출력.

## 4. 수정 대상 파일 (`File Scope`)
- 생성: `workspaces/ParallelMaple/RootDesk/MyDesk/InventoryLogic.mlua` + `.codeblock`
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/EnhanceLogic.mlua` (스페어 잠금 체크 1줄), `workspaces/ParallelMaple/RootDesk/MyDesk/PlayerDataLogic.mlua` (`#profile.inventory < 60` 상한)

## 5. Play Mode 완료 조건 (AC)
- [ ] 장비 60개까지 정상 획득 가능하며 61번째부터 차단됨.
- [ ] 잠금(`locked = true`) 설정된 장비는 판매, 분해, 일괄 분해, 스페어 복구 소모 모두에서 완벽히 보호됨.
- [ ] 1성 이상 강화되었거나 에픽 이상인 장비는 일괄 분해 시 삭제되지 않음.
