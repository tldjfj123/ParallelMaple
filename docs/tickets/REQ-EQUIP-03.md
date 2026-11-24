# REQ-EQUIP-03: 장비 세트 효과 서버 합산 (`보장/펜살/카루타/앱솔/아케인`)

- **도메인:** `EQUIP` · **유형:** `[FUNC]` · **우선순위:** `P1`
- **선행 티켓:** `REQ-EQUIP-02` · **병렬 가능:** `REQ-INVEN-01`, `REQ-SHOP-02`, `REQ-HUNT-01`과 병렬 가능

## 1. 목표 (User Story)
현대 메이플 24부위 풀 슬롯에서 본메 정통 **`보스 장신구 3/5/7/9셋` · `펜살리르 2~6셋` · `3카 5앱` · `3카 5앜`** 세트 효과가 자동으로 계산되어 스탯과 전투력에 반영된다.

## 2. 상세 요구사항 (FR)
1. `EquipDataLogic.SetEffects` 테이블에 5개 세트(`pensalir`, `karuta`, `absolab`, `arcane`, `boss_acc`)의 단계별 누적 보너스를 정의한다 (`docs/02-feature-specs.md` F4.3 및 `docs/03-data-reference.md` 3절 수치 준수).
2. `PlayerDataLogic:RecalculateStats(userId)`에서:
   - 현재 `profile.equipped`에 장착된 비파괴(`not isDestroyed`) 장비들의 `master.setId`별 개수를 집계. *(동일 `itemId` 중복 시 1개로만 카운트)*
   - 각 세트의 활성 단계 보너스(`str`, `dex`, `att`, `maxHp`, `maxHpPct`, `dmgPct`, `bossDmgPct`, `iedPct`, `critDmgPct`)를 최종 스탯에 합산.
   - `stats.activeSets` 배열에 `{ setId, name, count, maxCount, activeSummary }`를 담아 클라이언트 스냅샷에 전달.

## 3. 예외 / 엣지 케이스
- 세트 장착 장비 중 하나가 스타포스 파괴(`isDestroyed == true`)되면 즉시 장착 해제 및 세트 카운트에서 제외되어야 한다.
- `iedPct`(방어율 무시)는 덧셈이 아닌 원작 곱연산 `100 - (100 - cur) * (1 - val/100)`으로 합산한다.

## 4. 수정 대상 파일 (`File Scope`)
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/EquipDataLogic.mlua`, `workspaces/ParallelMaple/RootDesk/MyDesk/PlayerDataLogic.mlua`

## 5. Play Mode 완료 조건 (AC)
- [ ] 카루타 3부위(모자/상의/하의) + 앱솔 5부위(무기/장갑/신발/망토/견장) 장착 시 `카루타 3셋`과 `앱솔 5셋` 보너스가 동시에 합산된다.
- [ ] 보스 장신구 3/5/7/9부위 장착에 따라 단계별 스탯 및 보공/방무가 정확히 증가한다.
- [ ] 장비 1부위 해제 시 해당 세트 효과가 즉시 내려간다.
