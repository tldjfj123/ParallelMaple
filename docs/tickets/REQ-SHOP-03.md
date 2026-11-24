# REQ-SHOP-03: NPC 상점창(`B`) 3탭 UI (`ShopUIGroup.ui` + `UIShopController`)

- **도메인:** `SHOP` · **유형:** `[UI]` · **우선순위:** `P1`
- **선행 티켓:** `REQ-ARCH-02`, `REQ-SHOP-02` · **병렬 가능:** `REQ-SURV-01`, `REQ-SURV-03`, `REQ-HUNT-01`, `REQ-BOSS-03`과 병렬 가능

## 1. 목표 (User Story)
충돌 방지를 위해 독립된 `ShopUIGroup.ui` 파일로 빅뱅 이후~현대 공식 **`UIWindow4`** 스타일 NPC 상점창(`B`)을 구현하여, 플레이어가 잡화/강화/보조무기, 펫/자석펫, 보스 코인 교환을 한 창에서 3개 탭으로 편리하게 이용한다.

## 2. 상세 요구사항 (FR)
1. 별도 UI 그룹 `workspaces/ParallelMaple/ui/ShopUIGroup.ui` 생성 (빅뱅 이후~현대 공식 **`UIWindow4`** 프레임 스타일).
2. 신규 클라이언트 컨트롤러 `UIShopController` (`.mlua` + `.codeblock`):
   - 상단 3탭 전환: `[잡화/강화]` / `[펫 상점]` / `[보스 코인]`
   - 상단 보유 재화 바: 현재 탭에 맞춰 보유 메소 또는 5종 보스 코인 수량 표시
   - `[잡화/강화]` 탭: 품목 아이콘 + 이름 + 가격 + `[×1 구매]` / (소비용 `[×5 구매]`), `minJobTier` 미달 품목은 잠금 표시
   - `[펫 상점]` 탭: 현재 펫 수(`N / 3`) & 자석펫 상태(`잠김 / 해금완료`), 각 업그레이드 `[구매]` / `[보유완료]` 배지
   - `[보스 코인]` 탭: 보스별 필터(`자쿰/혼테일/카루타/스우/루시드`) + 장비 아이콘·스탯 요약·필요 코인 + `[교환]` 버튼
3. `AutoHuntComponent`에 `KeyboardKey.B` 단축키 연결 및 `UIIdleManager`에 `shop` 창 토글·`ESC` 전체닫기 등록.

## 3. 예외 / 엣지 케이스
- 메소/코인 부족 시 `[구매]`/`[교환]` 버튼 비활성화 스타일 표시 (클릭해도 서버에서 이중 검증).
- `ESC` 키로 다른 팝업과 동일하게 닫혀야 한다.

## 4. 수정 대상 파일 (`File Scope`)
- 생성: `workspaces/ParallelMaple/ui/ShopUIGroup.ui`, `workspaces/ParallelMaple/RootDesk/MyDesk/UIShopController.mlua` + `.codeblock`
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/AutoHuntComponent.mlua` (`B` 단축키 3줄), `workspaces/ParallelMaple/RootDesk/MyDesk/UIIdleManager.mlua` (창 라우터 등록)

## 5. Play Mode 완료 조건 (AC)
- [ ] `B` 키 또는 HUD 버튼으로 상점창이 열리고 `ESC`로 닫힌다.
- [ ] 3개 탭 전환 및 구매/교환 시 보유 메소·코인·인벤토리·펫이 즉시 갱신된다.
