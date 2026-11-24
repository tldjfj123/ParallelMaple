# REQ-HUNT-03: 사냥 도파민 이벤트 4종 (엘리트 몬스터 · 룬 · 30분 버닝 · 도파민 상자 3택1)

- **도메인:** `HUNT` · **유형:** `[FUNC]` · **우선순위:** `P1`
- **선행 티켓:** `REQ-HUNT-02` · **병렬 가능:** `REQ-BOSS-01`, `REQ-SURV-02`, `REQ-SHOP-03`과 병렬 가능

## 1. 목표 (User Story)
1인 방치 사냥 중에도 가끔 화면을 볼 때마다 도파민이 터지도록 **엘리트 몬스터(100마리), 경험치 2배 룬(10분), 30분 로테이션 버닝 필드, 사냥 도파민 상자(3택 1)** 4종을 구현한다.

## 2. 상세 요구사항 (FR)
1. **신규 `HuntingEventLogic` (`.mlua` + `.codeblock`) 구현**:
   - **① 엘리트 몬스터**: 일반 몹 1마리 처치 시 `profile.eliteKillCounter += 1`. `100` 도달 시 `0` 리셋 + 현재 맵에 거대 엘리트 몬스터 1마리 스폰 (`Scale 1.45x`, HP `15x`, EXP `10x`, 처치 시 주흔 `5x` + 큐브 100% + 장비 50% 확정 드랍).
   - **② 룬 (경험치 2배 3분)**: 개인 사냥터 체류 10분마다(최초 입장 3분 후 1회) 랜덤 발판에 룬 오브젝트 1개 스폰. 유저가 접근해 `RequestActivateRune(runeId)` 호출 시 `runeBuffUntilUtcMs = now + 180000` 설정 → 3분간 최종 경험치 `+100%`.
   - **③ 30분 로테이션 버닝 필드**: `math.floor(DateTime.UtcNow.Elapsed / (30 * 60 * 1000))` 결정론적 시드(Deterministic Seed)를 사용해 **모든 서버/인스턴스 룸에서 동일하게** 18개 챕터 중 5곳에 버닝 `3~10단계` (`+30% ~ +100%` 추경) 부여! *(별도 서버 통신 없이 Static/Instance 어디서나 100% 일치!)*
   - **④ 사냥 도파민 상자 (여정의 보물 컨버트)**: 몹 처치 시 `treasureGauge += (0.8 + currentChapter * 0.05)` (100 도달 시 `treasureGauge -= 100`, `treasurePendingCount = min(5, count + 1)`).
     - `RequestClaimTreasureCard(integer cardIndex)` (`1~3`): `treasurePendingCount > 0` 검증 → 차감 → 3장 카드 중 선택한 보상 지급 *(카드1: 주문의 흔적 대량 / 카드2: 레드·블랙 큐브 / 카드3: 대량 메소 — 챕터 비례 스케일링)* + 즉시 저장.

## 3. 예외 / 엣지 케이스
- 이미 엘리트 몬스터나 룬이 맵에 존재하면 중복 스폰하지 않는다.
- `treasurePendingCount == 0`이거나 `cardIndex`가 `1~3` 범위 밖이면 서버에서 즉시 거부.

## 4. 수정 대상 파일 (`File Scope`)
- 생성: `workspaces/ParallelMaple/RootDesk/MyDesk/HuntingEventLogic.mlua` + `.codeblock`
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/Monster.mlua` (사망 시 `HuntingEventLogic` 훅 호출), `workspaces/ParallelMaple/RootDesk/MyDesk/PlayerDataLogic.mlua` (경험치 계산 시 룬 +100% & 버닝 추경 합산)

## 5. Play Mode 완료 조건 (AC)
- [ ] 100마리 처치 시 거대 엘리트 몬스터가 등장하고 격파 시 큐브/주흔 대량 드랍이 터진다.
- [ ] 룬 발동 후 3분 동안 획득 경험치가 정확히 +100% 증가한다.
- [ ] 30분 단위 버닝 필드가 마을과 개인 사냥터 양쪽에서 동일한 5개 챕터로 표시된다.
- [ ] 도파민 상자 게이지 100% 시 3택 1 보상 수령 및 중복 수령 방지가 정상 작동한다.
