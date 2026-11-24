# REQ-HUNT-02: 18챕터 × 5스테이지(90스테이지) 스케일링 · 3,000마리 · 챕터 보스(60초 타임어택)

- **도메인:** `HUNT` · **유형:** `[FUNC]` · **우선순위:** `P1`
- **선행 티켓:** `REQ-HUNT-01` · **병렬 가능:** `REQ-BOSS-01`, `REQ-SHOP-03`, `REQ-SURV-01`과 병렬 가능

## 1. 목표 (User Story)
18개 사냥터를 `18챕터 × 5스테이지 = 90스테이지` 방치형 돌파·주차 구조로 확장하고, 스테이지당 3,000마리 처치 시 다음 단계 또는 60초 타임어택 챕터 보스에 도전할 수 있게 한다.

## 2. 상세 요구사항 (FR)
1. `WorldDataLogic.HuntingMaps` 순서를 `docs/03-data-reference.md` 4절 기준(`Ch.1` ~ `Ch.18`, Lv.160~180 `크리티아스 → 무기고2 → 지구방위본부 SF140`)으로 정렬하고 각 챕터에 `chapterIndex`, `chapterBossName`, `chapterBossSpriteRuid`, `specialtyTag` 추가.
2. **스테이지 스케일링 (`FieldSpawnerLogic` + `HuntingInstanceLogic`)**:
   - 개인 사냥터 맵 활성화 시 해당 유저의 `currentStage (1~5)` 배율을 몬스터에 적용:
     - HP 배율: `{ 1.00, 1.18, 1.38, 1.62, 1.90 }`
     - EXP/메소 배율: `{ 1.00, 1.15, 1.30, 1.48, 1.65 }`
   - 18개 맵 전체를 한꺼번에 스폰하지 않고 **유저가 현재 체류 중인 1개 챕터 맵만 동적 스폰/리셋**하여 메모리·CPU 대폭 절감!
3. **3,000마리 돌파 & 챕터 보스 타임어택 (60초)**:
   - 현재 최고 해금 스테이지(`currentChapter == maxUnlockedChapter and currentStage == maxUnlockedStage`)에서 몹 처치 시 `profile.stageKillCount += 1` (최대 `3000`).
   - `stageKillCount >= 3000` 도달 시:
     - `Stage 1~4`: `RequestChallengeChapterBoss()` 호출 시 즉시 `maxUnlockedStage += 1`, `stageKillCount = 0`으로 다음 스테이지 승급 & 몹 재스케일링!
     - `Stage 5`: 일반 몹 일시 숨김 → 중앙 발판에 **챕터 보스**(HP = Stage 5 일반 몹 × 35, 크기 1.6배) 소환 + **60초 타이머** 시작.
     - 60초 내 격파 시: 다음 챕터(`maxUnlockedChapter += 1, maxUnlockedStage = 1`) 해금 + 최초 돌파 보상(메소 + 주흔 + 큐브) 지급!
     - 60초 초과 또는 사망 시: 보스 제거 → 일반 몹 복구 + 패널티 없이 현재 `X-5` 스테이지 유지 (`stageKillCount`는 3,000 유지되어 언제든 재도전 가능!).
4. **주차 규칙**: 이미 해금된 모든 `(chapter, stage)`는 `profile.level >= chapter.reqLevel` 충족 시 언제든 자유 이동 가능.

## 3. 예외 / 엣지 케이스
- 챕터 보스 진행 중 맵 이동/귀환 시 보스 타이머와 보스 엔티티를 깨끗이 정리한다.
- 마지막 `Ch.18-5` 클리어 시 `maxUnlockedChapter`가 18을 초과하지 않도록 상한 클램프.

## 4. 수정 대상 파일 (`File Scope`)
- 수정: `workspaces/ParallelMaple/RootDesk/MyDesk/HuntingInstanceLogic.mlua`, `workspaces/ParallelMaple/RootDesk/MyDesk/FieldSpawnerLogic.mlua`, `workspaces/ParallelMaple/RootDesk/MyDesk/WorldDataLogic.mlua`

## 5. Play Mode 완료 조건 (AC)
- [ ] `Ch.1-1` → `Ch.1-5` 스테이지 변경 시 몬스터 HP·EXP·메소가 정확히 단계별로 증가한다.
- [ ] 3,000마리 달성*(QA용 디버그 단축 테스트 포함)* 후 `Stage 5` 챕터 보스 도전 시 60초 타이머와 거대 보스가 등장한다.
- [ ] 챕터 보스 격파 시 다음 챕터가 해금되고 최초 돌파 보상이 1회만 지급된다.
- [ ] 타임어택 실패 시 `stageKillCount`가 초기화되지 않고 일반 사냥으로 안전 복귀한다.
