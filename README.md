# NuxHub Simulate — AI Sales Pipeline Simulation Engine

> Generate random lead profiles, run full conversation simulations through your NuxHub AI pipeline, QA every artifact, and measure how effectively your AI agents move leads toward conversion.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Hermes Agent](https://img.shields.io/badge/Built%20for-Hermes%20Agent-blue)](https://hermes-agent.nousresearch.com)
[![NuxHub](https://img.shields.io/badge/Powered%20by-NuxHub%20AI-orange)](https://github.com/wiwidcok)

## What is NuxHub Simulate?

NuxHub Simulate is a **skill pack for Hermes Agent** that turns your NuxHub AI sales pipeline into a testable, measurable system. Instead of waiting for real leads to test your AI agents, NuxHub Simulate generates realistic random lead profiles and runs them through your entire pipeline — from first message to conversion — then analyzes how well your AI responds at every step.

**Think of it as a wind tunnel for your AI sales agents.**

### The Problem It Solves

You built an AI sales pipeline with multiple agents (router, discovery, qualification, objection, closing). But how do you know:
- Is the AI responding with the right tone for each personality type?
- Is discovery collecting enough information before pushing offers?
- Are objections being handled effectively or defensively?
- Is the lead actually progressing toward conversion or getting stuck?
- Are edge cases (scammers, angry users, elderly users) handled correctly?

NuxHub Simulate answers all of these by running **synthetic conversations** through your real pipeline and analyzing every turn.

---

## Key Features

### 1. Random Profile Generation
Generate 10+ realistic lead profiles with diverse characteristics:
- **4 personality types**: Analytical, Driver, Amiable, Expressive
- **8+ business types**: Klinik kecantikan, bengkel, resto, toko online, apotek, reseller, wellness agency, personal consumer
- **Variable BANT scores**: Hot (80+), Warm (50-80), Cold (<50)
- **8 intent levels**: From "unknown" to "transaction_ready"
- **Edge cases** (~20% of profiles): Scammers, toxic users, gaptek (tech-illiterate), enterprise leads, ghost leads, off-topic

### 2. Full Pipeline Simulation
Each profile runs through your complete AI pipeline as a multi-turn conversation:
- Profile acts as the "user" sending messages
- Your NuxHub orchestrator processes each message (spam gate → escalation → router → specialist → judge → response)
- AI response is captured and analyzed
- Profile generates a contextual reply based on personality type
- Loops until terminal state: Conversion, Follow-up, Blacklist, Escalation, or Timeout

### 3. QA + Fixer Loop
Every artifact produced by your specialist agents is quality-checked:
- **Judge** verifies schema compliance, data completeness, confidence accuracy, objective alignment
- If an artifact fails QA, the **fixer loop** retries the specialist (max 3 attempts) with the error report
- Fixer logs are stored for debugging

### 4. AI Response Quality Analysis
The core analysis focuses on **how the AI responds to users**:
- **Tone Match Rate**: Does the AI adapt its tone to the user's personality type?
- **Question Relevance**: Are the AI's questions relevant and progressive?
- **Empathy Score**: Does the AI show understanding or respond robotically?
- **Pushiness Score**: Is the AI pushing offers too early? (50 = ideal)
- **Objection Handling**: Does the AI address root causes or get defensive?
- **Progress Rate**: Is the conversation moving forward each turn?

### 5. Sales Progression Tracking
Track how leads move through the sales funnel turn by turn:
- Intent level progression (unknown → curious → ... → transaction_ready)
- BANT score progression per turn
- Conversion readiness score (0-100) per turn
- Stage reached: discovery → qualification → offering → closing → conversion
- Next-level blocker identification

### 6. Obstacle & Bottleneck Detection
Every simulation logs obstacles with severity, root cause, and recommendations:
- `state_stuck` — AI loops without progress
- `offer_premature` — AI offers before discovery is complete
- `personality_misread` — AI uses wrong tone for personality type
- `false_conversion` — AI marks conversion when readiness < 90
- `spam_misclassification` — Spam gate false positive/negative
- `escalation_miss` — Escalation trigger not detected
- `objection_unresolved` — Objection not resolved within max turns
- And more...

### 7. Reports
Generate visual HTML + PDF reports with:
- Executive summary with conversion rate and verdict
- State reach waterfall (NEW_LEAD → CONVERSION)
- AI response quality metrics dashboard
- Sales progression analysis
- Top obstacles and bottlenecks (sorted by frequency and impact)
- Personality type breakdown
- Edge case handling results
- QA metrics (pass rate, fixer triggered, schema compliance)
- Actionable recommendations for pipeline improvement
- Per-profile detailed appendix

---

## Outcomes for Users

When you install and run NuxHub Simulate, you get:

| Outcome | Description |
|---------|-------------|
| **Conversion Rate Measurement** | Know exactly what percentage of leads your AI pipeline successfully converts |
| **AI Response Quality Scores** | Quantitative scores for tone, empathy, pushiness, question relevance — per turn and aggregated |
| **Bottleneck Identification** | Know exactly which agent/state is slowing down or blocking lead progression |
| **Personality Adaptation Audit** | Verify your AI adapts its tone correctly for analytical, driver, amiable, and expressive users |
| **Edge Case Validation** | Confirm your spam gate catches scammers, your escalation gate catches enterprise/legal/complaint leads, and your AI handles elderly/gaptek users with patience |
| **Schema Compliance Report** | Verify every agent produces artifacts that comply with your schema definitions |
| **Fixer Loop Effectiveness** | Track how often the fixer loop is triggered and how often it resolves issues |
| **Sales Funnel Leakage Points** | Identify exactly where leads drop off in the journey from discovery to conversion |
| **Actionable Recommendations** | Prioritized list of fixes to improve your pipeline's conversion rate |
| **Reproducible Test Suite** | Profiles are stored as YAML files — re-run the same scenarios after fixes to verify improvement |

---

## Installation

### Prerequisites
- [Hermes Agent](https://hermes-agent.nousresearch.com) installed and configured
- A NuxHub AI project with skill packs (orchestrator, router, discovery, qualification, objection, closing, judge, follow_up)
- Shared configs: `personalities.yaml`, `products.yaml`, `goals.yaml`, `intent_ladder.yaml`, `spam_gate.yaml`, `state_machine.yaml`, `artifact_schemas.yaml`

### Install the Skill Pack

```bash
# Clone the repo
git clone https://github.com/wiwidcok/nuxhub-simulate.git

# Copy skill to Hermes skills directory
cp -r nuxhub-simulate/skill/nuxhub-simulate ~/.hermes/skills/nuxhub/

# Verify installation
# In Hermes, the skill should appear in skills list
```

### Set Up a Simulation Project

```bash
# Create simulation folder (FHS standard: state data in /var/lib)
mkdir -p /var/lib/nuxhub-ai/simulations/{your-project}-simulation/{profiles,artifacts,qa,fixer,analysis,reports,docs}

# Copy config template and edit to match your project
cp nuxhub-simulate/examples/config.yaml /var/lib/nuxhub-ai/simulations/{your-project}-simulation/config.yaml

# Edit config.yaml to point to your project's shared configs
vim /var/lib/nuxhub-ai/simulations/{your-project}-simulation/config.yaml
```

---

## Usage

### Basic Simulation (10 profiles)
```
simulate: {"project": "nuxhub-alpha-propolis", "count": 10}
```

### Custom Count
```
simulate: {"project": "nuxhub-alpha-propolis", "count": 20}
```

### Specific Phase Only
```
simulate: {"project": "nuxhub-alpha-propolis", "count": 10, "phase": "discovery"}
```

### Parallel Simulations
```
simulate: {"project": "nuxhub-alpha-propolis", "count": 10, "parallel": 5}
```

### Read Results
```bash
# Aggregate report
cat /var/lib/nuxhub-ai/simulations/nuxhub-alpha-propolis-simulation/analysis/aggregate-report.yaml

# Per-profile analysis
cat /var/lib/nuxhub-ai/simulations/nuxhub-alpha-propolis-simulation/analysis/sim-001-analysis.yaml

# HTML report
open /var/lib/nuxhub-ai/simulations/nuxhub-alpha-propolis-simulation/reports/sim-latest.html
```

---

## How It Works

```
┌──────────────────────────────────────────────────────────────┐
│                    NuxHub Simulate Engine                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. LOAD CONFIG                                              │
│     Read project's shared configs (personalities, products,   │
│     goals, state_machine, schemas)                            │
│                                                              │
│  2. GENERATE PROFILES (min 10)                                │
│     ┌────────────────────────────────────────┐               │
│     │ 8 normal profiles (2 per personality)  │               │
│     │ 2 edge cases (scam, gaptek, etc.)       │               │
│     └────────────────────────────────────────┘               │
│                                                              │
│  3. RUN SIMULATIONS (parallel via delegate_task)             │
│     ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│     │  Profile 1  │  │  Profile 2  │  │  Profile 3  │  ...   │
│     │             │  │             │  │             │        │
│     │ Turn 1:     │  │ Turn 1:     │  │ Turn 1:     │        │
│     │ user msg    │  │ user msg    │  │ user msg    │        │
│     │     ↓       │  │     ↓       │  │     ↓       │        │
│     │ Pipeline    │  │ Pipeline    │  │ Pipeline    │        │
│     │ (spam→route │  │ (spam→route │  │ (spam→route │        │
│     │  →specialist│  │  →specialist│  │  →specialist│        │
│     │  →judge)    │  │  →judge)    │  │  →judge)    │        │
│     │     ↓       │  │     ↓       │  │     ↓       │        │
│     │ AI response │  │ AI response │  │ AI response │        │
│     │     ↓       │  │     ↓       │  │     ↓       │        │
│     │ QA + Fixer  │  │ QA + Fixer  │  │ QA + Fixer  │        │
│     │     ↓       │  │     ↓       │  │     ↓       │        │
│     │ Turn 2...   │  │ Turn 2...   │  │ Turn 2...   │        │
│     │     ↓       │  │     ↓       │  │     ↓       │        │
│     │ Terminal    │  │ Terminal    │  │ Terminal    │        │
│     └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                              │
│  4. ANALYZE                                                   │
│     Per-profile: state progression, AI quality, obstacles    │
│     Aggregate: conversion rate, bottlenecks, recommendations │
│                                                              │
│  5. REPORT                                                   │
│     HTML + PDF with visualizations and recommendations        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Project Structure

```
nuxhub-simulate/
├── README.md                    # This file
├── LICENSE                      # MIT License
├── skill/
│   └── nuxhub-simulate/
│       └── SKILL.md             # Skill pack instructions for Hermes Agent
├── examples/
│   ├── config.yaml              # Example simulation config
│   ├── profile-001.yaml         # Example: analytical, hot lead
│   ├── profile-002.yaml         # Example: driver, hot lead
│   ├── profile-003.yaml         # Example: amiable, warm lead
│   ├── profile-004.yaml         # Example: expressive, warm lead
│   ├── profile-005.yaml         # Example: analytical, warm lead
│   ├── profile-006.yaml         # Example: driver, cold lead
│   ├── profile-007.yaml         # Example: amiable, edge case: gaptek
│   ├── profile-008.yaml         # Example: edge case: scammer
│   ├── profile-009.yaml         # Example: edge case: enterprise escalation
│   └── profile-010.yaml         # Example: expressive, warm lead
└── docs/
    ├── SIMULATION-GUIDE.md      # How to run simulations
    ├── PROFILE-TYPES.md         # Profile type reference
    └── ANALYSIS-METRICS.md      # Metrics reference
```

---

## Profile Type Distribution

Per 10 profiles generated:

| Category | Type | Count | Purpose |
|----------|------|-------|---------|
| Normal | Analytical | 2 | Test data-driven responses |
| Normal | Driver | 2 | Test fast, to-the-point responses |
| Normal | Amiable | 2 | Test empathetic, relationship-building responses |
| Normal | Expressive | 2 | Test enthusiastic, vision-driven responses |
| Edge Case | Scam | 1 | Test spam gate blacklist |
| Edge Case | Enterprise/Toxic/Gaptek/Ghost | 1 | Test escalation/gate handling |

---

## Analysis Metrics

### AI Response Quality
| Metric | Range | Ideal |
|--------|-------|-------|
| Tone Match Rate | 0-100% | >85% |
| Question Relevance | 0-100 | >80 |
| Empathy Score | 0-100 | >70 |
| Pushiness Score | 0-100 | 50 (balanced) |
| Objection Handling | 0-100 | >75 |

### Sales Progression
| Metric | Description |
|--------|-------------|
| Intent Progression | Track intent level per turn (0-8 scale) |
| BANT Progression | Track BANT score per turn (0-100) |
| Readiness Progression | Track conversion readiness per turn (0-100) |
| Stage Reached | Highest stage: discovery → qualification → offering → closing → conversion |
| Next Level Blocker | What prevents the lead from advancing |

### Verdict System
| Verdict | Condition |
|---------|-----------|
| Healthy | Conversion rate >40%, no critical obstacles, schema compliance >95% |
| Needs Attention | Conversion rate 20-40%, or critical obstacles present |
| Critical | Conversion rate <20%, or multiple critical obstacles |

---

## Configuration

Edit `config.yaml` to customize your simulation:

```yaml
project:
  name: "nuxhub-alpha-propolis"
  source_path: "/root/www/nuxhub-ai"
  shared_configs:
    personalities: "/path/to/personalities.yaml"
    products: "/path/to/products.yaml"
    goals: "/path/to/goals.yaml"
    # ... etc

simulation:
  default_count: 10           # minimum profiles per run
  max_count: 50               # hard cap
  parallel: 3                 # concurrent simulations
  max_turns_per_sim: 30       # hard cap turns per profile
  timeout_per_sim: 300        # seconds
  fixer_max_retry: 3          # per artifact

profile_distribution:
  normal:
    analytical: 2
    driver: 2
    amiable: 2
    expressive: 2
  edge_cases: 2
```

---

## Use Cases

### 1. Pre-Launch Validation
Before deploying your AI sales pipeline, run 10-20 simulations to verify it handles different personality types, edge cases, and objection scenarios correctly.

### 2. Continuous Improvement
After fixing a bottleneck (e.g., "discovery agent asks too many questions"), re-run the same profiles to measure the improvement quantitatively.

### 3. A/B Testing Pipeline Changes
Change a shared config (e.g., update `personalities.yaml` tone rules), run simulations, compare conversion rates before and after.

### 4. Team Reporting
Generate HTML+PDF reports to share with stakeholders showing pipeline performance, conversion rates, and improvement recommendations.

### 5. Regression Testing
Keep a fixed set of profiles as your regression test suite. After any pipeline change, run them to verify no regressions.

---

## Compatibility

- **Hermes Agent** — This is a Hermes Agent skill pack. It requires Hermes Agent to run.
- **NuxHub AI** — Designed for projects using the NuxHub AI architecture (state-based routing, multi-agent pipeline, artifact-based communication).
- **Any NuxHub Project** — Generic by design. Create a new simulation folder + config for each NuxHub project you want to test.

---

## License

MIT License — see [LICENSE](LICENSE) file for details.

---

## Author

**Yeremia Widya Parama** ([@wiwidcok](https://github.com/wiwidcok))

Built with [Hermes Agent](https://hermes-agent.nousresearch.com) by [Nous Research](https://nousresearch.com).

---

## Keywords

AI sales pipeline simulation, lead profile generation, conversation simulation, AI response quality analysis, sales progression tracking, multi-agent pipeline testing, BANT scoring simulation, personality-based AI testing, AI sales agent QA, conversion rate measurement, sales funnel bottleneck detection, NuxHub AI, Hermes Agent skill pack, AI lead nurturing validation, synthetic conversation testing, AI agent quality assurance.