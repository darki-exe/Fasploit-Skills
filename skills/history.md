# Roblox Exploit Ecosystem — Context & Terminology

> Use when asked about executor history, Hyperion/Byfron, why executors died, what UNC is, or any scene context question. Contains verified dates and facts about Synapse X, KRNL, V3rmillion, Script-Ware, and the Delta API dump.

**Source:** `skill.md v3` — sections §0  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 0. FUN FACTS & SCENE HISTORY

> Verified facts about the Roblox exploit ecosystem — what happened, when, and why it matters.

---

### The Hyperion / Byfron Acquisition — The Day Everything Changed

In **October 2022**, Roblox Corporation acquired **Byfron Technologies** for **$11.6 million**.
Byfron had previously shipped Hyperion — an anti-tamper system — to **Fortnite** and **Apex Legends** before Roblox even came calling.

On **May 3, 2023**, Hyperion went live in the 64-bit Roblox client.

What Hyperion actually does that makes executors hard:
- Kernel-adjacent integrity checks at the process level
- Behavioral heuristics monitoring API calls and memory access in real time
- Detects unauthorized code injection **before** a script can attach
- Crashes the client immediately on detection (no grace period, no warning)
- Broke Wine/Proton compatibility — Linux players lost access for a period
- In **May 2025**, Roblox expanded Hyperion to detect modified clients and issue account terminations automatically

Before Hyperion, running a free executor was a casual afternoon activity.
After Hyperion, maintaining a working executor requires reversing an obfuscated kernel-level system that gets patched constantly.
The arms race is still ongoing, but the cost-to-exploit ratio shifted decisively toward Roblox.

---

### Synapse X — The Fall of the King

**Synapse X** was the gold standard of paid Roblox executors for years. $20, premium features, the best script compatibility available, and a full debug library. Scripters used it even for legitimate game development and testing because its environment was just that clean.

On **October 27, 2023**, Synapse Softworks LLC announced the discontinuation of Synapse X as part of a **new partnership with Roblox Corporation**.
Synapse Softworks now works on Roblox's own anti-cheat and security infrastructure.
All user data was deleted from their records.

The announcement came less than a week after the V3rmillion shutdown.
The community had never seen two pillars collapse in the same week.

> Synapse X going from the most feared executor to literally building Roblox's defenses is one of the wildest pivots in gaming history.

---

### KRNL — The Free Tier Legend

**KRNL** launched in **2019** under the developer known as **Ice Bear**, distributed through **WeAreDevs**.
At its peak it ran a Level 8 engine with ~98% UNC compliance — almost unheard of for a free executor.

Its defining feature (and main complaint) was the **key system**: users had to grind through Linkvertise ad walls every 24 hours to get a temporary access key. Ice Bear monetized through Linkvertise ad revenue. Entire WeAreDevs threads existed just to complain about it.

The original team **shut KRNL down between 2023 and 2024** following Byfron's rollout. The official Ice Bear announcement is preserved on the Internet Archive.

What exists today under the "KRNL" name is unverified mirrors operated by unknown parties. None have proven a chain of custody from the original team. Some have been flagged as distributing different (potentially malicious) binaries under the KRNL brand.

---

### V3rmillion — The Hub Goes Dark

**V3rmillion** was the central forum of the Roblox exploit community for roughly 12 years. Scripts, executor releases, leaks, community drama, dev recruitment — everything went through V3rm.

On **October 23, 2023** — four days before the Synapse X announcement — the V3rmillion owners announced plans to shut down, delete all user data, and sell the domain. The stated reasons included admin burnout from constantly fighting scammers, removing malware disguised as executors, and moderating millions of posts. Hyperion making the whole scene less viable accelerated the decision.

A buyer was found on **November 13, 2023**. On November 19, the forum was reset and the domain redirected to a new site using the V3rmillion branding. The original community data is gone; a public archive was later published and is accessible via GitHub tools.

> October 2023 was the month V3rmillion announced shutdown, Synapse X closed, and Roblox's ban waves hit hundreds of thousands of accounts. The community called it the end of the Golden Age.

---

### UNC — Unified Naming Convention

Before UNC, cross-executor scripting was a nightmare. Every executor named its functions differently:

```lua
-- Just to check if a function is an executor closure, you needed:
local is_executor_closure = is_syn_closure or is_fluxus_closure or is_sentinel_closure
    or is_krnl_closure or is_proto_closure or is_calamari_closure
    or is_electron_closure or is_elysian_closure
```

**Script-Ware** introduced and documented UNC on **April 25, 2022** to solve this. The standard defined consistent function names across executors so scripts could be portable without 20-line compatibility checks.

On **May 4, 2024**, UNC was officially **discontinued** — its GitHub archived — following the broader collapse of the exploit scene triggered by Hyperion and the Synapse X partnership.

Several successors exist (sUNC and others), but none have reached the adoption the original had.
Delta, as seen in the API dump used in this document, still follows UNC naming conventions.

---

### Script-Ware — The Other Premium Player

Script-Ware was one of the dominant paid executors alongside Synapse X before the Byfron era. It was also the team that founded and maintained UNC.

Following the Hyperion rollout, Script-Ware was **shut down**. Being the founder of UNC didn't protect it — when the platform made injection fundamentally harder, the maintenance cost outweighed the revenue.

---

### The Delta Executor API Dump

The executor API used as the primary reference for this document is **Delta** (version 73801).
Delta is cross-platform — it supports Windows, Android, iOS, and macOS (`Delta.is_android()`, `Delta.is_ios()`, `Delta.is_mac()`).
Its API surface is large: **373 unique function objects**, **24 unique table objects** across the environment.

Most functions documented here exist on other modern executors under the same or aliased names.
Where function names vary, UNC-compliant portability patterns are noted.

---
