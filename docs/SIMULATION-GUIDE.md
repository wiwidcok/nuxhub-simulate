# Simulation Guide

## Prerequisites
1. Project source code ada di `<your-nuxhub-source-path>/`
2. Skill packs nuxhub terinstall di Hermes (`nuxhub-orchestrator`, `nuxhub-discovery`, dll)
3. Skill `nuxhub-simulate` terinstall

## Menjalankan Simulasi

### Basic Run (10 profiles)
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

Phase options:
- `all` (default) — full pipeline dari NEW_LEAD sampai terminal
- `discovery` — test discovery phase only
- `qualification` — test qualification only
- `offering` — test offering + objection
- `closing` — test closing + handoff
- `follow_up` — test follow up flow

### Custom Parallel Count
```
simulate: {"project": "nuxhub-alpha-propolis", "count": 10, "parallel": 5}
```

## Yang Terjadi Saat Simulasi

### 1. Profile Generation
Engine generate N random profiles dengan variasi:
- 4 personality types (analytical, driver, amiable, expressive)
- 8 business types (klinik, bengkel, resto, dll)
- Random BANT scenarios
- ~20% edge cases (scam, gaptek, toxic, dll)

Profiles disimpan di `profiles/profile-{NNN}.yaml`

### 2. Simulation Run (per profile)
Setiap profile di-run sebagai simulated conversation:
- Profile berperan sebagai "user" yang kirim pesan
- Pipeline nuxhub (orchestrator → specialist → judge) memproses
- AI response dicatat dan dianalisa
- Multi-turn sampai terminal state (conversion, blacklist, escalation, follow_up, timeout)

### 3. QA + Fixer Loop
Setiap artifact dari specialist agent di-QA:
- **Judge** cek schema compliance, data completeness, confidence, objective alignment
- Jika FAIL → **fixer loop** (max 3 retry)
- Fixer kirim error report ke specialist, specialist re-generate
- Re-judge hasil fix

### 4. Analysis
Setelah semua simulasi selesai:
- Per-profile analysis: state progression, AI response quality, obstacles
- Aggregate analysis: conversion rate, top obstacles, personality breakdown, edge case results
- Recommendations untuk perbaikan

### 5. Report
HTML+PDF report di-generate di `reports/`:
1. Executive summary
2. State reach waterfall
3. AI response quality metrics
4. Sales progression analysis
5. Top obstacles + bottlenecks
6. Personality breakdown
7. Edge case results
8. QA metrics
9. Recommendations
10. Per-profile detail (appendix)

## Membaca Hasil

### Aggregate Report
File: `analysis/aggregate-report.yaml`
- `conversion_rate` — % lead yang reach CONVERSION
- `state_reach_rate` — % lead yang reach setiap state
- `ai_quality_metrics` — rata-rata quality AI response
- `top_obstacles` — kendala tersering
- `top_bottlenecks` — bottleneck terbesar
- `recommendations` — saran perbaikan

### Per-Profile Analysis
File: `analysis/sim-{profile-id}-analysis.yaml`
- `state_progression` — trace state per turn
- `ai_response_analysis` — tone match, empathy, pushiness
- `sales_progression` — intent/bant/readiness progression
- `obstacles` — kendala di simulation ini
- `bottlenecks` — bottleneck di simulation ini
- `verdict` — pass/partial/fail

### HTML+PDF Report
File: `reports/sim-{timestamp}.html` + `.pdf`
- Visual report untuk sharing

## Tips
- Run minimum 10 profiles untuk coverage yang meaningful
- Run dengan `phase: "all"` untuk full pipeline test
- Check `aggregate-report.yaml` dulu, terus drill down ke per-profile untuk detail
- Gunakan recommendations dari aggregate report untuk fix issues
- Re-run setelah fix untuk verify improvement