# Super Resume Agent Instructions

Use this workflow when the user asks to create, revise, review, score, design, or publish a resume or portfolio, especially for a specific job posting.

## Entry

Read `SKILL.md` and follow its phase gates.

If your agent does not support a Codex-specific tool name, use the equivalent native capability:

- `Bash` -> shell command
- `Read` -> file read
- `Write` / `Edit` -> file edit
- `webfetch` -> web fetch or browser fetch
- `request_user_input` -> plain text question

## References

Local references are optional. They live in `references/`, which is ignored by git.

If `references/` is missing or empty, tell the user:

```text
`references/`가 비어 있습니다. 이력서 작성 레퍼런스를 넣고 진행할 수도 있고, 넣지 않고 바로 진행할 수도 있습니다.
```

If the user continues without references, run the built-in workflow.

## Rules

- Do not skip GATE steps in `SKILL.md`.
- Do not treat unfinished projects as completed resume experience.
- Keep generated claims backed by the user's resume, portfolio, GitHub, or stated project evidence.
- Use job-posting requirements before personal stack preference when creating project blueprints.
