# Resecurity RISK

The Resecurity RISK integration collects threat intelligence from the
[Resecurity RISK](https://www.resecurity.com/) REST API. It ships 4 data streams:

| Data stream | Source endpoint | Search term |
|---|---|---|
| Alert | `GET /alert/index` | optional |
| IOC | `GET /ioc/index` | not supported |
| Breach | `GET /breaches/index` | **required** |
| Dark Web | `GET /dark-web/index` | **required** |

## Requirements

You need a Resecurity RISK API key. Resecurity's API authenticates with an `Authorization: Basic`
header containing only the **base64-encoded API key** — no username/password pair, no separator.
Enter your key in the "API key" field of the integration policy; leave everything else as
configured by default unless your Resecurity tenant uses a non-default host.

## Breach and Dark Web require a search term

Unlike Alert and IOC, the Breach and Dark Web endpoints do not support listing all records — the
upstream API requires a search phrase (for example a company domain, brand name, or email
address to monitor). Configure this in the "Search query" field when adding the Breach or Dark
Web data stream. To monitor multiple terms, add multiple instances of the data stream, one per
term.

## Known limitation: no incremental collection

None of the 4 Resecurity RISK endpoints used by this integration expose a "since timestamp"
filter. Every poll re-walks pages from the start rather than fetching only new records. To bound
the work done per poll, each data stream has a `max_pages_per_poll` setting (default 10). Document
IDs are derived from the source record's own `id` field, so re-ingesting the same record on a
later poll overwrites the existing document rather than creating a duplicate — but if your feed
has more genuinely new records per interval than `max_pages_per_poll * 100` (the API's default
page size), increase `max_pages_per_poll` or shorten `interval` to keep up.

## Data streams

### Alert

Threat intelligence alerts — subject, content, category, confidence/risk scores, TLP status,
associated threat actors, geography, and any embedded IOC or TTP detail.

### IOC

Indicators of compromise — malware name, file hashes (MD5/SHA-256), detection ratio, and dates.

### Breach

Individual leaked-credential records (email, username, password/password hash) plus the breach
source's metadata (name, category, attack vector, compromised data types, geography).

### Dark Web

Dark web / underground-forum posts matching your search term — actor, category, country,
language, title, snippet, and source URL.
