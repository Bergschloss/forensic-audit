# 🛡️ Forensic Audit Skill

> **Professional Standard:** This framework has been validated through 30+ audit runs on a production Turborepo monorepo (Next.js, Fastify, Electron, Prisma/PostgreSQL). It represents a mature, systematic approach to automating QA, database integrity, and security auditing, and can be adopted as a standard auditing pipeline for medium-to-large scale commercial projects.

A three-phase, read-only forensic audit protocol designed for AI coding assistants (such as Antigravity, Claude, or other agentic frameworks). This skill guides the agent to systematically search the codebase for security vulnerabilities, race conditions, performance bottlenecks, and system resilience risks.

## 🌟 What It Does

The skill defines 3 sequential phases of detailed checking:

* **Phase 1: Security, Cryptography & Data Privacy**
  * **Authentication & Authorization:** Guarding against unauthenticated privilege escalation, validating JWT expiry/refresh rotation (RTR), and protecting against TOTP replay attacks.
  * **IDOR validation:** Auditing ownership checks on client-scoped endpoints.
  * **Cryptography & Secret Hygiene:** Checking AES-256 key configuration, Electron `safeStorage`, hardcoded secrets in source files, and GDPR compliance (PII deletion/export).
  * **Stripe Webhook Security:** Verifying webhook signature validation (`stripe.webhooks.constructEvent`).

* **Phase 2: Database Integrity, Transactions & Performance**
  * **Race conditions:** Finding TOCTOU (Time-of-Check to Time-of-Use) race conditions and ensuring atomic database transactions for critical updates (e.g. queue acceptance, voucher bookings).
  * **Query Optimization:** Auditing N+1 query loops, missing database indexes, and unbounded queries.
  * **Resource leaks:** Checking connection pooling limits and SSE listener cleanup.

* **Phase 3: Architecture, SOLID, Resilience & Lifecycle**
  * **Lifecycle & Timezones:** Verifying cron jobs are DST-safe and timezone-resilient.
  * **SOLID & Error Handling:** Checking structured JSON error logs (preventing database query leaks to the client) and fail-fast startup validations for missing environment variables.
  * **Desktop Integration:** Electron IPC bridge security (`contextIsolation`, `nodeIntegration`).

---

## How to Use

1. **Install:** Save `SKILL.md` into `~/.claude/skills/forensic-audit/SKILL.md` (create the folder if needed). Restart your agent session - the skill becomes available as `/forensic-audit`.
2. **Invoke with a defined scope:** Trigger it for one file (or one tightly-related file group) at a time - e.g. "/forensic-audit on src/routes/auth.ts". See Scope Discipline in SKILL.md - never point it at an entire repository in one pass.
3. **Read the report:** Each phase is written to a timestamped report in %USERPROFILE%\Desktop\_Reports\<timestamp>\ - never inside the project repository.
4. **Auditing a whole codebase?** See "Orchestrating a File-by-File Forensic Audit" below - generate a task list first, then run it one file at a time.

---

## 📋 Orchestrating a File-by-File Forensic Audit (Multi-Agent Pipeline)

For complex backend APIs or infrastructure codebases, doing a single repository scan for security flaws can miss subtle bugs (like a missing `await` before a database transaction or a missing TOTP replay guard). 

To ensure maximum focus and precision, we use a **File-by-File Forensic Audit Pipeline**. The template is provided in [tasks-forensic-example.md](./tasks-forensic-example.md).

### How to use this strategy:

1. **Ask the Agent to Scan & List:** 
   Feed the codebase to your AI assistant and ask it to scan the project files, identify all critical security endpoints, database services, schemas, and configurations, and generate a task list formatted like `tasks-forensic-example.md`. The LLM should determine which files are high-risk (auth, payments, crypto) and mark them for dual-run.
2. **Parallel Task Execution:**
   Instruct the orchestrator agent to spawn ALL `runA`/`runB` subagent tasks concurrently in a single batch. Each subagent operates in an isolated context, reads its target file, runs `/forensic-audit`, and writes its `FINAL.md`.
3. **Pipelined Merging (orchestrator does this directly):**
   As soon as both `runA` and `runB` for a dual-run scope complete, the orchestrator immediately merges them into `MERGED.md` — keeping the higher severity for shared findings and tagging unique ones. **Do not delegate merges** — the orchestrator handles this faster than spawning another subagent.
4. **Progress Logging:**
   The orchestrator updates `00_PROGRESS.md` with finding counts (`CRITICAL`/`HIGH`/`MEDIUM`/`LOW`) after each subagent completes its task.
5. **Consolidate to Master Report (orchestrator does this directly):**
   Once all tasks are done, the orchestrator reads all reports, deduplicates cross-file patterns, and outputs a consolidated `AUDIT_FINAL.md` sorted by severity. **Do not delegate the final merge** — only the orchestrator has full cross-file context for deduplication.

---

## 💻 Default Technology Stack

The default searches and rules in `SKILL.md` are set up for:
* **Backend:** Fastify API, JWT plugin, TOTP verification, Server-Sent Events (SSE).
* **Database & ORM:** Prisma ORM, PostgreSQL.
* **Desktop Client:** Electron IPC bridges (`ipcMain`, `ipcRenderer`).
* **Integrations:** Stripe SDK (Billing), Twilio SDK.

---

## 🔄 Custom Stack Adaptation

This skill is framework-agnostic. You can easily adapt it to any other technology stack (e.g., Python/Django, Go, Ruby on Rails, Express/Sequelize) by editing `SKILL.md` and modifying:
1. **Search queries (Ripgrep / Grep):** Update directory structures and ORM queries (e.g., change `prisma.findMany()` to match Sequelize, Mongoose, or Hibernate syntax).
2. **Security & Cryptography rules:** Replace the JWT/Stripe validation checkpoints with the specific libraries and middleware configuration of your chosen stack.
3. **Database transaction patterns:** Adjust checking rules to match your framework's transaction controls (e.g., Spring `@Transactional`, Django `transaction.atomic()`).
