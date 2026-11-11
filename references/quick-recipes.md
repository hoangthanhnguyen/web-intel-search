# Quick Reference: Query Recipes

Copy-paste starting points. Adapt each recipe to the target and source, and verify results against
a primary source. Engine syntax and limits are in `engines.md`; full dork categories are in `dorks.md`.
Sensitive-derived queries must be sanitized and shown for approval before sending (`SKILL.md` 0.6).

```
# Exact error / unique string
"ORA-12541: TNS:no listener"

# Exposed documents on a domain
site:example.com (filetype:pdf OR filetype:xlsx OR filetype:csv)

# Admin panels
site:example.com (inurl:admin OR inurl:login OR inurl:dashboard)

# Subdomains discovered by search engines
site:*.example.com -www

# Secrets in public repos (scope the OR group)
org:example AND ("AKIA" OR "api_key" OR "BEGIN RSA PRIVATE KEY")

# Exposed services by org (Shodan); review findings, do not connect to them
org:"Example Inc" port:3389,22,445,3306,5432,27017,9200,6379

# Certificate pivot (Censys)
host.services.tls.certificates.leaf_data.subject.organization: "Example Inc"

# Favicon pivot (Shodan)
http.favicon.hash:-1234567890

# Vulnerable product (FOFA)
app="Apache" && country="VN" && port="443"

# Recent breaches / leaks
site:pastebin.com "example.com" after:2025-01-01

# CVE enrichment
https://api.first.org/data/v1/epss?cve=CVE-2024-3400
https://services.nvd.nist.gov/rest/json/cves/2.0?cveId=CVE-2024-3400

# Reverse image
Yandex Images > Google Lens > Bing Visual Search > TinEye

# Username spread
https://whatsmyname.app/?q=<username> and manual verification
```

## Safety notes

- Index queries are passive; do not connect to discovered hosts without authorization and the
  separate action approval (`SKILL.md` 0.6).
- Never use discovered credentials, even to "just check". Detecting a secret is not permission to test it.
- Sanitize errors, hostnames, and customer data before sending a query to an external provider.
- Redact secrets, PII, and customer data before sharing results.
- A search hit is a lead, not proof; confirm with the vendor advisory, source code, or a lab test.
