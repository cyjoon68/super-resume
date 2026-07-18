---
name: super-resume
description: >
  Super Resume. 공고 기준으로 이력서/포트폴리오를 맞춤 수정하거나,
  이력서에 반영 가능한 문제해결 프로젝트 기획서를 생성하는 워크플로우 오케스트레이터.
  "super-resume", "Super Resume", "슈퍼 이력서", "resume-creator", "이력서 만들어줘",
  "이력서 수정해줘", "공고에 맞춰줘", "프로젝트 경험 생성해줘",
  "문제해결 경험 만들어줘" 요청 시 반드시 이 스킬을 사용할 것.
  "다시 실행", "수정", "업데이트", "재실행" 등 후속 요청에도 사용.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - webfetch
  - task
  - skill
  - request_user_input
---

# Super Resume — 경험 설계 + 공고 맞춤 이력서 오케스트레이터

공고 기준으로 현재 이력서/포트폴리오를 최적화하거나,
이력서에 반영 가능한 문제해결 프로젝트 기획서를 생성하는 전체 워크플로우를 조율한다.

---

## Reference Files

Reference files stay in this skill's local `references/` directory and are ignored by git.
If the directory is missing or empty, tell the user:
"스킬 폴더 안의 `references/`가 비어 있습니다. 이력서 작성 레퍼런스를 입력해 주시면 스킬 안의 `references/`에 저장하고 진행할 수 있고, 넣지 않고 바로 진행할 수도 있습니다."

Open only the relevant reference files from this skill's `references/` directory.
Use references during analysis, strategy, drafting, experience blueprint generation, and quality review when useful. These references do not replace the required Phase/GATE flow below.
If a needed reference file is missing, continue without it unless the user wants to add it first.

---

## Phase 0: 컨텍스트 확인

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

### 사용자 입력 수집

Phase 0은 반드시 두 단계로 나눈다.
선택 질문은 가능한 경우 Codex의 실제 질문 UI(`request_user_input`)로 표시한다.
일반 텍스트 목록과 수동 응답 요청만으로 끝내지 않는다.
`request_user_input`을 사용할 수 없는 환경에서만 동일한 문구를 일반 텍스트 질문으로 대체한다.

#### Phase 0-A: 입력 확인 + 작업 모드 선택

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

#### Phase 0-Reference Gate: 레퍼런스 확인

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

#### Phase 0-B: 생성할 산출물 선택

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

#### 추가 입력 수집

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

---

### GATE: Phase 0 → Phase 1 (분석 시작 확인)

**시점:** 사용자 입력 수집 완료 후, 분석 시작 전

**질문:** "입력을 확인했습니다. 분석을 시작할까요?"

**대기:** 사용자 응답 필수

**분기:**
- **Yes / 네 / 시작** → Phase 1(입력 분석) 진행
- **아니요 / 잠시만요** → "어떤 부분을 수정할까요?" 추가 질문 후 다시 GATE
- **종료 / 취소** → 워크플로우 종료

---

## Phase 1: 입력 분석

### Step 1-1: 이력서 파싱 (이력서가 제공된 경우)

1. 내부 모듈 `skills/resume-parser/SKILL.md`를 참조하여 이력서 파싱 절차 수행
2. 제공된 이력서 파일/텍스트를 파싱하여 구조화된 데이터로 변환
3. 입력이 PDF이면 `opendataloader-pdf`가 사용 가능한지 먼저 확인하고, 있으면 이를 우선 사용해 텍스트/구조/링크를 추출한다. 없거나 실패하면 `skills/resume-parser/SKILL.md`의 PDF fallback 절차를 따른다.
4. 입력이 PDF이면 텍스트 추출과 별도로 hyperlink annotation/link target을 추출하여 `_workspace/01_extracted_links.json`에 저장
5. 이력서 텍스트 또는 PDF annotation에서 GitHub 프로필 URL과 레포 URL을 추출하여 `personal_info.github`, `personal_info.links`, `projects[].url`에 저장
6. 이력서에 `GitHub`, `Github`, `깃허브`, `Git Hub` 같은 항목은 있으나 텍스트와 PDF annotation 어디에도 실제 URL이 없으면 `missing_inputs.github_url: true`와 `personal_info.github_label_present: true`로 기록
7. 프로필 이미지가 제공된 경우 `personal_info.profile_image`에 경로 저장
8. **말투 감지:** tone이 `"preserve"`(말투유지)인 경우, content-craft에 전달할 말투 분석을 위해 원본 이력서의 말투 특징을 `_workspace/01_tone_profile.json`에 저장
9. 출력: `_workspace/01_parsed_resume.json`, `_workspace/01_original_resume.md`

### Step 1-B: GitHub 기반 경험 수집 (이력서 없음, GitHub 링크만 — 시나리오 F)

> 이력서 없이 GitHub 링크만 제공된 경우, 이 Step으로 건너뛴다.

1. 내부 모듈 `skills/github-explorer/SKILL.md`를 참조하여 GitHub 경험 수집 절차 수행
2. 사용자가 제공한 GitHub 링크를 기반으로 저장소 탐색:
   - 모든 public 레포 목록 수집
   - 각 레포의 README, 기술 스택, 커밋 히스토리 분석
   - 주요 프로젝트 식별 (스타 수, 최근 활동 기준)
3. GitHub 데이터를 이력서 구조로 재구성:
   - `personal_info`: GitHub 프로필 정보 (이름, 링크, profile_image)
   - `summary`: (비어있음 — Phase 2에서 생성)
   - `skills`: 레포 기술 스택 집계
   - `projects`: 각 레포를 프로젝트로 변환 (STAR 초안 포함)
   - `experience`: (비어있음 — 사용자에게 요청 가능)
   - `education`: (비어있음)
4. 사용자에게 알림:
   > "GitHub에서 **N**개의 저장소를 찾았고, 그중 **M**개를 주요 프로젝트로 선별했습니다.
   > 이력서에 없는 정보(학력, 비공개 경험 등)는 추후 추가할 수 있습니다."
5. 출력: `_workspace/01_parsed_resume.json` (GitHub 기반), `_workspace/01_original_resume.md` (비어있거나 GitHub 요약)


### Step 1-2A: GitHub 링크 누락 확인 (GitHub 항목만 있는 경우)

> 이력서에 GitHub/Github/깃허브 항목은 있으나 텍스트와 PDF hyperlink annotation 어디에도 실제 URL이 없으면 Step 1-2를 조용히 건너뛰지 말고 사용자 확인을 거친다.

1. `_workspace/01_parsed_resume.json`의 `missing_inputs.github_url` 또는 `personal_info.github_label_present`를 확인. 단, PDF 입력이면 먼저 `_workspace/01_extracted_links.json`에 GitHub URL이 없는지 확인한다.
2. 값이 `true`이고 실제 GitHub URL이 없으면 사용자에게 질문:
   > "이력서에 GitHub 항목이 있지만 URL이 없습니다. GitHub 프로필이나 레포 링크를 제공하시겠어요?"
3. 사용자가 GitHub URL 제공 → `personal_info.github` 또는 `personal_info.links`에 저장하고 Step 1-2 실행
4. 사용자가 "없음 / 건너뛰기" 선택 → `_workspace/01_scenario.json`에 `github_skipped_reason: "github_label_without_url_user_skipped"` 저장 후 Step 1-2 건너뜀
5. 이 게이트의 목적은 GitHub 항목이 보이는 이력서를 silent skip하지 않는 것이다.

### Step 1-2: GitHub 저장소 탐색 (이력서에 GitHub 링크가 있을 경우)

> 이력서에 GitHub 링크가 있으면 **공고 URL 분석(Step 1-3)과 병렬로 실행**한다.

1. `_workspace/01_parsed_resume.json`에서 `personal_info.github`와 `personal_info.links`에서 GitHub URL 추출
2. 내부 모듈 `skills/github-explorer/SKILL.md`를 참조하여 GitHub 저장소 탐색 절차 수행
3. 이력서의 GitHub URL을 기반으로 저장소 탐색:
   - GitHub 프로필 URL → 사용자의 public 레포 목록 조회
   - 개별 레포 URL → 해당 레포 상세 분석
   - 공고 분석 결과(`job_analyses.json`)가 아직 없으면 기술 스택/도메인 기준으로만 필터링
   - 공고 분석 결과가 있으면 공고 요구사항과 매칭되는 레포 우선 선별
4. 출력: `_workspace/01_github_findings.json`
   - 매칭된 레포 목록 (각각 STAR 초안 포함)
   - 새로운 발견 사항 (이력서에 없는 프로젝트)
   - 중복 레포 표시 (이미 이력서에 있는 프로젝트)

### Step 1-3: 공고 URL 분석 (있을 경우)

1. 내부 모듈 `skills/job-analyzer/SKILL.md`를 참조하여 공고 분석 절차 수행
2. 모든 공고 URL을 병렬로 fetching하여 분석
3. 출력: `_workspace/01_job_analyses.json`, `_workspace/01_job_raw/`

### Step 1-4: 시나리오 분류

입력에 따라 작업 모드와 시나리오를 분류하고 `_workspace/01_scenario.json`에 저장:

```json
{
  "scenario": "A | B | C | D | E | F",
  "work_mode": "resume_based | experience_blueprint",
  "output_target": "resume | portfolio | both",
  "description": "설명",
  "has_resume": true|false,
  "has_portfolio": true|false,
  "has_github": true|false,
  "has_github_link_missing": true|false,
  "has_job_urls": true|false,
  "has_profile_image": true|false,
  "candidate_notes": {
    "military_service": "산업기능요원 보충역 | null",
    "availability": "string | null"
  },
  "design_request": false,
  "is_iteration": false,
  "tone": "professional | confident | concise | technical | collaborative | preserve",
  "missing_inputs": {
    "github_url": true|false
  }
}
```

**분류 기준:**
- **A (맞춤수정)**: 이력서/깃허브 + 공고 URL 1개 이상, 사용자가 수정에 동의
- **B (피드백)**: 이력서 or 깃허브만 있음, 사용자가 피드백 요청
- **C (디자인변경)**: 디자인/템플릿 변경 명시적 요청 (공고 URL 유무 무관)
- **D (반복수정)**: 이전 실행 결과 기반 후속 수정 요청
- **E (적합도 확인)**: 이력서/깃허브 + 공고 URL, 사용자가 "적합도만 확인" 요청 또는
  수정 전 기본으로 fit score 우선 표시
- **F (깃허브 기반 신규 작성)**: GitHub 링크만 제공, 이력서 없음. GitHub 저장소를 분석하여 이력서를 처음부터 작성

**작업 모드 기준:**
- **resume_based**: 현재 이력서/GitHub 기준으로 공고 맞춤 수정, 피드백, 디자인, PDF 출력을 진행한다. 기존 시나리오 A-F는 이 모드에서 유지된다.
- **experience_blueprint**: 공고 또는 목표 직군 기준으로 이력서/포트폴리오에 반영 가능한 문제해결 프로젝트 기획서를 생성한다. 현재 이력서/GitHub가 있으면 약점과 기술 스택을 참고하지만, 이력서가 없어도 진행할 수 있다.

**산출물 기준:**
- **resume**: 이력서만 생성/수정한다.
- **portfolio**: 포트폴리오만 생성/수정한다.
- **both**: 이력서와 포트폴리오를 함께 생성/수정한다.
- 입력 자료와 산출물은 독립적으로 판단한다. 이력서만 있어도 포트폴리오를 만들 수 있고, 포트폴리오만 있어도 이력서를 만들 수 있다.

---

### 예외 처리: URL에서 읽을 수 없는 섹션이 있는 경우

Phase 1 완료 후 `01_scenario.json`에 `fetch_warnings`가 있으면 (플랫폼 제한으로 일부 섹션을 읽지 못한 경우):

1. 사용자에게 추출 결과 요약을 보여줘라:
   ```
   📄 [플랫폼명] — [이름] 프로필
     ✅ 기본 정보, 기술 스택, 교육
     ⛔ 경력 (공개 제한)
     ⛔ 프로젝트 (공개 제한)

   🔔 경력과 프로젝트를 읽을 수 없습니다.
   PDF 파일을 업로드하거나 텍스트를 붙여넣어 주세요.
   ```
2. "이력서 PDF나 텍스트를 추가로 제공해주시겠어요?" 질문
   - **Yes / 네** → 추가 입력 받은 후 `01_parsed_resume.json` 업데이트
   - **아니요 / 지금대로 진행** → 읽은 내용만으로 진행 (누락 섹션은 빈 상태로 표시)

---

### GATE: Phase 1 → Phase 2 (전략 수립 확인)

**시점:** 이력서 파싱 + 공고 분석 + GitHub 탐색 완료 후

**질문 (시나리오별):**
- **work_mode = experience_blueprint:** 공고 분석 또는 목표 직군 분석 결과를 요약한 후 "이 기준으로 문제해결 프로젝트 기획서를 생성할까요?"
- **시나리오 A/E:** Fit Score를 계산하여 표시한 후 "이 공고에 맞춰서 이력서를 수정할까요?"
- **시나리오 B:** "분석이 완료되었습니다. 종합 피드백을 제공할까요?"
- **시나리오 C:** (디자인 변경 — 전략 생략, 바로 디자인 GATE로)
- **시나리오 F:** "GitHub 경험을 바탕으로 기본 이력서를 작성했습니다. 이제 어떻게 진행할까요?"

**대기:** 사용자 응답 필수

**분기:**
- **work_mode = experience_blueprint + Yes / 네 / 생성해줘** → Phase 2-X(Experience Blueprint) 진행
- **Yes / 네 / 수정해줘** → Phase 2(컨텐츠 전략) 진행
- **아니요 / 피드백만** → Phase 2-B(피드백)로 진행
- **종료** → Fit Score 리포트만 출력하고 종료

---

## Phase 2: 컨텐츠 전략

### Phase 2-X: Experience Blueprint — 문제해결 프로젝트 기획서 생성

> `work_mode: "experience_blueprint"`일 때 실행한다. 기존 이력서 맞춤 수정 Phase를 건너뛰고, 공고 적합도를 높일 수 있는 프로젝트 기획서를 생성한다.

1. 사용자가 제공한 관련 `references/` 파일이 있으면 열고 원칙을 적용한다.
2. 공고 URL이 있으면 `skills/job-analyzer/SKILL.md`로 공고의 필수/우대 스택, 서비스 도메인, 업무 맥락, 평가 키워드를 추출한다.
3. 현재 이력서/GitHub가 있으면 약점과 이미 보유한 경험을 참고한다. 없으면 공고/목표 직군 기준으로만 생성한다.
4. 프로젝트는 반드시 공고의 필수/우대 스택을 우선 사용한다. 공고에 없는 기술을 주력 스택으로 선택하지 않는다.
5. 각 프로젝트는 `서비스 맥락에서 발생하는 실제 문제 상황`과 `해결 방향`을 먼저 정의하고, 그 뒤 구현 명세를 만든다.
6. 기본으로 **서로 다른 4개**의 프로젝트 기획서를 생성한다. 공고가 여러 개면 공통 스택 기반 2개와 공고별 특화 2개를 우선 구성한다.
7. 기획서는 아이디어가 아니라 바로 구현 가능한 생성 명세여야 한다. 데이터 모델, API, 화면, 이벤트, 측정 지표, 테스트, 배포, 구현 지시서를 포함한다.
8. 완료 전 프로젝트를 완료 경험처럼 쓰지 않는다. 이력서 문장은 항상 "완성 후 사용할 수 있는 bullet"로 분리한다.
9. 기획서 생성 후에는 반드시 사용자 승인을 받아야 한다. 승인 전에는 프로젝트 구현, 레포 생성, 코드 작성, 파일 scaffold를 진행하지 않는다.
10. 출력:
   - `_workspace/06_experience_blueprints/01_{project_slug}.md`
   - `_workspace/06_experience_blueprints/02_{project_slug}.md`
   - `_workspace/06_experience_blueprints/03_{project_slug}.md`
   - `_workspace/06_experience_blueprints/04_{project_slug}.md`
   - `_workspace/06_experience_blueprints.json`
11. 4개 기획서를 하나의 md 파일에 합쳐서 끝내지 않는다. 프로젝트별 md 파일 4개가 실제로 생성되어야 한다.
12. 각 기획서는 고유한 `프로젝트명`과 파일명용 `slug`를 가져야 한다. `프로젝트명`은 문서 첫 H1으로 쓴다.

파일명 규칙:

```text
_workspace/06_experience_blueprints/
  01_onboarding_routine_ab_test.md
  02_workout_session_dropoff_dashboard.md
  03_community_activation_experiment.md
  04_ai_recommendation_feedback_loop.md
_workspace/06_experience_blueprints.json
```

`06_experience_blueprints.json`은 반드시 각 기획서의 이름과 파일 경로를 가진다:

```json
{
  "blueprints": [
    {
      "id": "BP-01",
      "name": "운동 초보자 온보딩 루틴 추천 실험 플랫폼",
      "slug": "onboarding_routine_ab_test",
      "file_path": "_workspace/06_experience_blueprints/01_onboarding_routine_ab_test.md"
    }
  ]
}
```

각 프로젝트 기획서 필수 항목:
- 프로젝트명
- slug
- 공고 매칭 키워드
- 공고 기술 스택 매핑
- 도메인 선택 이유
- 서비스 맥락에서 발생하는 실제 문제 상황
- 문제의 사용자/운영/비즈니스 영향
- 해결 방향
- 기술적 해결 방법
- MVP 범위
- 기술 스택
- 아키텍처/데이터 플로우
- 데이터 모델/엔티티
- API 명세
- 화면/사용자 플로우
- 상태 관리/캐싱/동기화 전략
- 일부러 만들 병목/제약
- 일부러 만들 병목/제약 해결 방법
- 측정 지표
- 이벤트 로깅 설계
- 테스트/부하테스트/관측 방법
- 배포 방식
- README 구조
- 포트폴리오 섹션 초안
- 완성 후 이력서 bullet 3-5개
- 구현 작업 분해
- 완료 기준
- 구현 에이전트용 작업 지시서

#### Job Stack Rule

문제해결 프로젝트는 해당 공고의 스택을 우선 사용한다.

- 공고의 필수 기술은 프로젝트 핵심 구현에 반드시 포함한다.
- 공고의 우대 기술은 차별점, 확장 기능, 성능 개선 포인트로 포함한다.
- 여러 공고가 주어지면 공통 스택을 기본으로 잡고, 공고별 특화 프로젝트를 따로 제안한다.
- 사용자의 선호 스택보다 공고 스택을 우선한다. 단, 사용자가 명시적으로 제한하면 그 제한 안에서 설계한다.
- 대체 기술이 필요하면 이유를 명시하고, 공고 스택과의 연결을 설명한다.

출력에는 반드시 아래 표를 포함한다:

```text
공고 기술 스택 매핑

| 공고 요구사항 | 프로젝트에서 쓰는 위치 | 증명 방식 |
|---|---|---|
| React | 상품 비교 피드/상세 화면 구현 | 화면 코드, 상태 관리, 성능 측정 |
| GraphQL | 상품/가격/히스토리 조회 API | schema, query, resolver |
| React Query | 가격 비교 데이터 캐싱 | staleTime, invalidation 전략 |
| Playwright | 구매 funnel E2E 테스트 | 테스트 코드와 CI 결과 |
```

#### Blueprint Completeness Rule

기획서는 다른 개발 에이전트가 추가 질문 없이 레포를 만들고 개발을 시작할 수 있을 정도로 구체화한다.

- DB가 있으면 schema를 명시한다.
- API가 있으면 endpoint, method, payload를 명시한다.
- 프론트가 있으면 화면 목록, 주요 상태, 사용자 플로우를 명시한다.
- 측정이 필요하면 event name과 event payload를 명시한다.
- 성능 개선 프로젝트면 before/after 측정 방법을 명시한다.
- 포트폴리오에 넣을 diagram 종류를 명시한다.
- 이력서 bullet은 완료 후 검증 가능한 수치 기준으로 작성한다.

#### Domain Problem Rule

프로젝트 기획서는 단순 기능 목록이 아니라 공고의 서비스 도메인에서 실제로 발생할 법한 문제 상황을 먼저 만든다. 이 항목 이름은 `서비스 맥락에서 발생하는 실제 문제 상황`으로 쓴다.

예시:

```text
서비스 맥락에서 발생하는 실제 문제 상황:
모바일 커머스에서 사용자는 상품 목록의 표시 가격만 보고 저렴하다고 판단하지만,
결제 단계에서 배송비, 쿠폰 조건, 옵션 추가금이 반영되며 최종 결제가가 달라진다.
이 차이가 반복되면 사용자는 추천 상품의 가격을 신뢰하지 못하고,
상품 상세 진입 후 바로 이탈하거나 결제 직전 단계에서 구매를 포기한다.

해결 방향:
상품 후보를 `상품가`, `배송비`, `쿠폰 적용가`, `옵션 추가금`, `최종 결제가` 기준으로 정규화한다.
추천 카드에는 최종 결제가와 가격 산정 근거를 함께 노출하고,
가격 변동이 큰 상품에는 최근 가격 변동 라벨을 표시한다.
전환 funnel을 수집해 비교 카드 노출 이후 상품 상세 진입률, 결제 진입률, 결제 직전 이탈률을 측정한다.
```

금지 표현:
- 사용자의 역량을 낮춰 보이게 하는 자기비하 표현
- 실제 완료하지 않은 일을 완료 경험처럼 보이게 하는 표현
- 프로젝트 기획을 허위 경험처럼 보이게 하는 표현
- 도메인 문제를 막연하거나 꾸민 이야기처럼 보이게 하는 표현

대체 표현:
- `서비스 맥락에서 발생하는 실제 문제 상황`
- `공고 적합도를 높일 프로젝트`
- `이력서에 반영 가능한 문제해결 경험`
- `포트폴리오로 확장 가능한 프로젝트`
- `완성 후 사용할 수 있는 bullet`

### GATE: Phase 2-X → 결과 제공

**질문:** "문제해결 프로젝트 기획서 4개가 각각 생성되었습니다. 어떤 프로젝트를 구현할까요?"

**대기:** 사용자 응답 필수

**분기:**
- 사용자가 프로젝트 선택 + 명시 승인 → 선택한 기획서의 `MVP 범위`, `기술 스택`, `아키텍처/데이터 플로우`, `데이터 모델/엔티티`, `API 명세`, `화면/사용자 플로우`, `구현 작업 분해`, `완료 기준`, `구현 에이전트용 작업 지시서`를 기준으로 구현한다.
- 사용자가 선택만 하고 승인이 모호함 → "선택한 프로젝트 구현을 시작해도 될까요?"라고 다시 묻는다.
- 추가 수정 요청 → 해당 프로젝트 기획서 수정 후 다시 GATE
- 종료 → 프로젝트별 md 파일 4개와 `_workspace/06_experience_blueprints.json`만 제공

### GATE: 프로젝트 구현 완료 → 병목/제약 해결 코드 추가

**질문:** "기획서의 `일부러 만들 병목/제약 해결 방법`대로 해결 코드를 추가할까요?"

**대기:** 사용자 응답 필수

**분기:**
- 승인 → 기획서의 `일부러 만들 병목/제약 해결 방법`, `기술적 해결 방법`, `테스트/부하테스트/관측 방법`, `측정 지표`를 기준으로 해결 코드를 추가한다.
- 수정 요청 → 어떤 병목/제약을 다르게 해결할지 확인한 뒤 반영한다.
- 거절/보류 → 병목/제약은 남겨두고 현재 구현 상태와 남은 TODO를 기록한다.

### 공통: Fit Score 표시 (이력서 + 공고 URL이 있을 때)

시나리오 A/E 모두에서, Phase 2 진입 시 **반드시 Fit Score를 먼저 계산하여 사용자에게 보여줘라.**

1. 내부 모듈 `skills/content-strategy/SKILL.md`를 참조하여 Step 2(적합도 산정)까지 실행
2. GitHub 탐색 결과(`_workspace/01_github_findings.json`)가 있으면 Fit Score 산정에 반영:
   - GitHub에서 발견된 추가 프로젝트/경험을 기술 스택 매칭과 경험 연관성 점수에 포함
   - "GitHub 발견 경험 포함 시 예상 점수"를 별도로 표시
3. Fit Score를 다음 형식으로 예쁘게 표시:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  적합도 분석: [회사명] — [포지션]

  기술 스택 매칭    ★★★★☆☆  6.5/10  — React는 ✅但 Go는 ❌
  경험 연관성       ★★★★★☆  7.0/10  — 유사 프로젝트 경험 있음
  경력 수준 적합성  ★★★★★★  8.0/10  — 요구 경력 5년 / 보유 6년
  키워드 밀도       ★★★★☆☆  6.0/10  — 핵심 키워드 7/12 매칭
  산업 도메인       ★★★☆☆☆  5.0/10  — 핀테크 경험 없음

  ───────────────────────────────────────────
  ★★★★☆☆  종합 적합도  6.5/10
  ───────────────────────────────────────────

  강점: React 생태계, 대규모 트래픽 경험
  약점: Go/핀테크 도메인 경험 부족
  개선 시 +1.5점: Go 사이드 프로젝트 추가, 결제 경험 강조
  ───────────────────────────────────────────
  🔍 GitHub 탐색 결과: 3개 레포 매칭
    ✅ kafka-pipeline (Kafka) — 이미 이력서에 있음 (자동 반영)
    🆕 go-cli-tool (Go) — 이력서에 없음 → 사용자 확인 필요
    🆕 kafka-streams-lab (Kafka) — 이력서에 없음 → 사용자 확인 필요
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

3. 출력: `_workspace/02_fit_score.json`

### GitHub 새로운 발견 사항 확인 (이력서에 없는 경험)

GitHub 탐색 결과에서 `new_discoveries`(이력서에 없는 경험)가 있으면
사용자에게 물어본 후 전략에 반영한다:

1. `_workspace/01_github_findings.json`의 `new_discoveries` 확인
2. `matched_repos` 중 `already_in_resume: false`인 레포도 새 발견으로 간주
3. 새 발견이 있으면 사용자에게 질문:

```
🔍 GitHub에서 이력서에 없는 경험을 발견했습니다:

  [1] go-cli-tool (Go, Docker)
      — CLI 자동화 도구, 8 stars
      → 공고 'Go 개발자' 요구사항과 매칭, Fit Score +0.8점 예상
      → 이력서에 추가할까요? (y/n)

  [2] kafka-streams-lab (Kafka, Java)
      — Kafka 스트리밍 처리 실습
      → 공고 'Kafka 경험' 요구사항과 매칭, Fit Score +0.5점 예상
      → 이력서에 추가할까요? (y/n)
```

- 사용자가 **y** → 새 발견을 전략에 포함. `content-strategy`에 추가 프로젝트로 전달
- 사용자가 **n** → 해당 발견을 제외. Fit Score 재계산 시에도 미반영
- 이미 이력서에 있는 경험(`already_in_resume: true`)은 묻지 않고 자동 포함

### 시나리오 A (맞춤수정)

1. Fit Score 표시 후 사용자에게 질문: "이 공고에 맞춰서 이력서를 수정할까요?"
2. Yes → 수정 전략 수립 계속 (content-strategy Step 3~5)
3. No → 종료 (Fit Score만 제공)
4. 출력: `_workspace/02_strategy.json`, `_workspace/02_fit_score.json`

### 시나리오 B (피드백)

1. 내부 모듈 `skills/quality-review/SKILL.md`를 참조하여 품질 검증 절차 수행
2. 이력서 데이터를 기반으로 종합 피드백 리포트 생성
3. **피드백 제공 후:** "이 피드백을 반영해서 수정할까요?" 질문
   - Yes → Phase 3-B(피드백 반영 수정)로 진행
   - No → 종료 (피드백 리포트만 출력)
4. 출력: `_workspace/04_review_report.json`

### 시나리오 C (디자인변경)

1. Phase 2에서 Fit Score 및 전략 수립 생략
2. 바로 Phase 4-B(디자인)로 이동

### 시나리오 D (반복수정)

1. 이전 실행 결과 확인 (`_workspace_prev/` 또는 `_workspace/`)
2. Fit Score 재계산 (변경사항 반영)
3. 사용자 피드백을 전략에 반영하여 전략 업데이트
4. 출력: `_workspace/02_strategy.json`, `_workspace/02_fit_score.json` (업데이트)

### 시나리오 E (적합도 확인)

1. Fit Score 계산 후 표시 (공통 section)
2. "이 공고에 맞춰서 수정할까요?" 질문
   - Yes → 시나리오 A로 전환 (Phase 2-A 계속)
   - No → 종료
3. 출력: `_workspace/02_fit_score.json`

### 시나리오 F (깃허브 기반 신규 작성)

> 이력서 없이 GitHub 링크만으로 시작. 경험 데이터는 GitHub에서 추출.

1. `_workspace/01_parsed_resume.json`에 GitHub 기반 경험 데이터가 있음
2. GitHub + 공고 URL이 모두 있으면 Fit Score 계산 (공통 section)
3. 사용자에게 질문:
   > "GitHub 경험을 바탕으로 기본 이력서를 작성했습니다.
   > 이제 공고에 맞춰서 수정할까요, 아니면 먼저 이력서에 추가할 정보(학력, 경력 등)를 알려주시겠어요?"
   - "공고에 맞춰서 수정": Phase 2-A로 (content-strategy로 전략 수립)
   - "추가 정보 입력": 사용자로부터 추가 경력/학력 등을 입력받아 `_workspace/01_parsed_resume.json` 업데이트 후 Phase 2-A로
4. **말투:** tone이 `"preserve"`인 경우 GitHub 분석 결과만으로는 원본 말투를 알 수 없으므로 `"professional"`로 폴백. 사용자에게 안내:
   > "이력서 원본이 없어 말투유지 대신 전문적인 말투를 사용합니다.
   > 다른 말투를 원하시면 알려주세요."
5. 출력: `_workspace/02_strategy.json`, `_workspace/02_fit_score.json`

---

### GATE: Phase 2 → Phase 3 (초안 작성 확인)

**시점:** 수정 전략 수립 완료 후

**질문:** "수정 전략을 준비했습니다. 첫 번째 버전(v1) 초안을 작성할까요?"

**대기:** 사용자 응답 필수

**분기:**
- **Yes / 네 / 작성해줘** → Phase 3-A(컨텐츠 작성) 진행
- **아니요 / 전략 수정** → "어떤 부분을 수정할까요?" 피드백 받고 전략 업데이트 후 다시 GATE
- **취소** → 종료

---

## Phase 3: 컨텐츠 작성

### Phase 3-A: 맞춤수정 컨텐츠 작성

1. 내부 모듈 `skills/content-craft/SKILL.md`를 참조하여 컨텐츠 작성 절차 수행
2. `_workspace/01_scenario.json`에서 `tone` 값을 읽어서 content-craft에 전달
3. tone이 `"preserve"`(말투유지)이면 `_workspace/01_tone_profile.json`(원본 말투 분석 결과)도 함께 전달하여 원본 말투를 모방하게 함
4. `_workspace/01_scenario.json`의 `output_target`에 따라 산출물을 작성한다.
   - `resume`: 이력서 초안 저장 → `_workspace/03_draft_resume_v1.md`
   - `portfolio`: 포트폴리오 초안 저장 → `_workspace/03_draft_portfolio_v1.md`
   - `both`: 이력서와 포트폴리오 초안을 모두 저장
5. 입력 자료와 산출물은 독립적이다. 이력서만 입력돼도 포트폴리오를 만들 수 있고, 포트폴리오만 입력돼도 이력서를 만들 수 있다.
6. `candidate_notes.military_service`가 있으면 기본 정보, 지원 조건, 또는 공고 적합성 섹션 중 자연스러운 위치에 명확히 반영한다.
7. 포트폴리오를 작성할 때(`output_target: "portfolio" | "both"`)는 전체 문서와 각 프로젝트를 아래 템플릿으로 작성한다:
   ```md
   # {이름} 포트폴리오

   **{직무/포지션}**

   ## {프로젝트명} - {한 줄 설명}

   ### 문제 01
   {일부러 만든 기술적 병목/제약을 실제 문제 상황처럼 서술}

   ### 해결
   {병목/제약 해결 방법을 기술적 해결로 서술}

   ### 문제 02
   {프로젝트가 해결하려는 사용자/운영/비즈니스 문제를 서술}

   ### 해결
   {도메인 구조, 기능 설계, 데이터 흐름으로 문제를 해결한 방법을 서술}
   ```
8. 각 프로젝트는 `문제 01`과 `문제 02`를 모두 포함한다. `문제 01`은 기술 면접용 문제, `문제 02`는 프로젝트 존재 이유/제품 문제로 작성한다.
9. 하나의 문제만 있으면 내용을 복제하지 말고 `일부러 만들 병목/제약`을 기술 문제로, 사용자/운영/비즈니스 영향을 제품 문제로 나눠 작성한다.
10. 프로젝트 기획서에서 포트폴리오를 만들 때는 아래 매핑을 따른다:
   - `문제 01`: `일부러 만들 병목/제약`
   - `문제 01`의 `해결`: `일부러 만들 병목/제약 해결 방법`, `기술적 해결 방법`, `검증`, `결과`
   - `문제 02`: `서비스 맥락에서 발생하는 실제 문제 상황`, `문제의 사용자/운영/비즈니스 영향`, `도메인 선택 이유`
   - `문제 02`의 `해결`: `해결 방향`, `아키텍처/데이터 플로우`, `데이터 모델/엔티티`, `API 명세`, `화면/사용자 플로우`
11. 포트폴리오의 문제 해결 경험에는 아래 정보가 반드시 들어가야 한다:
   - 문제 상황
   - 문제의 사용자/운영/비즈니스 영향
   - 원인
   - 해결 방향
   - 기술적 해결 방법
   - 일부러 만들 병목/제약
   - 일부러 만들 병목/제약 해결 방법
   - 검증
   - 결과
12. 전략 + 말투에 따라 선택된 산출물 내용 작성
13. 변경 로그 저장 → `_workspace/03_changelog_v1.json`

---

### GATE: Phase 3-A 피드백 (중간 버전 검토)

**시점:** v1 초안 작성 완료 후

**수행:** 사용자에게 v1 초안을 보여주고 변경사항 요약을 함께 제시

**질문:** "첫 번째 버전(v1) 초안입니다. 마음에 드시나요?"

```
주요 변경사항:
[변경사항 요약 — 바뀐 섹션과 이유]
```

**대기:** 사용자 응답 필수

**분기:**
- **좋아 / 계속 / 통과** → GATE: Phase 3 → 점수 향상(Phase 3-C)로 이동
- **수정 / 피드백** → 사용자 피드백 받아서 v2 생성 후 다시 GATE
- **취소 / 처음부터** → Phase 0으로 리셋

> 최대 3회 반복 (v1 → v2 → v3). 3회 초과 시 "더 수정해야 한다면 새로운 실행을 시작하는 게 좋겠습니다" 제안 후 강제 진행.

---

### Phase 3-B: 피드백 반영 수정 (시나리오 B 후속)

1. 내부 모듈 `skills/content-craft/SKILL.md`를 참조하여 컨텐츠 작성 절차 수행
2. `_workspace/01_scenario.json`에서 `tone` 값을 읽어서 content-craft에 전달
3. 피드백 리포트의 P0/P1 이슈를 반영하여 이력서 수정
4. 중간 버전 표시 및 피드백 수렴 (Phase 3-A와 동일)

---

### GATE: Phase 3 → 점수 향상 (Phase 3-C 진입 결정)

**시점:** 최종 버전(v1/v2/v3) 확정 후

**수행:** Fit Score 재계산

**질문 (Fit Score에 따라):**

- **8.0 미만:**
  > "현재 Fit Score가 **X.X/10**입니다. 목표 8.0까지 점수 향상을 진행할까요?
  > - **진행**: 약점([가장 낮은 항목])을 집중 개선
  > - **스킵**: 지금 상태로 품질 검증 진행"

- **8.0 이상:**
  > "Fit Score가 **X.X/10**으로 목표를 달성했습니다. 품질 검증을 진행할까요?
  > - **진행**: 품질 검증(Phase 4)으로 이동
  > - **추가 개선**: 1회 더 개선 사이클 실행"

**대기:** 사용자 응답 필수

**분기:**
- **진행 / 계속 / 개선해줘** → Phase 3-C(점수 향상) 진입
- **스킵 / 다음 / 괜찮아** → Phase 4(품질 검증)로 진행

---

### Phase 3-C: 점수 향상 루프 (Fit Score 8점 달성)

> 시나리오 **A(맞춤수정)** 와 **D(반복수정)** 에서만 실행한다.
> 시나리오 B(피드백), C(디자인변경), E(적합도 확인)는 이 Phase를 건너뛴다.

Phase 3-A/3-B에서 콘텐츠 수정이 완료된 후, Fit Score를 재계산하여
8.0 미만이면 **점수 향상**을 통해 Fit Score를 반복 개선한다.

#### 동작 원리

```
[초안 완료]
     │
     ▼ Fit Score 재계산
     │
     ┌────────────────────┐
     │  사이클 시작       │
     │  ───────────       │
     │  약점 차원 식별    │
     │  → 집중 개선       │
     │  → Fit Score 재계산 │
     │  → 진행 상황 표시   │
     └────────┬───────────┘
              │
              ▼ 사용자에게 질문 (매 사이클)
   "이번 사이클 결과입니다. 계속 개선할까요?"
              │
         ┌─────┴─────┐
         │ Yes       │ No
         ▼           ▼
   ┌── 다음 사이클 ──┐  Phase 4로 진행
   │  (최대 5회)    │
   │  정체 시 조기종료│
   └────────────────┘
```

#### Step 1: Fit Score 재계산
1. 내부 모듈 `skills/content-strategy/SKILL.md`를 참조하여 Step 2(적합도 산정)만 실행
2. 현재 초안 기준으로 Fit Score 재계산

#### Step 2: 점수 향상 수행 (매 사이클 사용자 선택)

**최초 진입 시:** Fit Score가 이미 8.0 이상이면 이 Phase를 건너뛰고 Phase 4로 진행한다.
**8.0 미만이면** 아래 루프에 진입한다.

각 사이클:

1. **약점 차원 식별:** 5개 항목 중 가장 점수가 낮은 1-2개 차원을 pinpoint
2. **원인 분석:** 왜 이 차원의 점수가 낮은지 구체적 근거 파악
   - 예: "키워드 밀도 6.0/10 → 공고에 'Kafka'가 필수인데 이력서에 없음"
3. **집중 개선:** 식별된 약점만 타겟팅하여 콘텐츠 수정
   - 내부 모듈 `skills/content-craft/SKILL.md`를 참조하여 특정 섹션만 수정 지시
   - 수정 범위를 최소화하라 — 관련 없는 부분은 건드리지 않는다
4. **Fit Score 재계산**
5. **사용자에게 진행 상황 표시:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  🔄 점수 향상 — 사이클 2/5

  이전: ★★★★☆☆  7.2/10
  현재: ★★★★★☆  8.4/10 (+1.2)

  개선 포인트:
  ✅ 키워드 밀도 6.0 → 8.5  ("Kafka" 경험 추가)
  ✅ 기술 스택 7.5 → 8.5   (Go 기본 문법 숙지 추가)

  남은 약점: 산업 도메인 6.0 (핀테크 경험)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

6. **매 사이클마다 사용자에게 질문:**

현재 점수와 관계없이 **매 사이클 종료 시** 항상 사용자에게 선택권을 준다:

   - **8.0 미만인 경우:**
     > "현재 **X.X/10**으로, 아직 목표 **8.0**에 도달하지 못했습니다.
     > 계속 개선할까요? 아니면 여기서 멈출까요?
     > - **계속 개선**: 다음 사이클 실행 (약점: [가장 낮은 항목] 집중)
     > - **여기까지**: 현재 상태로 최종 출력 진행"

   - **8.0 이상인 경우:**
     > "🎯 **Fit Score 8.0을 달성했습니다!** (현재: ★★★★★☆ X.X/10)
     > 계속 개선할까요? 아니면 여기서 마칠까요?
     > - **계속 개선**: 1회 더 개선 사이클 실행
     > - **여기까지**: 현재 상태로 최종 출력 진행"

   - 사용자가 **"계속 개선"** → 다음 사이클 실행 (최대 5회)
   - 사용자가 **"여기까지"** → Phase 4로 진행
   - 5회 초과 시: "최대 개선 사이클에 도달했습니다" → Phase 4로 진행
   - 개선 정체(2회 연속 +0.1 미만) 시: "더 이상 유의미한 개선이 어렵습니다" → Phase 4로 진행

#### Step 4: 최종 Fit Score 스냅샷 저장
- `_workspace/03_fit_score_history.json`에 개선 이력 저장
- 각 사이클의 before/after 점수와 변경 포인트 기록

```json
{
  "cycles": [
    {
      "cycle": 1,
      "before": 7.2,
      "after": 8.4,
      "improvements": ["키워드 밀도 +2.5", "기술 스택 +1.0"],
      "target_dimensions": ["keywords", "tech_stack"]
    },
    {
      "cycle": 2,
      "before": 8.4,
      "after": 8.7,
      "improvements": ["산업 도메인 +1.0"],
      "target_dimensions": ["domain"]
    }
  ],
  "final_score": 8.7,
  "total_improvement": 1.5
}
```

---

### GATE: Phase 3-C → Phase 4 (품질 검증 확인)

**시점:** 점수 향상 완료 또는 스킵 후

**질문:** "품질 검증을 진행할까요? (오타/문법/말투/ATS 호환성 검사)"

**대기:** 사용자 응답 필수

**분기:**
- **Yes / 네 / 검증해줘** → Phase 4-A(품질 검증) 진행
- **스킵 / 다음** → Phase 4-B(디자인 적용)로 이동

---

## Phase 4: 품질 검증 및 디자인

### Phase 4-A: 품질 검증

1. 내부 모듈 `skills/quality-review/SKILL.md`를 참조하여 품질 검증 절차 수행
2. 최종 초안 검증:
   - 오타/문법 검사
   - 말투/어조 분석
   - 내용 일관성 검증
   - ATS 호환성 검토
3. P0 이슈 자동 수정 → `_workspace/04_corrected_content.md`
4. 검증 리포트 저장 → `_workspace/04_review_report.json`
5. **사용자에게 검증 결과 요약 표시**

---

### GATE: Phase 4-A → Phase 4-B (디자인 적용 확인)

**시점:** 품질 검증 완료 후

**질문:** "검증 결과를 확인했습니다. 이제 디자인을 적용할까요?"

**대기:** 사용자 응답 필수

**분기:**
- **Yes / 네 / 적용해줘** → Phase 4-B(디자인 적용) 진행
- **수정 필요** → "어떤 부분을 수정할까요?" → Phase 3-A로 돌아가서 D 시나리오

---

### Phase 4-B: 디자인 적용

1. 내부 모듈 `skills/resume-designer/SKILL.md`를 참조하여 디자인 적용 절차 수행
2. 사용자에게 템플릿 선택 옵션 제공:
   - "어떤 스타일로 출력할까요?"
   - 1: **심플 (Classic)** — 흑백, 최소 디자인, 정보 전달 중심
   - 2: **모던 (Modern)** — 컬러 악센트, 깔끔한 라인, 여백 활용 (추천)
   - 3: **크리에이티브 (Creative)** — 독특한 레이아웃, 컬러 블록 (디자인 직군용)
   - 4: **ATS 최적화 (ATS-friendly)** — 키워드 가시성 극대화, 표/이미지 최소화
   - 사용자 선택 또는 직군 기반 추천
3. **프로필 이미지 처리:** `personal_info.profile_image`가 있으면 템플릿에 이미지 배치
   - 심플/ATS: 이미지 없음 (텍스트 중심)
   - 모던: 좌측 상단 원형 이미지
   - 크리에이티브: 헤더 영역에 이미지 통합
4. 템플릿 적용하여 마크다운 포맷팅
5. 출력: `_workspace/05_final_resume.md`

---

### GATE: Phase 4-B → Phase 5 (PDF 출력 확인)

**시점:** 디자인 적용 완료 후

**질문:** "디자인이 적용되었습니다. PDF로 출력할까요?"

**대기:** 사용자 응답 필수

**분기:**
- **Yes / 네 / 출력해줘** → Phase 5(PDF 출력) 진행
- **디자인 변경** → "다른 템플릿으로 변경할까요? 아니면 세부 조정이 필요할까요?" → Phase 4-B로 돌아가기
- **MD만** → PDF 생략, MD만 최종 제공

---

## Phase 5: PDF 출력

1. 내부 모듈 `skills/pdf-publisher/SKILL.md`를 참조하여 PDF 출력 절차 수행
2. 플랫폼에서 사용 가능한 방법으로 PDF 생성:
   - 가능: 자동 생성 → `_workspace/05_final_resume.pdf`
   - 불가능: 수동 변환 방법 안내
3. 프로필 이미지가 있는 경우 PDF에 포함 (base64 인코딩 또는 파일 참조)
4. **PDF가 생성된 경우:** 미리보기 요약 제공 (페이지 수, 파일 크기 등)

---

### GATE: Phase 5 → Phase 6 (최종 결과 확인)

**시점:** PDF 생성 완료 후

**질문:** "PDF 생성이 완료되었습니다. 최종 결과를 확인하시겠어요?"

**대기:** 사용자 응답 필수

**분기:**
- **Yes / 네 / 확인** → Phase 6(최종 결과 제공)
- **다시 수정** → "어느 단계로 돌아갈까요? (1: 내용수정 / 2: 디자인변경 / 3: 처음부터)"

---

## Phase 6: 최종 결과 제공

사용자에게 최종 결과물을 명확히 제시:

### 제공 항목
1. **최종 이력서**: `_workspace/05_final_resume.md` + (가능시) `_workspace/05_final_resume.pdf`
2. **변경사항 요약**: 무엇이 어떻게 바뀌었는지
3. **Fit Score 리포트**: 공고별 적합도 점수 (별점 + 항목별 분석)
4. **ATS 매칭 리포트**: 키워드 매칭률, 강점/약점
5. **피드백 요약** (시나리오 B): 주요 발견 사항과 개선 추천

### Fit Score 제공 형식
최종 결과에 Fit Score가 포함될 때는 다음 형식을 사용하라:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  📊 공고별 적합도 요약

  [회사명 A — 포지션]
  종합: ★★★★★☆  8.7/10  (개선 전 7.2 → 개선 후 8.7, +1.5)
  강점: React, TypeScript, 대규모 트래픽
  약점: Go 경험 부족 → 사이드 프로젝트로 보완 가능

  [회사명 B — 포지션]
  종합: ★★★★★★  9.2/10  (개선 전 8.5 → 개선 후 9.2, +0.7)
  강점: Java/Spring 전문성, 금융권 경험 완벽 매치
  약점: 없음
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 점수 향상 개선 이력 제공
점수 향상(Phase 3-C)이 실행된 경우, 개선 이력을 함께 제공하라:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  📈 점수 향상 개선 이력

  [회사명 A — 포지션]
  사이클 1: 7.2 → 8.4  (+1.2, 키워드/기술스택 집중)
  사이클 2: 8.4 → 8.7  (+0.3, 산업 도메인 보강)
  ──────────────────────────────────────────
  최종:   ★★★★★☆  8.7/10  (총 +1.5)
  사이클: 2회 / 목표: 8.0 ✅ 달성
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 후속 액션 제안
- "더 수정할 부분이 있나요?" → Phase 3으로 돌아가서 D 시나리오로 반복
- "새로운 공고로 다시 할까요?" → Phase 0으로 돌아가서 새 실행
- "디자인을 변경하고 싶나요?" → Phase 4-B로
- "완료" → 종료

---

## 데이터 전달 프로토콜

| Phase | 입력 | 출력 | 사용자 질문 필수? |
|-------|------|------|------------------|
| Phase 0 | 사용자 입력 | `01_scenario.json` | ✅ Phase→1 GATE |
| Phase 1 | Phase 0 출력 | `01_parsed_resume.json`, `01_job_analyses.json`, `01_github_findings.json` | ✅ Phase→2 GATE |
| Phase 2 | Phase 1 출력 | `02_fit_score.json`, `02_strategy.json` 또는 `04_review_report.json` | ✅ Phase→3 GATE |
| Phase 2-X | Phase 1 출력 + `work_mode: experience_blueprint` | `06_experience_blueprints/01_*.md` ... `04_*.md`, `06_experience_blueprints.json` | ✅ 기획서 선택 GATE |
| Phase 3 | Phase 2 출력 | `03_draft_resume_v{N}.md` 또는 `03_draft_portfolio_v{N}.md`, `03_changelog_v{N}.json`, `03_fit_score_history.json` | ✅ 피드백 GATE + 점수향상 GATE |
| Phase 4 | Phase 3 출력 | `04_review_report.json`, `04_corrected_content.md`, `05_final_resume.md` | ✅ 품질 GATE + 디자인 GATE |
| Phase 5 | Phase 4 출력 | `05_final_resume.pdf` | ✅ PDF GATE + 결과 GATE |

## 에러 핸들링

| 에러 유형 | 처리 |
|----------|------|
| Agent failure | 1회 재시도, 실패 시 해당 결과 없이 진행, 최종 보고서에 누락 명시 |
| Timeout | Phase별 제한 시간: Phase 1 (60s), Phase 2 (60s), Phase 3 (120s), Phase 4 (60s), Phase 5 (120s) |
| URL fetch 실패 | 실패한 URL 건너뛰고 나머지로 진행, 사용자에게 보고 |
| PDF 생성 실패 | 마크다운만 제공, 대체 방법 안내 |
| 데이터 충돌 | 출처 병기, 삭제 금지, 보고서에 상충 표시 |
| **GATE 무시** | **GATE에서 사용자 응답을 받지 않고 다음 Phase로 진행하지 마라. 사용자 응답이 없으면 다시 질문하라.** |

## 중간 버전 관리 규칙

1. 사용자에게 중간 버전을 보여줄 때는 **변경사항 요약**을 함께 제시하라
   - "이전 버전에서 변경된 점: ..."
2. 사용자 피드백은 다음 중 하나여야 한다:
   - "좋아, 계속 진행" → 다음 Phase로
   - "이 부분을 수정해줘: ..." → 피드백 반영 후 새 버전
3. 최대 반복 횟수: Phase 3에서 최대 3회 (무한 루프 방지)
4. 사용자가 3회 이상 수정을 요청하면: "더 수정해야 한다면 새로운 실행을 시작하는 게 좋겠습니다" 제안

## GATE 실행 규칙 (필수)

1. **모든 GATE는 반드시 실행하라.** GATE를 건너뛰고 다음 Phase로 진행하지 마라.
2. **사용자 응답을 기다려라.** 사용자가 응답할 때까지 같은 GATE를 유지하라.
3. **GATE 질문은 여러번 할 수 있다.** 사용자가 모호하게 응답하면 "죄송합니다, 다시 한번 여쭤볼게요"라고 말하고 명확한 선택지를 다시 제시하라.
4. **사용자가 이전 Phase로 돌아가길 원하면 해당 Phase로 돌아가라.** 순서를 강제하지 마라.
5. **사용자가 갑자기 종료를 원하면 워크플로우를 정리하고 종료하라.** 중간 데이터는 보존하라.

## 테스트 시나리오

### 정상 흐름 (맞춤수정 + 점수 향상)
1. 사용자: 이력서.md + 공고 URL 2개 제공
2. → GATE: 분석 시작 확인
3. → Fit Score 표시: ★★★★☆ 7.2/10, ★★★★★ 8.5/10
4. → GATE: "수정할까요?" → Yes
5. → GATE: "v1 초안을 작성할까요?" → Yes
6. → v1 표시 → GATE: "마음에 드시나요?" → 피드백 → v2
7. → GATE: "점수 향상을 진행할까요?" → Yes
8. → 사이클 1: 키워드 집중 개선 → 7.8→8.4 (+0.6)
9. → GATE: "계속 개선할까요?" → Yes
10. → 사이클 2: 산업 도메인 보강 → 8.4→8.7 (+0.3)
11. → GATE: "계속 개선할까요?" → "여기까지"
12. → GATE: "품질 검증을 진행할까요?" → Yes
13. → GATE: "디자인을 적용할까요?" → Yes
14. → GATE: 템플릿 선택 → "모던"
15. → GATE: "PDF로 출력할까요?" → Yes
16. → GATE: "최종 결과를 확인하시겠어요?" → Yes
17. → 변경사항 요약 + Fit Score 변화 (7.2→8.7, +1.5) + 개선 이력 + ATS 리포트

### 피드백 전용 흐름
1. 사용자: 이력서.md만 제공
2. → "피드백만 원하시나요?" → Yes
3. → GATE: "분석을 시작할까요?" → Yes
4. → GATE: "종합 피드백을 제공할까요?" → Yes
5. → 피드백 리포트 표시
6. → "이 피드백을 반영해서 수정할까요?" → Yes → 수정본
7. → 이후 모든 GATE 순차 실행

### 적합도 확인 흐름 (시나리오 E)
1. 사용자: 이력서.md + 공고 URL
2. → GATE: 분석 시작 확인 → Yes
3. → Fit Score 표시 (별점 + 항목별 분석 + 강약점)
4. → GATE: "이 공고에 맞춰서 수정할까요?"
5. → No → Fit Score 리포트만 출력하고 종료

### 문제해결 프로젝트 기획 흐름
1. 사용자: 공고 URL 또는 목표 직군 제공 + "프로젝트 문제해결 경험 생성해줘"
2. → 시작 질문에서 `experience_blueprint` 선택
3. → 공고 필수/우대 스택 분석
4. → `서비스 맥락에서 발생하는 실제 문제 상황`과 `해결 방향` 정의
5. → 공고 스택 기반 프로젝트 기획서 생성
6. → 데이터 모델/API/화면/이벤트/측정 지표/테스트/배포/README/이력서 bullet 포함
7. → `_workspace/06_experience_blueprints/01_*.md` ... `04_*.md`, `_workspace/06_experience_blueprints.json` 저장
8. → GATE: "이 중 어떤 프로젝트를 우선 구현할까요?"

### 디자인 변경 흐름
1. 사용자: "디자인만 바꿔줘"
2. → GATE: 템플릿 선택 → 선택
3. → GATE: 디자인 확인 → "PDF로 출력할까요?" → Yes
4. → 리디자인된 이력서 + PDF

### 에러 흐름
1. URL fetching 실패 (1개 성공, 1개 실패)
2. → 성공한 URL로만 진행
3. → Fit Score는 성공한 URL 기준으로만 계산
4. → 최종 보고서에 실패 URL 포함하여 사용자에게 알림
5. → 이후 모든 GATE는 정상 실행
