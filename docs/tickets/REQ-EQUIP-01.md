# REQ-EQUIP-01: KMS Lv.10~200 전사 전체 장비 DB & `msw-mcp` RUID 검증

- **도메인:** `EQUIP` · **유형:** `[DATA]` · **우선순위:** `P0`
- **선행 티켓:** `REQ-ARCH-01` · **병렬 가능:** `REQ-ARCH-03`, `REQ-STAT-01`, `REQ-BOSS-03`과 병렬 가능

## 1. 목표 (User Story)
[docs/03-data-reference.md](../03-data-reference.md)에 정의된 KMS 원작 Lv.10~200 전사 장비(한손검 17종, 전사 방어구 Lv.10~140, 카루타 하의, 앱솔/아케인 망토·견장, 보스 장신구 9셋, 보조무기/엠블렘 30·60·100)를 `EquipDataLogic`에 빠짐없이 등록하고 모든 아이콘/아바타 RUID를 `msw-mcp`로 검증한다.

## 2. 상세 요구사항 (FR)
1. `msw-mcp`의 `asset_search_resources`를 사용하여 `docs/03-data-reference.md`에 `*(T1.1 조회)*`로 남아있는 신규 장비들의 공식 MSW `avataritem` / `sprite` RUID를 조회하고 확정한다.
2. `EquipDataLogic.Items`에 모든 장비 마스터를 등록하며, 아래 신규 필드를 표준화한다:
   - `slot`: 24부위 표준 슬롯명 (`Weapon`, `SubWeapon`, `Emblem`, `Hat`, `Top`, `Pants`, `Shoes`, `Gloves`, `Cape`, `Shoulder`, `Ring`, `Pendant`, `Face`, `Eye`, `Earring`, `Belt`, `Pocket`, `Badge`, `Medal`, `AndroidHeart`)
   - `isOverall` (`boolean`): 한벌옷 여부 (`pensalir_top`, `absolab_top`, `arcane_top`, `rollico_armor` 등)
   - `setId` (`string | nil`): `"pensalir"`, `"karuta"`, `"absolab"`, `"arcane"`, `"boss_acc"`
   - `baseStr`, `baseDex`, `baseAtt`, `baseMaxHp`, `reqLevel`, `upgradeMax`, `canScroll`, `canStarforce`
3. 확정된 모든 RUID를 `docs/03-data-reference.md`에 함께 기록한다.

## 3. 예외 / 엣지 케이스
- 특정 저레벨 아이템의 정확히 일치하는 원작 RUID가 MSW 공식 에셋에 없을 경우, 같은 티어/외형의 검증된 전사 장비 RUID를 사용하고 문서에 메모한다. *(추측 해시 문자열 절대 금지!)*
- 잠재능력 풀 매핑(`GetPoolKeyForSlot`)이 신규 슬롯(`Pants`, `Cape`, `Shoulder`, `Face`, `Eye`, `Earring`, `Pendant`, `Belt` → `Armor` 또는 `Accessory`)을 빠짐없이 처리하도록 보강한다.

## 4. 수정 대상 파일 (`File Scope`)
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/EquipDataLogic.mlua`, `workspaces/ParallelMaple/RootDesk/MyDesk/EnhanceDataLogic.mlua` (`GetPoolKeyForSlot`), `docs/03-data-reference.md`

## 5. Play Mode 완료 조건 (AC)
- [ ] 등록된 모든 장비 마스터의 `iconRuid`가 인벤/장비창에서 깨짐(흰 박스/에러) 없이 정상 렌더링됨.
- [ ] `GetItemMaster` 및 `GetPoolKeyForSlot`이 24개 모든 슬롯 타입에 대해 `nil` 없이 유효 값을 반환함.
