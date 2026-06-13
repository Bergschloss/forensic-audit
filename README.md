# 🛡️ Forensic Audit Skill

> **Professional Standard:** This framework represents a mature, systematic approach to automating quality control (QA), database integrity, and security auditing. It can be safely adopted as a standard auditing pipeline for any medium-to-large scale commercial projects.

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

## 📋 Orchestrating a File-by-File Forensic Audit (Multi-Agent Pipeline)

For complex backend APIs or infrastructure codebases, doing a single repository scan for security flaws can miss subtle bugs (like a missing `await` before a database transaction or a missing TOTP replay guard). 

To ensure maximum focus and precision, we use a **File-by-File Forensic Audit Pipeline**. The template is provided in [tasks-forensic-example.md](file:///g:/Anti/forensic-audit_repo/tasks-forensic-example.md).

### How to use this strategy:

1. **Ask the Agent to Scan & List:** 
   Feed the codebase to your AI assistant and ask it to scan the project files, identify all critical security endpoints, database services, schemas, and configurations, and generate a task list formatted like `tasks-forensic-example.md`.
2. **Sequential Task Execution:**
   Instruct the orchestrator agent to execute this list **one-by-one**. For each file task, the main agent launches a subagent (or a fresh clean session) to analyze that file. This guarantees that the inspecting agent is completely focused on a single module without prior context pollution.
3. **File-Specific Reports:**
   Each subagent reads its assigned file, runs the `/forensic-audit` protocol, and writes its findings directly to a separate report `[NN]_[filename].md`.
4. **Progress Logging:**
   The orchestrator updates `00_PROGRESS.md` with finding counts (`CRITICAL`/`HIGH`/`MEDIUM`/`LOW`) after each subagent completes its task.
5. **Consolidate to Master Report:**
   Run a final task where the orchestrator merges all individual file reports, deduplicates overlapping issues, and outputs a consolidated `AUDIT_FINAL.md` sorted by severity.

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
