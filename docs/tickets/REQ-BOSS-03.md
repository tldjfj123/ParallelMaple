# REQ-BOSS-03: 보스 아레나 맵 5종(`Boss01~05.map`) & 패턴 1.2초 바닥 경고 연출

- **도메인:** `BOSS` · **유형:** `[MAP/FX]` · **우선순위:** `P1`
- **선행 티켓:** 없음 *(완전 병렬 가능!)* · **병렬 가능:** 사냥터 맵 작업(`REQ-HUNT-01`)과도 파일이 안 겹쳐 병렬 가능

## 1. 목표 (User Story)
5단계 레이드 보스 전용 아레나 맵 5종을 공식 메이플 보스 맵/배경으로 세팅하고, 보스 패턴의 1.2초 바닥 경고 텔레그래프 및 폭발 이펙트를 시각적으로 명확하게 구현한다.

## 2. 상세 요구사항 (FR)
1. 신규 보스 아레나 맵 5종 생성 (`MapComponent.InstanceMap = true`, 영문 파일명 필수):
   - `map/Boss01_Zakum.map` (자쿰의 제단)
   - `map/Boss02_Horntail.map` (혼테일의 동굴)
   - `map/Boss03_Karuta.map` (심연의 동굴 — 벨룸)
   - `map/Boss04_Lotus.map` (블랙헤븐 코어 — 스우)
   - `map/Boss05_Lucid.map` (몽환의 탑 — 루시드)
   - 각 맵 공통 필수 엔티티: `SpawnLocation` (유저 스폰), `BossSpawnLocation` (보스 스폰), 평탄하고 낙사 없는 발판, 각 보스 공식 BGM.
2. 각 보스 스프라이트 RUID를 `msw-mcp`로 조회·검증하고 `docs/03-data-reference.md`에 기록.
3. 신규 클라이언트 연출 로직 `BossPatternVisualLogic` (`.mlua` + `.codeblock`):
   - `OnShowTelegraph(string patternType, Vector3 pos, Vector2 size, number duration)`: 붉은 반투명 위험 영역이 `duration`(1.2초) 동안 서서히 채워지다 폭발 이펙트 + 경고/타격 SFX 재생 후 자동 제거.

## 3. 예외 / 엣지 케이스
- 보스 맵 양끝 벽 경계(`MapBounds`)를 단단히 막아 밀격 패턴에 맞아도 맵 밖으로 떨어지지 않도록 처리.

## 4. 수정 대상 파일 (`File Scope`)
- 생성: `workspaces/ParallelMaple/map/Boss01_Zakum.map` ~ `Boss05_Lucid.map`, `workspaces/ParallelMaple/RootDesk/MyDesk/BossPatternVisualLogic.mlua` + `.codeblock`
- 수정: `docs/03-data-reference.md` (보스 스프라이트/BGM RUID 기록)

## 5. Play Mode 완료 조건 (AC)
- [ ] 5개 보스 맵 모두 스폰·발판·카메라·양끝 벽이 정상이며 낙사 버그 0건.
- [ ] 패턴 경고 텔레그래프가 1.2초간 또렷하게 보인 뒤 폭발 이펙트와 함께 잔여 엔티티 없이 정리됨.
