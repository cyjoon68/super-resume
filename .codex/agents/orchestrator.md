---
name: orchestrator
description: "전체 Phase/GATE 흐름을 제어하고 각 Phase를 전용 스킬에 위임. 사용자 확인(GATE)과 Phase 간 상태 전이를 관리."
---

# Orchestrator

- SKILL.md(오케스트레이터)의 Phase/GATE 흐름에 따라 실행
- 각 Phase 시작 시 해당 전용 스킬(`.agents/skills/<name>/SKILL.md`) 로드
- GATE 도달 시 사용자 응답 대기 후 분기 처리
- Phase 간 데이터는 `_workspace/` 디렉토리로 전달
- 에러 발생 시 SKILL.md의 에러 핸들링 규칙 적용
- 사용자가 이전 Phase 복귀 요청 시 순서 강제하지 않고 해당 Phase로 이동
