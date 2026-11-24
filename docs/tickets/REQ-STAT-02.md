# REQ-STAT-02: 스탯창(`S`) AP 배분 UI · 활성 세트 · 자동포션 상태 표시

- **도메인:** `STAT` · **유형:** `[UI]` · **우선순위:** `P1`
- **선행 티켓:** `REQ-ARCH-02`, `REQ-STAT-01` · **병렬 가능:** `REQ-INVEN-01`, `REQ-SHOP-02`, `REQ-HUNT-01`과 병렬 가능

## 1. 목표 (User Story)
플레이어가 빅뱅 스타일 스탯창(`S`)에서 남은 AP와 `STR / DEX / INT / LUK` 수치를 확인하고 `[+]` / `[자동 배분]` / `[AP 초기화]` 버튼으로 편리하게 스탯을 찍을 수 있다.

## 2. 상세 요구사항 (FR)
1. `IdleHUDGroup.ui`의 스탯창 패널에 아래 UI 요소 배치:
   - 남은 AP 라벨 (`ABILITY POINT: N`)
   - `STR`, `DEX`, `INT`, `LUK` 4행 각각: 총합 수치(예: `124 (45+79)`) + **`[+]`** 버튼 (`+1`, Shift 또는 `[+10]` 보조 선택 가능)
   - **`[자동 배분]`** 버튼 (남은 AP > 0일 때 활성화)
   - **`[AP 초기화]`** 버튼 (보유 `ap_reset_scroll` 수량 표시)
   - 하단 확장 영역: 활성 세트 효과 요약(예: `카루타 3/4 · 앱솔 5/7`) + 자동 포션 ON/OFF 토글(`50% 이하 자동 사용`)
2. `UICharacterWindows:RefreshStatWindow()`가 최신 스냅샷의 `profile.baseStats`, `profile.unspentAp`, `stats.totalStr/Dex/Int/Luk`, `stats.activeSets`를 바인딩한다.

## 3. 예외 / 엣지 케이스
- `unspentAp == 0`일 때 `[+]` 버튼과 `[자동 배분]` 버튼은 비활성화(회색) 처리.
- 버튼 연타 시 스냅샷 갱신 전 중복 요청을 막기 위해 0.15초 디바운스 또는 서버 검증에 의존.

## 4. 수정 대상 파일 (`File Scope`)
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/UICharacterWindows.mlua`, `workspaces/ParallelMaple/ui/IdleHUDGroup.ui` *(주의: `.ui` 단독 수정 조율 필요)*

## 5. Play Mode 완료 조건 (AC)
- [ ] `S` 키로 스탯창을 열면 `STR/DEX/INT/LUK`와 남은 AP가 정확히 표시된다.
- [ ] `[+]` 및 `[자동 배분]` 클릭 시 즉시 스탯과 전투력이 갱신된다.
- [ ] `unspentAp == 0`이면 배분 버튼들이 깔끔하게 비활성화된다.
