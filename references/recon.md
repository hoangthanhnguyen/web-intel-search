# Recon Playbook

Recon means building a verified picture of a subject before deciding or acting. The subject is
whatever the question names: a domain, an IP range, an organization, a person, an email, a phone
number, an image, a repository, a cloud asset, or a threat actor. Pick the pack by question, not by
tool you happen to have.

**Boundary.** Passive collection from public indexes is research. Active probing (DNS brute force,
port scan, directory fuzzing, authenticated API calls, login attempts), credential validation, and
any change to a system are practical steps requiring written authorization and the A/B/C approval in
`SKILL.md` Section 0.6. Existing engagement scope is not approval to act now.

---

## 1. Subject selector

| Question sounds like | Subject pack | Start with |
|---|---|---|
| "What does this company expose?" | Domain / website, IP / ASN | DNS, RDAP, CT logs |
| "Who owns this IP / what else is on it?" | IP / ASN / network | RDAP, BGP, passive DNS |
| "What is this company's structure and risk?" | Organization | registry, filings, sanctions |
| "Who is this person / username?" | Person / username | identifier graph, accounts |
| "Is this email or phone real / exposed?" | Email / phone | MX/SMTP, breach indexes |
| "Where was this photo taken / who posted it?" | Image / video | EXIF, reverse image, geolocation |
| "What is leaked in this repo or org?" | Code / repository | code search, secret patterns |
| "What cloud assets belong to them?" | Cloud / external asset | buckets, mobile apps, API hosts |
| "Who is this actor / campaign?" | Threat actor | ATT&CK, vendor reporting, IOCs |
| "What happened at this event / place?" | Physical / event | archives, maps, ships/flights |

If the question names more than one subject, run one pack at a time and record how identifiers
connect. Do not blend subjects into one conclusion without evidence of a link.

---

## 2. Scope manifest

Write this before collecting. For passive-only work, authorization is "public sources"; the rest
still applies.

```
question:        one sentence, the decision it supports
subject:         domain / IP / org / person / email / phone / image / repo / asset / actor
authorization:   public-only / engagement scope (cite) / lab / unknown
targets:         exact identifiers and boundaries
exclusions:      subsidiaries, third parties, personal accounts, shared hosting
data classes:    what you may collect and what you must not retain
active actions:  none / list each (needs 0.6 approval)
rate/volume:     queries per minute, max pages, time budget
stop conditions: illegal content, personal data beyond need, scope ambiguity, unexpected access
retention:       where evidence lives, how long, who can read it
```

If authorization is unknown, stay passive and say so in the answer.

---

## 3. Universal loop

1. **Seed.** Start from the identifiers in the question: domain, IP, org name, username, email,
   phone, image hash, repo URL, bucket name, actor alias.
2. **Collect passively.** Public records, DNS, certificates, archives, search indexes, code search,
   vendor reporting. Record URL, date, and what it proves.
3. **Pivot on identifiers.** Each hit is a new seed. Keep a graph of nodes (identifier) and edges
   (evidence of a link), not a flat list.
4. **Verify links.** A shared certificate, registrant email, analytics ID, favicon hash, avatar
   hash, or ASN can link assets; shared hosting and CDNs produce false links. Require two
   independent signals before treating a link as real.
5. **Disambiguate.** Separate same-name people, brands, subsidiaries, and lookalike domains.
6. **Inventory.** Normalize into the schema in Section 7.
7. **Assess gaps.** State what could not be confirmed and what would change the conclusion.
8. **Report.** Answer first, then evidence with labels, then limits and next steps.

---

## 4. Subject packs

Each pack lists what to collect; detailed source lists are in `references/osint-targets.md`.

### 4.1 Domain / website
- Registration: RDAP/WHOIS, creation/expiry, registrar, registrant org (often redacted), historical
  WHOIS.
- DNS: A/AAAA, MX, NS, TXT (SPF, DMARC, verification tokens), CNAME, SOA; DNSSEC.
- Certificates: CT logs (crt.sh), SANs, issuer, first/last seen; pivot to sibling hosts.
- Subdomains: passive DNS, CT, archives; active brute force only with approval.
- Content: tech fingerprint, headers, favicon hash, robots.txt, sitemap, security.txt.
- History: Wayback CDX, archive.today, Common Crawl; removed pages and old subdomains.
- Exposure: indexed documents, admin panels, cloud buckets, pasted URLs.
- Reputation: Safe Browsing, URLhaus, PhishTank, PhishStats.

### 4.2 IP / ASN / network
- Ownership: RDAP, BGP, PeeringDB, ASN org, geolocation (treat as approximate).
- Reverse DNS and passive DNS; certificates presented on the IP.
- Exposure: Shodan/Censys/FOFA/ZoomEye services, banners, versions (banners can lie).
- Neighbors: same /24, same ASN, same certificate, same favicon. Exclude shared hosting and CDN
  ranges before attributing.
- Reputation: AbuseIPDB, GreyNoise, OTX, ThreatFox, Spamhaus.
- Open services: record, do not connect or authenticate.

### 4.3 Organization
- Legal identity: registry ID, jurisdiction, status, officers, filings.
- Brands and domains: trademark, app stores, official social accounts; verify against the registry
  record, not the logo.
- Corporate tree: subsidiaries, acquisitions, former names, joint ventures.
- Infrastructure: domains, ASNs, certificates, buckets, mobile apps, API hosts.
- People: public leadership and technical staff from official pages, filings, conference talks.
- Risk: sanctions/PEP (OpenSanctions), litigation, adverse media, breach mentions.
- Beneficial ownership: national registries, Open Ownership guidance; do not infer ownership from
  a shared address alone.

### 4.4 Person / username
- Identifiers: name variants, usernames, emails, phones, avatar hash, bio phrases, writing style.
- Accounts: WhatsMyName/Sherlock/Maigret sweep, then manual confirmation. A profile hit is a lead,
  not identity.
- Public records: company filings, court records, sanctions, conference bios, patents.
- Correlation: reused avatar, linked domains, timezone/activity, mutual references. Require
  multiple independent signals.
- Disambiguation: common names, transliterations, namesakes, impersonators. Never merge two
  people on a name match.
- Privacy: collect only what the lawful objective needs; no private accounts, no doxxing, no
  harassment, no publishing personal data. Record legal basis in the scope manifest.

### 4.5 Email / phone
- Email: syntax, MX, SMTP existence only where permitted (no spam, minimal probes), catch-all,
  Gravatar, PGP keyserver, breach exposure through HIBP or licensed providers.
- Phone: E.164 normalization, carrier/line type, public records, messaging presence (checks may
  notify the subject).
- Never use breach data or a discovered credential; report exposure, not dumps.

### 4.6 Image / video / geolocation
- Metadata: EXIF, GPS, device, timestamps, edit history; screenshots strip metadata.
- Reverse image: Google Lens, Yandex, Bing Visual Search, TinEye, Baidu; PimEyes only within legal
  and ethical limits.
- Geolocation: signage, plates, bollards, vegetation, architecture, shadows and sun angle
  (SunCalc), Street View, Mapillary, Overpass.
- Video: frame extraction, OCR/subtitles, audio fingerprint, keyframe metadata.
- Preserve the original file and hash; note that re-encoding changes it.

### 4.7 Code / repository
- Search public code (current GitHub syntax: `path:`, `content:`, `is:`), GitLab, Gitee,
  Sourcegraph, grep.app.
- Look for internal hostnames, tokens, CI config, IaC, package names, commit identities.
- Secret handling: detect and contextualize, never validate by default (see `secrets.md`).
- History: deleted files and old commits; forks and archived repos.
- Local clone/scan is a practical step; isolate and protect output.

### 4.8 Cloud / external asset
- Buckets: name patterns, public listing, permissions; do not download content.
- Mobile apps: store listings, permissions, embedded endpoints, certificate pinning.
- API hosts: docs, OpenAPI, error formats, versioning; do not fuzz without approval.
- Third parties: CDN, SaaS, status pages, support portals; separate owned from vendor-hosted.

### 4.9 Threat actor / campaign
- Naming: vendor aliases, MITRE groups, shared tooling, infrastructure clusters.
- Reporting: vendor research, CERT advisories, DFIR reports; note publication date and whether it
  is primary telemetry.
- TTPs: ATT&CK techniques, procedures, detection opportunities.
- Indicators: type, value, first/last seen, context, TTL; see `modules/10-community-conferences-research.md`
  threat-intelligence workflow.
- Attribution: present competing hypotheses and the evidence that would falsify each; do not state
  attribution as fact from one vendor blog.

### 4.10 Physical / event
- Flights: ICAO hex vs tail number, ADS-B history.
- Ships: IMO vs MMSI, port calls, AIS gaps (gaps are not proof of wrongdoing).
- Places: OpenStreetMap, Overpass, historical imagery, Street View, Mapillary.
- Events: official schedules, archives, local reporting; verify time zones.

---

## 5. Identifier graph

Keep nodes and edges:

```
node:  type=domain value=example.com first_seen=... source=...
edge:  from=example.com to=203.0.113.10 relation=resolves_to
       strength=strong evidence=DNS A record date=...
```

Link strength:
- **Strong:** direct DNS/CT record, registry filing, official page, signed document.
- **Medium:** same registrant email, same analytics ID, same favicon, same certificate.
- **Weak:** same ASN, same hosting provider, similar name, shared address. Weak links never justify
  attribution alone.

Store the graph in the answer or a file only with approval; no silent persistence.

---

## 6. Disambiguation checklist

- [ ] Two identifiers share more than a name (independent signal).
- [ ] The legal entity matches registry ID, not just brand.
- [ ] The domain is not a lookalike, parked, or expired.
- [ ] The person is not a namesake; check location, employer, and time period.
- [ ] The social account is not an impersonator; check creation date and cross-links.
- [ ] The IP is not shared hosting/CDN before attributing neighbors.
- [ ] The image is not a repost or AI-generated; check source and metadata.
- [ ] The evidence date matches the claim date.

If any box fails, the link stays a lead, not a fact.

---

## 7. Asset inventory schema

```json
{
  "subject": "example.com",
  "kind": "domain",
  "value": "api.example.com",
  "state": "resolved",
  "first_seen": "2026-11-12",
  "source": "crt.sh",
  "evidence": "source-reported",
  "attributes": {"ip": "203.0.113.10", "tech": ["nginx"], "tls_issuer": "..."},
  "links": [{"to": "203.0.113.10", "relation": "resolves_to", "strength": "strong"}],
  "notes": "not tested; no active probe",
  "owner_guess": null
}
```

Never fill `owner_guess` from a weak link. Leave it null and say why.

---

## 8. Evidence and confidence

Use the labels in `SKILL.md` Section 9.1. For recon specifically:
- A registry/CT/DNS record you read is **source-reported** or **code-inspected** (for a raw record),
  not "confirmed ownership".
- Two independent records agreeing is **corroborated**.
- Only your own approved active test is **reproduced**, with environment and date.
- State the fetch date; DNS and certificates change.

---

## 9. Active recon (approval required)

Only after written scope and the Section 0.6 approval:
- DNS brute force/subdomain enumeration, port scanning, directory fuzzing, API enumeration.
- Authenticated calls using your own test account, never a discovered credential.
- Rate limit, log every request, define stop conditions, and avoid destructive paths.
- Do not exploit, bypass access control, or pivot into third-party infrastructure.
- Report active results with tool, version, command, time, and target.

---

## 10. Report shape

Short answer:
```
Subject:
Question:
What we know:      3-7 bullets, each with evidence label and source/date
What we do not know:
Links:             strong / medium / weak
Next step:         passive / needs approval
```

Full report: use `SKILL.md` Section 9.3, add an asset inventory appendix and the identifier graph.

---

## 11. Pitfalls

- **Name matching.** The most common wrong conclusion; always disambiguate.
- **Shared infrastructure.** CDNs, hosting, and cloud ranges break IP-neighbor attribution.
- **Stale data.** WHOIS redaction, expired certs, reassigned IPs.
- **Tool output as truth.** Scanners misattribute orgs and versions.
- **Scope creep.** Pivoting into third parties, personal accounts, or acquired companies.
- **Evidence laundering.** Citing an aggregator that copied one blog as if it were independent.
- **Silent persistence.** Saving personal data or raw captures without approval and retention.
- **Over-collection.** More personal data increases risk without improving the answer.
