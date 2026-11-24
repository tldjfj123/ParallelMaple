# ParallelMaple (패러렐메이플)

> **MapleStory Worlds (MSW)** 기반 정통 빅뱅 감성 방치형 RPG  
> 공용 사냥터 자동/수동 사냥 + 3대 강화(주문의 흔적 / 스타포스 / 큐브) + 1인 보스전 & 오프라인 방치 보상

---

## 1. 폴더 구조

```text
ParallelMaple/
├── README.md                        # 프로젝트 소개 & 협업자 세팅 가이드
├── AGENTS.md                        # AI 바이브코딩 공통 규칙 (충돌 방지 / MSW 제약)
├── docs/
│   ├── 01-game-design.md            # 정식 게임 기획서 (비전, 코어 루프, As-Is vs To-Be)
│   ├── 02-feature-specs.md          # 기능별 상세 명세서 (F1~F8 유저 플로우·규칙·AC)
│   ├── 03-data-reference.md         # 마스터 데이터 & 검증된 MSW RUID 레퍼런스
│   ├── 04-ui-spec.md                # 빅뱅 스타일 UI 화면별 명세 & 단축키
│   ├── 05-architecture.md           # 코드 아키텍처, RPC 계약, DB 스키마, 모듈 분리 설계
│   ├── 06-roadmap.md                # 마일스톤 & 충돌 없는 독립 티켓 작업 보드
│   └── ui-references/               # 빅뱅 UI 레퍼런스 이미지
└── workspaces/
    └── ParallelMaple/               # MSW Maker 로컬 워크스페이스 (그룹 월드: 패러렐메이플)
        ├── Environment/
        ├── Global/
        ├── RootDesk/MyDesk/         # mLua 스크립트 (.mlua + .codeblock)
        ├── map/                     # 사냥터·마을·보스 맵 (.map)
        └── ui/                      # UI 그룹 (.ui)
```

---

## 2. 협업자 3분 세팅 가이드

### 최초 1회 세팅
1. **레포지토리 클론**
   ```bash
   git clone https://github.com/tldjfj123/ParallelMaple.git
   ```
2. **MSW 그룹 합류**
   - 팀장에게 MSW 그룹 초대 수락 + 등급 **`Creator`** (또는 `Manager`) 부여 확인
3. **MSW Maker에서 그룹 월드 연결**
   - MSW Maker 실행 → 그룹 월드 **`패러렐메이플`** 열기
   - 로컬 워크스페이스 경로 팝업이 뜨면 클론한 폴더 내부의 **`workspaces/ParallelMaple`** 지정
   - *(루트 폴더가 아니라 반드시 `workspaces/ParallelMaple`을 지정해야 합니다!)*

### 평소 작업 루틴 (Git + MSW Maker)
1. **작업 시작 전 (Pull & 동기화)**
   ```bash
   git pull origin main
   ```
   - MSW Maker에서 우클릭 → **`Refresh`** *(새 스크립트/맵/UI가 추가됐다면 **`Reimport All`**)*
2. **브랜치 작업 & 테스트**
   - `docs/06-roadmap.md`에서 담당 티켓을 고르고 기능 브랜치(`feat/<ticket-id>`)에서 작업
   - MSW Maker Play Mode에서 에러 로그 0건 확인
3. **커밋 & 푸시**
   - 작업 완료 시 Maker 저장 후 `git add` → `git commit` → `git push` → PR 또는 `main` 병합
   - *(주의: `.map`, `.ui` 파일은 두 명이 동시에 수정하면 Git 충돌 해결이 어렵습니다. `AGENTS.md`의 충돌 방지 규칙을 꼭 확인하세요!)*

---

## 3. 문서 읽는 순서 (AI 바이브코딩 기준)

새 작업을 시작할 때 AI 에이전트에게 아래 순서로 맥락을 읽히면 오해와 충돌 없이 바로 구현할 수 있습니다.

1. **[AGENTS.md](AGENTS.md)** — MSW 필수 제약사항, 서버/클라 보안 규칙, 파일 충돌 방지 규칙
2. **[docs/01-game-design.md](docs/01-game-design.md)** — 게임 방향성과 현재 구현 현황(As-Is / To-Be)
3. **[docs/06-roadmap.md](docs/06-roadmap.md)** — 이번에 맡을 티켓 ID, 수정 대상 파일, 완료 조건 확인
4. **[docs/02-feature-specs.md](docs/02-feature-specs.md)** + **[docs/05-architecture.md](docs/05-architecture.md)** — 해당 기능의 상세 스펙과 RPC/스키마 계약 확인
5. *(필요 시)* **[docs/03-data-reference.md](docs/03-data-reference.md)** (수치/RUID), **[docs/04-ui-spec.md](docs/04-ui-spec.md)** (UI 레이아웃)
