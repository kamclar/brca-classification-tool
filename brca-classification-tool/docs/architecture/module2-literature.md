# Modul 2: Literatura Search

## Účel

Vyhledání a extrakce důkazů z vědecké literatury pro kritéria, která nelze automaticky vyhodnotit (PS3/BS3, PP1/BS4, PS4).

## Architektura

```
┌─────────────────────────────────────────────────────────────────┐
│                    MODUL 2: LITERATURA SEARCH                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  VSTUP: BRCA1 c.181T>G                                          │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  2A: Strukturované databáze (nejdříve!)                  │   │
│  │                                                          │   │
│  │  → LOVD: funkční studie? splicing data?                  │   │
│  │  → BRCA Exchange: agregovaná evidence?                   │   │
│  │  → ClinVar submissions: co říkají jednotlivé laby?       │   │
│  └──────────────────────────────────────────────────────────┘   │
│                          │                                       │
│                          ▼                                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  2B: Vyhledání v literatuře                              │   │
│  │                                                          │   │
│  │  → PubMed search (E-utilities API)                       │   │
│  │  → Fetch abstrakty                                       │   │
│  │  → Fetch PMC full-text (kde dostupné)                    │   │
│  │  → Fetch bioRxiv/medRxiv (označené jako preprint)        │   │
│  └──────────────────────────────────────────────────────────┘   │
│                          │                                       │
│                          ▼                                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  2C: LLM extrakce                                        │   │
│  │                                                          │   │
│  │  → Mistral 7B / Claude API                               │   │
│  │  → Strukturovaný JSON výstup                             │   │
│  │  → Validace proti ENIGMA approved assays                 │   │
│  └──────────────────────────────────────────────────────────┘   │
│                          │                                       │
│                          ▼                                       │
│  VÝSTUP: Evidence pro PS3/BS3, PP1/BS4, PS4                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Co hledáme v článcích

### Funkční studie (PS3/BS3)

| Otázka | Proč je důležitá |
|--------|------------------|
| Testovali tuto konkrétní variantu? | Přímý důkaz |
| Jaký assay použili? | HDR, SGE, MANO-B, splicing... |
| Jakou buněčnou linii? | HAP1, HEK293... |
| Měli správné kontroly? | WT, known pathogenic |
| Jaký byl výsledek? | % aktivity, normal/abnormal |
| Je assay v ENIGMA Table 9? | Validovaný assay |

### Segregační data (PP1/BS4)

| Otázka | Proč je důležitá |
|--------|------------------|
| Kolik rodin studovali? | Síla důkazu |
| Segreguje varianta s rakovinou? | Kosegregace |
| Jaká byla metodika? | LOD skóre |

### Case-control (PS4)

| Otázka | Proč je důležitá |
|--------|------------------|
| Kolik pacientů vs kontrol? | Velikost studie |
| Jaký OR/RR? | Síla asociace |
| Byla populace matched? | Validita |

## LLM Extraction Prompt

```python
EXTRACTION_PROMPT = """
You are analyzing a scientific article about BRCA1/BRCA2 variants.
Extract information about functional studies for the variant: {variant}

IMPORTANT:
- Functional assays test protein/cell function IN VITRO: HDR, SGE, MANO-B, 
  splicing/minigene, transcription activation, cell viability, protein binding
- NOT functional assays: 
  - haplotype, population genetics, case-control, genotyping, sequencing
  - expression studies on tumor samples
  - clinical outcome studies
- The variant may be tested directly OR used as a control

Article text: {text}

Respond ONLY in valid JSON:
{
  "variant_mentioned": true/false,
  "tested_as": "direct/control/mentioned_only/NOT_FOUND",
  "functional_study_found": true/false,
  "assay_type": "HDR/SGE/MANO-B/splicing/transcription/other/NOT_FOUND",
  "cell_line": "cell line name or NOT_FOUND",
  "controls_used": ["list"] or "NOT_FOUND",
  "result": "normal/abnormal/intermediate/NOT_FOUND",
  "activity_percent": number or "NOT_FOUND",
  "conclusion": "pathogenic/benign/uncertain/NOT_FOUND",
  "key_quote": "direct quote max 150 chars"
}
"""
```

## Zdroje dat

### PubMed / PMC

- **API:** E-utilities (esearch, efetch)
- **Full-text:** PMC (Open Access subset)
- **Limitace:** ~40% článků má full-text v PMC

### bioRxiv / medRxiv

- **API:** Europe PMC (`SRC:PPR` filter)
- **Výhoda:** Nejnovější data
- **⚠️ Označit jako:** `is_peer_reviewed: false`

### LOVD

- **URL:** https://databases.lovd.nl/shared/genes/BRCA1
- **Výhoda:** Kurátorované funkční studie
- **API:** TODO zjistit

### BRCA Exchange

- **URL:** https://brcaexchange.org/
- **Výhoda:** Agregovaná data z více zdrojů
- **API:** REST API dostupné

## Status implementace

| Komponenta | Status | Poznámka |
|------------|--------|----------|
| PubMed search | ✅ Hotovo | E-utilities |
| PMC full-text fetch | ✅ Hotovo | Opravena extrakce PMC ID |
| LLM extrakce | 🔄 V vývoji | Prompt iterace |
| bioRxiv search | 🔄 V vývoji | Europe PMC API |
| LOVD lookup | 🔲 TODO | |
| BRCA Exchange | 🔲 TODO | |

## Testovací varianty

| Varianta | Gen | Očekáváno | Status testu |
|----------|-----|-----------|--------------|
| c.181T>G (C61G) | BRCA1 | Pathogenic | ✅ Testováno |
| c.5096G>A (R1699Q) | BRCA1 | Benign | 🔲 TODO |
| c.5207T>C (V1736A) | BRCA1 | Benign | 🔲 TODO |
| c.5095C>T (R1699W) | BRCA1 | Pathogenic | 🔲 TODO |
| c.9976A>T (K3326*) | BRCA2 | Benign | 🔲 TODO |
