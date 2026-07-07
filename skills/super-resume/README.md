<p align="center">◯ ─────────── ◯</p>

<h1 align="center">S U P E R&nbsp;&nbsp;R E S U M E</h1>

<p align="center">◯ ─────────── ◯</p>

<p align="center">
  <strong>이력서를 그냥 고치지 마세요. 공고에 맞는 증거 구조를 먼저 만드세요.</strong>
</p>

<p align="center">
  지원자의 이력서, 포트폴리오, GitHub, 채용 공고를 읽고<br>
  공고 적합도 분석부터 문장 수정, 프로젝트 경험 설계, 디자인/PDF 출력까지 이어주는<br>
  에이전트 워크플로우입니다.
</p>

<p align="center">
  <code>Codex</code> · <code>Claude Code</code> · <code>OpenCode</code> · <code>Cursor</code>
</p>

<p align="center">
  <em>이력서는 문장이 아니라 판단 자료입니다. 먼저 무엇을 증명해야 하는지 정해야 합니다.</em>
</p>

<div align="center">
<table width="720">
  <tr>
    <td width="50%" align="left"><strong>기술 스택</strong><br>공고의 기술을 실제로 다뤄봤는가?</td>
    <td width="50%" align="left"><strong>문제 해결</strong><br>프로젝트가 문제, 행동, 결과로 읽히는가?</td>
  </tr>
  <tr>
    <td width="50%" align="left"><strong>증거</strong><br>GitHub와 포트폴리오가 주장을 뒷받침하는가?</td>
    <td width="50%" align="left"><strong>보완</strong><br>부족한 경험은 어떤 프로젝트로 채울 수 있는가?</td>
  </tr>
</table>
</div>

---

## 빠른 시작

Codex 로컬 스킬:

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/cyjoon68/super-resume.git ~/.codex/skills/super-resume
```

Claude Code 전역 스킬:

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/cyjoon68/super-resume.git ~/.claude/skills/super-resume
```

OpenCode 전역 스킬:

```bash
mkdir -p ~/.config/opencode/skills
git clone https://github.com/cyjoon68/super-resume.git ~/.config/opencode/skills/super-resume
```

Cursor 전역 User Rules:

```bash
git clone https://github.com/cyjoon68/super-resume.git
cat super-resume/.cursor/rules/super-resume.mdc | pbcopy
```

Cursor Settings → Rules → User Rules에 붙여넣습니다.

참조 노트는 스킬 설치 폴더 안에만 추가:

```bash
mkdir -p ~/.codex/skills/super-resume/references
```

스킬 실행 중 `references/`가 비어 있으면 에이전트가 입력을 받아 스킬 폴더 안의 `references/`에 저장할 수 있습니다.
비워도 기본 워크플로우로 실행됩니다.

---

## 무엇을 하나요

```text
Resume / Portfolio / GitHub / Job Posting
        ↓
Parse → Analyze → Score → Strategize
        ↓
Draft → Review → Design → Publish
```

기능 역할

| 기능 | 역할 |
| --- | --- |
| 입력 분석 | 이력서, 포트폴리오, URL, PDF, GitHub를 구조화 |
| 공고 분석 | 필수/우대 스택, 도메인, 평가 키워드 추출 |
| Fit Score | 기술 스택, 경험 연관성, 키워드, 도메인 점수화 |
| GitHub 탐색 | 이력서에 없는 public 프로젝트 발견 |
| 콘텐츠 전략 | 요약, 프로젝트 bullet, 지원동기 방향 결정 |
| 프로젝트 기획 | 공고 적합도를 높일 문제해결 프로젝트 4개 생성 |
| 품질 검증 | 오타, 구체성, 수치, 링크, ATS 관점 점검 |
| 출력 | Markdown 디자인 적용 및 PDF 출력 |

---

## 워크플로우

```text
Phase 0  입력 확인
Phase 1  이력서/GitHub/공고 분석
Phase 2  Fit Score + 전략 수립
Phase 3  초안 작성 + 점수 향상
Phase 4  품질 검증 + 디자인
Phase 5  PDF 출력
Phase 6  최종 결과 제공
```

각 Phase 사이에는 GATE가 있습니다.
사용자 확인 없이 다음 단계로 넘어가지 않습니다.

```text
입력 확인 → 분석 시작?
분석 완료 → 전략 수립?
전략 완료 → 초안 작성?
초안 확인 → 점수 향상?
검증 완료 → 디자인?
디자인 완료 → PDF?
```

---

## 시나리오

| 시나리오 | 입력 | 출력 |
| --- | --- | --- |
| 공고 맞춤 수정 | 이력서 + 공고 URL | 공고 기준 수정본 |
| 피드백 | 이력서 또는 GitHub | 종합 피드백 |
| 디자인 변경 | 기존 이력서 | 디자인 적용본 |
| 적합도 확인 | 이력서 + 공고 URL | Fit Score 리포트 |
| GitHub 기반 작성 | GitHub 링크 | 신규 이력서 초안 |
| 문제해결 경험 생성 | 공고 URL 또는 목표 직군 | 프로젝트 기획서 4개 |

---

## 문제해결 프로젝트 기획

경험이 부족한 공고라면, 이력서에 바로 넣을 허위 문장을 만들지 않습니다.
대신 완성 후 이력서와 포트폴리오에 반영할 수 있는 프로젝트 기획서를 만듭니다.

기획서는 다음을 포함합니다:

- 서비스 맥락에서 발생하는 실제 문제 상황
- 공고 기술 스택 매핑
- 데이터 모델과 API
- 화면과 사용자 플로우
- 이벤트 로깅과 측정 지표
- 테스트와 부하 테스트
- 배포 방식
- README 구조
- 완성 후 사용할 수 있는 이력서 bullet
- 구현 에이전트용 작업 지시서

완료하지 않은 프로젝트를 완료 경험처럼 쓰지 않습니다.

---

## 참조 노트

`references/`는 스킬 설치 폴더 안에 두는 개인 학습 노트/이력서 작성 기준입니다.

```text
references/
  your-notes.md
  job-writing-rules.md
  portfolio-checklist.md
```

공개 repo에는 포함하지 않습니다.
저작권이 애매한 영상 요약, 강의 노트, 자막 기반 정리는 여기에만 두세요.

비어 있으면 에이전트가 이렇게 안내합니다:

```text
스킬 폴더 안의 `references/`가 비어 있습니다. 이력서 작성 레퍼런스를 입력해 주시면 스킬 안의 `references/`에 저장하고 진행할 수 있고, 넣지 않고 바로 진행할 수도 있습니다.
```

---

## 구조

```text
.
├── AGENTS.md
├── CLAUDE.md
├── SKILL.md
├── .cursor/
│   └── rules/
│       └── super-resume.mdc
├── agents/
│   ├── content-crafter.md
│   ├── content-strategist.md
│   ├── design-publisher.md
│   ├── input-analyzer.md
│   └── quality-reviewer.md
└── skills/
    ├── content-craft/
    ├── content-strategy/
    ├── github-explorer/
    ├── job-analyzer/
    ├── pdf-publisher/
    ├── quality-review/
    ├── resume-designer/
    └── resume-parser/
```

---

## 원칙

- 공고의 필수/우대 스택을 우선합니다.
- 이력서 문장은 근거가 있는 주장만 씁니다.
- GitHub 발견 경험은 사용자 확인 후 반영합니다.
- 모든 중간 결과는 `_workspace/`에 저장합니다.
- GATE 응답 없이는 다음 Phase로 진행하지 않습니다.
- 프로젝트 기획은 허위 경험이 아니라 완성 후 사용할 수 있는 증거 설계입니다.

---

## 라이선스와 참조

참조 노트는 사용자가 직접 채웁니다.
공개 repo에는 워크플로우와 에이전트 지시문만 포함됩니다.
