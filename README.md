# web-intel-search

A router-driven web intelligence and security research skill for OpenClaw, Claude Code, Codex,
Cursor, and other Agent Skills-compatible agents. It finds information, selects the right domain
module, and enforces a strict search + verification process.

**Not an attack tool.** It is a search and research playbook for authorized security work,
defensive research, bug bounty, appsec, architecture, learning, and journalism.

## What it does

- Routes a request to 1 primary + 0-2 secondary modules based on keywords and intent.
- Runs the Universal Search Loop proportionally: quick for narrow facts, deep for disputed or
  consequential questions (declared, not guessed silently).
- Enforces a Process Compliance Gate before answering: scope, sources, queries, evidence labels,
  contradictions, and redaction.
- Keeps traceability: sanitized queries, sources used, sources read but excluded, and approvals.
- Separates research from action: credential validation, active testing, reproduction, installs,
  and changes need the explicit A/B/C approval in `SKILL.md` Section 0.6 (once, or bounded
  similar actions for the current task).
- Ships 12 domain packs and 6 core reference files.

## Research vs action

The skill is research-first. It can plan an action but does not perform one because a lab or an
engagement already exists. Approval options: **A** once, **B** bounded similar actions for this
task, **C** decline. A grant is limited by stated target, operation, environment, data class and
destination, privilege, rate/cost, impact, and expiry; any change requires a new approval.

## Modules

| # | Module | Use when the user asks about |
|---|---|---|
| M1 | Bug Bounty | Programs, scope, platforms, VDP, rewards, triage, disclosure |
| M2 | Writeups & Experience | Bug stories, case studies, "how I found", lessons learned |
| M3 | CVE & PoC | CVE records, affected versions, PoCs, exploits, patch diffs, KEV/EPSS |
| M4 | Cheatsheets & Payloads | Payloads, wordlists, commands, bypasses, privesc references |
| M5 | Red Team | ATT&CK, adversary emulation, C2 docs, OPSEC, purple team |
| M6 | AppSec & Secure Code | SAST/DAST/SCA, code review, secure SDLC, threat modeling, LLM/GenAI |
| M7 | Security Architecture | Zero trust, cloud security, reference architectures, controls |
| M8 | Debugging & Reproduction | Stack traces, crashes, root cause, fuzzing, lab reproduction |
| M9 | Labs, CTF & Training | PortSwigger, HTB, TryHackMe, VulnHub, certifications |
| M10 | Community & Research | Blogs, conferences, researchers, newsletters, social |
| M11 | Vendor Advisories | PSIRT bulletins, patches, changelogs, fixed versions |
| M12 | Methodology & Reporting | PTES, OWASP WSTG, scoping, RoE, reports, retests |

## Core references

- `references/engines.md` - operator cheat sheets and API endpoints.
- `references/dorks.md` - categorized Google/GitHub/Shodan/cloud/paste dorks.
- `references/secrets.md` - secret regex catalog and safe validation notes.
- `references/tooling.md` - CLI, APIs, automation, and safety checklist.
- `references/quick-recipes.md` - copy-paste query recipes and safety notes.
- `references/osint-targets.md` - target-specific OSINT source packs and meta-hubs.

## Install

### OpenClaw / ClawHub

```bash
# from ClawHub after publishing
openclaw skills install @<owner>/web-intel-search

# or copy a local checkout into your OpenClaw skills directory
cp -r web-intel-search ~/.openclaw/skills/
```

### Claude Code

```bash
cp -r web-intel-search ~/.claude/skills/
```

### opencode

```bash
mkdir -p ~/.config/opencode/skills
cp -r web-intel-search ~/.config/opencode/skills/
```

### Other agents

Copy the folder into your agent's skills directory. The entry point is `SKILL.md`; supporting
files are resolved relative to the skill directory.

## Publish to ClawHub

```bash
npm i -g clawhub
clawhub login
clawhub skill publish ./web-intel-search --dry-run
clawhub skill publish ./web-intel-search \
  --slug web-intel-search \
  --name "Web Intel Search" \
  --version 1.9.2 \
  --tags "security,osint,search,bug-bounty,red-team,appsec,architecture" \
  --changelog "v1.9.2: Tor/I2P access procedure, observation-vs-ownership rule, and lawful substitutes for illegal sources"
```

Before publishing, review the skill and its files. ClawHub runs automated security scans; keep
the content defensive and free of secrets.

## Usage examples

```text
"Tìm writeup bug bounty IDOR trên HackerOne"           -> M2 (+M1)
"CVE-2024-3400 có PoC chưa, affected versions nào?"   -> M3 (+M11)
"Cheatsheet leo thang đặc quyền Linux"                 -> M4 (+M9)
"Thiết kế zero trust cho AWS"                          -> M7 (+M6)
"Debug crash trong binary CTF"                         -> M8 (+M9)
"Red team technique cho initial access"                -> M5 (+M10)
"So sánh OWASP ASVS và NIST 800-53"                    -> M6 + M7 + M12
```

Every response states the module used, the research depth, the sanitized queries with engine/date,
sources with evidence labels, caveats, and any approval used (or none). A narrow lookup gets a
concise answer plus traceability; a deep assessment or engagement deliverable can use the full
report template. Both include two source lists: links used for synthesis and links read but not
used (with the exclusion reason).

## Output contract

```text
module:                 M# <name>
objective:              one sentence
depth:                  quick / deep (and why)
authorization:          authorized / lab / public-only / unknown
authorization_evidence: actual scope/policy source, or "not established"
approval:               current-task A/B/C grant used (scope + limits), or none
queries:                exact sanitized queries/dorks + engine + date
sources_used:           URL + title + date + evidence label + claim supported
sources_read:           URL + title + date + reason read but not used
findings:               claim + evidence label + limits
pivots:                 next identifiers or sources to check
caveats:                gaps, contradictions, stale/unsupported claims
```

Evidence labels: `source-reported`, `corroborated`, `code-inspected`, `reproduced`. "Reproduced"
requires your own observed run with environment/date; a source's claim stays source-reported.

If no source was read but excluded, write `None` explicitly. Discovery-only pages belong under
`sources_read` with reason `discovery only`; do not hide them.

## Process guarantee

The skill does not allow a module to bypass the process. The router selects sources and recipes;
the Universal Search Loop, the Evidence labels, and the Process Compliance Gate remain mandatory.
The gate fails if the traceability block omits either used sources or read-but-not-used sources,
or if a practical action/queries went out without the required approval.

## Safety

- Research and public-source work first; practical testing needs written scope and the separate
  action approval.
- No credential validation by default; no destructive actions; no exploit execution against third
  parties.
- Sanitize sensitive-derived queries and show them before sending.
- Redact secrets and PII; respect disclosure timelines and terms of service.
- Verify third-party skills and tools before installing them.

## License

MIT-0 (ClawHub default). See `LICENSE`.
