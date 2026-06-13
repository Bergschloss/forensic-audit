# 📋 File-by-File Forensic Audit Pipeline (Tasks Example)

This file demonstrates how to orchestrate a systematic, file-by-file `/forensic-audit` pipeline. 

> [!IMPORTANT]
> **Core Concept:** To prevent context bloat and ensure the deepest possible analysis of critical code paths, security rules, and database interfaces, the audit is split into individual file tasks. 
> 
> The main agent (orchestrator) coordinates the pipeline sequentially, spawning isolated subagents (with clean contexts) to inspect each file individually, logging the results, and finally merging them into a consolidated report.
>
> **Dual-Run for High-Risk Files:** For the highest-risk security/financial files (auth, payments, crypto/secrets), run the audit twice (`a`/`b`) with independent subagents (fresh context, no shared memory), then merge into `MERGED.md` — keeping the higher severity for shared findings and tagging unique ones `[Run A only]` / `[Run B only]`. Lower-risk infra/config files get a single pass.

---

## 🛠️ Pipeline Rules

1. **Orchestrator Role:** The main agent coordinates execution. It sequentially spawns dedicated subagents to run the `/forensic-audit` protocol on each target file (or file pair, for dual-run tasks) one-by-one.
2. **Isolation:** Each target file is processed in a clean, isolated subagent context to maximize precision and avoid context pollution. For dual-run tasks, `a` and `b` subagents must not share context.
3. **Execution Prompt:** For each active task, the orchestrator instructs the subagent to read the target file contents, run `/forensic-audit` on it, and output the report to:
   `[Output Path]\[NN]_[filename_slug].md` (single-pass), or `[Output Path]\[NN]_[filename_slug]\runA\` / `runB\` (dual-run).
4. **Merge Strategy (dual-run tasks):** Once `a` and `b` are complete, merge their reports into `MERGED.md`:
   - Combine shared findings (keep the higher severity of the two).
   - Keep unique findings marked as `[Run A only]` or `[Run B only]`.
5. **Progress Logging:** After each file task (or merge) is completed, append a line in `00_PROGRESS.md`:
   `[NN] | [File Path] | CRITICAL: X | HIGH: Y | MEDIUM: Z | DONE`
6. **Final Consolidation:** Once all files are audited (and dual-run tasks merged), the orchestrator compiles all reports/MERGED.md files into a single master document (`AUDIT_FINAL.md`), deduplicates findings, and sorts them by severity: `CRITICAL` ➔ `HIGH` ➔ `MEDIUM` ➔ `LOW`.

---

## 🎯 Example Target Files

*Replace these placeholders with the critical security, database, and infrastructure files identified during your initial scan. Mark the highest-risk files (auth, payments, secrets/crypto) for dual-run.*

### Output Directory
`Output path: C:\Users\Username\Desktop\_Reports\audit_run32\`

### Checklist

- [ ] **01a_auth-routes** | Path: `src/routes/auth.ts` ➔ `01_auth-routes\runA\`
- [ ] **01b_auth-routes** | (Same file as 01a) ➔ `01_auth-routes\runB\`
- [ ] **01m_auth-routes-merge** | Merge 01a and 01b ➔ `01_auth-routes\MERGED.md`

- [ ] **02_session-routes** | Path: `src/routes/sessions.ts` ➔ `02_sessions.md`

- [ ] **03a_stripe-integration** | Path: `src/routes/stripe.ts` ➔ `03_stripe-integration\runA\`
- [ ] **03b_stripe-integration** | (Same file as 03a) ➔ `03_stripe-integration\runB\`
- [ ] **03m_stripe-integration-merge** | Merge 03a and 03b ➔ `03_stripe-integration\MERGED.md`

- [ ] **04a_crypto-helper** | Path: `src/lib/crypto.ts` ➔ `04_crypto-helper\runA\`
- [ ] **04b_crypto-helper** | (Same file as 04a) ➔ `04_crypto-helper\runB\`
- [ ] **04m_crypto-helper-merge** | Merge 04a and 04b ➔ `04_crypto-helper\MERGED.md`

- [ ] **05_sse-handler** | Path: `src/lib/sse-tickets.ts` ➔ `05_sse.md`
- [ ] **06_db-schema** | Path: `prisma/schema.prisma` ➔ `06_schema.md`
- [ ] **07_nginx-config** | Path: `nginx/nginx.conf` ➔ `07_nginx.md`
- [ ] **08_docker-compose** | Path: `docker-compose.prod.yml` ➔ `08_docker.md`

- [ ] **99_final-merge** | Merge all individual reports (and MERGED.md files from dual-run tasks) into `AUDIT_FINAL.md`
