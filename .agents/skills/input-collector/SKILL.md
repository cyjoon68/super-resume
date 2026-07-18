---
name: input-collector
description: >
  Phase 0. 사용자의 입력을 수집하고 작업 모드(resume_based / experience_blueprint),
  산출물(output_target), 레퍼런스, 지원 조건, 말투 등을 수집한다.
  시나리오 분류 및 _workspace/01_scenario.json 생성을 담당한다.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - webfetch
  - request_user_input
---

# Input Collector — 컨텍스트 확인 + 입력 수집

본격적인 분석 전, 사용자의 입력을 수집하고 작업 모드와 시나리오를 분류한다.
선택 질문은 가능한 경우 Codex의 실제 질문 UI(`request_user_input`)로 표시한다.
`request_user_input`을 사용할 수 없는 환경에서만 동일한 문구를 일반 텍스트 질문으로 대체한다.

## 컨텍스트 확인

```bash
_WORKSPACE_DIR="_workspace"
_SKILLS_DIR="skills"
_HAS_PREVIOUS=false
[ -d "$_WORKSPACE_DIR" ] && _HAS_PREVIOUS=true
echo "HAS_PREVIOUS: $_HAS_PREVIOUS"
mkdir -p "$_WORKSPACE_DIR"
```

- `_workspace/` 존재 + "다시 실행", "재실행" → **부분 재실행** (마지막 Phase부터 재개)
- `_workspace/` 존재 + 새 입력 → **새 실행** (기존 _workspace를 `_workspace_prev/`로 이동)
- `_workspace/` 미존재 → **초기 실행**

---

## Phase 0-A: 입력 확인 + 작업 모드 선택

먼저 사용자가 제공한 입력을 짧게 확인하고, 같은 질문에서 작업 모드를 선택하게 한다.

표시 형식:

```text
입력 확인:
이력서: {resume_path_or_summary}
포트폴리오: {portfolio_path_or_summary_or_없음}
GitHub: {github_url_or_없음}
공고: {job_url_or_target}

모드 선택
```

Codex 질문 UI를 사용할 수 있으면 다음 구조로 질문한다:

```text
질문: 어떤 방식으로 진행할까요?
선택지:
1. 현재 이력서 기준으로 공고에 맞추기
2. 이력서에 넣을 프로젝트/문제해결 경험 생성까지 같이 하기
```

- 1번 또는 "이력서 기준" → `work_mode: "resume_based"`
- 2번, "프로젝트 경험 생성", "문제해결 경험 생성" → `work_mode: "experience_blueprint"`
- 모드 선택 질문은 생략하지 않는다. 사용자가 이미 요청 방향을 말했어도 반드시 이 질문으로 `work_mode`를 확정한다.
- 사용자 응답 전에는 `resume_based` 또는 `experience_blueprint`를 기본값으로 확정하지 않는다.
- 이력서 없이 공고 URL/목표 직군만 제공하거나 프로젝트 기획을 요청한 경우에도, `experience_blueprint`를 추천할 수는 있지만 사용자 선택을 받아야 한다.

---

## Phase 0-Reference Gate: 레퍼런스 확인

이 체크는 `SKILL.md`가 있는 스킬 폴더 기준으로 실행한다.

```bash
_HAS_REFERENCES=false
_SKILL_REFERENCES_DIR="references"
[ -d "$_SKILL_REFERENCES_DIR" ] && [ "$(find "$_SKILL_REFERENCES_DIR" -type f | head -1)" ] && _HAS_REFERENCES=true
echo "HAS_REFERENCES: $_HAS_REFERENCES"
```

- `_HAS_REFERENCES=false`이면 사용자에게 알린다:
  - "스킬 폴더 안의 `references/`가 비어 있습니다. 이력서 작성 레퍼런스를 입력해 주시면 스킬 안의 `references/`에 저장하고 진행할 수 있고, 넣지 않고 바로 진행할 수도 있습니다."
- 사용자가 레퍼런스를 넣겠다고 하면 입력을 받아 `references/user-notes.md` 또는 사용자가 지정한 파일명으로 저장한다.
- 사용자가 바로 진행하겠다고 하거나 명시하지 않으면, 레퍼런스 없이 기본 워크플로우로 진행한다.

---

## Phase 0-B: 생성할 산출물 선택

작업 모드가 정해진 뒤 별도 질문으로 생성할 산출물을 선택하게 한다.

Codex 질문 UI를 사용할 수 있으면 다음 구조로 질문한다:

```text
질문: 무엇을 만들까요?
선택지:
1. 이력서
2. 포트폴리오
3. 이력서 + 포트폴리오
```

- 선택값은 `_workspace/01_scenario.json`의 `output_target`에 저장한다.
- `1. 이력서` → `output_target: "resume"`
- `2. 포트폴리오` → `output_target: "portfolio"`
- `3. 이력서 + 포트폴리오` → `output_target: "both"`
- 이력서만 제공돼도 이력서, 포트폴리오, 이력서+포트폴리오 생성 가능
- 포트폴리오만 제공돼도 이력서, 포트폴리오, 이력서+포트폴리오 생성 가능
- GitHub만 제공돼도 가능한 범위에서 동일하게 생성 가능

---

## 추가 입력 수집

1. **이력서, 포트폴리오, 이력서 URL, 포트폴리오 URL, 또는 GitHub 링크** (`resume_based`에서는 하나 이상 필수, `experience_blueprint`에서는 선택):
   - 이력서: 파일 경로(.md, .pdf) 또는 텍스트 붙여넣기
   - 포트폴리오: 파일 경로(.md, .pdf), 텍스트 붙여넣기, Notion/Figma/웹 URL
   - 이력서 URL: Google Docs, Notion, 개인 웹사이트 등에 호스팅된 이력서 링크 (webfetch로 자동 fetching)
   - GitHub 링크: GitHub 프로필 URL 또는 레포 URL (이력서 대신 GitHub 경험으로 시작)
   - 모두 제공 가능. GitHub 링크는 이력서/포트폴리오를 보완하는 추가 경험 소스로도 사용됨
2. **프로필 이미지** (선택): 이력서에 넣을 프로필 사진 파일 경로 또는 URL
   - "프로필 사진을 추가하시겠어요?"라고 질문
   - 없다면 건너뜀
3. **지원 조건** (선택): 병역, 산업기능요원, 보충역, 근무 가능 시점 등 지원 판단에 필요한 조건
   - 사용자가 제공하면 `_workspace/01_scenario.json`의 `candidate_notes`에 저장한다.
   - 예: `candidate_notes.military_service: "산업기능요원 보충역"`
   - 최종 이력서/포트폴리오에서 이 정보를 누락하지 않는다.
4. **공고 URL 또는 목표 직군** (`experience_blueprint`에서는 둘 중 하나 필수): 하나 이상의 채용 공고 링크 또는 목표 직군/회사/도메인
5. **요청 사항** (선택):
   - "공고에 맞춰서 수정해줘" (시나리오 A)
   - "피드백만 해줘" (시나리오 B) → 이력서/깃허브 있을 때 자동 제안
   - "디자인만 바꿔줘" (시나리오 C)
   - "적합도만 확인해줘" (시나리오 E) → 수정 없이 fit 점수만
   - "프로젝트 경험 생성해줘" / "문제해결 경험 만들어줘" (`experience_blueprint`)
   - (이전 실행의 후속) "여기서 수정해줘" (시나리오 D)
6. **말투** (선택, 기본값: 말투유지):
   - "어떤 말투로 작성할까요?"
   - 1: **전문적인** — 격식 있고 정확한 표현, 표준 업계 용어
   - 2: **자신감 있는** — 강한 액션 동사, 성과 중심, 주도적 표현
   - 3: **간결한** — 핵심만 간략하게, bullet point 중심
   - 4: **기술 중심** — 기술적 세부사항 강조, 아키텍처/성능 수치 중심
   - 5: **협업 중심** — 팀워크, 리더십, 크로스펑셔널 협업 강조
   - 6: **말투유지** — 이력서 원본의 말투를 분석해서 그대로 유지 (기본값). GitHub만 제공된 경우에는 professional로 폴백
   - 사용자가 선택하지 않으면 `"preserve"` (말투유지)를 기본값으로 사용

> **이력서만 제공된 경우:** 먼저 `output_target`을 확인하라.
> 이력서만 있어도 이력서, 포트폴리오, 이력서+포트폴리오를 만들 수 있다.
> 사용자가 산출물을 명시하지 않았으면 "무엇을 만들까요?"를 먼저 묻고,
> 사용자가 피드백만 원한다고 답한 경우에만 Phase 2-B(피드백)로 진행한다.

> **GitHub 링크만 제공된 경우 (이력서 없음):**
> "이력서 없이 GitHub 링크만으로도 진행할 수 있습니다.
> GitHub 저장소를 분석해서 경험을 추출한 뒤, 원하시는 방향으로 진행할게요."
> → Phase 1-B(GitHub 기반 경험 수집)로 진행.

> **이력서 + 공고 URL이 모두 제공된 경우:** Fit Score(적합도)를 먼저 보여주고
> "이 공고에 맞춰서 이력서를 수정할까요?"라고 질문하라.
> 사용자가 Yes → Phase 2-A(맞춤수정)로 진행. No → 종료.