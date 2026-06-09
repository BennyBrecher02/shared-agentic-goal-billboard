---
belongs_to_goal: G12   # by-goal parent (goal-wiring §0 tier-A — filename/topic names the goal)
pin_id: P-010
title: Local AI Week 1 install (Ollama + Claude Code Router + LiteLLM)
created: 2026-05-28
narrowed_in: RH-017 deep dive + 2026-05-27T23:43Z user inbox + ongoing token-leak conversations
why_paused: requires user gesture for `brew install ollama` (not in agent allow list); user marathon-session-tired
resume_criteria: user has a weekend free; agent has fresh budget for any complementary scripts; B-gate (A71 mutation-OS) PASSED already so C-gate is UNLOCKED
estimated_cost: $0 + ~3.5hr user setup time; ~50k tokens for CCR config + routing rules I write autonomously
leverage_g2: 10.0
ns_relation: COST SUBSTRATE — ends the token leak; 25-35% Claude burn reduction (text-only tasks only; visual audit stays on Claude per skepticism critique)
last_touched: 2026-05-28
state: pinned
---

# P-010 — Local AI Week 1 install

## TL;DR

Free path to 25-35% Claude burn reduction over 1 weekend. Foundation for everything in RH-017's 4-week rollout. **Visual audits stay on Claude** (local vision models hallucinate at ~60-70% accuracy — not OK for audit work).

## Sequence

### Step 1: User gestures (not in agent allow list)
```bash
brew install ollama
ollama pull qwen3-coder:30b   # ~17GB, one-time
ollama pull qwen3:1.5b        # ~1GB, fast small model for autocomplete
```

If running Ollama on M2 (LAN-exposed for M4 to route to):
```bash
ollama serve --host 0.0.0.0   # exposes :11434
# verify from M4: curl http://<M2_LAN_IP>:11434/api/tags
```

### Step 2: Agent-runnable (post settings-lift 2026-05-28)
```bash
npm install -g @musistudio/claude-code-router
# config: ~/.claude-code-router/config.json (agent writes)
ccr start
```

### Step 3: Routing rules (config file)

```json
{
  "providers": {
    "claude_api": "<your existing Claude API config>",
    "ollama_local": "http://localhost:11434",
    "ollama_m2":    "http://<M2_LAN_IP>:11434"
  },
  "default_route": "claude_api",
  "rules": [
    {"pattern": "audit|scrutiny|visual|capture|matrix|rubric", "provider": "claude_api"},
    {"pattern": "plan|design|architectural|memory|skill", "provider": "claude_api"},
    {"pattern": "single-file|rename|move|refactor (?!multi)", "provider": "ollama_local"},
    {"pattern": "comprehension|explain|summarize|draft (?!plan)", "provider": "ollama_local"},
    {"pattern": "boilerplate|scaffold|autocomplete", "provider": "ollama_local"}
  ]
}
```

### Step 4: LiteLLM router on Pi5 (optional, parallel track)

If Pi5 is ready as always-on infra node:
```bash
# On Pi5:
pip install litellm
litellm --host 0.0.0.0 --port 4000 --config /etc/litellm/config.yaml
```

LiteLLM becomes the central router; CCR queries LiteLLM; LiteLLM decides backend.

## What this DOES NOT include

- ❌ Visual audit work (Claude only — local vision insufficient)
- ❌ Plan synthesis (Claude — judgment quality)
- ❌ Memory rule writing (Claude — voice critical)
- ❌ Multi-file architectural reasoning (Claude — 1M context)
- ❌ EXO Mac sharding (separate pin; Phase 2)

## Expected impact

- $200/wk burn → effective ~$130-150/wk on routine work
- Weekly limit stretches from ~7d to ~9-10d
- Break-even on time invested: Month 2 of consistent use

## Cross-references

- `context/markdowns/research/rabbit-holes/RH-017-local-ai-deep-dive/findings-overview.md`
- `context/markdowns/research/rabbit-holes/RH-017-local-ai-deep-dive/sub-A-fully-free.md`
- `context/markdowns/research/rabbit-holes/RH-017-local-ai-deep-dive/sub-C-best-at-200wk-roof.md`
- P-011 (M2 cluster registration) — pairs with this; M2 hosts Ollama if used as inference node
- A71 mutation-OS — B-gate that unlocked C-gate (this pin)
