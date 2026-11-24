# REQ-STAT-01: 수동 AP 배분(`STR/DEX/INT/LUK`) & `(STR×4+DEX)`·맨손 기본공 공식

- **도메인:** `STAT` · **유형:** `[FUNC]` · **우선순위:** `P0`
- **선행 티켓:** `REQ-ARCH-03` · **병렬 가능:** `REQ-EQUIP-01`, `REQ-INVEN-01`, `REQ-BOSS-03`과 병렬 가능

## 1. 목표 (User Story)
플레이어가 레벨업할 때마다 AP +5를 받아 `STR / DEX / INT / LUK`에 직접 또는 자동 배분하고, 원작 본메 공식 `(STR×4+DEX) × ATT` 및 맨손 기본 공격력으로 전투할 수 있다.

## 2. 상세 요구사항 (FR)
1. **레벨업 AP 지급**: `PlayerDataLogic:GrantMonsterKillRewardWithDrops`의 레벨업 루프에서 1레벨 상승할 때마다 `profile.unspentAp = (profile.unspentAp or 0) + 5`.
2. **서버 RPC 3종 (`PlayerDataLogic`)**:
   - `@ExecSpace("Server") RequestAllocateAp(string statKey, integer amount)`:
     - `statKey`가 `"str"`, `"dex"`, `"int"`, `"luk"` 중 하나이고 `amount >= 1`이며 `amount <= profile.unspentAp`인지 검증.
     - `profile.baseStats[statKey] += amount`, `profile.unspentAp -= amount`, 스냅샷 푸시 & 즉시 저장.
   - `@ExecSpace("Server") RequestAutoAllocateAp()`:
     - `profile.unspentAp > 0`이면 전량 `profile.baseStats.str`에 가산 후 `unspentAp = 0`, 스냅샷 푸시 & 저장.
   - `@ExecSpace("Server") RequestResetAp()`:
     - 인벤토리/소비에 `ap_reset_scroll` 1개 이상 보유 시 1개 차감 → `str/dex/int/luk`를 모두 `4`로 초기화하고 `unspentAp = 5 + (profile.level - 1) * 5` 복구.
3. **스탯 & 데미지 공식 개편**:
   - `RecalculateStats`에서 `baseStats.str`/`dex`/`int`/`luk` + 전직/도감/장비/스타포스/잠재를 합산해 `totalStr`, `totalDex`, `totalInt`, `totalLuk` 계산.
   - 무기 미장착(맨손) 시 최소 기본 공격력 `baseAtt = 12` 보장(Lv.1 맨손 사냥 가능).
   - `PlayerAttack:CalcDamage`와 전투력 계산을 `statValue = (stats.totalStr * 4) + stats.totalDex` 기반으로 변경.

## 3. 예외 / 엣지 케이스
- `amount <= 0`, 소수점, 존재하지 않는 `statKey`, `amount > unspentAp` 요청은 서버에서 즉시 무시한다.
- `AP 초기화 주문서` 미보유 시 토스트 `"AP 초기화 주문서가 없습니다. 상점에서 구매하세요."` 출력.

## 4. 수정 대상 파일 (`File Scope`)
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/PlayerDataLogic.mlua`, `workspaces/ParallelMaple/RootDesk/MyDesk/PlayerAttack.mlua`

## 5. Play Mode 완료 조건 (AC)
- [ ] 레벨업 시 `unspentAp`가 정확히 +5 증가한다.
- [ ] `STR`/`DEX` 투자 시 `totalStr`/`totalDex` 및 전투력·실제 몬스터 타격 데미지가 상승한다.
- [ ] 무기 미장착 맨손 상태에서도 Lv.1 헤네시스 사냥터 1 몬스터에게 유효 데미지가 들어간다.
- [ ] 잘못된 RPC 값(`amount = -5`, `statKey = "hack"`) 호출 시 데이터 변조 없이 차단된다.
