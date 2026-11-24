# REQ-ARCH-02: `UIIdleManager` 기능별 4모듈 분리

- **도메인:** `ARCH` · **유형:** `[UI]` · **우선순위:** `P0`
- **선행 티켓:** 없음 · **병렬 가능:** `REQ-ARCH-01`과 완전 병렬 가능

## 1. 목표 (User Story)
1,454줄짜리 단일 `UIIdleManager.mlua`를 역할별 UI 컨트롤러로 분리하여, 이후 장비/인벤/스탯/강화/사냥터 UI를 2명이 동시에 작업해도 충돌하지 않게 만든다.

## 2. 상세 요구사항 (FR)
1. 아래 3개의 신규 클라이언트 전용 Logic 스크립트(`.mlua` + `.codeblock` 쌍)를 생성한다:
   - `UICharacterWindows`: 장비창(`E`), 인벤토리창(`I`), 스탯창(`S`), 스킬창(`K`) 이벤트 바인딩 및 렌더링 (`RefreshEquipWindow`, `RefreshInvenWindow`, `OnClickInvenAction`, `RefreshStatWindow`, `RefreshSkillWindow`)
   - `UIEnhanceController`: 강화·큐브 통합창(`U`/`C`) 상태·바인딩·렌더링 (`RefreshEnhancePanel`, `RenderDetailCard`, `RenderActionTab`, `OnClickAction`, `SetEnhanceLog`, `CurrentTab`, `SelectedUid`, `UseSafeguard`, `UseStarCatch`)
   - `UINavigationWindows`: 사냥터 이동창(`M`) 바인딩 및 렌더링 (`RefreshHuntMapWindow`, `HuntMapPage`, `DisplayedMaps`)
2. `UIIdleManager`는 **창 열기/닫기 라우터(`ToggleWindow`, `SetWindowOpen`, `CloseAllWindows`, `ToggleEnhancePanel`) + 메인 HUD(`RefreshHud`, `RefreshHudButtons`) + 우측 획득 로그(`PushGainLog`, `RenderGainLogs`) + 공용 유틸(`GetGradeColor`, `GetItemIcon`, `BuildStarRow`, `FindClientItem`, `IsClientItemEquipped`)** 만 유지한다.
3. `_UIIdleManager:RefreshAll()`은 현재 열려있는 각 컨트롤러의 Refresh 메서드를 위임 호출한다.

## 3. 예외 / 엣지 케이스
- 기존 `AutoHuntComponent`, `PlayerDataLogic`, `EnhanceLogic`이 호출하는 `_UIIdleManager:RefreshAll()`, `ToggleWindow()`, `ToggleEnhancePanel()`, `SetEnhanceOpen()`, `SetEnhanceLog()`, `CloseAllWindows()`, `PushGainLog()` 시그니처는 100% 하위 호환 유지.
- `BindUIEvents()` 중복 바인딩 방지 플래그를 각 컨트롤러별로 독립 관리한다.

## 4. 수정 대상 파일 (`File Scope`)
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/UIIdleManager.mlua`
- 생성: `UICharacterWindows.mlua` + `.codeblock`, `UIEnhanceController.mlua` + `.codeblock`, `UINavigationWindows.mlua` + `.codeblock`
- *(주의: `.ui` 파일은 이 티켓에서 건드리지 않는다.)*

## 5. Play Mode 완료 조건 (AC)
- [ ] 단축키 `E / I / S / K / M / U / C / ESC`가 모두 기존과 동일하게 창을 열고 닫음.
- [ ] 장비 장착, 포션 사용, 전리품 판매, 주문서/스타포스/큐브 실행, 사냥터 이동 버튼이 전부 정상 동작하며 콘솔 에러 0건.
- [ ] `UIIdleManager.mlua` 파일 크기가 500줄 이하로 축소됨.
