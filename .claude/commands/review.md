---
description: Run the python-code-reviewer subagent on the current PR diff (or staged changes if no PR exists).
argument-hint: "[optional: base branch, default origin/main]"
---

Invoke the `python-code-reviewer` subagent against the diff between `$1` (default: `origin/main`) and the current branch.

Steps:
1. Run `git fetch origin --quiet` then compute the diff via `git diff ${1:-origin/main}...HEAD --name-only` to enumerate changed files.
2. If no changes vs base, fall back to `git diff --cached --name-only`; if still empty, exit with "Nothing to review."
3. Delegate the review to the `python-code-reviewer` subagent. Pass it the file list and ask for the structured output defined in its prompt.
4. Print the agent's review verbatim. Do not post to GitHub from here — the slash command is local only.
