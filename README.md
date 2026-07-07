<p align="center">
  <img src="assets/logo.png" alt="Super Resume" width="180">
</p>

# Super Resume

Codex plugin for job-fit resume and portfolio workflows.

![Codex Plugin](https://img.shields.io/badge/Codex-Plugin-1f2937)
![Skills](https://img.shields.io/badge/Skills-Super%20Resume-2563eb)
![Version](https://img.shields.io/badge/version-v0.1.0-2563eb)

Super Resume is a Codex plugin that helps turn a resume, portfolio, GitHub evidence, and job posting into a structured proof workflow: analyze job fit, plan evidence, revise resume/portfolio content, generate project blueprints, review quality, and prepare PDF-ready output.

It is useful when a resume should prove fit for a specific job instead of only listing experience.

> Super Resume does not invent completed experience. Project blueprints are separated from completed resume bullets, and the workflow includes gates before drafting, implementation, design, PDF output, and finalization.

## Quick Start

Install the stable release:

```bash
codex plugin marketplace add cyjoon68/super-resume --ref v0.1.0
codex plugin add super-resume@super-resume-marketplace
```

Restart Codex, then invoke the plugin:

```text
@super-resume Tailor my resume and portfolio to this job posting.
```

For project experience planning:

```text
@super-resume Create portfolio-ready project experience for this job posting.
```

## What Changes In Codex

When you invoke `@super-resume`, Codex applies a gated resume workflow:

1. Ask which mode to use before proceeding.
2. Ask whether to create a resume, portfolio, or both.
3. Read resume, portfolio, GitHub, and job posting evidence.
4. Analyze job-fit gaps before drafting.
5. Create a strategy before rewriting content.
6. For project blueprint mode, generate four separate project plans.
7. For portfolio output, use `문제 01`, `해결`, `문제 02`, `해결`.
8. Verify quality before design or PDF output.
9. Keep unfinished project plans separate from completed experience.

## When To Use It

Use it for:

- Tailoring a resume to a specific job posting.
- Turning GitHub projects into evidence-backed resume bullets.
- Writing portfolio sections around problem solving.
- Planning missing project experience for a target role.
- Checking whether resume claims are supported by GitHub or portfolio evidence.
- Preparing design/PDF-ready resume output.

Skip it for:

- A tiny wording tweak.
- Generic career advice with no target role.
- Claims you cannot support with real work or planned implementation.

## Workflow Modes

Super Resume always asks which mode to use.

```text
1. 현재 이력서 기준으로 공고에 맞추기
2. 이력서에 넣을 프로젝트/문제해결 경험 생성까지 같이 하기
```

Mode 1 updates an existing resume/portfolio around a job posting.

Mode 2 creates project blueprints for missing evidence. After the blueprint is created, the workflow asks which project to implement, then later asks whether to add the code that resolves the planned bottleneck or constraint.

## Portfolio Structure

Portfolio project sections use this structure:

```md
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

`문제 01` is the technical interview problem.  
`문제 02` is the product, user, operations, or business reason the project exists.

## References

The public plugin does not include private reference notes.

If you want local references, add them inside the installed plugin skill folder:

```text
skills/super-resume/references/
```

When references are present, the workflow reads them before analysis or drafting. When references are missing, the workflow can proceed without them or ask you to add notes first.

## Install Options

Stable release:

```bash
codex plugin marketplace add cyjoon68/super-resume --ref v0.1.0
codex plugin add super-resume@super-resume-marketplace
```

Development version:

```bash
codex plugin marketplace add cyjoon68/super-resume --ref main
codex plugin add super-resume@super-resume-marketplace
```

Local development:

```bash
codex plugin marketplace add /path/to/super-resume
codex plugin add super-resume@super-resume-marketplace
```

Restart Codex after installing or updating the plugin.

## Repository Layout

```text
.codex-plugin/plugin.json
.agents/plugins/marketplace.json
assets/logo.png
skills/super-resume/SKILL.md
skills/super-resume/agents/
skills/super-resume/skills/
```

## Submission

Codex Plugin Directory submission notes are in [`SUBMISSION.md`](SUBMISSION.md).

Privacy notes are in [`PRIVACY.md`](PRIVACY.md).

## License

UNLICENSED.
