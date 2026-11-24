# REQ-SURV-01: HP 50% 자동 포션(1초 쿨, 25% 파엘 우선) & 무패널티 헤네시스 부활

- **도메인:** `SURV` · **유형:** `[FUNC]` · **우선순위:** `P1`
- **선행 티켓:** `REQ-ARCH-03` · **병렬 가능:** `REQ-SURV-02`, `REQ-SHOP-03`, `REQ-BOSS-03`과 병렬 가능

## 1. 목표 (User Story)
임시 무적 회복을 제거하고, 피격 시 HP 50% 이하에서 보유 포션을 자동 사용하며, 포션 소진으로 사망해도 일반 사냥터에서는 경험치 손실 없이 헤네시스로 안전 부활한다.

## 2. 상세 요구사항 (FR)
1. **임시 무적 제거**: `PlayerDataLogic:RecalculateStats`의 `Hp < totalMaxHp * 0.65` 시 강제 풀회복 코드를 제거한다. *(단, `Hp > totalMaxHp` 클램프만 유지)*
2. **신규 `SurvivalLogic` (`.mlua` + `.codeblock`) 구현**:
   - `OnPlayerDamaged(Entity playerEntity)` (서버 전용, `PlayerHit`에서 피격 직후 호출):
     - **사망 판정 (`Hp <= 0`)**:
       - 현재 보스전 중이면 `BossLogic:OnPlayerDeathInBoss(userId)`에 위임.
       - 일반 사냥터이면 `DeadUsers[userId] = true` 잠금 → `2.0초` 후 `Hp = MaxHp` 회복 + `HuntingInstanceLogic:RequestReturnTown` 호출 + 클라이언트 자동사냥 OFF + 토스트 `"체력이 0이 되어 헤네시스로 귀환했습니다. (경험치 손실 없음)"`.
     - **자동 포션 (`Hp > 0` & `profile.autoPotionEnabled`)**:
       - `Hp / MaxHp <= (profile.autoPotionThreshold or 0.50)`이고 내부 쿨타임(`1.0초`) 경과 시 발동.
       - 우선순위: `Hp / MaxHp <= 0.25` 위급 시 `power_elixir` 우선, 그 외 `red_potion` → `white_potion` → `blue_potion` → `power_elixir` 중 보유 > 0인 포션 1개 자동 차감 & HP 회복 & 스냅샷 푸시.
       - 보유 포션 0개 시 30초 쿨타임으로 토스트 `"⚠️ 보유한 포션이 없습니다! 상점(B)에서 보충하세요."`.
   - `@ExecSpace("Server") RequestSetAutoPotion(boolean enabled, number threshold)`: `0.1 <= threshold <= 0.9` 클램프 후 저장.

## 3. 예외 / 엣지 케이스
- 연타 피격으로 `OnPlayerDamaged`가 한 프레임에 여러 번 들어와도 1초 쿨타임과 `DeadUsers` 플래그로 포션 중복 소모·중복 부활을 차단한다.

## 4. 수정 대상 파일 (`File Scope`)
- 생성: `workspaces/ParallelMaple/RootDesk/MyDesk/SurvivalLogic.mlua` + `.codeblock`
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/PlayerHit.mlua`, `workspaces/ParallelMaple/RootDesk/MyDesk/PlayerDataLogic.mlua` (임시 회복 3줄 제거)

## 5. Play Mode 완료 조건 (AC)
- [ ] 피격으로 HP가 50% 이하가 되면 서버에서 포션 1개가 자동 차감되고 HP가 회복된다.
- [ ] HP가 25% 이하로 떨어졌을 때 파워 엘릭서가 있으면 우선 소모된다.
- [ ] 포션 0개 상태에서 HP가 0이 되면 경험치 감소 없이 2초 후 헤네시스로 부활하고 자동사냥이 꺼진다.
