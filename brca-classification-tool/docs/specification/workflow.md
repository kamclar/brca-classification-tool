# Workflow

## Současný postup laboratoře

```
VCF soubor (výstup z NGS)
        │
        ▼
┌───────────────────────────────────┐
│  Filtry laboratoře                │
│  • Frekvence (gnomAD)             │
│  • Quality (QUAL, DP, GQ)         │
│  • Impact (HIGH, MODERATE)        │
└───────────────────────────────────┘
        │
        ▼
Prioritizované varianty
(chr, start, end, ref, alt)
        │
        ▼
Přepis do c. a p. notace
        │
        ▼
┌───────────────────────────────────┐
│  MANUÁLNÍ KLASIFIKACE             │
│  (toto chceme podpořit)           │
└───────────────────────────────────┘
        │
        ▼
Finální klasifikace
(Pathogenic / Likely Pathogenic / VUS / Likely Benign / Benign)
```

## Navrhovaný workflow s nástrojem

```
Prioritizované varianty
        │
        ▼
┌───────────────────────────────────┐
│  NÁŠ NÁSTROJ                      │
│                                   │
│  Modul 1: Automatická kritéria    │
│  Modul 2: Literatura search       │
│                                   │
└───────────────────────────────────┘
        │
        ▼
Strukturovaný report s důkazy
        │
        ▼
┌───────────────────────────────────┐
│  ROZHODNUTÍ GENETIKA/LÉKAŘE       │
│  (nástroj NENAHRAZUJE)            │
└───────────────────────────────────┘
        │
        ▼
Finální klasifikace
```

## Typy variant podle dostupnosti dat

| Typ | Popis | Co nástroj může udělat |
|-----|-------|------------------------|
| **Známá P/LP/B/LB** | V ClinVar s jasnou klasifikací | Zobrazit existující klasifikaci |
| **Existující VUS** | V ClinVar jako VUS | Hledat nové důkazy v literatuře |
| **Nová varianta** | Nikde v databázích | In silico predikce + literatura |
