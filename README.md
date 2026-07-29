<p align="center">
  <img src="assets/logo.png" alt="Super Resume" width="180">
</p>

# Super Resume

Codex plugin for job-fit resume and portfolio workflows.

![Codex Plugin](https://img.shields.io/badge/Codex-Plugin-1f2937)
![Skills](https://img.shields.io/badge/Skills-Super%20Resume-2563eb)
![Version](https://img.shields.io/badge/version-v0.1.0-2563eb)

Super Resume reads resumes, portfolios, GitHub evidence, and job postings, then guides a structured workflow: job-fit analysis, evidence planning, content revision, project blueprint generation, quality review, and PDF-ready output.

Designed for cases where a resume should prove fit for a specific role rather than just listing experience.

## Quick Start

```bash
codex plugin marketplace add cyjoon68/super-resume --ref v0.1.0
codex plugin add super-resume@super-resume-marketplace
```

Restart Codex and invoke:

```
@super-resume Tailor my resume and portfolio to this job posting.
```

For project experience planning:

```
@super-resume Create portfolio-ready project experience for this job posting.
```

## How It Works

The workflow runs in gated phases, each requiring user confirmation before proceeding:

1. **Mode selection** — resume-based tailoring or project experience blueprint.
2. **Output target** — resume, portfolio, or both.
3. **Input collection** — resume, portfolio, GitHub links, job postings.
4. **Analysis** — parse resume, explore GitHub repos, analyze job requirements.
5. **Fit score** — tech stack match, experience relevance, keyword density, domain fit.
6. **Strategy** — decide what to emphasize, add, or restructure.
7. **Drafting** — rewrite content following the strategy and selected tone.
8. **Score improvement** — iterative fit score boosting (up to 5 cycles).
9. **Quality review** — grammar, tone consistency, ATS compatibility.
10. **Design & PDF** — template selection and PDF export.

## Blueprint Mode

When generating project experience, the workflow:

- Analyzes job posting required/preferred stacks.
- Defines a real-world problem scenario in the service domain.
- Creates 4 separate project blueprints with data models, APIs, events, and metrics.
- Keeps unfinished plans separate from completed resume bullets.
- Asks for approval before any implementation begins.

## Reference Notes

The repository includes reference notes under `references/` used by agents during analysis, strategy, and drafting:

- `core.md` — first-screen requirements, failure patterns, motivation structure
- `checklist-formulas.md` — verification items, sentence patterns
- `competency-signals.md` — FE/BE skill signals, writing maturity levels
- `experience-blueprints.md` — portfolio project selection criteria
- `portfolio.md` — section structure, relationship to resume
- `projects.md` — STAR structure, quantification, technical depth

Each agent loads the relevant references before its phase. Missing references are skipped with a log entry.

## Repository Layout

```
.codex-plugin/plugin.json
.agents/plugins/marketplace.json
.agents/skills/           — 15 sub-skills
.codex/agents/            — 8 agent definitions
references/               — 6 reference note files
assets/logo.png
SKILL.md
AGENTS.md
SUBMISSION.md
PRIVACY.md
```

## Install Options

Stable release:

```bash
codex plugin marketplace add cyjoon68/super-resume --ref v0.1.0
codex plugin add super-resume@super-resume-marketplace
```

Development:

```bash
codex plugin marketplace add cyjoon68/super-resume --ref main
codex plugin add super-resume@super-resume-marketplace
```

Local:

```bash
codex plugin marketplace add /path/to/super-resume
codex plugin add super-resume@super-resume-marketplace
```

Restart Codex after installing or updating.

## License

[MIT](LICENSE).
