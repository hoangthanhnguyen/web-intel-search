# Secret Pattern Catalog

Use for defensive review, authorized assessments, and leak triage. Discovery and validation are
separate steps: **detect by default, do not validate**. Never use a discovered secret to access data.
A read-only GET can still return customer, financial, or account data, so "read-only" does not make
it a safe identity check. Any validation requires the separate A/B/C approval and written scope in
`SKILL.md` Section 0.6. Redact values in all reports (first/last 4 chars only).

## Provider tokens

| Provider | Pattern | Notes |
|---|---|---|
| AWS Access Key ID | `AKIA[0-9A-Z]{16}` / `ASIA[0-9A-Z]{16}` | Long-term / temporary |
| AWS Secret Key | `(?i)aws_secret_access_key\s*[=:]\s*[A-Za-z0-9/+=]{40}` | 40 chars |
| AWS Session Token | `(?i)aws_session_token\s*[=:]\s*[A-Za-z0-9/+=]{100,}` | Temporary |
| GCP API Key | `AIza[0-9A-Za-z\-_]{35}` | 39 chars |
| GCP Service Account | `"type"\s*:\s*"service_account"` | JSON key file |
| GitHub PAT (classic) | `ghp_[0-9A-Za-z]{36}` | |
| GitHub PAT (fine-grained) | `github_pat_[0-9A-Za-z_]{82}` | |
| GitHub OAuth/App | `gh[ousr]_[0-9A-Za-z]{36}` | |
| GitLab PAT | `glpat-[0-9A-Za-z\-_]{20}` | |
| Slack token | `xox[baprs]-[0-9A-Za-z\-]{10,}` | |
| Slack webhook | `https://hooks\.slack\.com/services/T[0-9A-Z]+/B[0-9A-Z]+/[0-9A-Za-z]+` | |
| Stripe live | `(sk|rk)_live_[0-9A-Za-z]{20,}` | High impact |
| Stripe test | `(sk|rk)_test_[0-9A-Za-z]{20,}` | Lower impact |
| Twilio | `SK[0-9a-fA-F]{32}` / `AC[0-9a-fA-F]{32}` | API key / account SID |
| SendGrid | `SG\.[0-9A-Za-z\-_]{22}\.[0-9A-Za-z\-_]{43}` | |
| Mailgun | `key-[0-9a-zA-Z]{32}` | |
| OpenAI | `sk-(proj-)?[A-Za-z0-9\-_]{20,}` | |
| Anthropic | `sk-ant-[A-Za-z0-9\-_]{20,}` | |
| Hugging Face | `hf_[A-Za-z0-9]{30,}` | |
| npm | `npm_[A-Za-z0-9]{36}` | |
| PyPI | `pypi-AgEIcHlwaS5vcmc[A-Za-z0-9\-_]{50,}` | |
| Docker Hub | `dckr_pat_[A-Za-z0-9\-_]{20,}` | |
| DigitalOcean | `do[op]_v1_[a-f0-9]{64}` | |
| Heroku | `(?i)heroku[0-9a-f]{32}` | |
| Cloudflare API Token | `[A-Za-z0-9\-_]{40}` | Too generic; confirm via context |
| JWT | `eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+` | Decode payload, never trust |
| Telegram Bot | `[0-9]{8,10}:AA[0-9A-Za-z\-_]{33}` | |
| Discord Webhook | `https://discord(app)?\.com/api/webhooks/[0-9]+/[A-Za-z0-9\-_]+` | |
| Discord Bot | `[MN][A-Za-z\d]{23}\.[\w-]{6}\.[\w-]{27}` | |
| Google OAuth Client | `[0-9]+-[0-9A-Za-z_]{32}\.apps\.googleusercontent\.com` | |
| Facebook Token | `EAACEdEose0cBA[0-9A-Za-z]+` | |
| Twitter Bearer | `AAAAAAAAAAAAAAAAAAAAA[0-9A-Za-z%]{40,}` | |
| Shopify | `shpat_[0-9a-fA-F]{32}` / `shpca_` / `shppa_` | |
| Atlassian | `ATATT[0-9A-Za-z\-_]{20,}` | |
| Datadog | `[0-9a-f]{32}` with `api_key`/`app_key` context | Context needed |
| Terraform Cloud | `[A-Za-z0-9]{14}\.atlasv1\.[A-Za-z0-9\-_]{60,}` | |
| Pulumi | `pul-[a-f0-9]{40}` | |
| Vault Token | `hvs\.[A-Za-z0-9\-_]{20,}` / `s\.[A-Za-z0-9]{24}` | |
| Supabase | `sbp_[a-f0-9]{40}` | |
| Vercel | `vercel_[A-Za-z0-9]{20,}` | |
| Netlify | `nfp_[A-Za-z0-9]{40}` | |
| Postman | `PMAK-[A-Za-z0-9\-_]{50,}` | |

## Keys and certificates

```
-----BEGIN (RSA |EC |DSA |OPENSSH |PGP )?PRIVATE KEY( BLOCK)?-----
-----BEGIN CERTIFICATE-----
-----BEGIN ENCRYPTED PRIVATE KEY-----
ssh-rsa AAAA[0-9A-Za-z+/]+
ssh-ed25519 AAAA[0-9A-Za-z+/]+
```

## Connection strings

```
(mongodb|mongodb\+srv|mysql|postgres|postgresql|redis|rediss|amqp|amqps|mssql|oracle|jdbc):\/\/[^\s"'<>]+
https?://[^/\s:@]+:[^/\s:@]+@[^/\s]+
(?i)(smtp|imap|pop3)://[^\s"'<>]+
(?i)(jdbc:mysql|jdbc:postgresql|jdbc:oracle|jdbc:sqlserver)://[^\s"'<>]+
```

## Generic assignments (triage with context)

```
(?i)(password|passwd|pwd|pass)\s*[=:]\s*["']?[^\s"',;]{6,}
(?i)(secret|secret_key|client_secret|app_secret)\s*[=:]\s*["']?[^\s"',;]{6,}
(?i)(api[_-]?key|apikey|access[_-]?key)\s*[=:]\s*["']?[^\s"',;]{8,}
(?i)(auth[_-]?token|access[_-]?token|refresh[_-]?token|bearer)\s*[=:]\s*["']?[^\s"',;]{8,}
(?i)(private[_-]?key)\s*[=:]\s*["']?[^\s"',;]{8,}
(?i)(aws_access_key_id|aws_secret_access_key|aws_session_token)\s*[=:]\s*["']?[^\s"',;]{8,}
```

## High-entropy detection

Regex alone misses unknown providers. Score candidate strings:
- Length >= 20, mixed case + digits, or base64/hex charset.
- Shannon entropy > 4.0 (base64), > 3.0 (hex).
- Surrounding context contains assignment, `token`, `key`, `secret`, `auth`, `Bearer`.
- Exclude hashes/IDs that match known formats (UUID, MD5, SHA1, SHA256, commit SHAs, asset hashes).
- Deduplicate by SHA-256 of the value; aggregate by file/repo.

## Candidate validation (separate approval; narrow checks only)

Default: report "not validated". Consider a check only if it changes a decision and written scope
exists. Before proposing one, name the provider/account, exact endpoint, response fields, and side
effects, then use the A/B/C flow in `SKILL.md` Section 0.6.

| Provider | Candidate check (verify current docs) | Important caution |
|---|---|---|
| AWS | `sts get-caller-identity` | Confirms caller identity; never list/read resources |
| GitHub | `GET /user` | Check token scopes; no repository content |
| GitLab | `GET /api/v4/user` | |
| Slack | `auth.test` | |
| OpenAI | `GET /v1/models` | Metadata only; do not invoke models |
| Anthropic | `GET /v1/models` | Metadata only |
| npm | `GET /-/whoami` | |
| PyPI | token info endpoint | |
| Stripe | No data-free check in this catalog | `/v1/balance` returns account balance; `/v1/charges` returns customer charges. Treat both as data access, not identity |
| SendGrid | `GET /v3/user/account` | Returns account details |
| Datadog | `GET /api/v1/validate` | |

Rules:
- Only with written scope **and** the specific action approval; never reuse a grant beyond its limits.
- Never pivot from a key to customer data, billing actions, or destructive operations.
- If authorization or a data-free check is ambiguous, report without validation.
- Stop on unexpected data. Redact; recommend rotation; notify the owner through the agreed channel.
- Never place the raw secret in an agent answer, log, query, or file.

## Scanner commands

Local scanning is a practical step; run it only with approval, in an isolated workspace, and do not
upload findings. TruffleHog verifies credentials through provider APIs by default — disable that
unless validation is separately approved. Keep `--results` so unverified detections are still shown.

```bash
gitleaks detect --source . --report-format json --report-path gitleaks.json
trufflehog filesystem . --no-verification --results=verified,unverified,unknown --json
trufflehog git file://repo --no-verification --results=verified,unverified,unknown --json
git log -p --all | grep -Ei 'api[_-]?key|secret|password|token'
```
Scan output contains live secrets in plaintext. Treat it as sensitive: restrict permissions, do not
paste it into chat, and delete it after the approved task when the retention rule allows.
