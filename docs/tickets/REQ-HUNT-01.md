# REQ-HUNT-01: 사냥터 18개 `InstanceMap` 설정 & 1인 개인 사냥터 `RoomService` 전이

- **도메인:** `HUNT` · **유형:** `[MAP/FX]` · **우선순위:** `P0`
- **선행 티켓:** `REQ-ARCH-03` · **병렬 가능:** `REQ-BOSS-03`과 병렬 가능 *(단, 사냥터 `.map` 18개 단독 작업 필수!)*

## 1. 목표 (User Story)
헤네시스 마을(`TileMapTemplate`)만 공용 Static Room으로 유지하고, 사냥터 18개(`HenesysHunt1`, `Hunt01~17`)는 유저별 **1인 개인 사냥터 인스턴스 룸(`HuntRoom_<userId>`)**으로 전환하여 스틸 0%와 서버 리소스 최적화를 달성한다.

## 2. 상세 요구사항 (FR)
1. **맵 속성 변경 (단독 작업)**:
   - `HenesysHunt1.map` 및 `Hunt01_StrayDog.map` ~ `Hunt17_Excavation4.map` 총 18개 맵의 `MapComponent.InstanceMap = true` 설정.
   - `TileMapTemplate.map`(헤네시스 마을)은 `InstanceMap = false` (Static Room) 유지.
2. **신규 `HuntingInstanceLogic` (`.mlua` + `.codeblock`) 방 전이 구현**:
   - `RequestEnterHuntingStage(integer chapter, integer stage)`:
     - 현재 유저가 **Static Room(마을)**에 있다면:
       1. `profile.currentChapter = chapter`, `profile.currentStage = stage`, `profile.lastHuntingMapName = mapName` 기록
       2. **`SaveProfileAsync(userId, callback)` 완료 후** `_RoomService:GetOrCreateInstanceRoom("HuntRoom_" .. userId, all18HuntMapNames)` → `_RoomService:MoveUsersToInstanceRoom(roomKey, { userId })` 실행!
     - 이미 **내 개인 사냥터 룸** 안에 있다면: 방 재이동 없이 즉시 `_TeleportService`로 대상 챕터 맵 이동 + 스테이지 갱신!
   - `RequestReturnTown()`:
     - 현재 개인 사냥터 룸에 있다면 `SaveProfileAsync(userId, callback)` 완료 후 `_RoomService:MoveUsersToStaticRoom({ userId })`로 헤네시스 귀환.
3. **인스턴스 룸 진입 시 자동 워프**:
   - `HandleUserEnterEvent`가 개인 사냥터 룸에서 발동하면 `LoadProfileForUser` 완료 직후 저장된 `profile.lastHuntingMapName`으로 자동 텔레포트 및 해당 챕터/스테이지 몹 스폰.

## 3. 예외 / 엣지 케이스
- **Static ↔ Instance 데이터 유실 절대 금지**: 반드시 `SaveProfileAsync` 콜백 성공 이후에만 `MoveUsersToInstanceRoom` / `MoveUsersToStaticRoom`을 호출한다.
- 방 이동 중 연타 방지를 위해 `TransitioningUsers[userId]` 잠금 플래그(3초 타임아웃) 적용.

## 4. 수정 대상 파일 (`File Scope`)
- 생성: `workspaces/ParallelMaple/RootDesk/MyDesk/HuntingInstanceLogic.mlua` + `.codeblock`
- 수정: `workspaces/ParallelMaple/map/HenesysHunt1.map`, `workspaces/ParallelMaple/map/Hunt01~17.map` (`InstanceMap = true`), `workspaces/ParallelMaple/RootDesk/MyDesk/PlayerDataLogic.mlua` (기존 `RequestTeleportMap`을 `HuntingInstanceLogic`에 위임)

## 5. Play Mode 완료 조건 (AC)
- [ ] 멀티플레이 테스트(2클라)에서 두 유저가 헤네시스 마을에서는 서로 보이고, 같은 `Ch.1` 사냥터에 들어가면 서로 격리된 1인 공간에서 사냥한다.
- [ ] 사냥터 룸 안에서 `Ch.1 → Ch.2` 이동 시 마을을 거치지 않고 즉시 이동한다.
- [ ] 사냥터 ↔ 마을 왕복 시 레벨·경험치·메소·인벤토리가 100% 유실 없이 유지된다.
