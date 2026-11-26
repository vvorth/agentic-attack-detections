# Contributing

Thank you for helping. This is a free, open-source project for security teams, and outside reports are how the rules get better.

## What helps most

- **False positives.** A rule fired on something harmless. Tell us the rule and the shape of the event.
- **Missed detections.** A framework did something the rules did not catch.
- **New indicators.** An address, domain, hash or string from a published report.
- **Fixes.** A query that does not run, a rule that cannot fire, a broken conversion.
- **New frameworks.** Follow [docs/adding-a-framework.md](docs/adding-a-framework.md).

Open an [issue](https://github.com/ThreatDefence/agentic-attack-detections/issues) with the matching form, or send a pull request.

Never include customer data, real user names, internal host names or internal addresses. Describe the shape of the event instead.

## Setting up

```bash
pip install -r tools/requirements.txt
```

Optional tools unlock more checks: `suricata` and `openssl` for the Suricata tests, `zeek` for the Zeek tests, and `dotnet` for the KQL check.

## Before you open a pull request

Run these and commit everything they regenerate:

```bash
python3 tools/build_feeds.py           # rebuild feeds/ and the per-framework lists from iocs/indicators.csv
python3 tools/convert.py               # rebuild generated/ from the Sigma rules
.venv-s1/bin/python tools/convert_sentinelone.py   # rebuild generated/sentinelone/; venv from tools/requirements-sentinelone.txt
.venv-cortex/bin/python tools/convert_cortex.py    # rebuild generated/cortex_xdr/; venv from tools/requirements-cortex.txt
python3 tools/build_attack.py          # rebuild the ATT&CK coverage page and Navigator layers
python3 tools/validate.py              # Sigma, KQL, Elastic, Splunk, Suricata, YARA, Zeek, feeds
python3 tests/suricata/run_tests.py    # one flow per Suricata rule
python3 tests/sigma/run_tests.py       # Sigma rules against test events in tests/sigma/fixtures
python3 tests/kql/run_tests.py         # hunts and analytics against sample rows; needs the Kusto emulator, see tests/kql/README.md
python3 tools/build_waf.py             # rebuild waf/ from waf/signatures.yaml and iocs/indicators.csv
python3 tests/waf/run_tests.py         # every WAF signature on Cloudflare, AWS WAF and Azure WAF
```

CI runs the same checks on every pull request.

## Rules for content

**Indicators** go in `iocs/indicators.csv`, the single source for every feed.
- Every row names its source and links to it. The value must appear in that source.
- Addresses get a `review_by` date 180 days after the report.
- Set `confidence` as the README describes: high, medium or low.
- Do not edit files under `feeds/` or `generated/` by hand. They are built.

**Suricata rules**
- Use the framework's SID range, listed in [docs/adding-a-framework.md](docs/adding-a-framework.md).
- Give every rule that inspects content a fast pattern. A `pcre` on its own runs on every transaction.
- Add a case to `tests/suricata/run_tests.py` that makes the rule fire, and a near miss that must stay silent. The test run fails if a rule has no case.

**KQL**
- Use real table and column names. `tools/validate.py` checks them against Microsoft's table reference.
- If a hunt uses a table the checker does not know, add it with `python3 tools/kqlcheck/fetch_schema.py <TableName>`.
- Test cases are welcome: sample rows the query must return, and near misses, in the format described in [tests/kql/README.md](tests/kql/README.md).

**Sigma**
- One behavior per rule, with `falsepositives` filled in honestly.
- Keep a rule's file path stable: its ID is derived from the path.
- Test events are welcome: a match and a near miss in `tests/sigma/fixtures/`, in the format described in [tests/sigma/README.md](tests/sigma/README.md).

**ATT&CK**: every rule names at least one technique from the release pinned in `tools/attack-enterprise.json`, and CI fails one that names none or a retired one. Pick the technique for what the rule sees, not for everything the framework can do.
- Sigma: `attack.<tactic>` and `attack.tNNNN` tags.
- Sentinel analytics: `tactics` and `relevantTechniques`.
- KQL hunts: a `// ATT&CK: T1190 (Exploit Public-Facing Application), ...` line in the header comment, after the attribution line.
- Suricata: `mitre_tactic_id`, `mitre_tactic_name`, `mitre_technique_id` and `mitre_technique_name` in `metadata`, one group per technique, as the Emerging Threats rules write them.
- YARA: `mitre_att = "T1505.003"` in `meta` (comma separated for more than one).
- Zeek: `ATT&CK: T1595.002.` at the end of the notice's `##` comment.

**Every signature string** should come from the framework's source code at a named commit, or from a report that published it. Say which in the pull request.

## What to expect

We aim to reply to issues and pull requests within two working days. If a change does not fit the project, we will say so and explain why.

Everyone taking part is expected to follow the [code of conduct](CODE_OF_CONDUCT.md).

By contributing you agree that your contribution is released under the [MIT license](LICENSE).
