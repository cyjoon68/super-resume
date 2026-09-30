<p align="center">
  <img src="assets/logo.png" alt="Super Resume" width="180">
</p>

# Super Resume

공고 맞춤 이력서, 포트폴리오, 자기소개서 워크플로우를 위한 Codex 플러그인입니다.

![Codex Plugin](https://img.shields.io/badge/Codex-Plugin-1f2937)
![Skills](https://img.shields.io/badge/Skills-Super%20Resume-2563eb)
![Version](https://img.shields.io/badge/version-v0.1.0-2563eb)

Super Resume는 이력서, 포트폴리오, GitHub 근거, 채용 공고를 읽고 구조화된 워크플로우로 안내합니다. 공고 적합도 분석, 근거 계획, 콘텐츠 수정, 경험 기반 자기소개서 작성, 프로젝트 기획서 생성, 품질 검증, PDF 출력까지 이어집니다.

경험을 나열하는 이력서가 아니라 특정 직무에 대한 적합성을 증명하는 이력서가 필요한 경우를 위해 설계했습니다.

## 빠른 시작

```bash
codex plugin marketplace add cyjoon68/super-resume --ref v0.1.0
codex plugin add super-resume@super-resume-marketplace
```

Codex를 재시작하고 호출합니다:

```
@super-resume 이 공고에 맞춰 이력서와 포트폴리오를 수정해줘.
```

경험 기반 자기소개서가 필요할 때:

```text
@super-resume 이 공고에 맞는 자기소개서를 내 이력서와 포트폴리오 근거로 작성해줘.
```

프로젝트 경험 기획이 필요할 때:

```
@super-resume 이 공고에 맞는 포트폴리오용 프로젝트 경험을 만들어줘.
```

## 작동 방식

워크플로우는 단계별 GATE로 진행되며, 다음 단계로 넘어가기 전에 사용자 확인을 받습니다.

1. **모드 선택** — 이력서 기반 맞춤 또는 프로젝트 경험 기획서.
2. **산출물 선택** — 이력서, 포트폴리오, 자기소개서를 각각 독립 선택.
3. **입력 수집** — 이력서, 포트폴리오, GitHub 링크, 채용 공고.
4. **분석** — 이력서 파싱, GitHub 저장소 탐색, 공고 요구사항 분석.
5. **Fit Score** — 기술 스택 일치, 경험 연관성, 키워드 밀도, 도메인 적합도.
6. **전략** — 강조할 것, 추가할 것, 재구성할 것 결정.
7. **작성** — 선택한 산출물 재작성. 자기소개서는 공고 카테고리, 근거 기반 소제목, N개 경험 단락으로 구성.
8. **점수 향상** — Fit Score 반복 개선 (최대 5회).
9. **품질 검증** — 문법, 말투 일관성, ATS 호환성.
10. **디자인 & PDF** — 선택한 산출물에 하나의 템플릿 적용, 산출물별 PDF 출력 (선택).

## Blueprint 모드

프로젝트 경험을 생성할 때 워크플로우는 다음을 수행합니다:

- 공고의 필수/우대 기술 스택을 분석합니다.
- 서비스 도메인에서 실제 문제 시나리오를 정의합니다.
- 데이터 모델, API, 이벤트, 측정 지표를 포함한 프로젝트 기획서 4개를 생성합니다.
- 미완성 계획은 완료된 이력서 경험과 분리합니다.
- 구현 시작 전 반드시 승인을 받습니다.
- 구현과 완료 근거가 검증된 뒤에만 선택한 산출물 작성으로 돌아갑니다.

## 레퍼런스 노트

저장소의 `references/`에는 분석, 전략, 작성 단계에서 에이전트가 사용하는 레퍼런스 노트가 있습니다:

- `core.md` — 첫 화면 요건, 실패 패턴, 지원동기 구조
- `checklist-formulas.md` — 검증 항목, 문장 패턴
- `competency-signals.md` — FE/BE 역량 신호, 문장 성숙도 단계
- `experience-blueprints.md` — 포트폴리오 프로젝트 선정 기준
- `portfolio.md` — 섹션 구조, 이력서와의 관계
- `projects.md` — STAR 구조, 수치화, 기술 깊이

각 에이전트는 자기 단계 전에 관련 레퍼런스를 읽습니다. 누락된 레퍼런스는 로그를 남기고 건너뜁니다.

## 저장소 구조

```
.codex-plugin/plugin.json
.agents/plugins/marketplace.json
skills/                    — 하위 스킬 15개
.codex/agents/            — 에이전트 정의 8개
references/               — 레퍼런스 노트 6개
assets/logo.png
SKILL.md
AGENTS.md
SUBMISSION.md
PRIVACY.md
```

## 설치 옵션

안정 버전:

```bash
codex plugin marketplace add cyjoon68/super-resume --ref v0.1.0
codex plugin add super-resume@super-resume-marketplace
```

개발 버전:

```bash
codex plugin marketplace add cyjoon68/super-resume --ref main
codex plugin add super-resume@super-resume-marketplace
```

로컬:

```bash
codex plugin marketplace add /path/to/super-resume
codex plugin add super-resume@super-resume-marketplace
```

설치 또는 업데이트 후 Codex를 재시작하세요.

## 라이선스

[MIT](LICENSE).
