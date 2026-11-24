# REQ-BOSS-04: 보스 입장창(`T`) · 보스 상단 HUD · 클리어/실패 결과 팝업 (`BossUIGroup.ui`)

- **도메인:** `BOSS` · **유형:** `[UI]` · **우선순위:** `P1`
- **선행 티켓:** `REQ-ARCH-02`, `REQ-BOSS-01` · **병렬 가능:** `REQ-SHOP-03`, `REQ-SURV-03`, `REQ-HUNT-04`와 병렬 가능

## 1. 목표 (User Story)
독립된 `BossUIGroup.ui` 파일로 보스 입장창(`T`), 보스전 상단 전용 HUD(보스 HP 바 / 남은 시간 / 데스카운트), 클리어·실패 결과 모달을 구현한다.

## 2. 상세 요구사항 (FR)
1. 별도 UI 그룹 `workspaces/ParallelMaple/ui/BossUIGroup.ui` 생성 (빅뱅 이후~현대 공식 **`UIWindow4`** 프레임 + 보스 HP 바 스타일).
2. 신규 클라이언트 컨트롤러 `UIBossController` (`.mlua` + `.codeblock`):
   - **① 보스 입장 선택창 (`T`)**:
     - 5단계 보스 카드 리스트 (`자쿰 / 혼테일 / 카루타 / 스우 / 루시드`)
     - 카드별 표시: 보스 아이콘, 요구 레벨, 권장 전투력, 남은 일일 클리어 횟수(`2/2`, `1/1`), 획득 코인·핵심 드랍 요약, `[입장하기]` 버튼.
   - **② 보스전 상단 HUD** (`BossLogic:OnBossStateUpdate` 수신 시 활성화):
     - 보스 이름 + 대형 붉은 HP 바 (`현재 HP / 최대 HP · XX.X%`)
     - 남은 제한시간 (`09:42`) + 남은 데스카운트 (`♥ ♥ ♥ ♥ ♥`)
     - 우측 `[포기/퇴장]` 버튼 (확인 팝업 후 `RequestLeaveBoss()`).
   - **③ 결과 모달** (`BossLogic:OnBossResult` 수신 시 표시):
     - 클리어: 소요 시간 + 획득 코인/메소/큐브/직드랍 장비 아이콘 + `[헤네시스 귀환]` 버튼.
     - 실패: 실패 사유(데스카운트 소진 / 시간 초과) + `"일일 입장 횟수는 차감되지 않았습니다"` 안내 + `[헤네시스 귀환]` 버튼.
3. `AutoHuntComponent`에 `KeyboardKey.T` 단축키 연결 및 `UIIdleManager`에 `boss` 창 토글·`ESC` 닫기 등록.

## 3. 예외 / 엣지 케이스
- 보스전 입장 중에는 `T` 입장창과 `M` 사냥터 이동창 열기를 차단하고 `[포기/퇴장]`으로만 나가도록 일관화.

## 4. 수정 대상 파일 (`File Scope`)
- 생성: `workspaces/ParallelMaple/ui/BossUIGroup.ui`, `workspaces/ParallelMaple/RootDesk/MyDesk/UIBossController.mlua` + `.codeblock`
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/AutoHuntComponent.mlua` (`T` 단축키 3줄), `workspaces/ParallelMaple/RootDesk/MyDesk/UIIdleManager.mlua` (창 라우터 등록)

## 5. Play Mode 완료 조건 (AC)
- [ ] `T` 키로 보스 입장창이 열리고 레벨/횟수 미달 보스는 입장 버튼이 비활성화된다.
- [ ] 보스전 입장 시 상단 HP 바·타이머·데스카운트가 실시간으로 정확히 갱신된다.
- [ ] 클리어/실패 결과 모달이 정상 표시되고 `[헤네시스 귀환]`으로 마을 복귀된다.
