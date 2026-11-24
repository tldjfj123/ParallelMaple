# REQ-SURV-03: 오프라인 방치 정산 팝업 UI (`OfflinePopupGroup.ui` + `UIOfflinePopup`)

- **도메인:** `SURV` · **유형:** `[UI]` · **우선순위:** `P1`
- **선행 티켓:** `REQ-ARCH-02`, `REQ-SURV-02` · **병렬 가능:** `REQ-SHOP-03`, `REQ-BOSS-04`, `REQ-HUNT-01`과 병렬 가능

## 1. 목표 (User Story)
유저가 1분~12시간 오프라인 후 재접속하면 빅뱅 이후~현대 공식 **`UIWindow4`** 스타일 정산 모달이 자동으로 뜨고, 경과 시간·기준 사냥터·획득 예정 보상을 확인한 뒤 `[보상 수령하기]` 버튼 한 번으로 정산받는다.

## 2. 상세 요구사항 (FR)
1. 독립 UI 파일 `workspaces/ParallelMaple/ui/OfflinePopupGroup.ui` 생성 (공식 **`UIWindow4`** 중앙 모달).
2. 신규 클라이언트 컨트롤러 `UIOfflinePopup` (`.mlua` + `.codeblock`):
   - `OfflineRewardLogic:OnShowOfflineRewardPopup(rewardJson)` 수신 시 팝업 활성화.
   - 표시 항목:
     - 경과 시간 포맷 (`예: 5시간 24분 / 최대 12시간`, 12시간 도달 시 `MAX` 배지 강조)
     - 정산 사냥터 & 스테이지 (`예: Ch.13-3 커닝타워`) + 예상 처치 수 (`예: 7,420마리`)
     - 보상 아이콘 + 숫자 4칸: **경험치**, **메소**, **주문의 흔적**, **수상한 큐브**
   - 하단 대형 **`[보상 수령하기]`** 버튼 → 클릭 즉시 버튼 비활성화(중복 클릭 방지) + `_OfflineRewardLogic:RequestClaimOfflineReward()` 호출 → 팝업 닫기 + 획득 사운드 재생.

## 3. 예외 / 엣지 케이스
- 팝업을 수령하지 않고 `ESC`로 닫았더라도 `pendingOfflineReward`가 서버에 남아 있으므로, HUD 상단에 작은 `[오프라인 보상 미수령]` 배지를 띄워 다시 열 수 있게 처리.

## 4. 수정 대상 파일 (`File Scope`)
- 생성: `workspaces/ParallelMaple/ui/OfflinePopupGroup.ui`, `workspaces/ParallelMaple/RootDesk/MyDesk/UIOfflinePopup.mlua` + `.codeblock`

## 5. Play Mode 완료 조건 (AC)
- [ ] 오프라인 정산 이벤트 수신 시 팝업이 화면 중앙에 깔끔하게 뜨고 모든 수치가 콤마 포맷으로 표시된다.
- [ ] `[보상 수령하기]` 클릭 시 보상이 즉시 HUD에 반영되고 팝업이 닫힌다.
