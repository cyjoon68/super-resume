# Super Resume for Claude Code

Use `SKILL.md` as the main workflow.

Trigger this workflow when the user asks for:

- resume creation or revision
- portfolio creation or revision
- job-posting fit analysis
- GitHub-based resume drafting
- project or problem-solving experience blueprints
- resume PDF or design output

If a Codex-specific instruction appears, use the Claude Code equivalent. For example, ask a normal question instead of `request_user_input` when needed.

Do not skip the GATE steps in `SKILL.md`.

Local `references/` files are optional and ignored by git. If they are missing, tell the user they can add references or continue without them.
