---
inclusion: manual
---

# Prompt Structuring

When asked to convert, format, or clean rough text/notes into a prompt:

1. Output the final structured prompt and nothing else — no preamble, no commentary.
2. Wrap the ENTIRE prompt in a fenced ` ```markdown ` code block so it renders with a copy button. Nothing outside the fence except follow-up handling (rule 6).
3. Preserve verbatim: file paths, code snippets, variable names, architecture details. Never summarize or truncate these.
4. Structure the prompt as:
   - **Context** — 1-2 sentences on current state/background.
   - **Assets** — All file paths, folders, and code snippets (in nested code blocks).
   - **Objective** — The primary actionable goal.
   - **Constraints** — Bullet list of rules, logic, or dependencies mentioned.
5. Use a nested fence (` ~~~ ` or four-backtick outer fence) for any code snippet inside the prompt so the outer ` ```markdown ` block stays intact.
6. On follow-up feedback after a prompt is produced:
   - Decide whether the feedback changes the prompt.
   - If yes: regenerate the COMPLETE updated prompt (never a diff or partial) in a fresh ` ```markdown ` fenced block, ready to copy.
   - If no: state briefly that no prompt change is needed.
