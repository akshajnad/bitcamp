# ECHO — Entity-based Cyber History Observer

> A noir pixel-art city where every building is a server, every road is a network connection, and a cyberattack left its echo behind. You are the investigator. A swarm of AI specialists are your field team. Together you reconstruct what happened before critical infrastructure collapses.

---

## Concept

Something happened last night. Money vanished. A building went dark. A citizen was terminated without authorization. The attack is over — but its **echo** remains: traces in logs, fragments of deleted files, anomalous traffic that rippled from machine to machine before anyone noticed.

You have **10 in-game hours** to reconstruct that echo before evidence degrades.

The city *is* the file system. Buildings are servers. Roads are network connections. Abandoned buildings are deleted data. Graffiti is malware. And scattered across the city are four AI specialist agents — **LOGIS, NETARA, CARVER, and CHRONO** — who move through the environment, investigate systems, and broadcast their findings. A central intelligence called **ECHO** aggregates their work into evolving hypotheses, guiding you while requiring your verification to act.

The attack propagated like an echo through connected systems. Intelligence propagates the same way — from specialist to specialist, building toward a complete picture. When you have enough, you take it to the courtroom.

**Target Prizes:**
- Best Game Jam Track — fully playable mystery game
- Best Digital Forensics (Cipher Tech) — teaches 4+ real forensics concepts
- Best Gamification — turns forensics into a tactile game loop
- Best UI/UX — noir pixel city with visible AI agents is unlike anything judges have seen
- Best Moonshot — first forensics game with embodied multi-agent swarm
- Best Use of Featherless.ai — entire agent swarm runs on open-source LLMs
- Best Data Visualization (Peraton) — the city IS a live visualization of system activity

---

## Tech Stack

| Layer     | Tech                                                                   |
|-----------|------------------------------------------------------------------------|
| Frontend  | Phaser 3, Next.js 16, Bun                                              |
| Backend   | FastAPI, LangGraph, Python                                             |
| LLM       | Featherless.ai (OpenAI-compatible, open-source models)                 |
| Assets    | Kenney 1-Bit Pack (CC0, 16×16 px) — monochrome base + Phaser tinting  |
| Map Tool  | Tiled Map Editor → JSON → Phaser Tilemap                               |

**Tileset:** [Kenney 1-Bit Pack](https://kenney.nl/assets/1-bit-pack) — CC0, 1,060 tiles, 16×16, monochrome top-down city with buildings, roads, interiors, and characters.

**Why 1-Bit for ECHO:**
The monochrome base makes the city feel like a **digital ghost** — a memory of a system that once ran. Nothing looks alive until tinting breathes color into it. Color = meaning: the city literally shifts from black-and-white into color as agents investigate. The binary aesthetic mirrors the forensics theme — every tile is either signal or noise, known or unknown.

**Rendering approach — color as meaning:**
All color applied at runtime via Phaser's `setTint()` per building/road based on forensics state:

| State | Tint | Effect |
|---|---|---|
| Unknown / unvisited | `0x334455` (cold blue-grey) | Fades into background |
| Agent investigating | `0x7b61ff` (violet) | Pulses while agent is inside |
| Clean / confirmed | `0xc8b89a` (warm amber) | Comes alive, readable |
| Compromised | `0xff2200` (hard red) | Bleeds into adjacent tiles |
| Evidence found | `0xffcc00` (gold) | Bursts then settles |
| Offline / deleted | `0x111111` (near-black) | Almost invisible |

---

## City ↔ Forensics Mapping

| City Element        | Forensics Concept              | Visual State                                        |
|---------------------|-------------------------------|-----------------------------------------------------|
| Buildings           | Machines / servers             | Lit = active, dark = offline, red = compromised      |
| Roads               | Network connections            | Circuit-trace pulse along edges                     |
| Power grid          | System dependencies            | Blackout = dependency failure                       |
| Citizens (NPCs)     | Running processes              | Walk normally; freeze/dissolve = killed             |
| AI Agent NPCs       | Specialist investigators       | Move to buildings, animate while inspecting         |
| Abandoned buildings | Deleted / corrupted files      | Boarded up, debris, recoverable furniture           |
| Graffiti            | Malware signatures             | Hidden encoded messages on walls                    |
| Security cameras    | System logs                    | Blinking = active log, smashed = wiped              |
| City archives       | Registry artifacts             | Filing cabinets, ownership records                  |
| Underground tunnels | Hidden / encrypted partitions  | Trapdoors, only visible when discovered             |
| Blackouts           | Denial-of-service events       | Block-wide darkness, flickering                     |
| Delivery trucks     | Suspicious network traffic     | Slow-moving glowing sprites between buildings       |

---

## The Agent Swarm — MiroFish Architecture

ECHO uses a **MiroFish-style multi-agent system**: a swarm of specialized AI agents that operate independently in parallel, share discoveries through a common evidence pool, and produce emergent collective intelligence without a central coordinator directing their work. A meta-agent (ECHO) observes the shared pool and synthesizes hypotheses for the player.

Each specialist is an **embodied NPC** in the city — you watch them walk between buildings, see their investigation animations, and receive their findings as speech bubbles and evidence cards.

### The Four Specialists

| Agent | NPC Name | Specialty | What They Do |
|---|---|---|---|
| Log Analyst | **LOGIS** | Access logs, authentication events | Walks to buildings, reads visitor logs, flags anomalous logins, detects brute-force patterns |
| Network Analyst | **NETARA** | Traffic patterns, lateral movement | Patrols road segments, reads delivery manifests, maps suspicious connections between nodes |
| File Analyst | **CARVER** | Deleted data, steganography | Enters abandoned buildings, excavates debris, decodes graffiti, reconstructs fragmented files |
| Timeline Analyst | **CHRONO** | Event sequencing, temporal correlation | Moves between buildings comparing timestamps, draws connection lines when sequences match |

### ECHO — Central Intelligence (Meta-Agent)

ECHO doesn't investigate directly. It observes what the four specialists discover, aggregates their findings into evolving hypotheses, and surfaces them to the player via the `ECHOPanel`. ECHO:
- Synthesizes findings from all four specialists into a coherent attack narrative
- Updates its confidence meter (0–100%) as evidence accumulates
- Surfaces hypotheses: *"Based on LOGIS and NETARA's findings, the initial vector was likely the Bank Vault"*
- Occasionally produces **red herrings** — ~20% of its flags are wrong, mirroring how real AI tools need human verification
- Requires player confirmation before any finding is moved to the accusation board

### MiroFish Pattern — How It Works

```
┌─────────────────────────────────────────────────────────┐
│                   Shared Evidence Pool                  │
│   (append-only log of all specialist findings)          │
└────────┬──────────┬──────────┬──────────┬──────────────┘
         │          │          │          │
      LOGIS      NETARA     CARVER     CHRONO
    (parallel  (parallel  (parallel  (parallel
     subgraph)  subgraph)  subgraph)  subgraph)
         │          │          │          │
         └──────────┴──────────┴──────────┘
                         │
                       ECHO
                  (meta-agent, reads
                   shared pool, synthesizes
                   → hypothesis stream
                   → player guidance)
```

**Key properties:**
- All four specialists run **concurrently** (LangGraph parallel branches)
- No specialist waits on another — they each query the scenario state independently
- When LOGIS finds something relevant to NETARA's domain, it appends a `cross_ref` tag to the shared pool — NETARA picks it up on its next cycle without direct communication
- ECHO runs on a slower cadence (every N findings) to synthesize, not to direct
- Agents can **disagree** — conflicting findings are surfaced to the player as unresolved contradictions

### Agent Attributes

```python
id:           str   # "logis" | "netara" | "carver" | "chrono" | "echo"
specialty:    str
confidence:   float  # 0.0–1.0 — how certain this agent is in its current findings
findings:     list[Finding]   # evidence items discovered so far
current_task: str             # what the agent is currently investigating
position:     tuple[int,int]  # (x, y) grid position in city — drives NPC sprite
cross_refs:   list[str]       # tags pointing to findings other agents should check
```

### Featherless.ai Setup (Backend)

Each specialist and ECHO runs on the same Featherless.ai endpoint, using the same model but with different system prompts:

```python
from langchain_openai import ChatOpenAI

def make_agent_llm() -> ChatOpenAI:
    return ChatOpenAI(
        model=os.getenv("ECHO_MODEL", "Qwen/Qwen3-235B-A22B"),
        api_key=os.getenv("FEATHERLESS_API_KEY"),
        base_url=os.getenv("FEATHERLESS_BASE_URL", "https://api.featherless.ai/v1"),
    )

# One instance per agent — same model, different prompts
logis_llm   = make_agent_llm()
netara_llm  = make_agent_llm()
carver_llm  = make_agent_llm()
chrono_llm  = make_agent_llm()
echo_llm    = make_agent_llm()  # meta-agent, sees full shared evidence pool
```

---

## Building Roster

Each scenario seeds 12–16 buildings. Types drawn from:

### Server Types
| ID | Building | Forensics Role |
|----|----------|----------------|
| `srv_web` | Web Agency | Public-facing server, high inbound traffic |
| `srv_db` | Bank Vault | Database server, sensitive records |
| `srv_auth` | City Hall | Authentication server, login events |
| `srv_mail` | Post Office | Mail server, communication logs |
| `srv_file` | Public Library | File server, document storage |
| `srv_admin` | Police Station | Admin console, elevated privileges |

### Endpoint Types
| ID | Building | Forensics Role |
|----|----------|----------------|
| `end_exec` | Mayor's Mansion | Executive workstation |
| `end_dev` | Workshop | Developer machine, code artifacts |
| `end_intern` | Boarding House | Intern/low-privilege endpoint |
| `end_finance` | Counting House | Finance workstation, transaction logs |

### Infrastructure
| ID | Building | Forensics Role |
|----|----------|----------------|
| `infra_router` | Crossroads Tavern | Network router, all traffic passes through |
| `infra_dns` | Town Crier Tower | DNS server, name resolution logs |
| `infra_vpn` | Underground Tunnel | VPN endpoint, encrypted egress |
| `infra_backup` | Warehouse | Backup server, snapshot history |

---

## Building Attributes

```python
id:               str    # unique slug
building_type:    str    # "server" | "endpoint" | "infrastructure"
name:             str    # display name
status:           str    # "clean" | "compromised" | "offline" | "unknown"
activity_log:     list[LogEntry]   # timestamped access/event records
deleted_files:    list[FileEntry]  # recoverable via file carving mechanic
registry:         dict             # ownership, install history, config keys
graffiti:         str | None       # base64-encoded steganographic payload
connections:      list[str]        # building IDs this node connects to
traffic_anomaly:  float            # 0.0–1.0, NETARA-computed suspicion score
agent_visited:    list[str]        # which specialist agents have inspected this building
```

---

## Gameplay Loop

### 1 — The Incident Report
Player receives a case briefing. The four specialist NPCs spawn at the city entrance and begin fanning out autonomously. ECHO speaks: *"Four agents deployed. I'll synthesize their findings. You have 10 hours."*

### 2 — Watch the Swarm Work
Specialist NPCs move through the city independently. Speech bubbles appear as they make discoveries:
- LOGIS at Bank Vault: *"4,000 login attempts from a single IP. 03:17 AM."*
- NETARA on east road: *"Unusual outbound volume to the Underground Tunnel. No matching return traffic."*
- CARVER in abandoned alley: *"Graffiti decodes to: `DROP TABLE transactions;`"*
- CHRONO: *"LOGIS's timestamp and NETARA's traffic spike are 4 minutes apart. Same direction."*

Findings stream into the `EventFeed` and are pinned to the city map.

### 3 — Investigate Yourself
Click any building to open `BuildingInspectModal` and dig deeper than the agents went:

| Tab | Mechanic | Forensics Concept |
|-----|----------|--------------------|
| **Visitor Log** | Scroll entries, flag suspicious ones | Log analysis |
| **Recovered Files** | Drag debris to restore deleted files | File carving |
| **City Archives** | Browse registry keys, spot unauthorized installs | Registry forensics |
| **Walls** | Decode graffiti cipher | Steganography |

### 4 — Follow the Roads
Roads glow with traffic data. Intensity = volume, color = time of day. Click any road to see the delivery truck manifest (timestamps, data volume, source/destination). Confirmed anomalies get flagged on the map.

### 5 — Reconstruct the Timeline
`TimelineScrubber` spans 10 in-game hours (18:00–04:00). Scrub to watch buildings light up as activity happened. The moment the attack jumped from machine to machine is visible as a red wave. The agent NPCs replay their own positions at each hour — you see when CHRONO first detected the anomaly.

### 6 — Verify ECHO's Hypotheses
ECHO periodically surfaces a hypothesis card: *"Hypothesis: origin = Bank Vault, vector = credential stuffing, path = Bank → Router → Mail. Confidence: 67%."* Player can **Accept** (adds to accusation board), **Reject** (removes from consideration), or **Investigate Further** (sends the relevant specialist back in).

### 7 — Make Your Case
`EvidenceBoard` accumulates confirmed findings. When accusation threshold is met, `CourtroomModal` opens — pixel-art courtroom, judge reviews:
- Origin building (patient zero)
- Attack path (ordered building list)
- Responsible party (suspect name)

Score = accuracy of all three. Partial credit available.

---

## Forensics Mechanics (Detail)

### File Carving
Abandoned buildings have pixel debris sprites. Drag debris → resolves into recovered `FileEntry` (name, type, partial content, timestamp). CARVER may already be inside doing the same — player and agent can recover different fragments.

### Steganography
Graffiti on building walls → click → pixel grid mini-game. Select every Nth pixel to extract a binary string, decoded by CARVER into readable text.

### Traffic Analysis
Road segments → click → volume-over-time chart (Phaser `Graphics`, no external lib). NETARA's finding appears as an annotation on the same chart.

### Log Timeline Correlation
Pin two buildings' logs side-by-side in timeline view. Matching timestamps draw a connector line. CHRONO does this automatically for its highest-suspicion pairs — player can do it manually for any pair.

---

## LangGraph Architecture

```
Case Briefing Request
        ↓
[Scenario Generator Node]  — LLM authors mystery: buildings, logs, planted evidence,
                             attack path, red herrings, correct answer key
        ↓
[City Seeder Node]         — populates building attributes, traffic, deleted files
        ↓
[WebSocket: init]          — sends full city state + agent spawn positions to frontend
        ↓
┌──────────────────────────────────────────────────┐
│  MiroFish Swarm (parallel LangGraph branches)    │
│                                                  │
│  [LOGIS Node]   [NETARA Node]  [CARVER Node]     │
│      ↓               ↓              ↓            │
│  log analysis   traffic scan   file recovery     │
│      └───────────────┴──────────────┘            │
│               Shared Evidence Pool               │
│                      ↓                           │
│              [CHRONO Node]                       │
│           temporal correlation                   │
└──────────────────────┬───────────────────────────┘
                       ↓
              [ECHO Meta-Agent Node]
          synthesizes → hypothesis stream
                       ↓
         WebSocket: echo:hypothesis → ECHOPanel
                       ↓
              [Investigation Loop]
   ├─ /building/inspect   → [Log Retrieval Node]
   ├─ /traffic/analyze    → [Traffic Node]
   ├─ /echo/verify        → [ECHO re-evaluation Node]
   └─ /accusation/submit  → [Scoring Node]
                       ↓
            [WebSocket: verdict]
```

---

## Visual Style

**Palette:**
| Role | Color |
|------|-------|
| Background / base | `#0a0a0f` |
| Unknown building | `0x334455` (cold blue-grey) |
| Agent investigating | `0x7b61ff` (violet pulse) |
| Clean / confirmed | `0xc8b89a` (warm amber) |
| Compromised | `0xff2200` with bleed |
| ECHO hypothesis | `0x00ffcc` (cyan) |
| Evidence confirmed | `0xffcc00` (gold) |

**Kenney 1-Bit Pack — Echo Rendering:**
- Tileset is monochrome — white sprites on transparent background, 16×16 px
- World starts near-invisible — unvisited buildings pulse at 3% opacity
- First-inspect: 400ms snap from near-invisible to full tint ("echo sharpens into reality")
- Rain: `ParticleEmitter`, white 1×2 px rects, 85° angle, alpha 0.3
- Corruption bleed: `Graphics` mask expands 1 tile/sec outward from compromised buildings
- Roads: circuit-trace style, tinted `0x00ffcc`, animated pulse redrawn each frame

**Agent NPC Sprites (1-bit characters, tinted per agent):**
| Agent | Tint | Walk Animation |
|---|---|---|
| LOGIS | `0x44aaff` (blue) | 4-frame walk cycle |
| NETARA | `0x00ffcc` (cyan) | 4-frame walk cycle |
| CARVER | `0xff9900` (amber) | 4-frame walk cycle |
| CHRONO | `0xcc44ff` (purple) | 4-frame walk cycle |

**Animations:**
| Event | Animation |
|-------|-----------|
| Agent arrives at building | NPC sprite walks to entrance, building tint pulses violet |
| Agent finds evidence | Gold particle burst from building, speech bubble appears |
| Building compromised | Red pulse → bleed seeps onto adjacent roads |
| Building cleared | White flash → returns to normal tint |
| ECHO hypothesis | Cyan glow radiates from ECHOPanel across implicated buildings |
| Agents cross-referencing | Dashed line drawn between two buildings the agents are correlating |
| NPC terminated | Sprite freeze → dissolve into pixel dust |

---

## Scenario Structure

```python
{
  "title": str,                      # "The Midnight Withdrawal"
  "briefing": str,                   # Mayor's incident report text
  "duration_hours": int,             # 10
  "buildings": list[Building],       # seeded city state
  "traffic_records": list[Traffic],  # timestamped road activity
  "answer_key": {
    "origin_building": str,
    "attack_path": list[str],
    "suspect": str,
    "technique": str                 # "phishing" | "sql_injection" | "insider" | ...
  },
  "red_herrings": list[str],         # building IDs with planted false evidence
  "agent_assignments": {             # which specialist starts at which building
    "logis": str,
    "netara": str,
    "carver": str,
    "chrono": str
  },
  "echo_hypotheses": list[Hypothesis]  # pre-seeded hypotheses ECHO surfaces during play
}
```

---

## Backend Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/investigate` | POST | Create investigation session, return `session_id` |
| `/investigate/{id}/ws` | WebSocket | Stream: `init` → agent finding events → `verdict` |
| `/echo/query` | POST | Natural language question to ECHO meta-agent |
| `/echo/verify` | POST | Player challenges an ECHO hypothesis — re-evaluates |
| `/building/{id}/inspect` | GET | Logs, files, registry, graffiti for a building |
| `/traffic/analyze` | GET | Traffic manifest + anomaly scores for a road |
| `/accusation/submit` | POST | Score accusation against answer key |

---

## Demo Script (90 seconds)

1. Load: *"The Midnight Withdrawal"* — city appears, dark and rainy
2. Four agent NPCs fan out from the entrance across the city
3. LOGIS reaches Bank Vault → speech bubble: *"4,000 logins. 03:17AM. One IP."*
4. NETARA on east road → *"Heavy outbound to the VPN tunnel. Nothing came back."*
5. CHRONO: dashed line appears between Bank and Router → *"4-minute gap. Same direction."*
6. Scrub timeline to 03:17 → red wave jumps Bank → Router → Mail Server
7. ECHO panel: *"Hypothesis: Bank Vault origin, credential stuffing, path via Router. 71% confidence."* Player clicks Accept
8. Player clicks Underground Tunnel → City Archives → registered to `g.harlow`
9. CARVER decodes graffiti on Mail Server: `DROP TABLE transactions;`
10. Submit accusation → courtroom → 100% accuracy — city lights up clean, echo resolved

---

## Prize Positioning

| Prize | Argument |
|-------|----------|
| **Best Game Jam** | Fully playable noir mystery, win condition, scoring, two scenarios |
| **Best Digital Forensics** | Log analysis, file carving, steganography, traffic analysis, registry forensics |
| **Best Gamification** | Every forensics concept is a tactile mechanic; swarm agents make invisible work visible |
| **Best UI/UX** | Noir pixel city with visible AI agents — literally watching intelligence propagate |
| **Best Moonshot** | First forensics game with embodied MiroFish multi-agent swarm |
| **Best Featherless.ai Use** | Five agents (LOGIS, NETARA, CARVER, CHRONO, ECHO) all powered by Featherless open-source LLMs |
| **Best Data Visualization** | The city IS the data — attack propagation, agent movement, and evidence recovery are all spatial |
