# 📋 File-by-File Forensic Audit Pipeline (Tasks Example)

This file demonstrates how to orchestrate a systematic, file-by-file `/forensic-audit` pipeline. 

> [!IMPORTANT]
> **Core Concept:** To prevent context bloat and ensure the deepest possible analysis of critical code paths, security rules, and database interfaces, the audit is split into individual file tasks. 
> 
> The main agent (orchestrator) coordinates the pipeline sequentially, spawning isolated subagents (with clean contexts) to inspect each file individually, logging the results, and finally merging them into a consolidated report.

---

## 🛠️ Pipeline Rules

1. **Orchestrator Role:** The main agent coordinates execution. It sequentially spawns dedicated subagents to run the `/forensic-audit` protocol on each target file one-by-one.
2. **Isolation:** Each target file is processed in a clean, isolated subagent context to maximize precision and avoid context pollution.
3. **Execution Prompt:** For each active task, the orchestrator instructs the subagent to read the target file contents, run `/forensic-audit` on it, and output the report to:
   `[Output Path]\[NN]_[filename_slug].md`
4. **Progress Logging:** After each file task is completed, append a line in `00_PROGRESS.md`:
   `[NN] | [File Path] | CRITICAL: X | HIGH: Y | MEDIUM: Z | DONE`
5. **Final Consolidation:** Once all files are audited, the orchestrator compiles all separate reports into a single master document (`AUDIT_FINAL.md`), deduplicates findings, and sorts them by severity: `CRITICAL` ➔ `HIGH` ➔ `MEDIUM` ➔ `LOW`.

---

## 🎯 Example Target Files

*Replace these placeholders with the critical security, database, and infrastructure files identified during your initial scan.*

### Output Directory
`Output path: C:\Users\Username\Desktop\_Reports\audit_run32\`

### Checklist

- [ ] **01_auth-routes** | Path: `src/routes/auth.ts` ➔ `01_auth.md`
- [ ] **02_session-routes** | Path: `src/routes/sessions.ts` ➔ `02_sessions.md`
- [ ] **03_stripe-integration** | Path: `src/routes/stripe.ts` ➔ `03_stripe.md`
- [ ] **04_crypto-helper** | Path: `src/lib/crypto.ts` ➔ `04_crypto.md`
- [ ] **05_sse-handler** | Path: `src/lib/sse-tickets.ts` ➔ `05_sse.md`
- [ ] **06_db-schema** | Path: `prisma/schema.prisma` ➔ `06_schema.md`
- [ ] **07_nginx-config** | Path: `nginx/nginx.conf` ➔ `07_nginx.md`
- [ ] **08_docker-compose** | Path: `docker-compose.prod.yml` ➔ `08_docker.md`

- [ ] **99_final-merge** | Merge all individual markdown reports into `AUDIT_FINAL.md`
