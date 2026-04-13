# Modul 1: ACMG Engine

## Účel

Automatické vyhodnocení ACMG kritérií, která lze zjistit z databází a in silico predikcí.

## Architektura

```
┌─────────────────────────────────────────────────────────────────┐
│                        MODUL 1: ACMG ENGINE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  VSTUP                                                          │
│  ─────                                                          │
│  Varianta: BRCA1 c.181T>G (p.Cys61Gly)                         │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   gnomAD     │  │   ClinVar    │  │  In silico   │          │
│  │   Lookup     │  │   Lookup     │  │  Predikce    │          │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘          │
│         │                 │                 │                   │
│         ▼                 ▼                 ▼                   │
│  ┌──────────────────────────────────────────────────┐          │
│  │              ENIGMA Rules Engine                  │          │
│  │                                                   │          │
│  │  • Frequency rules (BA1, BS1, PM2)               │          │
│  │  • Variant type rules (PVS1)                     │          │
│  │  • Domain rules (BP1, PP3)                       │          │
│  │  • Similar variant rules (PS1, PM5)              │          │
│  └──────────────────────────────────────────────────┘          │
│                          │                                      │
│                          ▼                                      │
│  VÝSTUP                                                         │
│  ─────                                                          │
│  {                                                              │
│    "criteria": [...],                                           │
│    "total_points": 9,                                           │
│    "suggested_class": "Likely Pathogenic"                       │
│  }                                                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Datové zdroje

### gnomAD

- **Verze:** v2.1 (exomes), v3.1 (genomes)
- **Subset:** non-cancer
- **Klíčové metriky:**
  - FAF (Filtering Allele Frequency)
  - popmax (bez ASJ a FIN)
  - Read depth

### ClinVar

- **API:** E-utilities
- **Hledáme:** 
  - Existující klasifikace
  - Podobné varianty (PS1, PM5)

### In silico predikce

| Nástroj | Kritérium | Práh |
|---------|-----------|------|
| BayesDel (no AF) | PP3 | ≥0.28 |
| BayesDel (no AF) | BP4 | ≤0.15 |
| SpliceAI | PP3 | ≥0.2 |
| SpliceAI | BP4/BP1 | ≤0.1 |

## Implementace kritérií

### Frekvenční kritéria

```python
def get_frequency_criteria(gnomad_data):
    faf = gnomad_data.filtering_allele_frequency
    depth = gnomad_data.read_depth
    
    if depth < 20:
        return None  # Nespolehlivé
    
    if faf > 0.001:  # >0.1%
        return {"BA1": "Stand-alone Benign"}
    
    if faf > 0.0001:  # >0.01%
        return {"BS1_Strong": -4}
    
    if faf >= 0.00002:  # ≥0.002%
        return {"BS1_Supporting": -1}
    
    if gnomad_data.allele_count == 0 and depth >= 25:
        return {"PM2_Supporting": 1}
    
    return None
```

### PVS1 (Loss of Function)

```python
def get_pvs1_score(variant, transcript_info):
    if variant.type not in ['frameshift', 'nonsense', 'splice_site']:
        return None
    
    # NMD escape check
    if is_last_exon(variant) or near_end(variant):
        return {"PVS1_Strong": 4}  # Downgrade
    
    if is_first_exon_no_start(variant):
        return {"PVS1_Moderate": 2}
    
    return {"PVS1": 8}  # Very Strong (default)
```

### BP1 (mimo funkční doménu)

```python
BRCA1_DOMAINS = {
    'RING': (2, 101),
    'coiled_coil': (1391, 1424),
    'BRCT': (1650, 1857)
}

def get_bp1_score(variant, spliceai_score):
    if spliceai_score > 0.1:
        return None  # Možný splicing efekt
    
    if not in_functional_domain(variant, BRCA1_DOMAINS):
        return {"BP1_Strong": -4}
    
    return None
```

## Status implementace

| Kritérium | Status | Poznámka |
|-----------|--------|----------|
| BA1 | 🔲 TODO | |
| BS1 | 🔲 TODO | |
| PM2 | 🔲 TODO | |
| PVS1 | 🔲 TODO | Potřeba exon-specific tabulka |
| PP3/BP4 | 🔲 TODO | BayesDel + SpliceAI |
| BP1 | 🔲 TODO | |
| PS1/PM5 | 🔲 TODO | ClinVar lookup |
