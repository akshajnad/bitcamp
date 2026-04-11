# ECHO — Frontend Agent Instructions

> Noir pixel-art city forensics sim. The city is the file system. A cyberattack left its echo behind. Four AI specialist agents move through the city reconstructing what happened. A central intelligence — ECHO — synthesizes their findings for the player.

## Commands
| Task | Command |
|------|---------|
| Dev | `bun dev` |
| Build | `bun build` |
| Lint | `bun lint` |
| Format | `bun format` |

## Key Conventions
- Phaser pages must be `"use client"` with `next/dynamic` + `ssr: false`
- Linter: Biome (`biome.json`) — not ESLint
- React Compiler is enabled — avoid manual `useMemo`/`useCallback` unless profiling shows a need
- Next.js 16 has breaking changes — read `node_modules/next/dist/docs/` before using unfamiliar APIs

## Project Structure
```
src/
  components/
    ECHOPanel.tsx              # ECHO meta-agent: hypothesis stream, confidence meter, verify/reject
    EvidenceBoard.tsx          # Confirmed evidence cards + accusation builder
    TimelineScrubber.tsx       # Bottom bar — scrub 10 in-game hours, replay agent positions
    BuildingInspectModal.tsx   # Click building → logs, deleted files, registry, steg tabs
    NetworkMap.tsx             # D3 force graph — road/traffic connections between buildings
    CourtroomModal.tsx         # Final accusation submission + verdict screen
    EventFeed.tsx              # Real-time stream of all agent findings + player actions
    GameCanvas.tsx             # Phaser 3 wrapper (dynamic import, ssr: false)
  game/
    scenes/WorldScene.ts       # Main Phaser scene — noir city, buildings, agent NPCs, processes
    scenes/BootScene.ts        # Asset loading
    bridge/EventBridge.ts      # Singleton — React ↔ Phaser via forensics:* events
    map/CityGenerator.ts       # Procedural noir city from Kenney 1-Bit tileset
    systems/AgentNPCSystem.ts  # Manages 4 specialist NPC sprites: movement, animation, speech
    systems/CorruptionSystem.ts # Infection spread animation (red glow, tile-by-tile bleed)
    systems/TrafficSystem.ts   # Delivery truck / suspicious traffic visualization on roads
    effects/                   # Rain particles, neon pulse, corruption bleed, gold burst
  hooks/
    useInvestigation.ts        # WebSocket state: scenario, evidence, agent findings, timeline
    useTimeline.ts             # Timeline scrubber state + city replay at hour T
  services/
    wsClient.ts                # Socket.IO client
  types/
    index.ts                   # Frontend types
    backend.ts                 # Backend response types
```

## The Agent Swarm (MiroFish Architecture)

ECHO uses a **MiroFish-style multi-agent swarm**: four specialists run in parallel with no central coordinator. Each is an embodied NPC in the city — players *watch* them work. A meta-agent (ECHO) synthesizes their findings into hypotheses.

### Specialist Agents — NPC Sprites

| Agent | Role | Tint | What Player Sees |
|---|---|---|---|
| **LOGIS** | Log Analyst | `0x44aaff` (blue) | Walks to buildings, reads logs, speech bubble on anomaly |
| **NETARA** | Network Analyst | `0x00ffcc` (cyan) | Patrols roads, flags traffic spikes, traces lateral movement |
| **CARVER** | File Analyst | `0xff9900` (amber) | Enters abandoned buildings, excavates debris, decodes graffiti |
| **CHRONO** | Timeline Analyst | `0xcc44ff` (purple) | Moves between buildings drawing dashed correlation lines |

Each specialist is managed by `AgentNPCSystem.ts`. On each backend `agent:finding` event:
1. Move NPC sprite to the relevant building (pathfinding via OccupancyGrid)
2. Play inspect animation (2s loop while agent is "inside")
3. Show speech bubble with finding summary
4. Emit `forensics:evidence_found` to React

### ECHO Meta-Agent — `ECHOPanel.tsx`

ECHO observes the shared evidence pool and synthesizes specialist findings into hypotheses. It does **not** move — it's the voice in the player's earpiece, rendered in `ECHOPanel.tsx`.

**Panel responsibilities:**
- Streams hypothesis cards: *"Origin likely Bank Vault. 71% confidence."*
- Each hypothesis has **Accept / Reject / Investigate Further** buttons
  - Accept → pins to `EvidenceBoard`
  - Reject → removes from consideration, ECHO notes it
  - Investigate Further → backend sends the relevant specialist back to that building
- Confidence meter (0–100%) updates as evidence accumulates
- ~20% of ECHO's hypotheses are red herrings — UI must show confidence, never certainty
- Player can type a free-form question: *"What do LOGIS and NETARA agree on?"*

All ECHO interaction hits `POST /echo/query` or `POST /echo/verify` — no direct Featherless calls from the browser.

## EventBridge Events (`forensics:*`)

| Event | Direction | Payload |
|---|---|---|
| `forensics:building_inspect` | React → Phaser | `{ building_id: string }` |
| `forensics:evidence_found` | Phaser → React | `{ evidence: Evidence, agent_id: string }` |
| `forensics:traffic_highlight` | React → Phaser | `{ from: string, to: string, hour: number }` |
| `forensics:timeline_scrub` | React → Phaser | `{ hour: number }` |
| `forensics:agent_move` | Backend → Phaser | `{ agent_id: string, building_id: string }` |
| `forensics:agent_finding` | Backend → React | `{ agent_id: string, finding: Finding, building_id: string }` |
| `forensics:corruption_spread` | Backend → Phaser | `{ building_ids: string[], intensity: number }` |
| `forensics:echo_hypothesis` | Backend → React | `{ hypothesis: Hypothesis, confidence: number }` |
| `forensics:accusation_submit` | React → Backend | `{ origin_building: string, attack_path: string[], suspect: string }` |
| `forensics:building_cleared` | Backend → Phaser | `{ building_id: string }` — building tint heals back to amber |
| `forensics:agent_correlate` | Backend → Phaser | `{ agent_id: string, from: string, to: string }` — dashed line between two buildings |

## City ↔ Forensics Mapping
| City Element | Forensics Concept |
|---|---|
| Buildings | Machines / servers |
| Roads | Network connections |
| Citizens (NPCs) | Running processes |
| AI Agent NPCs (LOGIS etc.) | Specialist investigators — embodied intelligence |
| Abandoned buildings | Deleted / corrupted files |
| Graffiti | Malware signatures |
| Security cameras | System logs |
| City archives | Registry artifacts |
| Underground tunnels | Hidden / encrypted partitions |
| Blackouts | Denial-of-service events |
| Delivery trucks | Suspicious network traffic |

## Tileset — Kenney 1-Bit Pack
- **Source:** https://kenney.nl/assets/1-bit-pack — CC0, 1,060 tiles, 16×16 px, monochrome
- All tiles are white-on-transparent — color comes entirely from `setTint()` at runtime
- City starts near-invisible (`0x111111` tint, 3% opacity pulse) — the echo of a city
- Color = forensics state: `0x334455` unknown, `0x7b61ff` agent inside, `0xc8b89a` clean, `0xff2200` compromised, `0xffcc00` evidence
- Roads rendered as circuit traces — tinted `0x00ffcc`, animated pulse along length each frame
- Agent NPC sprites use 1-bit character tiles, each tinted to their agent color

## Noir Visual Style
- Background: `#0a0a0f` — world is nearly dark at start
- The city *emerges from darkness* as agents investigate — echoes resolve into color
- Rain: `ParticleEmitter`, white 1×2 px rects, 85° angle, alpha 0.3
- Corruption bleed: red tint radiates tile-by-tile outward from compromised buildings, 1 tile/sec
- Reveal animation: 400ms snap from near-invisible to full tint when first inspected
- ECHO hypothesis: cyan glow pulses across implicated buildings when hypothesis is surfaced
- Agent correlation: dashed line drawn between two buildings CHRONO is correlating
- Camera: top-down, pan + zoom on click

## Featherless.ai (Backend — for reference)
All five agents (LOGIS, NETARA, CARVER, CHRONO, ECHO) call Featherless.ai via the backend. The frontend never calls Featherless directly.

```
NEXT_PUBLIC_WS_URL=http://localhost:8000
```

Backend env:
```
FEATHERLESS_API_KEY=fl-...
ECHO_MODEL=Qwen/Qwen3-235B-A22B
FEATHERLESS_BASE_URL=https://api.featherless.ai/v1
```
