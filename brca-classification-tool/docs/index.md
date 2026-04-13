# BRCA Classification Tool

## O projektu

Podpůrný nástroj pro klasifikaci germline variant v genech **BRCA1** a **BRCA2** podle [ENIGMA VCEP ACMG/AMP guidelines v1.2.0](https://cspec.genome.network/cspec/ui/svi/doc/GN092).

### Co nástroj dělá

1. **Automaticky vyhodnotí** dostupná ACMG kritéria (frekvence, typ varianty, in silico predikce)
2. **Prohledá literaturu** pro funkční studie a další důkazy
3. **Připraví podklady** pro finální rozhodnutí genetika/lékaře

### Co nástroj NEDĚLÁ

- Nenahrazuje lidský úsudek
- Neposkytuje finální klasifikaci automaticky
- Není certifikovaný pro klinické použití (zatím)

## Proč tento nástroj?

### Problém VUS

- **~5% BRCA testů** v USA končí výsledkem VUS (Variant of Uncertain Significance)
- V ClinVar je **>5000 BRCA2 variant** klasifikovaných jako VUS
- **85% missense variant** v BRCA2 jsou VUS

### Problém nových variant

- Laboratoře pravidelně nacházejí varianty, které nikdo nikdy neviděl
- Pro tyto varianty neexistují data v databázích ani literatuře
- Klasifikace vyžaduje ruční práci a expertízu

## Dvoustupňový přístup

```
┌─────────────────────────────────────────────────────────────┐
│  VSTUP: Prioritizovaná varianta (po filtrech lab)           │
│  chr, start, end, ref, alt → c. a p. notace                 │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  MODUL 1: Automatická ACMG kritéria                         │
│                                                             │
│  ✓ gnomAD frekvence → BA1/BS1/PM2                          │
│  ✓ Typ varianty → PVS1                                     │
│  ✓ Pozice v doméně → BP1/PP3                               │
│  ✓ BayesDel/SpliceAI → PP3/BP4                             │
│  ✓ Podobné varianty v ClinVar → PS1/PM5                    │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  MODUL 2: Vyhledání v literatuře                            │
│  (pro VUS nebo neznámé varianty)                            │
│                                                             │
│  ? Funkční studie → PS3/BS3                                │
│  ? Segregační data → PP1/BS4                               │
│  ? Case-control → PS4                                      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  VÝSTUP: Strukturovaný report                               │
│                                                             │
│  • Nalezená kritéria s důkazy                              │
│  • Navržené skóre                                          │
│  • Zdroje a reference                                      │
│                                                             │
│  → FINÁLNÍ ROZHODNUTÍ NA GENETIKOVI/LÉKAŘI                 │
└─────────────────────────────────────────────────────────────┘
```

## Stav vývoje

| Komponenta | Status |
|------------|--------|
| Specifikace požadavků | 🔄 V jednání |
| Modul 1: ACMG Engine | 🔲 Plánováno |
| Modul 2: Literatura search | 🔄 Prototyp |
| UI | 🔲 Plánováno |
