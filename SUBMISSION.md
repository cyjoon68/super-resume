# Codex Plugin Directory Submission

## Plugin

- Name: Super Resume
- Repository: https://github.com/cyjoon68/super-resume
- Version: v0.1.0
- Category: Productivity
- Logo: `assets/logo.png`
- Privacy policy: https://github.com/cyjoon68/super-resume/blob/main/PRIVACY.md

## Description

Super Resume is a Codex workflow plugin that reads a user's resume, portfolio, GitHub evidence, and job posting, then guides job-fit analysis, resume/portfolio revision, project experience planning, design review, and PDF output.

## Capabilities

- Skills-only plugin
- No MCP server
- No OAuth
- No external API owned by this plugin
- No server-side data storage

## Test prompts

```text
Use $super-resume to tailor my resume to this frontend engineer job posting.
```

Expected:

- Ask which mode to use before proceeding.
- Ask what output to create: resume, portfolio, or both.
- Analyze job-fit evidence before drafting.

```text
Use $super-resume to create portfolio-ready project experience for this job posting.
```

Expected:

- Ask which mode to use before proceeding.
- In project blueprint mode, create 4 separate project plans.
- Include `일부러 만들 병목/제약` and `일부러 만들 병목/제약 해결 방법`.

```text
Use $super-resume to write a portfolio section from this project blueprint.
```

Expected:

- Use `문제 01`, `해결`, `문제 02`, `해결`.
- Map `문제 01` from the planned technical bottleneck/constraint.
- Map `문제 02` from the user, operations, or business problem.

## Review notes

The plugin is intended for user-controlled resume and portfolio drafting. It should not represent unfinished project plans as completed experience. The workflow includes explicit gates before drafting, project implementation, design/PDF output, and finalization.
