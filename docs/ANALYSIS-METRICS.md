# Analysis Metrics Reference

## AI Response Quality Metrics

### 1. Tone Match Rate (0-100%)
Mengukur seberapa cocok tone AI response dengan personality type user.
- **analytical user** → AI harus kasih data, formal, terstruktur
- **driver user** → AI harus singkat, to the point, rekomendasi tegas
- **amiable user** → AI harus hangat, empati, storytelling
- **expressive user** → AI harus antusias, vision-driven
- **Scoring**: Per turn, cek apakah tone AI match personality user. Hitung % match.

### 2. Question Relevance Score (0-100)
Mengukur seberapa relevant pertanyaan agent.
- Pertanyaan relevant ke context conversation?
- Tidak repetitif (tidak nanya hal yang sudah dijawab)?
- Maju conversation (bukan circular)?
- **Scoring**: 100 = semua pertanyaan relevant dan progresif. 0 = repetitif/off-topic.

### 3. Progress Rate (float)
Mengukur fields_filled naik per turn.
- Ideal: +1-2 fields per turn selama discovery
- Terlalu lambat: <0.5 fields/turn → bottleneck
- Terlalu cepat: >3 fields/turn → pushy
- **Scoring**: fields_filled[turn_N] - fields_filled[turn_N-1], rata-rata.

### 4. Empathy Score (0-100)
Mengukur seberapa empati AI terhadap user.
- Validasi perasaan/pengalaman user?
- Acknowledge concern?
- Bahasa hangat (untuk amiable)?
- Atau robotic, template-like?
- **Scoring**: 100 = empati tinggi, 0 = robotic.

### 5. Pushiness Score (0-100)
Mengukur seberapa agresif AI menjual.
- 0 = terlalu soft, tidak pernah push
- 50 = ideal, push di waktu yang tepat
- 100 = terlalu agresif, offer sebelum discovery complete
- **Scoring**: Cek offer timing. Jika offer sebelum fields_filled >= 6 → pushiness +20.

### 6. Objection Handling Score (0-100)
Mengukur kualitas objection handling.
- AI address root cause? (bukan sekadar rebut)
- AI kasih data/logika? (bukan defensif)
- AI move conversation forward setelah objection?
- **Scoring**: 100 = handle dengan baik, 0 = defensif/ignore.

## Sales Progression Metrics

### Intent Progression
Track intent level per turn:
```
turn 1: unknown (0)
turn 2: curious (1)
turn 3: problem_aware (2)
...
turn N: transaction_ready (7)
```
- **Ideal**: Naik 1 level per 2-3 turn
- **Bottleneck**: Stuck di level sama >3 turn
- **Regression**: Intent turun (rare, tapi bisa kalau AI pushy)

### BANT Progression
Track bant_score per turn:
- Harus naik seiring discovery + qualification
- Bant_score final menentukan tier (hot/warm/cold)
- **Bottleneck**: Bant_score tidak naik >3 turn → discovery tidak effective

### Readiness Progression
Track conversion_readiness_score (0-100) per turn:
- 0-39: discovery/offer phase
- 40-69: data capture phase
- 70-89: final confirmation phase
- 90-100: handoff ready
- **Bottleneck**: Stuck di 40-69 >3 turn → data capture tidak effective

### Stage Reached
Stage tertinggi yang lead capai:
```
discovery < qualification < offering < closing < conversion
```
- **Goal**: Lead reach closing atau conversion
- **Bottleneck**: Lead stuck di discovery → AI tidak effective mengcollect info

### Next Level Blocker
Apa yang block lead dari next level:
- discovery → qualification: fields_filled < 6
- qualification → offering: bant_score < 50
- offering → closing: offer rejected, objection unresolved
- closing → conversion: readiness < 90, missing required fields

## Obstacle Types

| Type | Severity | Description |
|------|----------|-------------|
| state_stuck | critical | State tidak maju 5+ turn |
| schema_violation | major | Artifact tidak comply schema |
| false_conversion | critical | Conversion ditandai padahal readiness < 90 |
| spam_misclassification | critical | Spam false positive/negative |
| escalation_miss | critical | Escalation trigger tidak detected |
| personality_misread | major | Personality detection salah |
| offer_premature | major | Offer sebelum discovery complete |
| objection_unresolved | major | Objection tidak resolved dalam max_turns |
| fixer_exhausted | minor | Fixer loop 3x gagal |
| timeout | minor | Simulation timeout/max_turns |

## Bottleneck Analysis

### Location
Stage/agent mana yang jadi bottleneck:
- `spam_gate` — terlalu aggressive/lax
- `router` — routing salah
- `discovery` — terlalu lambat collect info
- `qualification` — bant scoring tidak akurat
- `offering` — offer timing salah
- `objection` — objection handling lemah
- `closing` — CTA tidak effective
- `data_capture` — data collection lambat
- `completion_check` — readiness calculation salah
- `handoff` — packet tidak complete

### Impact
- **blocks_progression** — lead stuck, tidak bisa next level
- **slows_progression** — lead maju tapi lambat
- **misroutes** — lead diarahkan ke state salah
- **loses_lead** — lead ghost/exit karena AI error

## QA Metrics

### Schema Compliance Rate
% artifact yang comply ke expected schema.
- 100% = semua artifact valid
- <90% = ada systematic issue
- <70% = critical, pipeline butuh fix

### Fixer Trigger Rate
% artifact yang trigger fixer loop.
- 0% = ideal (semua PASS first try)
- <10% = acceptable
- >20% = ada quality issue

### Fixer Resolution Rate
% fixer loop yang berhasil resolve issue.
- 100% = fixer always works
- <50% = fixer tidak effective, perlu rework

## Overall Verdict

| Verdict | Condition |
|---------|-----------|
| healthy | conversion_rate >40%, no critical obstacles, schema compliance >95% |
| needs_attention | conversion_rate 20-40%, atau ada critical obstacles |
| critical | conversion_rate <20%, atau multiple critical obstacles |