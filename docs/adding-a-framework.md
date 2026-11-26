# Adding a framework

Every framework follows the same shape, so the tools, CI and feeds pick it up without changes beyond one line in the feed builder.

## Layout

```text
frameworks/<name>/
  README.md                 what the framework is, what is in the folder, quick start
  source-code-signals.md    what its code leaves behind, with file references and how hard each signal is to remove
  coverage.md               behavior by platform
  hunting-order.md          where to start
  sentinel/hunting/*.kql    numbered KQL hunts, NN-short-name.kql
  sentinel/analytics/*.yaml Sentinel scheduled rule templates
  sigma/<category>/*.yml    Sigma rules
  suricata/<name>.rules     hand-written rules
  yara/<name>.yar           YARA rules
```

`frameworks/<name>/suricata/<name>-ip.rules` and `frameworks/<name>/zeek/intel/<name>.intel` are generated. Do not edit them by hand.

## Steps

1. Add the framework to `FRAMEWORKS` in `tools/build_feeds.py`: display name, aliases, a one-line description, the actor label used for the framework's own operators, and the next free Suricata address-rule SID base.
2. Add its indicators to `iocs/indicators.csv` with `framework` set to its folder name. An indicator shared with another framework keeps one row with both names, space separated.
3. Put rules that would work for any agentic framework in `behaviors/` rather than in the framework folder. Coding-agent abuse, tool bursts, proxy rotation and post-exploitation patterns usually belong there.
4. Give every rule its ATT&CK techniques in the format [CONTRIBUTING.md](../CONTRIBUTING.md) lists for its kind.
5. Run `python3 tools/build_feeds.py`, `python3 tools/convert.py`, `tools/convert_sentinelone.py` and `tools/convert_cortex.py` (each in its own venv, see their requirements files), `python3 tools/build_attack.py` and `python3 tools/validate.py`, and commit everything they regenerate. Add one case per Suricata rule, and a near miss, to `tests/suricata/run_tests.py`: the run fails if a rule has no case.
6. Add the framework to the tables in the root `README.md`.

## Number ranges

Suricata rule IDs (SIDs) come from the block 3800000 to 3819999, the range requested for this project at [sidallocation.org](https://sidallocation.org) so the rules don't collide with other rule sets. Each framework gets a block of 1,000.

| Framework | Suricata rules | Suricata address rules |
|---|---|---|
| ARTEX | 3800001 to 3800499 | from 3800500 |
| Cairn | 3801001 to 3801499 | from 3801500 |
| PentAGI | 3802001 to 3802499 | from 3802500 |
| HexStrike | 3803001 to 3803499 | from 3803500 |
| CyberStrikeAI | 3804001 to 3804499 | from 3804500 |
| Strix | 3805001 to 3805499 | from 3805500 |
| Hermes Agent | 3806001 to 3806499 | from 3806500 |
| Agent Zero | 3807001 to 3807499 | from 3807500 |
| RedAmon | 3808001 to 3808499 | from 3808500 |
| Xalgorix | 3809001 to 3809499 | from 3809500 |
| Pentest Swarm AI | 3810001 to 3810499 | from 3810500 |
| Console-level frameworks added November 25, 2026 | blocks of 100 from 3811000: rules base + 1 to base + 49 | from base + 50 |
| Shannon | 3813001 to 3813099 | from 3813500 |
| PentestGPT | 3814001 to 3814499 | from 3814500 |
| Cyber-AutoAgent | 3815001 to 3815499 | from 3815500 |
| CAI | 3816001 to 3816499 | from 3816500 |
| Frameworks added November 26, 2026 with a block of 100 | see below | |
| Next framework | no full block left: smaller blocks, see the list below | |

The block from 3811000 to 3812399 is split into blocks of 100 for frameworks that were added from the exposure snapshot with few or no rules: AutoCVE 3811000, CyberStrike 3811100, Vulnerator AI 3811200, Decepticon 3811300, AIDA 3811400, BugTraceAI 3811500, Z3r0 3811600, AIRecon 3811700, LuaN1aoAgent 3811800, open-kritt 3811900, REDCELL 3812000, T3MP3ST 3812100, WormXGPT 3812200 and Chufeng Agent Core 3812300. AutoCVE's rules are 3811001 to 3811049 and its address rules start at 3811050, and so on. A framework that outgrows its 100 moves to a full block of 1,000. The rest of that block went to frameworks added on November 26, 2026: HackSynth 3812400, VulnBot 3812500, AutoPT 3812600, PentestAgent (the arXiv 2411.05185 code) 3812700, PentestAgent (GH05TCREW) 3812800 and Dark Moon 3812900. The block from 3817000 is split the same way: NyxStrike 3817000, sapper-v2 3817100, NeuroSploit 3817200, Vigilo 3817400, Pentest Copilot 3817500, cyber-harness 3817600, CHYing-agent 3817700 and Hexstrike-redteam 3817900; 3817300 and 3817800 stay free. The block from 3818000 is split the same way: pentest-ai 3818000, LazyOwn 3818100, H-Pentest 3818200, Incalmo 3818300, Virga 3818400, Nebula 3818500, MCP Kali Server 3818600, burp-ai-agent 3818700, Pentest Harness 3818800 and Mitsu 3818900. The block from 3819000 is split the same way: Blitz Strike 3819000, Sage (Mythic) 3819100, 0security 3819200, BreachWeave 3819300, ARIA-X 3819400, Clearwing 3819500, numasec 3819600, Agentic Bug Hunter 3819700, PentestCode 3819800 and RAPTOR 3819900. That fills the range requested for this project: frameworks added after this get smaller blocks carved from unused space. Decepticon keeps its block at 3811300 now that it has a full folder.

Rules in `behaviors/suricata/` use numbers inside the Cairn block (3801010 to 3801059), where they were first published. Sigma and Sentinel rule ids are UUIDv5 values derived from the file path, so they stay stable as long as a file is not renamed.

### Blocks of 20

The range requested for this project is full, so frameworks added from November 26, 2026 get a block of 20 carved from unused space. Hand-written Suricata rules use base + 1 to base + 15, and the address-rule base (the `sid` in `FRAMEWORKS`) is base + 16; address rules are generated only for frameworks with IP indicators. Shannon's block was cut to 3813001 to 3813099 (it uses three rules) to make room for them, and its address rules still start at 3813500.

| Framework | Block |
|---|---|
| hackingBuddyGPT | 3813100 to 3813119 |
| AutoPentester | 3813120 to 3813139 |
| MAPTA | 3813140 to 3813159 |
| PentesterFlow | 3813160 to 3813179 |
| Pentest Agent Suite | 3813180 to 3813199 |
| Cain | 3813200 to 3813219 |
| KaliGPT | 3813220 to 3813239 |
| Pensar Apex | 3813240 to 3813259 |
| MCP for Security | 3813260 to 3813279 |
| Guardian | 3813280 to 3813299 |
| Cybermes | 3813300 to 3813319 |
| ARTEMIS | 3813320 to 3813339 |
| CTF Agent | 3813340 to 3813359 |
| OpenHack | 3813360 to 3813379 |
| RedAI | 3813380 to 3813399 |
| Cochise | 3813400 to 3813419 |
| Deadend CLI | 3813420 to 3813439 |
| Ares | 3813440 to 3813459 |
| Offensive prompt packs (no Suricata rules) | 3817300 to 3817319 |

3813460 to 3813499, 3817320 to 3817399 and 3817800 to 3817899 stay free for further blocks of 20.
