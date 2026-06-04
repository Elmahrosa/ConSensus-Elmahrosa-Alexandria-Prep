# BOB — Security Architecture, Backup Architecture & Documentation Structure
**ConSensus Elmahrosa Alexandria Preparation — ELMAHROSA OS v2.0**

**Agent:** @BOB (Chief Security & Knowledge Officer)  
**Standards Enforced:** Zero Trust, encryption at rest + in transit, principle of least privilege, immutable backups.  
**KPI Focus:** Recovery readiness (RTO/RPO targets), documentation coverage (>90% for new systems), security compliance (no critical findings pre-launch).

---

## 1. Security Architecture

**Zero Trust Model for ELMAHROSA OS + ConSensus Prep:**
- Never trust, always verify. Every agent action, every dashboard access, every API call, every git push is authenticated and authorized.
- Assume breach: segment networks/services, encrypt everything, short-lived credentials, continuous monitoring/audit.

**Layers & Controls:**

**Identity & Access (Least Privilege)**
- Founder (KING_TEOS) = root / god-mode (approvals, key ceremonies).
- NEXUS = delegated executive (priority, roadmap, resource, escalation). Can approve but not mint root keys.
- Agents (@TARA etc.): Tool access only (web search, write to specific memory paths, no secret access). Run in sandboxed processes where possible.
- Human collaborators: Per-repo GitHub roles (read for most, write for assigned modules). No prod secrets.
- Service accounts: One per component (mission-control-api, vc-issuer, event-reg). Scoped tokens.

**Authentication**
- v0.1: Strong passphrase + TOTP for Mission Control (streamlit-auth or similar). Rotate quarterly.
- GitHub: SSO + 2FA enforced org-wide. No PATs with broad scopes; use fine-grained.
- Future: OIDC + hardware keys (Yubikey) for founder/NEXUS. Passwordless where possible.

**Authorization (RBAC + ABAC)**
- Mission Control pages: NEXUS sees all; SPIKE sees engineering + risks; SARA sees only outreach CRM; BOB sees audit + archives.
- Data: Memory layers partitioned. TARA writes intel layer only; cannot touch projects without NEXUS task.
- TEOS key management: Issuer keys ceremony witnessed (multi-party if possible). Never single person. Hot keys for testnet only; cold for mainnet.

**Data Protection**
- Encryption at rest: All DBs (Postgres) with disk encryption (LUKS or cloud KMS). Application-level where sensitive (VC claims, partner notes).
- Encryption in transit: TLS 1.3 everywhere (public site + internal mTLS for services). Git over SSH.
- Secrets: Never in git, env, or images. Use:
  - Doppler / Infisical / 1Password CLI for dev/CI.
  - Kubernetes secrets + external-secrets-operator or Vault in prod.
  - BOB audits every new secret addition.
- PII / Partner data: Minimized. Store only what's needed for outreach (name, role, org, public email, notes). Delete on request or after 2 years inactivity. Mark as "sovereign data" in governance.

**Network & Infrastructure**
- Public site: Cloudflare (DDoS, WAF) + origin shield if possible. Or sovereign CDN.
- Internal services: Private network or tailscale/Headscale for zero-trust overlay. No public exposure of Mission Control or VC admin endpoints.
- TEOS nodes: Validator keys offline for signing where possible; sentry architecture.
- CI/CD runners: Ephemeral, no persistent secrets beyond job. OIDC federation to cloud if used.
- Supply chain: Dependabot + OSV scanner + SLSA provenance for builds (future).

**Logging, Monitoring, Incident Response**
- All agent decisions, tool calls, state changes, approvals logged with timestamp + actor + hash chain (for BOB immutability).
- Central log: Loki or Postgres audit table + git commits as source of truth.
- Alerts: Critical (key access, failed auth >N, roadmap change without approval) → NEXUS + founder pager (simple email/SMS first).
- IR plan: 1. Contain (revoke tokens), 2. Assess (BOB + SPIKE), 3. Notify (NEXUS/founder), 4. Recover + post-mortem in archive. No public disclosure without legal.

**Compliance & Sovereignty**
- Align to Egypt data protection law + emerging DPI safeguards (DIAL, UN, G20).
- All repos: TESL or equivalent sovereign license (see Elmahrosa-Core).
- Audit: BOB runs quarterly repo + infra audit. Pre-event full security review.
- Never expose: As per OS policy — no creds, keys, tokens in any output or file committed.

**Current Risks (pre-mitigation) & Mitigations (this prep):**
- Broad GitHub PATs or leaked keys in sessions: BOB to rotate/audit all current PATs this week. Use fine-grained + expiration.
- Mission Control v0.1 weak auth: Hardcode strong dev password + note "replace before any real partner data".
- Shared memory in git: .gitignore for any local secret files; BOB pre-commit hook recommendation.
- Live demo VC keys: Use throwaway test keys only. Never mainnet keys at event.

---

## 2. Backup Architecture (Immutable, Recoverable, Sovereign)

**Principles:** 3-2-1-1-0 (3 copies, 2 media, 1 offsite, 1 immutable/airgapped, 0 errors on restore test). RPO <1h for critical (memory, CRM, keys metadata), RTO <4h for event ops.

**What to Backup:**
- All source (git — primary is GitHub + local mirrors).
- Shared memory layers (Postgres dumps + JSON exports committed to git + BOB archive).
- Event registrations / CRM data (DB + exports).
- TEOS chain state (snapshots + validator keys — keys never in backup, only encrypted metadata).
- Docs + archives (this prep folder + all Elmahrosa repos).
- Config (infra as code, but no secrets).

**Tools & Flows (v0.1 → v1):**
- Git: GitHub (primary) + local bare clones on multiple machines + BOB offline USB/hard drive (immutable snapshots monthly).
- Postgres: pg_dump daily (cron in docker) → encrypted (age or gpg with founder + NEXUS pubkeys) → object storage (self-hosted MinIO or sovereign S3-compatible in Egypt DC) + airgapped copy.
- Files/docs: rsync or restic to encrypted repo (restic is excellent for immutability + dedup). Same targets.
- TEOS: Native snapshot + export; test restore on separate node quarterly.
- Secrets: 1Password or Vault export (encrypted) to airgap only; never automated.
- Verification: BOB runs monthly restore drill on a clean VM. Document RTO/RPO achieved. Log in archive.

**Storage Locations (Sovereign preference):**
1. Primary: GitHub + prod DB disk.
2. Hot copy: Local dev machines (encrypted volumes) + VPS snapshot.
3. Offsite/immutable: Restic/MinIO in secondary Egyptian or EU region DC (or friend's basement with encrypted drive). Monthly WORM-like (append-only bucket policy if supported).
4. Cold/airgap: Quarterly full encrypted tar to external HDD kept by founder or trusted offline location. Never connected except for restore test.

**Retention:**
- Daily: 14 days
- Weekly: 3 months
- Monthly immutable: 1 year+
- Event-specific: Permanent + on-chain hash of final archive.

**Recovery Playbook (high-level):**
1. NEXUS declares incident or scheduled restore.
2. BOB + SPIKE pull latest git + latest encrypted backup.
3. Decrypt (multi-person if keys split).
4. Restore DB + files.
5. Verify hashes + sample data.
6. Re-deploy services.
7. Post-incident: Update this doc + archive report.

**Current Gaps:** No automated restic yet for this prep. BOB will stand up basic script in Sprint 0.

---

## 3. Documentation Structure (Knowledge Management)

**Canonical Location for this Prep:** `elmahrosa-repos/ConSensus-Elmahrosa-Alexandria-Prep/`

**Top-Level Structure (enforced by BOB):**
```
ConSensus-Elmahrosa-Alexandria-Prep/
├── README.md                 # Executive overview + how to navigate + NEXUS status
├── docs/
│   ├── TARA-10-DPI-Partners-Africa-MENA.md
│   ├── SARA-Outreach-Strategy.md
│   ├── JUNE-Keynote-Press-X-Content.md
│   ├── RAJA-14Day-Campaign-Strategy.md
│   ├── NEXUS-Master-Roadmap.md (to be written by NEXUS)
│   └── memory/               # Layer exports (founder-profile.json, elmahrosa-kb.md, teos-kb.md, active-projects.json)
├── architecture/
│   ├── SPIKE-System-Architecture.md
│   ├── ADRs/                 # Architecture Decision Records (001-*.md)
│   └── diagrams/             # .mmd + exported png/svg
├── campaign/
│   ├── assets/               # Images, copy variants
│   └── RAJA-14Day-Campaign-Strategy.md
├── security/
│   ├── BOB-Security-Backup-Doc-Structure.md  (this)
│   ├── audit-logs/           # Timestamped findings
│   └── compliance/           # Checklists, DPI safeguard mappings
├── reports/
│   └── NEXUS-Executive-Dashboard-*.md
├── scripts/                  # BOB backup/restore, audit, memory sync helpers (python or sh)
└── .gitignore                # Never commit secrets, local DBs, .env
```

**Global / OS-wide (for future BOB maintenance):**
- Elmahrosa-Core or new `elmahrosa-os` repo: 
  - AGENTS.md or ELMAHROSA_OS.md (the prompt itself — already created at ~ /ELMAHROSA_OS_v2.0.md and skill)
  - Runbooks/
  - Memory layers (versioned)
  - Security policy + incident response
  - Backup manifests + restore tests log

**Documentation Standards (BOB):**
- Every major artifact gets owner, date, version, status (Draft/Approved/Superseded).
- Use markdown with clear headings, tables for lists/KPIs, mermaid for diagrams.
- Update memory layers at end of every significant session (NEXUS responsibility, BOB enforces).
- Pre-commit or CI check: no TODOs without owner/deadline in new docs; links must resolve.
- Archive: Old versions moved to /archive/ with date. Never delete history.

**Coverage Target for this prep:** 100% of agent outputs (TARA/SARA/JUNE/RAJA/SPIKE/BOB/NEXUS) documented here. 100% of new code has inline comments + ADR if architectural.

**Access:** Public for event docs (under sovereign license). Internal for security, raw CRM, keys metadata (git private or access-controlled).

**Maintenance Cadence:** BOB weekly sweep during active prep. Full audit before event.

**Handoffs/Dependencies:**
- All agents: Commit outputs to this structure (or subdirs) using clear filenames. Never scatter.
- SPIKE: Add scripts/ for memory export/import + backup automation.
- NEXUS: Approve this structure + any exceptions.

**Recovery Readiness Current (self-assessed):** 
- Git: Strong (multiple clones + GitHub).
- Docs: This init run is being captured here — good.
- DBs: Not yet stood up — target RPO 1h once Postgres live (Sprint 0-1).
- Test restore: Scheduled for end of Sprint 1 by BOB.

**Next BOB actions:**
- Create the full dir skeleton + README + .gitignore (partially done via prior).
- Add pre-commit or simple lint for no-secrets.
- Stand up basic restic or rsync + age encryption script.
- Import the TARA/SARA/etc. docs we just produced into memory layer exports if structured.

**Status:** Security + backup + doc structure defined and partially implemented. System is sovereign by design from day one. No critical exposures in current artifacts.

All future commands or code changes in this initiative must pass BOB review gate (or explicit NEXUS waiver logged).

This keeps the OS honest.
