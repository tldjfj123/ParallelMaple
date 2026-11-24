# REQ-SURV-02: `DateTime.UtcNow` 기반 최대 12시간 오프라인 방치 보상 정산

- **도메인:** `SURV` · **유형:** `[FUNC]` · **우선순위:** `P1`
- **선행 티켓:** `REQ-ARCH-03` · **병렬 가능:** `REQ-SURV-01`, `REQ-SHOP-03`, `REQ-BOSS-03`과 병렬 가능

## 1. 목표 (User Story)
유저가 게임을 꺼두어도 마지막으로 사냥하던 챕터·스테이지 기준으로 최대 12시간까지 80% 효율의 오프라인 보상이 정확하게 누적되고, 재접속 시 중복 수령 없이 정산받을 수 있다.

## 2. 상세 요구사항 (FR)
1. **신규 `OfflineRewardLogic` (`.mlua` + `.codeblock`) 구현**:
   - `ComputeOfflineRewardOnLogin(string userId)` (`LoadProfileForUser` 완료 직후 서버에서 호출):
     - `nowUtcMs = DateTime.UtcNow.Elapsed`
     - `lastUtcMs = profile.lastSaveUtcMs or 0`; `lastUtcMs <= 0`이면 스킵.
     - `elapsedSec = math.floor((nowUtcMs - lastUtcMs) / 1000)`
     - `elapsedSec < 60`이면 스킵 (단, 기존 미수령 `profile.pendingOfflineReward`가 있으면 팝업 재전송).
     - `cappedSec = math.min(elapsedSec, 12 * 3600)` (최대 43,200초).
     - 기준 맵 = `profile.lastHuntingMapName` (없으면 `"HenesysHunt1"`), 스테이지 = `profile.currentStage or 1`.
     - 분당 기대 처치 수: `killsPerMin = 18 * (1 + (profile.jobTier or 0) * 0.15)` *(스타포스 미달 맵이면 `× 0.65`)*.
     - 총 예상 처치 수: `kills = math.max(1, math.floor(killsPerMin * (cappedSec / 60) * 0.80))`.
     - 보상 산출 (`exp`, `meso`, `trace`, `suspiciousCube` — `docs/02-feature-specs.md` F8.1 공식 준수).
     - `profile.pendingOfflineReward`에 저장 후 클라이언트 RPC `OnShowOfflineRewardPopup(json)` 전송.
   - `@ExecSpace("Server") RequestClaimOfflineReward()`:
     - `local r = profile.pendingOfflineReward; if not r then return end`
     - **즉시 `profile.pendingOfflineReward = nil` 소거** *(더블클릭 중복 수령 원천 차단!)*.
     - 경험치(레벨업 + AP +5 루프 포함), 메소, 주문의 흔적, 수상한 큐브 지급 → `SaveProfileAsync` → `PushSnapshotToClient`.

## 3. 예외 / 엣지 케이스
- 서버 재시작 후 재접속해도 `DateTime.UtcNow.Elapsed` 기반이라 오차가 없어야 한다.
- 시계 역전 등으로 `elapsedSec < 0`이면 무시하고 `lastSaveUtcMs = nowUtcMs`로 보정.
- Lv.200 만렙 유저는 `exp = 0`으로 처리하고 메소/주흔/큐브만 정상 지급.

## 4. 수정 대상 파일 (`File Scope`)
- 생성: `workspaces/ParallelMaple/RootDesk/MyDesk/OfflineRewardLogic.mlua` + `.codeblock`
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/PlayerDataLogic.mlua` (`LoadProfileForUser` 훅 3줄)

## 5. Play Mode 완료 조건 (AC)
- [ ] 60초 미만 재접속 시 오프라인 보상 팝업이 뜨지 않는다.
- [ ] `lastSaveUtcMs`를 20시간 전으로 세팅해 접속하면 정확히 12시간(43,200초)으로 상한 캡이 적용된다.
- [ ] 수령 요청을 2번 연속 보내도 보상은 정확히 1회만 지급된다.
- [ ] 오프라인 경험치로 다중 레벨업 시 레벨과 `unspentAp`(+5/Lv)가 정확히 올라간다.
