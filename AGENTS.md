# CipherChat --- AI Agent Instructions

**Primary Source of Truth:** [`PROJECT_CONTEXT.md`](PROJECT_CONTEXT.md)

All AI coding assistants (including Claude Web, Antigravity Desktop, and any other agents) working on CipherChat must strictly follow these instructions.

---

## 1. Core Operating Principles

1. **Inspect Before Changing:** Always inspect the actual repository, git status, and existing files before proposing or making changes. Never assume a file or feature exists just because it is described in planning documents.
2. **One Small Change at a Time:** Break work down into small, reviewable steps. Explain what you are doing in plain, beginner-friendly language.
3. **Plan and Wait for Approval:** Always present a clear plan explaining the intended change, and wait for explicit user approval before modifying or creating files.
4. **Do Not Commit:** Never run `git commit` or commit files. The developer makes and manages all Git commits.
5. **Never Invent Cryptography:** Encryption must use an established, vetted, and maintained library/protocol selected after documented evaluation. Never write custom crypto algorithms or claim placeholder encryption is secure.
6. **No Phantom Tests:** Never claim tests pass without running them. If tests cannot be run, clearly state that they were not run and explain why.
7. **Report Commands and Outputs:** Always report the exact files changed and the exact commands run with their real output.
8. **Never Leak Secrets:** Keep secrets, private keys, session tokens, passwords, and `.env` files out of source control and logs.

---

## 2. Session Initialization Routine

At the start of every session:
1. Read [`PROJECT_CONTEXT.md`](PROJECT_CONTEXT.md) and [`docs/STATUS.md`](docs/STATUS.md).
2. Inspect the actual repository files, git status, and package scripts.
3. Report:
   - What already exists
   - What is incomplete or broken
   - The smallest next implementation task
   - Acceptance criteria and tests for that task
4. Wait for user approval before writing or editing code.

---

## 3. Task Handoff Template

When completing a task or handing off work between sessions, summarize using this structure:

```text
Task:
Goal and acceptance criteria:
Files inspected:
Files changed:
Design decisions:
Commands run and results:
Tests added/updated:
Known limitations or risks:
Open questions:
Suggested next task:
```
