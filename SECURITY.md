# Security Policy

This document covers two distinct concerns:

1. **Reporting vulnerabilities** in this project.
2. **AI agent security rules** for any code in this repo that drives an LLM, tool, or agent.

The detailed reviewer checklist lives at [`.claude/context/ai-agent-security.md`](.claude/context/ai-agent-security.md). General development security rules live at `~/.claude/rules/development-rules.md` (user-global) and are not duplicated here.

---

## 1. Reporting a vulnerability

**Do not file public issues for security problems.**

Email `jsimonelli16@gmail.com` with:
- A description of the vulnerability and its impact
- Steps to reproduce (proof-of-concept welcome)
- Affected versions / commits
- Your contact for follow-up

Expect acknowledgement within 72 hours. Fixes for confirmed issues are prioritized; coordinated disclosure timelines are negotiable.

### Scope

In scope: code in this repository, its dependencies as configured here, and the CI/CD workflows in `.github/`.

Out of scope: third-party services this repo integrates with (report to those vendors directly), social engineering, physical attacks, and findings that require already-compromised credentials.

---

## 2. AI agent security rules

This project may invoke LLMs, expose tools to an agent, or persist agent memory. Apply these rules from the [OWASP AI Agent Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html). The reviewer enforces them on every PR.

### 2.1 Tool least-privilege

- Grant each tool only the minimum permissions needed for its task.
- Read-only by default. Write/delete/network access requires explicit justification.
- Maintain allowlists for permitted paths and operations.
- Block dangerous file patterns: `*.env`, `*.key`, `*.pem`, `*secret*`.
- Never expose a single "shell" tool to the agent.
- Sanitize tool parameters before logging or displaying.

### 2.2 Prompt-injection defense

- Treat every external input as hostile: user messages, retrieved docs, API responses, scraped web content, emails.
- Use clear delimiters between system instructions and untrusted data.
- Apply content filtering for known injection patterns.
- For high-risk content, use a separate LLM call to validate or summarize before insertion.
- Untrusted content must never directly drive tool-use decisions.

### 2.3 Memory & context hygiene

- Validate and sanitize data before writing to agent memory.
- Per-user / per-session isolation. No shared memory across tenants.
- Memory entries expire (default 24h) and have size caps (~100 items, ~5000 chars each).
- Redact PII (SSN, credit cards, passwords, API keys) before persistence.
- Use cryptographic checksums for integrity verification on long-lived memory.

### 2.4 Human-in-the-loop

- Classify each action: `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`.
- Auto-approve only `LOW` (read-only, idempotent).
- `MEDIUM+` queues for human approval with a preview of the exact action.
- Maintain an immutable audit trail of agent decisions.
- Provide a user-initiated interrupt and rollback path.

### 2.5 Output validation & guardrails

- Schema-validate every agent output (Pydantic / JSON Schema).
- PII filter on responses before display.
- Detect exfiltration patterns: oversized outbound payloads, suspicious URL encoding, base64 of internal data.
- Rate-limit agent actions (~100 calls / 60s baseline).
- Validate tool-call names and arguments against allowlists before execution.

### 2.6 Monitoring & observability

- Log every decision, tool call, and outcome to an immutable store.
- Redact secrets from logs (passwords, API keys, tokens, full PII).
- Alerts:
  - tool-call rate > 30/min
  - ≥ 5 failed tool calls
  - any injection-pattern detection
  - ≥ 3 sensitive-data accesses
  - session cost > $10 (denial-of-wallet)

### 2.7 Multi-agent

- Trust tiers: `UNTRUSTED`, `INTERNAL`, `PRIVILEGED`, `SYSTEM`.
- Allowlist recipients per agent.
- Sign and verify inter-agent messages.
- Message TTL (~5 min) prevents replay.
- Circuit breakers (5 failures → open) prevent cascading failure.

### 2.8 Data protection

- Classify data: `PUBLIC`, `INTERNAL`, `CONFIDENTIAL`, `RESTRICTED`.
- `RESTRICTED` fully redacted in any agent context; `CONFIDENTIAL` partially masked.
- Minimize sensitive data in prompts — pass IDs, fetch on demand.
- Encrypt at rest and in transit.
- Enforce retention and deletion per GDPR / CCPA where applicable.

---

## 3. LLM sub-processors

When this repo is read or modified through Claude Code, Codex, or any LLM CLI, file contents are sent to a third-party LLM provider. If this repo handles customer or regulated data:

- Sign a Data Processing Agreement with the AI vendor.
- Use zero-retention API endpoints where available.
- Document which directories are safe to include in agent context and which are not (use `.claudeignore` or equivalent).

## 4. Right to erasure

Deleting a file does not remove it from git history or from LLM provider logs. Proper deletion requires:

1. History rewrite with `git filter-repo` or BFG Repo-Cleaner.
2. Force-push and notify all clones.
3. Rotate any credentials that may have been exposed (assume compromised even on private repos).
4. Request deletion from LLM vendor conversation logs.

## 5. Audit controls

Recommended for any repo with shipped artifacts:

- Branch protection on `main` requiring PR review and signed commits.
- `CODEOWNERS` for sensitive folders (`.github/workflows/`, `.claude/agents/`, anything handling auth or secrets).
- Dependabot enabled (`.github/dependabot.yml`).
- `gitleaks` or `trufflehog` as a pre-commit hook.

## 6. Compliance

This template itself makes no compliance claims. SOC 2, HIPAA, GDPR, and similar frameworks depend on operational practices — documented agreements, data residency decisions, access controls — that live outside this repo.

---

*Adapted from the [OWASP AI Agent Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html) and [aakashg/product-growth-team-os SECURITY.md](https://github.com/aakashg/product-growth-team-os/blob/main/SECURITY.md).*
