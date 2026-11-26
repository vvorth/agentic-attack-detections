# Agentic Attack Detections

[![validate](https://github.com/ThreatDefence/agentic-attack-detections/actions/workflows/validate.yml/badge.svg)](https://github.com/ThreatDefence/agentic-attack-detections/actions/workflows/validate.yml)
[![release](https://img.shields.io/github/v/release/ThreatDefence/agentic-attack-detections)](https://github.com/ThreatDefence/agentic-attack-detections/releases)
[![license](https://img.shields.io/github/license/ThreatDefence/agentic-attack-detections)](LICENSE)

Open-source detection rules, hunting queries and indicators for security teams. They cover agentic attack frameworks: tools that hand an intrusion to AI agents, which plan, run real attack tools and keep going until they reach a goal.

Free to use under the MIT license. Fixes, false-positive reports and new indicators are welcome.

## What is here

- **Framework folders.** ARTEX and its AntiFox fork, Cairn, PentAGI, HexStrike, CyberStrikeAI, Strix, Hermes Agent, Agent Zero, RedAmon, Xalgorix, Pentest Swarm AI, Decepticon, Shannon, PentestGPT, Cyber-AutoAgent, CAI, HackSynth, VulnBot, AutoPT, PentestAgent (the paper's and GH05TCREW's), Dark Moon, NyxStrike, sapper-v2, NeuroSploit, Vigilo, Pentest Copilot, cyber-harness, CHYing-agent, Hexstrike-redteam, pentest-ai, LazyOwn, H-Pentest, Incalmo, Virga, MCP Kali Server, ARIA-X, Mitsu, burp-ai-agent, RAPTOR, Blitz Strike, Nebula, Clearwing, BreachWeave, numasec, Pentest Harness, Agentic Bug Hunter, 0security, Sage (Mythic), PentestCode, hackingBuddyGPT, AutoPentester, MAPTA, PentesterFlow, Pentest Agent Suite, Cain, KaliGPT, Pensar Apex, MCP for Security, Guardian, Cybermes, ARTEMIS, CTF Agent, OpenHack, RedAI, Cochise, Deadend CLI and Ares, plus four offensive prompt packs. Each folder is built from a review of the framework's source code and the public reporting on its use. Fourteen more frameworks whose consoles were found on the internet have smaller folders that say what the console and the tool look like, with the code behind each signal: AutoCVE, CyberStrike, Vulnerator AI, Decepticon, AIDA, BugTraceAI, Z3r0, AIRecon, LuaN1aoAgent, open-kritt, REDCELL, T3MP3ST, WormXGPT and Chufeng Agent Core.
- **Shared behaviors.** Rules for what these frameworks have in common, whatever the tool or the model behind it.
- **Indicators and feeds.** One CSV of 580 sourced indicators, and the feeds generated from it.
- **Exposed consoles.** A dated list of addresses where one of these consoles answers on the internet, in [`exposure/`](exposure), with a [count per framework](#exposed-consoles). It is a record of what is exposed, not a detection feed.

Formats: KQL for Microsoft Sentinel and Defender XDR, Sigma, Suricata, Zeek, YARA and STIX 2.1, plus WAF rules for Cloudflare, AWS WAF and Azure WAF. Splunk has one app for every framework. Elastic has one import with a rule for every framework.

## Try it in a minute

With Suricata installed, run the ARTEX rules over the sample capture in this repository:

```bash
git clone https://github.com/ThreatDefence/agentic-attack-detections
cd agentic-attack-detections
mkdir -p out
suricata -r frameworks/artex/tests/artex-sample.pcap -S frameworks/artex/suricata/artex.rules -l out -k none
cat out/fast.log
```

You should see alerts such as:

```text
[1:3800001:1] TD ARTEX user agent artex-<role>/1.0
[1:3800020:1] TD ARTEX placeholder probe path
[1:3800023:2] TD Unsigned alg none JWT in URI
```

The capture is test traffic written from the framework source code and published reports. It is not a recorded intrusion.

## Start here

| You have | Do this |
|---|---|
| Microsoft Sentinel | [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FThreatDefence%2Fagentic-attack-detections%2Fmain%2Fdeploy%2Fsentinel%2Fagentic-attack-detections.json) One deployment creates every analytics rule (disabled until you enable them), every hunting query and the `AGENTIC_IOCs` watchlist. Give it the workspace name. [`deploy/sentinel/`](deploy/sentinel) has one template per framework too. Run the hunts in each folder's `hunting-order.md` order |
| Defender XDR | Run the hunts from `frameworks/*/sentinel/hunting/` in advanced hunting. They work where the tables exist |
| Suricata | Add the `.rules` files from `frameworks/*/suricata/` and `behaviors/suricata/`. HTTP rules need cleartext or decrypted traffic. DNS and TLS SNI rules work on encrypted traffic |
| Zeek | Install the behaviors package with `zkg install https://github.com/ThreatDefence/agentic-attack-detections` (or load `behaviors/zeek/agentic-behaviors/` by path), set `server_nets` to your server ranges ([setup](behaviors/zeek/README.md)), load the framework scripts `frameworks/artex/zeek/artex/`, `frameworks/cyberstrikeai/zeek/cyberstrikeai/`, `frameworks/hexstrike/zeek/hexstrike/`, `frameworks/autocve/zeek/autocve/` and `frameworks/luan1aoagent/zeek/luan1aoagent/`, and feed `frameworks/*/zeek/intel/*.intel` to the Intel framework |
| YARA | Scan container layers, home folders, `/tmp` and web roots with `frameworks/*/yara/` |
| Any SIEM with Sigma | Convert `frameworks/*/sigma/` and `behaviors/sigma/` with `sigma convert`, or take the conversions in [`generated/`](generated) |
| Splunk | Install [`agentic-attack-detections-splunk.tgz`](https://github.com/ThreatDefence/agentic-attack-detections/releases/latest/download/agentic-attack-detections-splunk.tgz) from the latest release (Apps > Install app from file). One app has the searches for every framework, the indicator lookups and a dashboard with a card per framework. Point the `aad_indexes` macro at your indexes. [Setup](deploy/splunk) |
| SentinelOne | Take the converted queries in [`generated/sentinelone/`](generated/sentinelone): `s1ql/` for Deep Visibility searches and STAR custom rules, `powerquery/` for the Singularity Data Lake. 229 endpoint rules convert (process, file, registry and DNS). Proxy, web server and cloud rules don't: [`SUMMARY.md`](generated/sentinelone/SUMMARY.md) lists which rules converted and why the rest didn't. Load the hashes, domains and addresses in [`feeds/`](feeds) as threat intelligence. The queries are converted but not tested on SentinelOne data, so run them as hunts first |
| Cortex XDR or XSIAM | Take the XQL queries in [`generated/cortex_xdr/`](generated/cortex_xdr) and save the ones you want as BIOC rules. 209 endpoint rules convert (process, file and registry). DNS, proxy, web server and cloud rules don't: [`SUMMARY.md`](generated/cortex_xdr/SUMMARY.md) lists which rules converted and why the rest didn't. Load the indicators in [`feeds/`](feeds) as IOC rules. The queries are converted but not tested on Cortex data, so run them as hunts first |
| Elastic | Import [`agentic-attack-detections.ndjson`](https://github.com/ThreatDefence/agentic-attack-detections/releases/latest/download/agentic-attack-detections.ndjson) through Security > Rules > Import, or post it to `/api/detection_engine/rules/_import`. One file carries every framework and the shared behaviors, all disabled: the Sigma rules as query rules, the correlations as ES\|QL rules, the hand-written ARTEX rules, and an indicator match rule. For that one, load [`agentic-attack-indicators.ndjson`](https://github.com/ThreatDefence/agentic-attack-detections/releases/latest/download/agentic-attack-indicators.ndjson) with `POST logs-ti_agentic_attacks-default/_bulk`. Point the index patterns at your own data before you enable them. [Details](generated/elastic_ndjson) |
| A WAF: Cloudflare, AWS WAF or Azure WAF | Deploy the rules in [`waf/`](waf): framework user agents, probe markers and unsigned JWT probes to block, and reported addresses to challenge. A WAF is optional. It adds blocking at the edge to the detections, it doesn't replace them |
| A threat intelligence platform | Import `feeds/stix2.1-bundle.json`, or the per-framework lists in `feeds/by-framework/`. Steps for MISP and OpenCTI, and which fields arrive: [docs/threat-intelligence-platforms.md](docs/threat-intelligence-platforms.md) |

## Coverage

| | KQL hunts | Sentinel analytics | Sigma | Suricata | YARA | Zeek | Indicators |
|---|---|---|---|---|---|---|---|
| [ARTEX](frameworks/artex) | 15 | 3 | 16 | 46 | 4 | script, intel | 247 |
| [Cairn](frameworks/cairn) | 6 | 2 | 3 | 26 | 6 | intel | 73 |
| [PentAGI](frameworks/pentagi) | 5 | 2 | 3 | 10 | 3 | intel | 39 |
| [HexStrike](frameworks/hexstrike) | 4 | 2 | 3 | 7 | 3 | script, intel | 22 |
| [CyberStrikeAI](frameworks/cyberstrikeai) | 5 | 2 | 7 | 11 | 4 | script, intel | 74 |
| [Strix](frameworks/strix) | 3 | 1 | 3 | 5 | 3 | intel | 49 |
| [Hermes Agent](frameworks/hermes) | 4 | 2 | 3 | 7 | 4 | intel | 79 |
| [Agent Zero](frameworks/agent-zero) | 2 | | 1 | 3 | 1 | intel | 8 |
| [RedAmon](frameworks/redamon) | 3 | | 3 | 10 | 1 | intel | 19 |
| [Xalgorix](frameworks/xalgorix) | 3 | | 3 | 10 | 1 | intel | 24 |
| [Pentest Swarm AI](frameworks/pentestswarm) | 3 | | 3 | 11 | 1 | intel | 15 |
| [Decepticon](frameworks/decepticon) | 3 | 1 | 6 | 15 | 6 | intel | 31 |
| [Shannon](frameworks/shannon) | 3 | 1 | 3 | 3 | 5 | intel | 21 |
| [PentestGPT](frameworks/pentestgpt) | 3 | 1 | 3 | 9 | 3 | intel | 23 |
| [Cyber-AutoAgent](frameworks/cyber-autoagent) | 4 | 1 | 8 | 8 | 4 | intel | 19 |
| [CAI](frameworks/cai) | 3 | 1 | 6 | 14 | 4 | intel | 22 |
| [HackSynth](frameworks/hacksynth) | 3 | 1 | 4 | 1 | 2 | intel | 10 |
| [VulnBot](frameworks/vulnbot) | 3 | 1 | 3 | 5 | 2 | intel | 15 |
| [AutoPT](frameworks/autopt) | 3 | 1 | 4 | 6 | 2 | intel | 8 |
| [PentestAgent (paper code)](frameworks/pentestagent) | 3 |  | 3 | 3 | 2 | intel | 11 |
| [PentestAgent (GH05TCREW)](frameworks/ghostcrew-pentestagent) | 3 | 1 | 3 | 3 | 1 | intel | 10 |
| [DarkMoon](frameworks/darkmoon) | 4 | 1 | 4 | 12 | 3 | intel | 29 |
| [NyxStrike](frameworks/nyxstrike) | 3 | 1 | 4 | 17 | 3 | intel | 38 |
| [sapper-v2](frameworks/sapper-v2) | 3 | 1 | 3 | 6 | 2 | intel | 24 |
| [NeuroSploit](frameworks/neurosploit) | 4 | 1 | 9 | 22 | 4 |  | 30 |
| [Vigilo](frameworks/vigilo) | 3 | 1 | 4 | 18 | 2 | intel | 15 |
| [Pentest Copilot](frameworks/pentest-copilot) | 3 | 1 | 4 | 11 | 2 | intel | 23 |
| [cyber-harness](frameworks/cyber-harness) | 3 | 1 | 4 | 16 | 3 | intel | 34 |
| [CHYing-agent](frameworks/chying-agent) | 3 | 1 | 3 | 8 | 3 | intel | 19 |
| [Hexstrike-redteam](frameworks/hexstrike-redteam) | 3 | 1 | 4 | 10 | 3 | intel | 34 |
| [pentest-ai](frameworks/pentest-ai) | 3 | 1 | 5 | 24 | 2 | intel | 38 |
| [LazyOwn](frameworks/lazyown) | 3 | 1 | 5 | 23 | 5 | intel | 30 |
| [H-Pentest](frameworks/h-pentest) | 3 | 1 | 3 | 7 | 2 | intel | 15 |
| [Incalmo](frameworks/incalmo) | 4 | 1 | 5 | 13 | 3 | intel | 31 |
| [Virga](frameworks/virga) | 4 | 1 | 4 | 14 | 4 | intel | 30 |
| [MCP Kali Server](frameworks/mcp-kali-server) | 3 | 1 | 5 | 8 | 3 | intel | 15 |
| [ARIA-X](frameworks/aria-x) | 3 | 1 | 4 | 8 | 2 | intel | 22 |
| [Mitsu](frameworks/mitsu) | 3 | 1 | 7 | 5 | 3 | intel | 26 |
| [Custom AI Agent (burp-ai-agent)](frameworks/burp-ai-agent) | 3 | 1 | 5 | 17 | 2 | intel | 26 |
| [RAPTOR](frameworks/raptor) | 3 | 1 | 4 | 21 | 3 | intel | 39 |
| [Blitz Strike](frameworks/blitzstrike) | 3 | 1 | 5 | 15 | 2 | intel | 26 |
| [Nebula](frameworks/nebula) | 3 | 1 | 5 | 15 | 3 | intel | 39 |
| [Clearwing](frameworks/clearwing) | 3 | 1 | 5 | 20 | 3 | intel | 26 |
| [BreachWeave](frameworks/breachweave) | 4 | 1 | 5 | 22 | 4 | intel | 37 |
| [numasec](frameworks/numasec) | 3 | 1 | 5 | 10 | 4 | intel | 18 |
| [Pentest Harness](frameworks/pentest-harness) | 3 | 1 | 4 | 12 | 2 | intel | 18 |
| [Agentic Bug Hunter](frameworks/agentic-bug-hunter) | 4 | 1 | 6 | 23 | 2 | intel | 31 |
| [0security](frameworks/zerosecurity) | 3 | 1 | 7 | 27 | 2 | intel | 40 |
| [Sage (Mythic)](frameworks/sage-mythic) | 4 | 1 | 7 | 9 | 4 | intel | 38 |
| [PentestCode](frameworks/pentestcode) | 4 | 1 | 6 | 9 | 3 |  | 21 |
| [hackingBuddyGPT](frameworks/hackingbuddygpt) | 4 | 1 | 5 | 7 | 3 | intel | 16 |
| [AutoPentester](frameworks/autopentester) | 3 | 1 | 2 | 9 | 2 | intel | 17 |
| [MAPTA](frameworks/mapta) | 3 |  | 2 |  | 3 | intel | 11 |
| [PentesterFlow](frameworks/pentesterflow) | 3 | 1 | 4 | 4 | 3 | intel | 18 |
| [Pentest Agent Suite](frameworks/pentest-agent-suite) | 3 | 1 | 5 | 14 | 4 | intel | 23 |
| [Cain](frameworks/cain) | 3 | 1 | 4 | 6 | 2 | intel | 17 |
| [KaliGPT](frameworks/kaligpt) | 3 | 1 | 7 | 3 | 3 | intel | 14 |
| [Pensar Apex](frameworks/apex) | 3 | 1 | 5 | 5 | 3 | intel | 23 |
| [MCP for Security](frameworks/mcp-for-security) | 3 | 1 | 3 | 8 | 3 | intel | 17 |
| [Guardian](frameworks/guardian) | 3 | 1 | 6 | 4 | 4 | intel | 13 |
| [Cybermes](frameworks/cybermes) | 3 | 1 | 5 | 12 | 3 | intel | 24 |
| [ARTEMIS](frameworks/artemis) | 3 | 1 | 6 | 4 | 4 | intel | 19 |
| [CTF Agent](frameworks/ctf-agent) | 3 | 1 | 6 | 3 | 2 | intel | 13 |
| [OpenHack](frameworks/openhack) | 3 | 1 | 8 | 14 | 3 | intel | 20 |
| [RedAI](frameworks/redai) | 3 | 1 | 2 |  | 4 | intel | 12 |
| [Cochise](frameworks/cochise) | 3 | 1 | 3 | 1 | 3 | intel | 9 |
| [Deadend CLI](frameworks/deadend-cli) | 3 | 1 | 5 | 8 | 2 | intel | 16 |
| [Ares](frameworks/ares) | 4 | 1 | 9 | 6 | 6 | intel | 20 |
| [Offensive prompt packs](frameworks/prompt-packs) |  |  | 4 |  | 8 | intel | 25 |
| [AutoCVE](frameworks/autocve) | | | | 2 | | script | |
| [LuaN1aoAgent](frameworks/luan1aoagent) | | | | 4 | | script | |
| [Shared behaviors](behaviors) | 25 | 6 | 26 | 18 | | package | |

Every rule names its MITRE ATT&CK techniques. [docs/attack-coverage.md](docs/attack-coverage.md) lists them per framework with links to the rules, and [`generated/attack/`](generated/attack) has an ATT&CK Navigator layer per framework and one for everything.

The other eleven console-level folders (CyberStrike, Vulnerator AI, AIDA, BugTraceAI, Z3r0, AIRecon, open-kritt, REDCELL, T3MP3ST, WormXGPT, Chufeng Agent Core) have no rules yet: they describe the console and the tool, and their exposure searches are in `exposure/`. Suricata counts are hand-written rules; each framework also has address rules generated from its indicators. Some indicators belong to more than one framework. Every framework is in the [Splunk app](deploy/splunk). ARTEX also has Elastic rules and ES\|QL queries, and an osquery pack.

## Frameworks

| Framework | What it is | Seen in |
|---|---|---|
| [ARTEX](frameworks/artex) | Go planner and worker agents with a web console. AntiFox is the same code renamed | Storm-0940, Storm-3358, VAPOR PANDA and criminal groups, against Microsoft 365, Okta, AWS, Azure, web applications and Active Directory |
| [Cairn](frameworks/cairn) | Python engine that drives Claude Code, Codex and Pi coding agents in Kali containers | Online retailers, payment sites, Japanese web applications and WSUS servers |
| [PentAGI](frameworks/pentagi) | Go multi-agent platform that runs Kali tools in a container per engagement | Intrusions GTIG and Mandiant reported since August 2026, and a PaperCut exploitation operator's infrastructure |
| [HexStrike](frameworks/hexstrike) | Python MCP server that gives any AI assistant about 150 security tools over an unauthenticated API | NetScaler exploitation discussion in 2025, and intrusions GTIG and Mandiant reported |
| [CyberStrikeAI](frameworks/cyberstrikeai) | Go operator console whose agents run about 90 tools, with its own implants, C2 listeners and webshell manager | FortiGate mass-exploitation infrastructure, an AWS intrusion, and education organizations in Africa and Latin America |
| [Strix](frameworks/strix) | Python AI pentest agent that runs teams of agents in a Kali sandbox, on any model | The retail card-theft campaign Gambit reported, where it found the flaws Cairn exploited |
| [Hermes Agent](frameworks/hermes) | A legitimate general-purpose agent turned into an attack console: approvals off, a bundled jailbreak skill, red-team personas | The Gambit retail operator, the Carbonato Docker botnet, and Thailand's Ministry of Finance |
| [Agent Zero](frameworks/agent-zero) | General-purpose agent in a Kali container with a built-in hacker profile and an open web UI | Threat-actor experimentation GTIG reported, with no published indicators |
| [RedAmon](frameworks/redamon) | Self-hosted penetration testing platform: a web console, a recon pipeline and an agent that exploits from a Kali sandbox with Metasploit | No public report of use in an attack. Covered from its source code |
| [Xalgorix](frameworks/xalgorix) | Go binary with a web dashboard whose agent tests web applications and APIs without approval steps and re-exploits each finding | No public report of use in an attack. Covered from its source code |
| [Pentest Swarm AI](frameworks/pentestswarm) | Go command-line tool that runs recon, exploit and report agents over about 40 security tools | No public report of use in an attack. Covered from its source code |
| [Shannon](frameworks/shannon) | TypeScript white-box AI pentester: Temporal workers drive model agents that read the code and test the running application with a headless browser | No public report of use in an attack. Covered from its source code |
| [PentestGPT](frameworks/pentestgpt) | Python agent from the USENIX Security 2024 paper that drives the Claude Code or Codex CLI with permission checks off | No public report of use in an attack. Covered from its source code |
| [Cyber-AutoAgent](frameworks/cyber-autoagent) | Python agent on the Strands SDK that runs a penetration test from a Kali container without approval steps, with a bundled Langfuse stack | No public report of use in an attack. Covered from its source code |
| [CAI](frameworks/cai) | Alias Robotics agent framework whose red team and bug bounty agents run shell commands, with an optional HTTP API and session-log telemetry | No public report of use in an attack. Covered from its source code |
| [HackSynth](frameworks/hacksynth) | Research CTF agent: a planner model writes one bash command per step, run in a Kali container | No public report of use in an attack. Covered from its source code |
| [VulnBot](frameworks/vulnbot) | Research multi-agent pentester that runs Kali tools over SSH, with an optional unauthenticated knowledge base API | No public report of use in an attack. Covered from its source code |
| [AutoPT](frameworks/autopt) | Research web pentest agent that runs public vulhub exploits, with an unauthenticated Flask console on port 5000 | No public report of use in an attack. Covered from its source code |
| [PentestAgent (paper code)](frameworks/pentestagent) | Research agent from arXiv 2411.05185: nmap, CVE lookup and a public exploit run with the model's commands. Not the GH05TCREW tool | No public report of use in an attack. Covered from its source code |
| [PentestAgent (GH05TCREW)](frameworks/ghostcrew-pentestagent) | Python pentest agent (GH05TCREW/pentestagent) with a terminal UI, agent crews, Kali sandbox images and an unauthenticated MCP server | No public report of use in an attack. Covered from its source code |
| [DarkMoon](frameworks/darkmoon) | OpenCode-based agent stack with 51 attack prompts, a root toolbox container and an unauthenticated MCP server | No public report of use in an attack. Covered from its source code |
| [NyxStrike](frameworks/nyxstrike) | HexStrike AI derivative with a web interface, a decision engine and about 200 tools over MCP | No public report of use in an attack. Covered from its source code and scan data |
| [sapper-v2](frameworks/sapper-v2) | Contest kit that coordinates several coding assistants around a shared engagement graph, with a web board | No public report of use in an attack. Covered from its source code and scan data |
| [NeuroSploit](frameworks/neurosploit) | Rust multi-model harness with hundreds of specialist agents, an out-of-band channel and a web interface | No public report of use in an attack. Covered from its source code and scan data |
| [Vigilo](frameworks/vigilo) | Agent-driven web and white-box assessment platform published by CERT-EU; its grey-box agent names itself in every request | A legitimate assessment tool; no public report of misuse. Covered from its source code and scan data |
| [Pentest Copilot](frameworks/pentest-copilot) | BugBase web application whose agent runs commands in Kali, drives a browser agent and integrates with Mythic C2; the first account to register owns it | No public report of use in an attack. Covered from its source code and scan data |
| [cyber-harness](frameworks/cyber-harness) | chainreactors Go agent harness (formerly AIScan): a scan agent running the chainreactors scanners, code audit, and a Web Hub that controls nodes | No public report of use in an attack. Covered from its source code and scan data |
| [CHYing-agent](frameworks/chying-agent) | Contest agent on the Claude Agent SDK with an unauthenticated dashboard on every interface and fixed web terminal markers | No public report of use in an attack. Covered from its source code |
| [Hexstrike-redteam](frameworks/hexstrike-redteam) | Copy of HexStrike with the BOAZ payload evasion framework behind five API routes, bound to every interface with no auth | No public report of use in an attack. Covered from its source code |
| [pentest-ai](frameworks/pentest-ai) | Python CLI and MCP server whose every request carries its name and a per-install id; its compose file serves the dashboard without auth | No public report of use in an attack. Covered from its source code |
| [LazyOwn](frameworks/lazyown) | Python red team framework with a multi-operator C2, HTTP beacons on a Gmail-like path, a DNS channel, and about 190 MCP tools for AI agents | No public report of use in an attack. Covered from its source code |
| [H-Pentest](frameworks/h-pentest) | Small contest pentest platform: an unauthenticated FastAPI backend with the Docker socket mounted, and a Vue console | No public report of use in an attack. Covered from its source code |
| [Incalmo](frameworks/incalmo) | CMU research system for multistage attacks: a planner model drives Caldera Sandcat implants through an unauthenticated Flask C2 | No public report of use in an attack. Covered from its source code |
| [Virga](frameworks/virga) | Experimental Go C2 whose beacon can embed a local language model; unauthenticated MCP servers and admin API | No public report of use in an attack. Covered from its source code |
| [MCP Kali Server](frameworks/mcp-kali-server) | Flask API plus MCP bridge that lets an AI assistant run any command on a Kali host; no authentication, packaged in Kali | No public report of use in an attack. Covered from its source code |
| [ARIA-X](frameworks/aria-x) | Python engine borrowing ARTEX, Cairn and PentAGI designs; console on 8787 and its own ARIA-X user agent on every request | No public report of use in an attack. Covered from its source code |
| [Mitsu](frameworks/mitsu) | Terminal AI pentest agent with a 165-tool catalog, driving Codex or Claude Code over a local MCP server | No public report of use in an attack. Covered from its source code |
| [Custom AI Agent (burp-ai-agent)](frameworks/burp-ai-agent) | Burp Suite extension (Custom AI Agent) that lets models and MCP clients drive Burp and adds AI scanners with fixed payload markers | No public report of use in an attack. Covered from its source code |
| [RAPTOR](frameworks/raptor) | Claude Code research framework for code and binary analysis, a self-naming web scanner, and exploit and patch generation | No public report of use in an attack. Covered from its source code |
| [Blitz Strike](frameworks/blitzstrike) | MCP server for AI coding clients that runs a one-call web engagement with fixed user agents, canaries and redirect probes | No public report of use in an attack. Covered from its source code |
| [Nebula](frameworks/nebula) | Operator workstation by BerylliumSec whose agents run commands in a local Kali container, with an interception proxy | No public report of use in an attack. Covered from its source code |
| [Clearwing](frameworks/clearwing) | Network pentest agent and source-code vulnerability hunter with a Kali container, signed payloads and callback listeners | No public report of use in an attack. Covered from its source code |
| [BreachWeave](frameworks/breachweave) | Multi-agent pentest platform from Chinese automated-attack contests, with Kali solver containers and NPS tunnel hand-off | No public report of use in an attack. Covered from its source code |
| [numasec](frameworks/numasec) | Terminal security agent built on OpenCode, with its own AppSec probe user agent and directory fuzzer markers | No public report of use in an attack. Covered from its source code |
| [Pentest Harness](frameworks/pentest-harness) | Rebrand of the dsh agent harness with a pentest persona and a local web UI with no login | No public report of use in an attack. Covered from its source code and scan data |
| [Agentic Bug Hunter](frameworks/agentic-bug-hunter) | Bug bounty toolkit for coding agents with fixed verifier user agents, a password sprayer and OOB markers | No public report of use in an attack. Covered from its source code |
| [0security](frameworks/zerosecurity) | Harness for persistent AI security agents (0security) with fixed LLM probe prompts and a server-mode console | No public report of use in an attack. Covered from its source code |
| [Sage (Mythic)](frameworks/sage-mythic) | AI operator inside the Mythic C2 framework whose tasks leave fixed SAGE_ proof markers on Windows hosts | No public report of use in an attack. Covered from its source code |
| [PentestCode](frameworks/pentestcode) | OpenCode fork with thirteen pentest agents, injection probe markers and pivot and persistence recipes | No public report of use in an attack. Covered from its source code |
| [hackingBuddyGPT](frameworks/hackingbuddygpt) | TU Wien research framework (wintermute) for LLM-driven Linux and Windows privilege escalation, web, API and Active Directory testing | No public report of use in an attack. Covered from its source code |
| [AutoPentester](frameworks/autopentester) | Research fork of the 2024 PentestGPT that runs the model's commands and Metasploit on the host without approval | No public report of use in an attack. Covered from its source code |
| [MAPTA](frameworks/mapta) | UCL research agent: a gpt-5 main agent directs sandbox and validator agents against web targets | No public report of use in an attack. Covered from its source code |
| [PentesterFlow](frameworks/pentesterflow) | Human-in-the-loop terminal agent for pentests and bug bounty with a shell, a browser and a Burp bridge | No public report of use in an attack. Covered from its source code |
| [Pentest Agent Suite](frameworks/pentest-agent-suite) | Bug bounty framework of about 50 agents, skills and hooks for seven coding assistants, with a stealth browser | No public report of use in an attack. Covered from its source code |
| [Cain](frameworks/cain) | Cloud-focused agent on the Claude Agent SDK with recon, test and report stages, Chinese clouds included | No public report of use in an attack. Covered from its source code |
| [KaliGPT](frameworks/kaligpt) | Command-line AI assistant for Kali with several model providers, aimed at students | No public report of use in an attack. Covered from its source code |
| [Pensar Apex](frameworks/apex) | Pensar's terminal pentest agent for black-box and white-box web testing, open core of a commercial console | No public report of use in an attack. Covered from its source code |
| [MCP for Security](frameworks/mcp-for-security) | Archived collection of 23 stdio MCP servers that let an AI assistant run nmap, nuclei, sqlmap and other tools | No public report of use in an attack. Covered from its source code |
| [Guardian](frameworks/guardian) | AI pentest CLI whose planner, tool-selector, analyst and reporter agents run YAML tool workflows | No public report of use in an attack. Covered from its source code |
| [Cybermes](frameworks/cybermes) | Bug bounty and red team framework built on Hermes Agent, with about two hundred playbooks | No public report of use in an attack. Covered from its source code |
| [ARTEMIS](frameworks/artemis) | Stanford research agent: a supervisor that starts and watches vulnerability-hunting sub-agents | No public report of use in an attack. Covered from its source code |
| [CTF Agent](frameworks/ctf-agent) | CTF solver that races several models in parallel Docker sandboxes | No public report of use in an attack. Covered from its source code |
| [OpenHack](frameworks/openhack) | Local source-code security scanner whose agents map, review, challenge and judge findings | No public report of use in an attack. Covered from its source code |
| [RedAI](frameworks/redai) | Terminal workbench where Claude Code or Codex agents scan code and validate flaws in a browser or simulator | No public report of use in an attack. Covered from its source code |
| [Cochise](frameworks/cochise) | Research prototype that drives an assumed-breach attack on an Active Directory lab from an attacker host | No public report of use in an attack. Covered from its source code |
| [Deadend CLI](frameworks/deadend-cli) | Black-box web pentest agent: a Bun terminal UI driving a Python agent with sandboxed tools | No public report of use in an attack. Covered from its source code |
| [Ares](frameworks/ares) | Dreadnode's Rust multi-agent platform that chains impacket, NetExec and certipy against Active Directory; it ships its own detections | No public report of use in an attack. Covered from its source code |
| [Offensive prompt packs](frameworks/prompt-packs) | Four projects with no agent code (Claude-Red, pentest-ai-agents, codex-redteam-mode, vibe-pentest) that turn a coding agent into an attack tool | No public report of use in an attack. Covered from their files |
| [AutoCVE](frameworks/autocve) | Python platform whose agents audit source code for new CVEs and write reports, with a web console and an API | No public report of use in an attack. 256 consoles exposed |
| [CyberStrike](frameworks/cyberstrike) | TypeScript offensive security harness with a browser extension, an injection prober and a callback server. Not CyberStrikeAI | No public report of use in an attack |
| [Vulnerator AI](frameworks/vulnerator-ai) | OODA-loop pentest agent: a Hermes Agent brain driving Kali and Playwright tools over MCP | No public report of use in an attack |
| [Decepticon](frameworks/decepticon) | Python LangGraph red team agents that run commands in a Kali sandbox and start Sliver or Havoc C2 on demand; telemetry on unless the operator declines | No public report of use in an attack. Covered from its source code |
| [AIDA](frameworks/aida) | Platform that turns a model into a pentester working from its own tool container or Exegol | No public report of use in an attack |
| [BugTraceAI](frameworks/bugtraceai) | Self-hosted web pentest suite with agentic scanning, a web interface and an API | No public report of use in an attack |
| [Z3r0](frameworks/z3r0) | Red team workbench with specialist agents, sandboxed tools and an egress proxy pool | No public report of use in an attack |
| [AIRecon](frameworks/airecon) | Recon agent on a local Ollama model with a Kali sandbox and its own search instance | No public report of use in an attack |
| [LuaN1aoAgent](frameworks/luan1aoagent) | Pentest agent with graph-based planning (鸾鸟), a web workbench and a chisel tunnel | No public report of use in an attack |
| [open-kritt](frameworks/open-kritt) | Vulnerability research tool whose agents find and validate flaws in code | No public report of use in an attack |
| [REDCELL](frameworks/redcell) | LangGraph agents that run a pentest end to end in a Kali container, with chisel pivoting | No public report of use in an attack |
| [T3MP3ST](frameworks/t3mp3st) | Multi-agent red teaming harness with a command and control dashboard | No public report of use in an attack |
| [WormXGPT](frameworks/wormxgpt) | Hacker-themed web terminal and MCP bridge calling security tools. Not the criminal WormGPT service | No public report of use in an attack |
| [Chufeng Agent Core](frameworks/chufeng) | Agent workspace (楚锋 Agent Core) for CTF, forensics and pentest tasks; no public code | No public report of use in an attack. Built from scan data |

Each folder has a README, the hunts and rules, and a `source-code-signals.md` that explains what the framework leaves behind and where in its code each signal comes from.

## Exposure tracking as of 26 Nov 2026

This project now tracks exposure as well as detection: which of these frameworks have consoles reachable on the internet, how many there are, and how the numbers move over time.

![Active autonomous offensive frameworks: live consoles for the top 30 on 26 November 2026](docs/images/exposure-status-2026-11-26.png)

## Exposed consoles

Hosts on the internet that serve the web console of each framework, found in internet-wide scan data with the fingerprints the rules here use: favicons, page titles, page text that survives a rename, and certificates.

<!-- exposed-consoles:start -->
An active host is one seen in the seven days to November 25, 2026. Hosts the scan data flags as honeypots are left out of both counts.

| Framework | Active hosts | Seen in 30 days | Honeypots left out |
|---|---|---|---|
| [ARTEX](frameworks/artex) and AntiFox | 2,478 | 3,273 | 0 |
| [Hermes Agent](frameworks/hermes) | 1,220 | 2,226 | 0 |
| [CyberStrikeAI](frameworks/cyberstrikeai) | 492 | 669 | 513 |
| [PentAGI](frameworks/pentagi) | 309 | 493 | 0 |
| [AutoCVE](frameworks/autocve) | 147 | 256 | 0 |
| [Agent Zero](frameworks/agent-zero) | 113 | 173 | 0 |
| [HexStrike](frameworks/hexstrike) | 30 | 35 | 0 |
| [RedAmon](frameworks/redamon) | 22 | 38 | 0 |
| [Cairn](frameworks/cairn) | 19 | 32 | 0 |
| [Xalgorix](frameworks/xalgorix) | 6 | 12 | 0 |
| [CyberStrike](frameworks/cyberstrike) | 5 | 8 | 0 |
| [Strix](frameworks/strix) | 4 | 13 | 0 |
| [AIDA](frameworks/aida) | 4 | 5 | 0 |
| [BugTraceAI](frameworks/bugtraceai) | 3 | 3 | 0 |
| [Chufeng Agent Core](frameworks/chufeng) | 2 | 4 | 0 |
| [Z3r0](frameworks/z3r0) | 1 | 4 | 0 |
| [Decepticon](frameworks/decepticon) | 1 | 3 | 0 |
| [AIRecon](frameworks/airecon) | 1 | 2 | 0 |
| [LuaN1aoAgent](frameworks/luan1aoagent) | 1 | 1 | 0 |
| [open-kritt](frameworks/open-kritt) | 1 | 1 | 0 |
| [T3MP3ST](frameworks/t3mp3st) | 1 | 1 | 0 |
| [Vulnerator AI](frameworks/vulnerator-ai) | 0 | 21 | 0 |
| [REDCELL](frameworks/redcell) | 0 | 1 | 0 |
| [WormXGPT](frameworks/wormxgpt) | 0 | 1 | 0 |
| [Pentest Swarm AI](frameworks/pentestswarm) | 0 | 0 | 0 |
| All, each host counted once | 4,699 | 6,982 | 513 |
<!-- exposed-consoles:end -->

- An exposed console is not proof of an attack. Researchers, students and security teams run these tools, and some hosts are decoys.
- More than half of the pages titled CyberStrikeAI are flagged as honeypots, which is why that count is so much lower than a title search suggests. The flagged hosts stay in the list with `honeypot` set to `yes`.
- One network that is not flagged holds more than a third of the remaining CyberStrikeAI hosts and of the PentAGI hosts. That looks like one mass deployment or a set of decoys, not that many operators ([#21](https://github.com/ThreatDefence/agentic-attack-detections/issues/21)).
- The ARTEX count includes 229 services found by the Basic auth realm or the `X-Artex-Auth` header put in front of the console. The realm match is probable, not certain: Artex is also a company name ([details](exposure/README.md#artex-behind-basic-authentication)).
- Consoles behind Basic auth named after the tool are common: 63 realms named Cairn, 14 CyberStrikeAI, 8 PentAGI and a few for other frameworks. They now have searches, marked probable, and are added to the counts at the next full snapshot ([plain-word searches](exposure/README.md#plain-word-searches)).
- These counts come from Shodan alone. Consoles on ports Shodan has not scanned are not in them.
- Hermes Agent and Agent Zero are general-purpose agents, so their numbers count exposed agents, not attack consoles. Hermes is counted on its default port only.
- Consoles served by host name only, or on a port that has not been scanned, are not counted.

The addresses, with the port and the fingerprint that found each one, are in [`exposure/consoles.csv`](exposure/consoles.csv). The counts of every snapshot are kept in [`exposure/history.csv`](exposure/history.csv), so the numbers can be followed over time.

## Behaviors shared by agentic frameworks

[`behaviors/`](behaviors) holds rules that do not depend on any one framework.

| Behavior | Rules |
|---|---|
| An attack tool started under an AI agent CLI (Claude Code, Qwen Code, Kimi CLI, Codex, opencode, Hermes, Cairn's workers), found from the process tree | KQL hunt, Sentinel analytics rule, Sigma |
| Servers calling AI model APIs: Chinese models (DeepSeek, Qwen, GLM, Kimi, MiniMax, Doubao), gateways such as OpenRouter, and Western APIs | KQL hunt, Sigma, Zeek |
| An AI agent's page reader on your application next to attack traffic from the same address | KQL hunt, Sigma, Suricata, Zeek |
| One site probed through hundreds of rotating proxy addresses | KQL hunt, Zeek |
| Configuration and backup file theft, and probes of WSUS administration services | KQL hunt, Sentinel analytics rule, Sigma, Suricata, Zeek |
| An attack agent proving it can write to a container registry: a push to a repository named after the agent (`artex-i<id>`, `hermes-h<id>-writecheck`, `csai-push-probe`) or after a write test | Sigma, Suricata |
| Kerberos user enumeration and SMB or WinRM fan-out from one address | KQL hunt, Zeek |
| A server calling back to the address that was probing it | KQL hunt, Suricata (PentAGI worker ports), Zeek |
| AWS credentials used at machine speed from one outside address | KQL hunt |
| Servers looking up out-of-band testing domains (interactsh, Burp Collaborator, dnslog.cn), which means a blind payload ran on them | KQL hunt, Sigma, Suricata, Zeek |
| Password spray at a fixed rhythm, and a spray that ends in a successful sign-in | KQL hunt, Sentinel analytics rule, Sigma |
| Applications that guess secrets, change their own access, ask for tokens to many resources in minutes or call Microsoft Graph in scripted bursts | KQL hunt, Sentinel analytics rule, Sigma |
| Bulk Key Vault secret reads, Azure validate sweeps and Kudu sign-ins | KQL hunt, Sigma |
| AWS permission testing: DryRun bursts, denied sweeps, CloudWatch searches for secrets and Cognito sign-up abuse | KQL hunt, Sentinel analytics rule, Sigma |
| Several unrelated user agent families from one address inside a minute | KQL hunt |
| Web or database services starting shells or download tools, and scripts written into web folders | KQL hunt, Sigma |
| Servers calling webhook, callback and tunnel services, or downloading from a bare address | KQL hunt, Sigma |
| Kerberoasting, DCSync, WinRM fan-out and remote tasklist fan-out | KQL hunt, Sigma |
| Windows system binary names such as svchost.exe running from the wrong folder | Sigma |
| Unsigned (alg none) tokens sent to a web application | Sigma |

The rows from the password spray down were first written for ARTEX. Nothing in them depends on ARTEX, so they now sit here and run for every framework. [`frameworks/artex/README.md`](frameworks/artex/README.md#moved-to-shared-behaviors) lists where each one went.

## Indicators and feeds

| Path | Contents |
|---|---|
| [`iocs/indicators.csv`](iocs/indicators.csv) | 580 indicators. Each row has its source, confidence, a review date for addresses and the framework it belongs to. This file is the single source for every feed |
| [`feeds/`](feeds) | Generated from the CSV: a STIX 2.1 bundle, a Sentinel watchlist, and plain lists of addresses, domains, URLs, user agents and hashes. The plain lists carry medium and high confidence rows only. Low confidence rows, which include legitimate services such as AI model APIs, need their context, so they stay in the CSV, the watchlist and the STIX bundle. [`feeds/by-framework/`](feeds/by-framework) has the lists per framework |
| [`docs/attack-coverage.md`](docs/attack-coverage.md) | Which rules cover which ATT&CK technique, per framework, with the [Navigator layers](generated/attack) beside it. `tools/build_attack.py` rebuilds both |
| [`generated/`](generated) | Every Sigma rule converted to Splunk SPL, Sentinel and Defender XDR KQL, and Elastic ES\|QL and Lucene, plus [`elastic_ndjson/`](generated/elastic_ndjson) where all of them are one Kibana import. [`SUMMARY.md`](generated/SUMMARY.md) shows which rule converted to which backend |
| [`waf/`](waf) | Cloudflare, AWS WAF and Azure WAF rules built from [`waf/signatures.yaml`](waf/signatures.yaml) and the CSV. The [README](waf/README.md) covers when a WAF helps and how to deploy |
| [`exposure/`](exposure) | Addresses where one of these consoles answers on the internet, with the fingerprint that found each one. A record of what is exposed, kept out of the feeds above. `tools/exposure_snapshot.py` rebuilds it |
| [`tools/`](tools) | Feed builder, WAF builder, release file builder, Sigma converter and validator |
| [`docs/`](docs) | [Adding a framework](docs/adding-a-framework.md) |

Each framework folder also has a Zeek intel file and Suricata address rules, generated from the same CSV.

### Downloads

Each [release](https://github.com/ThreatDefence/agentic-attack-detections/releases) carries the same content as ready-made files. These links always point at the newest release:

| File | Contents |
|---|---|
| [`agentic-attack-detections.rules`](https://github.com/ThreatDefence/agentic-attack-detections/releases/latest/download/agentic-attack-detections.rules) | Every Suricata rule in one file |
| [`agentic-attack-detections-yara.zip`](https://github.com/ThreatDefence/agentic-attack-detections/releases/latest/download/agentic-attack-detections-yara.zip) | The YARA rules, one file per framework |
| [`agentic-attack-detections-stix2.1.json`](https://github.com/ThreatDefence/agentic-attack-detections/releases/latest/download/agentic-attack-detections-stix2.1.json) | The STIX 2.1 bundle |
| [`agentic-attack-detections-indicators.zip`](https://github.com/ThreatDefence/agentic-attack-detections/releases/latest/download/agentic-attack-detections-indicators.zip) | `iocs/indicators.csv` and every feed |
| [`agentic-attack-detections-waf.zip`](https://github.com/ThreatDefence/agentic-attack-detections/releases/latest/download/agentic-attack-detections-waf.zip) | The WAF rules for Cloudflare, AWS WAF and Azure WAF |
| [`SHA256SUMS.txt`](https://github.com/ThreatDefence/agentic-attack-detections/releases/latest/download/SHA256SUMS.txt) | SHA-256 of each file |

To keep the Suricata rules current with `suricata-update`:

```bash
suricata-update add-source agentic-attack-detections https://github.com/ThreatDefence/agentic-attack-detections/releases/latest/download/agentic-attack-detections.rules
suricata-update
```

## How the rules are built and checked

- **From source.** A signature string comes from the framework's code at a named commit, or from a report that published it. `source-code-signals.md` gives the file and line.
- **From reports.** Every indicator names the report it came from, and each value was checked against that report.
- **Tested.** Each Suricata rule has a test flow and a near miss. Sigma rules are run against test events, and hunts and analytics rules against sample rows in Microsoft's Kusto emulator, where cases have been written: [`tests/sigma/`](tests/sigma) and [`tests/kql/`](tests/kql) have the formats, and new cases are welcome. Every KQL query is parsed with Microsoft's parser and its columns checked. The Zeek packages run against test captures. CI runs the validator and the Suricata, Sigma and KQL tests on every push.
- **Run on real data.** Version 2.7.0 was run over 30 days of production telemetry to remove rules that could not fire and to quiet rules that were noisy.

No rule has been tested against a captured intrusion. Treat every rule as experimental until it has run on your own data.

## Confidence

| Level | Meaning | How to use it |
|---|---|---|
| High | Specific to a framework or its campaigns: its user agents, command lines, paths, prompt strings, probe artifacts, reported C2 and skimmer domains, file hashes | Alert |
| Medium | Seen in a framework's activity but changeable or shared: reported addresses, generic agent user agents, prompt text shared with similar tools | Alert after baselining, or hunt |
| Low | Common or legitimate: AI model APIs, request-collection services, agent CLI user agents, hosting ASNs, proxy providers | Context for another finding only |

Addresses from cloud and VPS providers are reassigned. Every address has a `review_by` date 180 days after it was reported. Addresses past that date are left out of the plain lists, the Zeek intel files and the Suricata address rules; they stay in the CSV and the STIX bundle as history. Check before you block.

## Testing

```bash
pip install -r tools/requirements.txt
python3 tools/validate.py                              # Sigma, KQL, ES|QL, TOML, Splunk, Suricata SIDs, YARA, Zeek, osquery, feeds
python3 -m unittest discover -s tests/feeds            # indicator confidence, expiry and SID policy
python3 tests/sigma/run_tests.py                       # Sigma rules against test events
python3 tests/kql/run_tests.py                         # hunts and analytics against sample rows; needs the Kusto emulator
lab/lab.sh up && lab/lab.sh test                       # Splunk and Elastic in Docker, every rule against test events: lab/README.md
python3 tools/convert.py                               # regenerate generated/
python3 -m venv .venv-s1 && .venv-s1/bin/pip install -r tools/requirements-sentinelone.txt
.venv-s1/bin/python tools/convert_sentinelone.py       # regenerate generated/sentinelone/ (needs pySigma 0.11, so its own venv)
python3 -m venv .venv-cortex && .venv-cortex/bin/pip install -r tools/requirements-cortex.txt
.venv-cortex/bin/python tools/convert_cortex.py        # regenerate generated/cortex_xdr/ (needs pySigma 0.11, so its own venv)
python3 tools/build_waf.py                             # regenerate waf/
python3 tests/waf/run_tests.py                         # offline checks of the WAF rules on every platform
python3 tools/build_release.py                         # build the release downloads into dist/
python3 tests/suricata/run_tests.py                    # one flow per Suricata rule; needs suricata and openssl
bash frameworks/artex/tests/run_network_tests.sh       # needs zeek and suricata on the PATH
bash behaviors/zeek/tests/run_tests.sh                 # needs zeek on the PATH
bash frameworks/cyberstrikeai/tests/run_zeek_tests.sh  # needs zeek on the PATH
bash frameworks/hexstrike/tests/run_zeek_tests.sh      # needs zeek on the PATH
bash frameworks/autocve/tests/run_zeek_tests.sh        # needs zeek on the PATH
bash frameworks/luan1aoagent/tests/run_zeek_tests.sh   # needs zeek on the PATH
```

`validate.py` checks KQL with Microsoft's parser when `dotnet` is installed, and Zeek scripts when `zeek` is.

## Contributing

Pull requests are welcome. The most useful contributions are false-positive reports, missed detections and new indicators with a source. See [CONTRIBUTING.md](CONTRIBUTING.md) for the checks to run, and [docs/adding-a-framework.md](docs/adding-a-framework.md) to add a framework.

Where the project is going is in the [roadmap](ROADMAP.md).

## Getting help

Open an [issue](https://github.com/ThreatDefence/agentic-attack-detections/issues). There are forms for a false positive, a missed detection, a new indicator and a new framework. Describe the shape of the event; never paste customer data. For a security problem in the repository itself, see [SECURITY.md](SECURITY.md).

## License

MIT. See [LICENSE](LICENSE). Each indicator names its source in `iocs/indicators.csv`. Use them in your own products and feeds.

## Disclaimer

Provided as is, without warranty. A match is a lead, not a verdict. Absence of a match does not mean absence of compromise.

## Credits

Started and maintained by [ThreatDefence](https://threatdefence.com), with fixes from [@jack51706](https://github.com/jack51706), [@tridev-tej](https://github.com/tridev-tej), [@Bropdrop](https://github.com/Bropdrop), [@blitzcrieg1](https://github.com/blitzcrieg1) and [@nerdyhacks](https://github.com/nerdyhacks). Background on ARTEX: the [advisory](https://threatdefence.com/article/artex-agentic-attack-framework-detection-hunting) and the [source code signals article](https://threatdefence.com/article/artex-source-code-signals-deep-hunting).

## History

This repository was `artex-detections`. GitHub redirects the old address. The ARTEX rules moved from the root to [`frameworks/artex/`](frameworks/artex); `iocs/indicators.csv` and `feeds/` kept their paths.
