# Profile Types Reference

## Personality Types (4 tipe)

### 1. Analytical
- **Ciri**: Minta data spesifik, angka, statistik. Bahasa formal, terstruktur.
- **Signals**: "berapa", "gimana caranya", "apa buktinya", "data nya dong"
- **Tone rules**: Kasih data, jangan lebay, struktur: fakta → data → kesimpulan
- **Objection style**: Minta ROI calculation, case study, perbandingan
- **Expected AI behavior**: Agent harus kasih data, bukan klaim kosong

### 2. Driver
- **Ciri**: Mau cepat, to the point, decisive. Pesan pendek.
- **Signals**: "langsung aja", "berapa lama", "kapan mulai", "mau", "perlu"
- **Tone rules**: Kasih opsi 2-3, rekomendasi tegas, highlight speed + hasil
- **Objection style**: "Terlalu lama", "Ada yang lebih cepat?"
- **Expected AI behavior**: Agent harus singkat, jangan bertele-tele

### 3. Amiable
- **Ciri**: Cerita panjang, relationship-focused, butuh koneksi personal.
- **Signals**: "saya rasa", "menurut saya", "pengalaman", cerita personal
- **Tone rules**: Bangun rapport, validasi perasaan, storytelling, bahasa hangat
- **Objection style**: "Nanti saya diskusi keluarga dulu", "Saya mikir-mikir dulu"
- **Expected AI behavior**: Agent harus empati, jangan buru-buru

### 4. Expressive
- **Ciri**: Antusias, vision-driven, big picture, suka ide baru.
- **Signals**: "bayangkan", "potensinya", banyak seru (!), emoji
- **Tone rules**: Match energi, gambarkan visi, jangan terlalu technical
- **Objection style**: "Tapi nanti gimana?", "Besar ya potensinya?"
- **Expected AI behavior**: Agent harus antusias tapi profesional

## BANT Scenarios

### Budget
| Value | Score | Tier Impact |
|-------|-------|-------------|
| besar (>20jt) | 25 | hot candidate |
| ada (5-20jt) | 20 | warm/hot |
| terbatas (<5jt) | 10 | warm/cold |
| nihil | 0 | cold |

### Authority
| Value | Score |
|-------|-------|
| sendiri | 25 |
| atasan | 15 |
| staf | 10 |
| ragu | 5 |

### Need
| Value | Score |
|-------|-------|
| urgent | 25 |
| medium | 20 |
| low | 10 |
| tidak_sadari | 5 |

### Timeline
| Value | Score |
|-------|-------|
| segera | 25 |
| 1bulan | 20 |
| 3bulan | 10 |
| tidak_tahu | 5 |

## Intent Levels (8 rungs)
```
0: unknown         — "Halo", "info dong"
1: curious         — "Boleh info", "produk apa"
2: problem_aware   — "Susah", "butuh solusi"
3: solution_aware  — "Butuh iklan", "butuh konsultasi"
4: product_aware   — "Harganya berapa", "paketnya apa"
5: intent_confirmed — "Mau", "saya ambil", "gimana ordernya"
6: data_complete   — "Sudah kasih nomor", "sudah kasih alamat"
7: transaction_ready — "Produk jelas", "kontak jelas", "metode bayar jelas"
8: converted        — "Handoff sent", "order created"
```

## Edge Cases (~20% dari total profiles)

### gaptek
- Typo berat, bahasa campur Indo+daerah
- Pesan singkat, tidak jelas
- Test: AI harus sabar, toleransi, tidak anggap spam

### scam
- Pattern: transfer, rekening, pinjaman, ALL CAPS urgency
- Test: Spam gate harus detect (+50 → +100 → blacklist)

### toxic
- Kata: bodoh, goblok, anjing, bangsat, penipu
- Test: Toxic detection (+20), 2x → escalation

### enterprise
- Kata: enterprise, perusahaan, korporat, banyak cabang
- Test: Escalation gate harus trigger → human handoff

### ghost
- User balas 1-2 turn terus hilang
- Test: Follow-up flow harus trigger

### off_topic
- Pesan tidak relate ke bisnis
- Test: AI harus arahkan kembali, tidak spam-flag langsung

## Objection Types (Propolis Context)

| Type | Trigger | Root Cause | Expected AI Response |
|------|---------|------------|---------------------|
| mahal | "Mahal juga ya" | Value belum jelas | ROI framing, cost per unit |
| belum_yakin_khasiat | "Apa benar efektif?" | Trust issue | Testimonial, studi kasus |
| nanti_dulu | "Saya pikir-pikir dulu" | Belum ready | Gentle follow-up, no pressure |
| sudah_punya_brand_lain | "Sudup pakai brand X" | Switching cost | Differentiation, value unique |
| tidak_ada_budget | "Belum ada budget" | Budget block | Alternatif, konsultasi gratis |
| belum_bisa_keputusan | "Tanya keluarga dulu" | Authority gap | Support, info untuk share |
| perlu_tanya_dokter | "Dokter sy dulu" | Trust+medical | Info ilmiah, komposisi |
| takut_efek_samping | "Takut efek samping" | Safety concern | Info keamanan, dosis |