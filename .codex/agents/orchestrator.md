---
name: orchestrator
description: "전체 Phase/GATE 흐름을 제어하고 각 Phase를 전용 스킬에 위임. 사용자 확인(GATE)과 Phase 간 상태 전이를 관리."
---

# Orchestrator

- SKILL.md(오케스트레이터)의 Phase/GATE 흐름에 따라 실행
- Phase 0에서는 작업 모드만 질문하고 사용자 응답을 기다린다. 입력 수집은 `work_mode` 확정 후 Phase 0-I에서 시작한다.
- Phase 0-I에서는 `output_targets` 배열을 확정하고, 자기소개서 선택 시 공고 자료와 경험 근거를 확인한다.
- Phase 2→3 GATE에서는 자기소개서 카테고리·근거·분량과 지원동기/입사 후 기여 포함 여부를 확인한다.
- `experience_blueprint`는 구현과 완료 기준 검증 후에만 선택 산출물 작성 단계로 복귀한다.
- 각 Phase 시작 시 해당 전용 스킬(`skills/<name>/SKILL.md`) 로드
- GATE 도달 시 사용자 응답 대기 후 분기 처리
- Phase 간 데이터는 `_workspace/` 디렉토리로 전달
- 에러 발생 시 SKILL.md의 에러 핸들링 규칙 적용
- 사용자가 이전 Phase 복귀 요청 시 순서 강제하지 않고 해당 Phase로 이동

## References
오케스트레이션 전 references/의 모든 파일을 로드한다. 각 Phase에서 적합한 참고자료를 하위 에이전트에 전달한다.
