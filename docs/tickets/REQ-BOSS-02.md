# REQ-BOSS-02: 보스 몬스터 컴포넌트 & 5종 시그니처 패턴 AI (`BossMonster`)

- **도메인:** `BOSS` · **유형:** `[FUNC]` · **우선순위:** `P1`
- **선행 티켓:** `REQ-BOSS-01` · **병렬 가능:** `REQ-BOSS-03`, `REQ-BOSS-04`, `REQ-HUNT-04`와 병렬 가능

## 1. 목표 (User Story)
5단계 레이드 보스 각각이 고유의 체력·방어율(PDR)·시그니처 패턴(1.2초 경고 → `MaxHp` 비율 피해)을 구사해, 자동사냥과 수동 회피·포션 관리의 긴장감을 제공한다.

## 2. 상세 요구사항 (FR)
1. 신규 컴포넌트 `BossMonster` (`.mlua` + `.codeblock`) 구현 (`@Sync` 프로퍼티: `BossId`, `MaxHp`, `Hp`, `Pdr`, `IsDead`):
   - `HandleHitEvent`: 데미지 반영 → `BossLogic:NotifyBossHpChanged(userId, Hp, MaxHp)` 호출 → `Hp <= 0` 시 `BossLogic:OnBossDefeated(userId)` 트리거.
2. 보스별 패턴 2~3종 주기 실행 (서버 `OnUpdate` / 타이머 기반):
   - **`zakum` (자쿰)**: ① 팔 내려찍기(플레이어 위치 기준 경고 존 `1.2초` → 최대 HP 30% 피해) / ② 낙석 3곳(경고 `1.2초` → 최대 HP 20%)
   - **`horntail` (혼테일)**: ① 전방 브레스(넓은 박스 `1.2초` → 최대 HP 35%) / ② 꼬리 내려치기(하단 존 `1.2초` → 최대 HP 30%)
   - **`karuta` (벨룸)**: ① 땅속 잠복 후 돌출(플레이어 X 추적 경고 `1.2초` → 최대 HP 50%) / ② 낙석(최대 HP 25% + 1.5초 이동 둔화)
   - **`lotus` (스우)**: ① 다중 낙하물(최대 HP 35%) / ② 전방 밀격 레이저(경고 `1.2초` → 최대 HP 45% + 넉백)
   - **`lucid` (루시드)**: ① 초승달 검기 투사체(최대 HP 35%) / ② 시한 폭탄 경고(2.0초 후 폭발 → 최대 HP 55%)
3. 패턴 발동 시 `BossPatternVisualLogic:OnShowTelegraph(patternType, pos, size, duration)` 멀티캐스트/클라 RPC 호출 후 `1.2초` 뒤 판정 실행.

## 3. 예외 / 엣지 케이스
- 플레이어가 무적 시간(부활 후 3초) 중이면 패턴 피해를 0으로 무시한다.
- 보스가 사망하거나 유저가 퇴장하면 진행 중이던 모든 지연 패턴 타이머를 즉시 취소한다.

## 4. 수정 대상 파일 (`File Scope`)
- 생성: `workspaces/ParallelMaple/RootDesk/MyDesk/BossMonster.mlua` + `.codeblock`

## 5. Play Mode 완료 조건 (AC)
- [ ] 5종 보스 각각에서 최소 2개 패턴이 1.2초 예고 후 정확한 범위와 `MaxHp` 비율 피해로 적중한다.
- [ ] 경고 존 밖으로 이동/점프 회피 시 피해를 받지 않는다.
- [ ] 보스 처치 시 `BossLogic:OnBossDefeated`가 정확히 1회 호출된다.
