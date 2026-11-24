# REQ-BOSS-01: 1인 레이드 보스 5단계 서버 코어 (`BossLogic`) & 방무/보공 공식

- **도메인:** `BOSS` · **유형:** `[FUNC]` · **우선순위:** `P1`
- **선행 티켓:** `REQ-ARCH-03`, `REQ-EQUIP-01` · **병렬 가능:** `REQ-HUNT-02`, `REQ-SHOP-03`, `REQ-BOSS-03`과 병렬 가능

## 1. 목표 (User Story)
자쿰·혼테일·카오스 루타비스·스우·루시드 5단계 1인 레이드 보스의 입장/제한시간(10분)/데스카운트(5회)/KST 00:00 일일 횟수 초기화/클리어 보상(코인·큐브·직드랍 장비)을 서버 권위로 구현하고, `PlayerAttack`에 보공·방무(PDR) 공식을 연동한다.

## 2. 상세 요구사항 (FR)
1. `WorldDataLogic.Bosses`에 5종 보스 마스터 등록 (`docs/02-feature-specs.md` F6.1 기준):
   - `zakum` (Lv.50, 2회/일, HP 250K, PDR 40%, 하급 코인 ×5, 직드랍 풀: 자쿰 투구/아쿠아틱/응축/실버블라썸)
   - `horntail` (Lv.120, 2회/일, HP 3.5M, PDR 60%, 중급 코인 ×8, 직드랍 풀: 이피아/데이모스/혼목/뱃지/숄더/데아 이어링)
   - `karuta` (Lv.150, 1회/일, HP 25M, PDR 120%, 루타비스 코인 ×10, 직드랍 풀: 카루타 4부위/매커네이터/골든클로버)
   - `lotus` (Lv.160, 1회/일, HP 120M, PDR 200%, 앱솔 코인 ×10, 직드랍 풀: 앱솔 7부위/핑크빛 성배)
   - `lucid` (Lv.200, 1회/일, HP 600M, PDR 300%, 아케인 코인 ×10, 직드랍 풀: 아케인 7부위)
2. **신규 `BossLogic` (`.mlua` + `.codeblock`) 구현**:
   - **KST 일일 초기화**: `GetKstDateString()` (`DateTime(DateTime.UtcNow.Elapsed + 9*3600*1000)` → `YYYY-MM-DD`) 비교 → 날짜 변경 시 `profile.bossDailyClears = {}` 리셋.
   - `@ExecSpace("Server") RequestEnterBoss(string bossId)`: 레벨 & 남은 일일 클리어 횟수 검증 → `SaveProfileAsync` 후 1인 보스 인스턴스 룸(`BossRoom_<userId>`)으로 이동 & 600초 타이머 + 데스카운트 5 초기화.
   - `OnBossDefeated(string userId)`:
     - **클리어 시에만** `bossDailyClears[bossId] += 1` 차감.
     - 확정 보스 코인 + 메소 + 큐브 + 보스 직드랍(예: 15% 확률로 해당 보스 장비 1종) 지급 & 즉시 저장.
     - `OnBossResult` 전송 → 5초 후 또는 `[마을 귀환]` 클릭 시 헤네시스 복귀.
   - `OnPlayerDeathInBoss(string userId)`: 데스카운트 -1; 남았으면 2초 후 보스 맵 스폰 지점 HP 100% + 3초 무적 부활; 0이면 **횟수 차감 없이** 실패 처리(`OnBossResult`) 및 귀환.
   - 600초 타임아웃 또는 `RequestLeaveBoss()` 시에도 **횟수 차감 없이** 실패/퇴장 처리.
3. **`PlayerAttack:CalcDamage` 보스 보정 추가**:
   - `defender`에 `BossMonster` 컴포넌트가 있으면:
     - `dmgMult = (1 + (dmgPct + bossDmgPct) / 100) * (1 + finalDmgPct / 100)`
     - `effectiveDef = bossPdr * (1 - iedPct / 100)`
     - `iedMult = math.max(0.05, 1 - effectiveDef / 100)`

## 3. 예외 / 엣지 케이스
- 보스전 도중 유저 접속 종료 시 타이머를 즉시 해제해 메모리 누수를 방지한다.
- 인벤토리 60칸이 가득 찬 상태에서 보스 직드랍이 당첨되어도 코인·메소·큐브는 정상 지급되고 인벤 가득 안내 토스트 출력.

## 4. 수정 대상 파일 (`File Scope`)
- 생성: `workspaces/ParallelMaple/RootDesk/MyDesk/BossLogic.mlua` + `.codeblock`
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/WorldDataLogic.mlua`, `workspaces/ParallelMaple/RootDesk/MyDesk/PlayerAttack.mlua`

## 5. Play Mode 완료 조건 (AC)
- [ ] 레벨 미달 및 일일 클리어 소진 시 보스 입장이 차단된다.
- [ ] 방무(IED)와 보공(BossDmg) 잠재가 보스 실제 데미지에 공식대로 반영된다.
- [ ] 실패/포기/시간초과 시 일일 횟수가 차감되지 않고, 클리어 시에만 차감되며 코인·보상이 저장된다.
- [ ] KST 00:00 날짜 변경 시 일일 클리어 횟수가 자동 리셋된다.
