# ACMG kritéria pro BRCA1/BRCA2

## Zdroj

[ENIGMA VCEP Specifications v1.2.0](https://cspec.genome.network/cspec/ui/svi/doc/GN092)

## Bodový systém

| Síla | Pathogenic body | Benign body |
|------|-----------------|-------------|
| Very Strong | +8 | -8 |
| Strong | +4 | -4 |
| Moderate | +2 | -2 |
| Supporting | +1 | -1 |
| BA1 (Stand-alone) | — | Automaticky Benign |

## Klasifikace podle skóre

| Třída | Skóre | Název |
|-------|-------|-------|
| 5 | ≥10 | Pathogenic |
| 4 | 6–9 | Likely Pathogenic |
| 3 | 0–5 | VUS |
| 2 | -1 až -6 | Likely Benign |
| 1 | ≤-7 | Benign |

## ENIGMA-specifické úpravy

> ⚠️ ENIGMA má vlastní specifikace, které se liší od základních ACMG guidelines!

| Kritérium | Původní ACMG | ENIGMA BRCA | Důvod |
|-----------|--------------|-------------|-------|
| PM2 | Moderate (+2) | Supporting (+1) max | BRCA varianty časté |
| BP1 | Supporting (-1) | Strong (-4) | Mimo domény = silný důkaz |
| PP1 | Supporting (+1) | Škála dle LR | LR≥350 = Very Strong |
| PVS1 | Very Strong (+8) | Dle exonu | NMD escape = downgrade |

## Automatizovatelná kritéria (Modul 1)

| Kritérium | Zdroj dat | Automatizace |
|-----------|-----------|--------------|
| BA1 | gnomAD FAF >0.1% | ✅ Plně |
| BS1 | gnomAD FAF >0.01% | ✅ Plně |
| PM2 | gnomAD AC=0, depth≥25 | ✅ Plně |
| PVS1 | Typ varianty + exon | ✅ Plně |
| PP3/BP4 | BayesDel, SpliceAI | ✅ Plně |
| BP1 | Pozice mimo doménu + SpliceAI≤0.1 | ✅ Plně |
| PS1/PM5 | ClinVar lookup | ✅ Plně |

## Manuální kritéria (Modul 2 — literatura)

| Kritérium | Typ důkazu | Automatizace |
|-----------|------------|--------------|
| PS3/BS3 | Funkční studie | 🔄 AI extrakce z článků |
| PP1/BS4 | Segregace v rodinách | 🔄 AI extrakce z článků |
| PS4 | Case-control | 🔄 AI extrakce z článků |
| PM3 | Fanconi anemia | ❌ Manuální |
| PP4/BP5 | Multifaktoriální LR | ❌ Manuální |

## Funkční domény

### BRCA1

| Doména | Aminokyseliny | Význam |
|--------|---------------|--------|
| RING | 2–101 | E3 ubiquitin ligase |
| Coiled-coil | 1391–1424 | Protein-protein interakce |
| BRCT | 1650–1857 | Fosfopeptid vazba |

### BRCA2

| Doména | Aminokyseliny | Význam |
|--------|---------------|--------|
| PALB2 binding | 21–39 | Vazba PALB2 |
| BRC repeats | 1002–2085 | Vazba RAD51 |
| DBD | 2481–3186 | Vazba DNA |
| NLS | 3263–3269 | Jaderná lokalizace |

## Reference

- [ENIGMA Specifications Document v1.2](https://cspec.genome.network/cspec/ui/svi/doc/GN092)
- [Parsons et al. 2024 - AJHG](https://www.cell.com/ajhg/fulltext/S0002-9297(24)00257-X)
