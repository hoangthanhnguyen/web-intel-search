# M8 - Debugging, Crash Triage & Vulnerability Reproduction

**Purpose:** Find the root cause of errors, crashes, failed exploits, or unexpected behavior, and
plan safe reproduction in an isolated lab. Emphasis on exact-error search, stack traces,
debuggers, sanitizers, fuzzing, and version pinning. Any run is a separate approved handoff.

**Triggers:** debug, gỡ lỗi, stack trace, backtrace, crash, core dump, segfault, reproduce,
reproduction, root cause, fuzzing, gdb, lldb, pwndbg, gef, WinDbg, ASan, UBSan, valgrind, strace,
ltrace, binary analysis, "why does this fail", "cannot reproduce", "works on my machine".

**Process contract:** Always run the Universal Search Loop (Section 1) and the Process Compliance
Gate (Section 0.5) and the action-approval rules (Section 0.6). This module supplies
sources and queries only; it does not authorize practical actions.

---

## Primary sources

### Error and issue search
- Exact error message in quotes; remove volatile parts (paths, IDs, timestamps).
- Stack Overflow, GitHub Issues, vendor bug trackers, mailing lists, Reddit.
- Official docs, changelogs, known-issues pages, release notes.
- Source code and tests (grep the error string in the repo).
- CVE/NVD/OSV for crashes that are security vulnerabilities.

### Debugging and instrumentation
| Tool | Use |
|---|---|
| **gdb + pwndbg/gef** | Linux userland debugging |
| **lldb** | macOS/LLVM |
| **WinDbg / x64dbg** | Windows |
| **rr** | Record/replay |
| **strace / ltrace** | Syscall/library tracing |
| **perf / bpftrace** | Performance and kernel tracing |
| **valgrind** | Memory errors |
| **ASan / UBSan / TSan / MSan** | Sanitizers |
| **AFL++ / libFuzzer / honggfuzz / OSS-Fuzz** | Fuzzing |
| **Ghidra / IDA / Binary Ninja / radare2/rizin** | Reverse engineering |
| **CyberChef** | Encoding/decoding and data transforms |
| **objdump / readelf / nm / strings** | Binary inspection |

### Reproduction sources
- Vendor advisory + patch diff (M11).
- Public PoC (M3) - review before running.
- Docker/VM images with the exact vulnerable version.
- Reproducible builds, lockfiles, and pinned dependencies.
- CTF/lab environments (M9).

---

## Query recipes

```text
# Exact error search
"Segmentation fault" "gdb" backtrace
"undefined reference to" "version"
"AttributeError" "module" "version"
"panic:" "runtime error" "go"
"Unhandled exception" "stack trace"
"error CS" "ASP.NET Core"
"ORA-12541" "TNS:no listener"

# Crash / vulnerability reproduction
"CVE-2024-3400" reproduce
"heap-buffer-overflow" ASAN
"use-after-free" "reproducer"
"stack overflow" "crash" "PoC"
"core dump" "gdb" "bt full"
"fuzzing" "crash" "AFL"

# Version and environment
"works on my machine" "version mismatch"
"cannot reproduce" "environment"
"dependency conflict" "lockfile"
"undefined behavior" "compiler"
"docker" "reproduce" "exact version"

# Source and issues
site:github.com inurl:issues "crash" "reproduce"
site:stackoverflow.com "error" "solution"
site:bugs.chromium.org "crash"
site:bugzilla.mozilla.org "crash"

# Binary analysis
"Ghidra" "decompile" "function"
"radare2" "analysis" "binary"
"pwndbg" "heap" "tcache"
```

---

## Workflow

1. **Capture the exact failure.** Full error message, stack trace, command, input, environment,
   version, and whether it is reproducible. Keep the raw capture local; it may contain secrets.
2. **Sanitize before any external search.** Remove credentials, tokens, cookies, connection
   strings, customer data, private hostnames, usernames, and proprietary code. Replace volatile
   values with placeholders while keeping the public technical structure (error code, function
   name, framework, version). Show the exact sanitized query and provider for approval (Section
   0.6) before sending. Search language, framework, and version.
3. **Check known issues and changelogs.** The bug may be fixed in a later version or documented as
   a limitation.
4. **Propose a minimal reproducer.** Reduce input, dependencies, and steps; pin versions. Building
   and running it is an approved practical handoff.
5. **Plan isolation.** Container/VM, snapshot, no production credentials, network restrictions.
6. **Plan instrumentation.** Logs, assertions, sanitizers, debugger, memory/state inspection. Do
   not attach to, modify, or run tools on a system without that approval.
7. **Form hypotheses for testing.** Change one variable at a time and record results.
8. **Identify the likely root cause** from source, patch, and evidence; distinguish code-inspected
   from reproduced.
9. **Map to a fix or mitigation.** Patch, config change, dependency upgrade, or workaround.
10. **Describe the regression test** and reproduction steps; writing/running it needs approval.

---

## What to extract

| Field | Example |
|---|---|
| Symptom | SIGSEGV in `parse_header` |
| Environment | Ubuntu 24.04, app 1.2.3, glibc 2.39 |
| Reproducer | input file + command |
| Root cause | integer overflow -> undersized allocation |
| Evidence | ASan report, gdb backtrace, patch diff |
| Fix | upgrade to 1.2.4 / bounds check |
| Detection | crash signature, log pattern |
| Evidence | code-inspected (patch + source) or reproduced if an approved run was observed |

---

## Pitfalls

- **Version mismatch.** "Latest" may not be the affected version.
- **Environment differences.** libc, compiler flags, CPU features, locale, timezone.
- **Non-determinism.** Race conditions, ASLR, heap layout, network timing.
- **Optimized builds.** Inlining and optimization hide frames; use debug builds.
- **Missing symbols.** Install debug symbols or build with `-g`.
- **Sanitizer false positives.** Verify with a second tool.
- **Unsafe PoCs.** Malicious or destructive samples; use a disposable VM.
- **Production debugging.** Attaching to production can cause outages; use staging.

---

## Ethics

- Debug and reproduce only on systems you own or are authorized to test.
- Use isolated labs; never attach debuggers to third-party production systems.
- Treat crash dumps and memory as sensitive; they may contain secrets or PII. Sanitize before
  sending anything to an external provider.
- Do not publish exploits for unpatched vulnerabilities; follow coordinated disclosure.
- Cleanup of artifacts and snapshots is an approved action; propose it rather than deleting silently.

---

## Output contract

Extends the shared contract in `SKILL.md` Section 13. Research output only; practical actions are a separate approved handoff (Section 0.6).

```
module:                 M8 Debugging
objective:              root-cause and reproduce <failure>
authorization:          authorized / lab / public-only / unknown
authorization_evidence: actual scope/policy source (module default: lab / owned system), or not established
approval:               current-task A/B/C approval used (scope + limits), or none
queries:                exact sanitized queries/dorks + engine + date (never invented)
sources_used:           error search, docs, issues, source, patch, mapped to the claim it supports
sources_read:           URLs opened but excluded, with reason (off-scope, stale, duplicate, low confidence, inaccessible, discovery only)
findings:               reproducer, backtrace, root cause, fix, regression test; label each claim source-reported / corroborated / code-inspected / reproduced with limits
pivots:                 M3 CVE, M11 patch, M9 lab
caveats:                environment/version differences, non-determinism, evidence limits
```
