# AGENTS.md — ParallelMaple AI 바이브코딩 공통 규칙

> 이 프로젝트는 **2명 이상의 협업자가 AI 에이전트로 풀 바이브코딩**을 진행하는 MapleStory Worlds(MSW) 프로젝트입니다.  
> 모든 AI 에이전트는 코드를 수정하거나 생성하기 전에 반드시 이 문서를 따릅니다.

---

## 1. 작업 전 필수 확인 체크리스트

1. **문서 우선 확인**: 구현할 티켓을 [docs/07-requirements-index.md](docs/07-requirements-index.md)와 해당 개별 티켓 파일(`docs/tickets/REQ-*.md`)에서 먼저 읽고, 필요 시 상세 명세([docs/02-feature-specs.md](docs/02-feature-specs.md))와 스키마/RPC 계약([docs/05-architecture.md](docs/05-architecture.md))을 참조합니다.
2. **티켓 스코프 준수**: 할당된 티켓의 **"수정 대상 파일"** 범위를 벗어나지 않습니다. 다른 티켓이 수정 중인 공용 파일을 임의로 리팩토링하지 않습니다.
3. **한글 파일명 금지**: 새로 생성하는 스크립트, 맵, UI, 문서, 이미지 파일명은 반드시 **영문 ASCII**(`PascalCase` 또는 `kebab-case`)를 사용합니다. *(macOS NFD / Windows NFC Git 충돌 방지)*

---

## 2. MapleStory Worlds (MSW) 기술 필수 규칙

### 2.1 스크립트 파일 쌍 (`.mlua` + `.codeblock`)
- `workspaces/ParallelMaple/RootDesk/MyDesk/`의 모든 스크립트는 **`.mlua`와 `.codeblock` 파일이 쌍으로 존재**합니다.
- 새 스크립트를 로컬에서 직접 추가할 경우, MSW Maker에서 `Reimport All` 시 정상 인식되도록 **MSW Maker에서 빈 스크립트를 생성하거나 기존 `.codeblock` 메타 형식을 정확히 복제하되 고유 `Id`를 새로 발급**해야 합니다.
- 스크립트 이름/클래스명을 바꿀 때는 `.mlua` 내부 `script <Name> extends ...`와 `.codeblock`의 `Name` 필드를 일치시켜야 합니다.

### 2.2 실행 공간 & 서버 권한 보안 (`@ExecSpace`)
- **모든 재화 증감, 경험치/레벨업, 드랍 판정, 강화 확률 판정, 보스 클리어/보상, 장비 장착/분해, 상점 거래, 데이터 저장은 반드시 `@ExecSpace("ServerOnly")` 또는 `@ExecSpace("Server")`에서 처리**합니다.
- `@ExecSpace("Server")` RPC 메서드는 클라이언트가 보낸 `userId`를 신뢰하지 않고 항상 암묵적 매개변수 **`senderUserId`** 를 사용합니다.
- 컴포넌트 RPC(`PlayerAttack` 등)에서는 `senderUserId == self.Entity.PlayerComponent.UserId` 소유권 검증을 수행합니다.
- 클라이언트(`ClientOnly` / `Client`)는 UI 렌더링, 입력 처리, 로컬 이펙트/사운드, 예측 쿨타임 표시만 담당합니다.

### 2.3 시간 계산 (오프라인 보상 / 일일 초기화)
- `_UtilLogic.ServerElapsedSeconds`는 **서버 인스턴스 기동 후 경과 초**이므로 서버가 재시작되면 0으로 초기화됩니다.
- 오프라인 방치 시간 계산, 일일 보스 입장 횟수 초기화(KST 00:00) 등 **영속적인 실제 시각 비교에는 절대 `ServerElapsedSeconds`를 쓰지 말고 `DateTime.UtcNow` (Epoch 밀리초/초)** 를 사용합니다.

### 2.4 `DataStorageService` 스키마 호환성
- 현재 저장 키: `"idle_save_v1"` (단일 JSON 인코딩 문자열).
- 프로필 스키마에 새 필드를 추가할 때는 **기존 저장 데이터(`version = 2` 등)를 깨뜨리지 않도록 `LoadProfileForUser`에 안전한 기본값 마이그레이션 로직을 반드시 추가**합니다.
- JSON 직렬화 전 `SanitizeForJson` 규칙을 유지하며, 빈 테이블이 `nil`로 생략될 수 있으므로 클라이언트(`OnReceiveSnapshot`)와 서버 로드 시 `field = field or {}` 방어 코드를 둡니다.

### 2.5 MSW RUID 규칙
- 검증되지 않은 임의의 RUID 해시 문자열을 추측해서 넣지 않습니다.
- [docs/03-data-reference.md](docs/03-data-reference.md)에 등록된 검증 RUID를 우선 사용하고, 새 에셋이 필요하면 `msw-mcp`의 `asset_search_resources` 등으로 조회 후 문서에 함께 기록합니다.

---

## 3. 2인 협업 Git 충돌 방지 규칙

### 3.1 `.map` / `.ui` 파일은 동시 수정 금지
- `workspaces/ParallelMaple/map/*.map`과 `workspaces/ParallelMaple/ui/*.ui`는 대용량 JSON 포맷이라 Git 3-way merge가 매우 취약합니다.
- **한 번에 한 사람만 특정 `.map` 또는 `.ui` 파일을 수정**합니다.
- 새 UI 창을 추가할 때는 기존 `.ui` 파일을 거대하게 키우기보다 **가능하면 별도 `.ui` 그룹 파일**(예: `BossUIGroup.ui`, `ShopUIGroup.ui`, `OfflinePopupGroup.ui`)로 분리하는 것을 권장합니다.

### 3.2 거대 단일 스크립트 분리 원칙 (Phase 0 최우선)
- 현재 `UIIdleManager.mlua` (1,454줄)와 `GameDataLogic.mlua` (1,337줄)는 충돌 위험이 매우 높습니다.
- [docs/05-architecture.md](docs/05-architecture.md)의 모듈 분리 설계에 따라 **Phase 0(모듈 분리)을 완료한 뒤 기능 개발을 병렬 진행**합니다.
- 이후 새 기능 추가 시 거대 매니저 파일에 코드를 덧붙이지 않고 **전용 Logic/UI 스크립트**로 분리합니다.

### 3.3 커밋 전 자가 검증
1. `git status`로 의도하지 않은 `.map` 부동소수점 재저장 노이즈(메이커가 열기만 해서 0.00001 바뀐 파일)가 섞이지 않았는지 확인합니다. 관련 없는 `.map` 노이즈는 `git checkout -- <file>`로 되돌립니다.
2. 변경된 스키마, RPC, 데이터 테이블, 완료된 티켓이 있다면 **`docs/` 해당 문서도 같은 커밋/PR에서 함께 갱신**합니다.
3. MSW Maker Play Mode에서 변경한 기능의 정상 동작 및 콘솔 에러 0건을 확인합니다.

---

## 4. 코딩 컨벤션 (mLua)

- **네이밍**
  - Logic / Component 스크립트: `PascalCase` (예: `BossLogic`, `ShopLogic`, `UIEnhanceController`)
  - 메서드 / 프로퍼티: `PascalCase` (예: `RequestBuyItem`, `RecalculateStats`)
  - 클라이언트 → 서버 RPC: `Request<Action>` (예: `RequestEnterBoss`, `RequestDismantleItems`)
  - 서버 → 클라이언트 RPC: `On<Event>` 또는 `Notify<Event>` (예: `OnReceiveSnapshot`, `OnBossStateChanged`)
  - 로컬 변수 / JSON 테이블 키: `camelCase` (예: `bossCoins`, `dailyBossEntries`)
- **방어 코드**
  - 엔티티 접근 전 항상 `isvalid(entity)` 및 컴포넌트 `isvalid(entity.TransformComponent)` 체크
  - `_GameDataLogic:EnsureInit()` (또는 분리된 데이터 모듈의 `EnsureInit()`)를 모든 서버/클라 진입점에서 보장
- **로그**
  - 서버 디버그 로그는 `log("[ScriptName] message")` 형식 통일, 매 프레임(`OnUpdate`) 내부 `log()` 호출 금지
