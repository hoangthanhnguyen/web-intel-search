# Tooling & Automation

Research first. These commands are references for an approved practical step, not instructions to
run them. Install only what is needed, pin versions, verify sources, and prefer package managers
over `curl | bash`. Use API keys from environment variables or a secret manager. Active scanning,
credential validation, downloads/uploads, and any change to a system require the separate approval
in `SKILL.md` Section 0.6; existing engagement authorization alone is not enough.

## 1. Web fetch and crawl

```bash
# Plain fetch
curl -sSL -A "Mozilla/5.0" "https://example.com" -o page.html
curl -sSL "https://example.com/api" -H "Accept: application/json" | jq .

# Extract text/links (choose one)
lynx -dump -nolist page.html
w3m -dump page.html
pandoc -f html -t plain page.html
python3 -c "import bs4,sys;print(bs4.BeautifulSoup(open(sys.argv[1]).read(),'html.parser').get_text())" page.html

# Playwright (JS-rendered pages); installing it is a practical action needing approval.
# Firecrawl / Crawl4AI are agent-friendly crawlers with extraction APIs.
```
Do not send internal pages, authenticated URLs, or sensitive-derived queries to third-party
extraction/crawl services without sanitizing and approval (Section 0.6). Never submit a URL that
embeds credentials or tokens.

## 2. General search APIs

```bash
# Google Programmable Search
curl -s "https://www.googleapis.com/customsearch/v1?key=$GCS_KEY&cx=$GCS_CX&q=site:example.com+filetype:pdf"

# Brave Search
curl -s "https://api.search.brave.com/res/v1/web/search?q=example" -H "X-Subscription-Token: $BRAVE_KEY"

# Tavily
curl -s https://api.tavily.com/search -H "Content-Type: application/json" \
  -d '{"api_key":"'"$TAVILY_KEY"'","query":"example","search_depth":"advanced"}'

# Exa
curl -s https://api.exa.ai/search -H "Content-Type: application/json" \
  -H "x-api-key: $EXA_KEY" -d '{"query":"example","numResults":10}'

# SerpAPI
curl -s "https://serpapi.com/search.json?q=example&api_key=$SERPAPI_KEY"

# Perplexity
curl -s https://api.perplexity.ai/chat/completions -H "Authorization: Bearer $PPLX_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"sonar-pro","messages":[{"role":"user","content":"example"}]}'
```

## 3. Cyberspace search

Searching an index (Shodan/Censys/FOFA) is passive; querying a specific target, downloading
results, or pivoting to live hosts can become active. Confirm authorization and scope first.

```bash
pip install shodan censys   # installation is a practical step
shodan init "$SHODAN_API_KEY"
shodan search --fields ip_str,port,org,product 'org:"Example Inc"'
shodan host 203.0.113.10
shodan count 'ssl:"example.com"'
curl -s https://internetdb.shodan.io/203.0.113.10 | jq .

censys search 'services.tls.certificates.leaf_data.names: example.com' --index hosts
censys view 203.0.113.10

# FOFA / ZoomEye / Hunter / Quake: use official web UI or API with your key.
```

## 4. Subdomain and asset discovery

Discovery is active against the target's DNS/infrastructure. Use passive sources first
(CT logs, passive DNS, archives); brute force and port scanning need written scope plus the
Section 0.6 approval.

```bash
# Passive first
amass enum -passive -d example.com -o amass.txt
gau example.com | tee urls.txt
waybackurls example.com | tee wayback.txt
curl -s 'https://crt.sh/?q=%25.example.com&output=json' | jq -r '.[].name_value' | sort -u

# Active enumeration/scanning: authorization + separate approval required
subfinder -d example.com -all -silent | dnsx -silent -a -resp
subfinder -d example.com -silent | httpx -silent -title -tech-detect -status-code
naabu -host example.com -top-ports 1000 -silent
nuclei -l urls.txt -severity critical,high -o nuclei.txt

# bbot (all-in-one, respects scope/rate limits)
bbot -t example.com -f subdomain-enum web-basic -o ./bbot-out
```

## 5. GitHub and code

```bash
gh search code 'org:example "AKIA"'
gh search repos 'topic:osint --sort stars'
gh api 'search/code?q=filename:.env+org:example'

# Local clone/scan is a practical step: approved, isolated, no credential use.
gitleaks detect --source repo --report-format json --report-path leaks.json
# TruffleHog verifies credentials by calling provider APIs by default. Use no-verification
# unless validation is separately approved. --results is still required because unverified
# results are otherwise filtered out.
trufflehog git file://repo --no-verification --results=verified,unverified,unknown --json
git log -p --all | grep -Ei 'api[_-]?key|secret|password|token'
```
Do not use a discovered credential. A "read-only" provider endpoint can still return customer or
financial data; see `secrets.md` validation notes.

## 6. DNS, certificates, archives

```bash
dig +short example.com A
dig +short example.com MX
dig +short example.com TXT
dig +short example.com NS
whois example.com
curl -s 'https://crt.sh/?q=%25.example.com&output=json' | jq -r '.[].name_value' | sort -u
curl -s 'https://web.archive.org/cdx/search/cdx?url=example.com/*&output=json&fl=timestamp,original,statuscode&collapse=urlkey&limit=500' | jq .
curl -s 'https://index.commoncrawl.org/CC-MAIN-2025-30-index?url=example.com/*&output=json' | jq .
```

## 7. Identity and OSINT

These tools probe third-party services about a person/account and may notify the subject or violate
site terms. Use them only for a lawful, approved objective; document the legal basis and minimize
collected data.

```bash
pipx install sherlock-project
sherlock username --timeout 10 --print-found

pipx install maigret
maigret username --html

pipx install holehe
holehe email@example.com

pipx install phoneinfoga
phoneinfoga scan -n "+84912345678"

# Image metadata (local; preserve the original file)
exiftool image.jpg
identify -verbose image.jpg
ffmpeg -i video.mp4 -vf fps=1 frame_%04d.png
```

## 8. Threat intelligence

```bash
# VirusTotal
curl -s "https://www.virustotal.com/api/v3/domains/example.com" -H "x-apikey: $VT_KEY" | jq .
curl -s "https://www.virustotal.com/api/v3/files/$SHA256" -H "x-apikey: $VT_KEY" | jq .

# AbuseIPDB
curl -s "https://api.abuseipdb.com/api/v2/check?ipAddress=203.0.113.10" \
  -H "Key: $ABUSE_KEY" -H "Accept: application/json" | jq .

# GreyNoise
curl -s "https://api.greynoise.io/v3/community/203.0.113.10" -H "key: $GREYNOISE_KEY" | jq .

# AlienVault OTX
curl -s "https://otx.alienvault.com/api/v1/indicators/domain/example.com/general" | jq .

# NVD / EPSS / KEV / OSV
curl -s "https://services.nvd.nist.gov/rest/json/cves/2.0?cveId=CVE-2024-3400" | jq .
curl -s "https://api.first.org/data/v1/epss?cve=CVE-2024-3400" | jq .
curl -s "https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json" | jq '.vulnerabilities[] | select(.cveID=="CVE-2024-3400")'
curl -s https://api.osv.dev/v1/query -d '{"package":{"name":"lodash","ecosystem":"npm"}}' | jq .

# Exploit-DB local
searchsploit apache 2.4
searchsploit -w apache 2.4
```

## 9. Automation patterns

### Rate limiting and retries
```python
import time, random, requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

session = requests.Session()
retry = Retry(total=3, backoff_factor=1.5, status_forcelist=[429, 500, 502, 503, 504])
session.mount("https://", HTTPAdapter(max_retries=retry))
session.headers["User-Agent"] = "Mozilla/5.0 (compatible; research-bot/1.0)"

def polite_get(url, **kw):
    time.sleep(1 + random.random())
    return session.get(url, timeout=20, **kw)
```

### Deduplication
```python
from urllib.parse import urlsplit, urlunsplit, parse_qsl, urlencode

TRACKING = {"utm_source","utm_medium","utm_campaign","utm_term","utm_content","fbclid","gclid"}

def canonical(url):
    p = urlsplit(url)
    q = [(k,v) for k,v in parse_qsl(p.query, keep_blank_values=True) if k.lower() not in TRACKING]
    path = p.path.rstrip("/") or "/"
    return urlunsplit((p.scheme.lower(), p.netloc.lower(), path, urlencode(q), ""))
```

### Result record schema
Track results in the answer; save a file only with approval. Use evidence labels, not a score.
```json
{
  "query": "site:example.com filetype:pdf",
  "engine": "google",
  "timestamp": "2026-10-20T02:00:00Z",
  "url": "https://example.com/report.pdf",
  "title": "Annual Report",
  "snippet": "...",
  "evidence": "source-reported",
  "limits": "not opened in full",
  "pivots": ["whois", "certificate", "author"]
}
```

### Parallel query batching
- Batch 3-4 engines/queries, sequential within a batch, 1-2s delay.
- Respect per-API quotas; cache responses keyed by canonical query.
- Stop retrying on 401/403; rotate only where ToS permits.
- Persist raw responses only with approval of location and content.

## 10. Safety checklist

- Research first; practical steps need written authorization plus Section 0.6 approval.
- Scope every action: target/account, operation, environment, exclusions, rate, time, stop condition.
- No credential use without a separately approved, narrowly scoped check; no data exfiltration.
- Sanitize sensitive-derived queries and show them before sending.
- Redact secrets and PII in outputs.
- Verify tools/skills before install; prefer signed releases and pinned versions.
- Run untrusted tools in a container/VM without credentials.
- Track queries and actions in the answer for the engagement report; no silent persistence.
