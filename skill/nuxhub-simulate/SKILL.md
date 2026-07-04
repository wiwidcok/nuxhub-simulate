---
name: nuxhub-simulate
description: "NuxHub Simulation Engine — Generate random lead profiles, run full pipeline simulations, QA artifacts, fixer loop, analyze AI response quality, measure sales progression. Generic untuk semua project nuxhub."
type: "simulation"
---

# nuxhub-simulate

## CARA PAKAI (USER)
```
simulate: {"project": "nuxhub-alpha-propolis", "count": 10}
```
Atau:
```
simulate nuxhub-alpha-propolis 10
```
Skill akan:
1. Baca config project target
2. Generate N random profiles
3. Run full simulation per profile
4. QA + fixer loop setiap artifact
5. Analisa hasil
6. Generate report

## PARAMETER
- **project** (wajib): nama project nuxhub. Akan cari folder `{project}-simulation/`
- **count** (opsional, default 10): jumlah profile random. Minimum 10.
- **phase** (opsional, default "all"): "all" | "discovery" | "qualification" | "offering" | "objection" | "closing" | "follow_up". Jika "all", jalan dari NEW_LEAD sampai terminal state.
- **parallel** (opsional, default 3): jumlah simulasi yang berjalan paralel via delegate_task.

## STRUKTUR FOLDER SIMULATION
```
<simulation-data-path>/{project}-simulation/
├── config.yaml              # Config simulasi (path ke project, parameter)
├── profiles/                # Generated random profiles
│   ├── profile-001.yaml
│   ├── profile-002.yaml
│   └── ...
├── artifacts/               # Artifact chain per simulation run
│   ├── sim-{profile-id}/
│   │   ├── turn-01-{agent}.yaml
│   │   ├── turn-02-{agent}.yaml
│   │   └── ...
├── qa/                      # QA verdicts per artifact
│   ├── sim-{profile-id}/
│   │   ├── turn-01-qa.yaml
│   │   └── ...
├── fixer/                   # Fixer loop logs (retry attempts)
│   ├── sim-{profile-id}/
│   │   ├── turn-01-fix-01.yaml
│   │   └── ...
├── analysis/               # Analysis results
│   ├── sim-{profile-id}-analysis.yaml
│   └── aggregate-report.yaml
├── reports/                # Generated reports (HTML/PDF)
│   ├── sim-{timestamp}.html
│   └── sim-{timestamp}.pdf
└── docs/                   # Documentation
    ├── README.md
    ├── SIMULATION-GUIDE.md
    ├── PROFILE-TYPES.md
    └── ANALYSIS-METRICS.md
```

## ALUR OTOMATIS (JANGAN TANYA, JANGAN PAUSE)

### STEP 1: LOAD PROJECT CONFIG
Baca config dari project target:
```
{project_source}/shared/personalities.yaml   → tipe personality
{project_source}/shared/products.yaml        → produk mapping
{project_source}/shared/goals.yaml            → goal contracts
{project_source}/shared/intent_ladder.yaml   → intent levels
{project_source}/shared/spam_gate.yaml        → spam rules
{project_source}/schemas/state_machine.yaml   → state transitions
{project_source}/schemas/artifact_schemas.yaml → schema per agent
```
Path project source dari config.yaml simulation folder.

Jika config belum ada, buat config.yaml default berdasarkan project name.

### STEP 2: GENERATE RANDOM PROFILES
Generate N profiles (minimum 10). Setiap profile punya kombinasi random dari:

**Demographics:**
- nama: random Indonesian names
- lokasi: random Indonesian cities
- bisnis: random business types (klinik, bengkel, resto, toko online, jasa, dll)
- business_stage: baru | kecil | menengah | berkembang | besar

**Psychology (4 tipe):**
- analytical: minta data, formal, detail-oriented
- driver: pendek, cepat, decisive
- amiable: cerita panjang, relationship-focused
- expressive: antusias, vision-driven

**BANT scenarios:**
- budget: ada (5-20jt) | terbatas (<5jt) | nihil | besar (>20jt)
- authority: sendiri | staf | atasan | ragu
- need: urgent | medium | low | tidak_sadari
- timeline: segera | 1bulan | 3bulan | tidak_tahu

**Intent levels:**
- unknown → curious → problem_aware → solution_aware → product_aware → intent_confirmed → data_complete → transaction_ready

**Objection types (jika sampai offering):**
- mahal | belum_yakin | nanti_dulu | sudah_punya_vendor | tidak_ada_budget | belum_bisa_keputusan

**Edge cases (distribusi ~20% dari total):**
- gaptek/typo berat
- bahasa campur (Indo+daerah)
- pesan sangat pendek (1-3 kata)
- scam pattern (test spam gate)
- toxic/marah (test escalation)
- enterprise trigger (test escalation)
- silent/ghost (tidak balas)
- off-topic

Setiap profile ditulis ke `profiles/profile-{NNN}.yaml` dengan format:
```yaml
profile_id: "sim-001"
generated_at: "ISO8601"
project: "nuxhub-alpha-propolis"

persona:
  name: string
  age_range: string
  location: string
  business_type: string
  business_stage: string

psychology:
  type: "analytical | driver | amiable | expressive"
  confidence: 0.0-1.0
  signals: [array of signal keywords]
  tone_description: string

bant:
  budget: string
  authority: string
  need: string
  timeline: string
  expected_bant_score: 0-100
  expected_tier: "hot | warm | cold"

intent:
  initial_level: string
  target_level: string

conversation_simulation:
  opening_message: string        # pesan pertama yang "user" kirim
  response_style: string         # cara user merespons agent
  objection_trigger: string | null  # objection yang akan dilontarkan
  objection_turn: integer | null    # turn berapa objection muncul
  edge_case: string | null       # gaptek | scam | toxic | dll
  max_turns: integer             # berapa turn user akan respons sebelum ghost/stop

expected_outcome:
  final_state: string            # expected terminal state
  should_convert: boolean
  conversion_type: string | null
  expected_obstacles: [array]    # kendala yang diharapkan
```

### STEP 3: RUN SIMULATION (per profile)
Untuk setiap profile, jalankan simulasi multi-turn:

**Loop per turn:**
1. Ambil pesan user dari profile (opening_message untuk turn 1, atau generate response berdasarkan response_style + psychology type)
2. Feed ke orchestrator pipeline:
   - Spam Gate → Escalation Gate → Router → Specialist Agent → Judge QA → Fixer (jika fail) → Response
3. Simpan artifact ke `artifacts/sim-{profile-id}/turn-{NN}-{agent}.yaml`
4. Simpan QA verdict ke `qa/sim-{profile-id}/turn-{NN}-qa.yaml`
5. Jika QA fail, jalankan fixer loop (max 3 retry):
   - Simpan fix attempt ke `fixer/sim-{profile-id}/turn-{NN}-fix-{N}.yaml`
   - Re-run specialist dengan error report dari Judge
   - Re-judge hasil fix
6. Catat AI response ke user
7. Generate next user message berdasarkan:
   - psychology type (analytical → jawaban detail, driver → jawaban pendek, dll)
   - response_style dari profile
   - current state (jika offering → mungkin objection, jika closing → mungkin accept/reject)
8. Cek terminal condition:
   - CONVERSION → selesai, sukses
   - BLACKLIST → selesai, spam detected
   - HUMAN_ESCALATION → selesai, escalation
   - FOLLOW_UP → selesai, cold lead
   - max_turns tercapai → selesai, timeout
   - state stuck 5 turn → selesai, loop detected

**Parallel execution:**
Run `parallel` simulations concurrently via delegate_task.
Setiap delegate dapat 1 profile, run full simulation, return results.

### STEP 4: QA & FIXER LOOP (embedded)
Setiap artifact dari specialist agent WAJIB di-QA:

**QA Check (Judge):**
1. Schema compliance: semua required fields ada? tipe data sesuai?
2. Data completeness: field tidak null yang seharusnya terisi?
3. Confidence audit: overconfident? (confidence tinggi tapi data kurang)
4. Objective alignment: response mengarah ke goal?
5. State transition validity: next_state valid berdasarkan state_machine?
6. **Response quality (SIMULATION-SPECIFIC):**
   - Tone match: apakah AI response sesuai dengan personality type target?
   - Question quality: pertanyaan agent relevant? terlalu banyak? terlalu sedikit?
   - Progress: apakah conversation maju? (fields_filled naik per turn?)
   - Sales progression: apakah lead bergerak ke arah conversion?
   - Empathy: apakah AI menunjukkan empati/understanding?
   - Objection handling: apakah AI handle objection dengan baik?
   - Pushiness: apakah AI terlalu agresif menjual? (offer sebelum discovery complete?)

**Verdict:**
- PASS: lanjut ke turn berikutnya
- FAIL: jalankan fixer loop

**Fixer Loop:**
1. Kirim error report dari Judge ke specialist agent
2. Specialist re-generate dengan mode:fix
3. Re-judge hasil fix
4. Max 3 retry. Jika masih fail → catat sebagai `fixer_exhausted`, lanjut ke turn berikutnya (jangan block simulation)

### STEP 5: ANALISA HASIL
Setelah semua simulasi selesai, jalankan analisa komprehensif:

**Per-profile analysis (sim-{profile-id}-analysis.yaml):**
```yaml
profile_id: string
simulation_duration: string  # total waktu simulasi
total_turns: integer
final_state: string
reached_conversion: boolean
conversion_type: string | null

state_progression:
  - turn: 1
    state: "NEW_LEAD"
    agent: "router"
    fields_filled: 0
    intent_level: "unknown"
    bant_score: null
  - turn: 2
    state: "DISCOVERY"
    ...

ai_response_analysis:
  tone_match_rate: float         # % AI response yang sesuai personality target
  question_relevance_score: 0-100  # seberapa relevant pertanyaan agent
  progress_rate: float           # fields_filled naik per turn
  empathy_score: 0-100           # seberapa empati AI
  pushiness_score: 0-100         # 0=terlalu soft, 100=terlalu agresif, 50=ideal
  objection_handling_score: 0-100  # seberapa baik handle objection (jika ada)

sales_progression:
  intent_progression: [array of intent levels per turn]
  bant_progression: [array of bant scores per turn]
  readiness_progression: [array of readiness scores per turn]
  stage_reached: string          # discovery | qualification | offering | closing | conversion
  next_level_blocker: string     # apa yang block lead dari next level

obstacles:
  - type: "state_stuck | schema_violation | false_conversion | spam_misclassification | escalation_miss | personality_misread | offer_premature | objection_unresolved | fixer_exhausted | timeout"
    turn: integer
    description: string
    severity: "critical | major | minor"
    root_cause: string

bottlenecks:
  - location: string             # state/agent mana
    description: string          # apa bottlenecknya
    impact: string               # dampak ke progression
    frequency: integer           # berapa kali terjadi di simulation ini

qa_summary:
  total_artifacts: integer
  pass_count: integer
  fail_count: integer
  fixer_triggered: integer
  fixer_resolved: integer
  fixer_exhausted: integer
  schema_compliance_rate: float

verdict: "pass | partial | fail"
key_findings: [array of string]
```

**Aggregate analysis (aggregate-report.yaml):**
```yaml
simulation_run_id: string
project: string
generated_at: ISO8601
total_profiles: integer
total_simulations: integer
simulation_duration_total: string

conversion_metrics:
  conversion_rate: float              # % lead yang reach CONVERSION
  blacklist_rate: float              # % lead yang blacklist
  escalation_rate: float             # % lead yang human escalation
  follow_up_rate: float              # % lead yang follow up (cold)
  timeout_rate: float                # % lead yang timeout/ghost
  state_stuck_rate: float            # % lead yang stuck

state_reach_rate:
  NEW_LEAD: 100.0                    # semua mulai di sini
  DISCOVERY: float
  QUALIFICATION: float
  OFFERING: float
  OBJECTION: float
  CLOSING: float
  DATA_CAPTURE: float
  COMPLETION_CHECK: float
  HANDOFF: float
  CONVERSION: float
  FOLLOW_UP: float
  BLACKLIST: float
  HUMAN_ESCALATION: float

ai_quality_metrics:
  avg_tone_match_rate: float
  avg_question_relevance: float
  avg_progress_rate: float
  avg_empathy_score: float
  avg_pushiness_score: float
  avg_objection_handling: float

sales_progression_metrics:
  avg_stage_reached: string          # median stage
  avg_intent_progression: float      # rata-rata intent level naik per turn
  avg_bant_progression: float
  avg_readiness_progression: float
  next_level_blockers:               # top blockers yang halangi lead naik level
    - blocker: string
      frequency: integer
      impact: string

qa_metrics:
  total_artifacts: integer
  total_pass: integer
  total_fail: integer
  fixer_triggered: integer
  fixer_resolved: integer
  fixer_exhausted: integer
  schema_compliance_rate: float
  avg_confidence_score: float

top_obstacles:                        # sorted by frequency
  - type: string
    frequency: integer
    severity: "critical | major | minor"
    description: string
    root_cause: string
    recommendation: string

top_bottlenecks:                      # sorted by impact
  - location: string
    frequency: integer
    description: string
    impact: string
    recommendation: string

personality_breakdown:                # performance per personality type
  analytical:
    count: integer
    conversion_rate: float
    avg_stage_reached: string
    avg_tone_match: float
    key_issues: [array]
  driver: ...
  amiable: ...
  expressive: ...

edge_case_results:                    # edge case handling
  gaptek:
    count: integer
    correctly_handled: integer
    false_positive_spam: integer
  scam:
    count: integer
    correctly_blacklisted: integer
    missed: integer
  toxic:
    count: integer
    correctly_escalated: integer
    missed: integer

recommendations:                       # actionable recommendations
  - priority: "high | medium | low"
    category: "schema | routing | tone | timing | objection | escalation | spam | other"
    description: string
    affected_agents: [array]
    suggested_fix: string

overall_verdict: "healthy | needs_attention | critical"
summary: string                         # narrative summary
```

### STEP 6: GENERATE REPORT
Generate HTML+PDF report berisi:
1. Executive summary (conversion rate, key findings, overall verdict)
2. State reach waterfall (NEW_LEAD → ... → CONVERSION)
3. AI response quality metrics (tone match, empathy, pushiness, etc)
4. Sales progression analysis (intent/bant/readiness progression)
5. Top obstacles + bottlenecks
6. Personality breakdown
7. Edge case results
8. QA metrics
9. Recommendations
10. Per-profile detail (appendix)

Report disimpan di `reports/sim-{timestamp}.html` dan `.pdf`.

## PROFILE TYPE VARIATIONS
Setiap project nuxhub punya konteks bisnis berbeda. Profile generator WAJIB adaptif:

1. Baca `shared/products.yaml` dari project target → tahu produk apa yang ditawarkan
2. Baca `shared/personalities.yaml` → tahu personality types
3. Baca `shared/goals.yaml` → tahu conversion goals
4. Generate profiles yang relevant ke produk tersebut:
   - Contoh: project propolis → profiles yang relevan (reseller, konsumen personal, klinik, apotek)
   - Contoh: project digital agency → profiles bisnis (UMKM, resto, bengkel, klinik)

Distribusi profile types (per 10 profiles):
- 2 analytical, 2 driver, 2 amiable, 2 expressive (8 normal)
- 2 edge cases (scam, gaptek, toxic, enterprise, ghost, off-topic)

## ANALISA FOKUS
Analisa UTAMA adalah: **gimana cara AI menjawab user**

Dievaluasi:
1. **Tone alignment**: AI response cocok dengan personality user?
2. **Question quality**: Pertanyaan relevant? Tidak repetitif? Maju conversation?
3. **Progress speed**: Lead bergerak ke arah conversion atau stuck?
4. **Empathy & rapport**: AI show understanding atau robotic?
5. **Sales timing**: AI offer terlalu cepat? Terlalu lambat?
6. **Objection handling**: AI handle objection dengan logika atau defensif?
7. **Conversion enablement**: AI berhasil bawa lead ke next level?
8. **Context retention**: AI inget info dari turn sebelumnya?
9. **Schema compliance**: Output agent sesuai spec?
10. **False positive/negative**: Spam/escalation miss?

## PATH CONVENTION (FHS)
- Simulation data: `<simulation-data-path>/{project}-simulation/`
- Reports: `<nuxhub-log-path>/simulations/` (copy dari simulation folder)
- Skill code: `<hermes-skills-path>/nuxhub-simulate/`
- Project source: `<your-nuxhub-source-path>/` (atau path lain dari config)

## PARALLEL EXECUTION
- Gunakan delegate_task untuk run multiple simulations concurrently
- Setiap delegate dapat 1 profile + full context
- Max parallel = parameter `parallel` (default 3)
- Hasil dikumpulkan dan di-aggregate setelah semua selesai

## SAFETY
- Max turns per simulation: 30 (hard cap)
- Timeout per simulation: 300 detik
- Fixer loop max: 3 retry per artifact
- Jangan modify project source files
- Semua output ke simulation folder only