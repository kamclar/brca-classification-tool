# Přehled architektury

## Vysokoúrovňový pohled

```
┌─────────────────────────────────────────────────────────────────────┐
│                         BRCA Classification Tool                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐             │
│  │   VSTUP     │    │   MODULY    │    │   VÝSTUP    │             │
│  │             │    │             │    │             │             │
│  │ • VCF       │───▶│ • Modul 1   │───▶│ • JSON      │             │
│  │ • HGVS      │    │ • Modul 2   │    │ • Report    │             │
│  │             │    │             │    │             │             │
│  └─────────────┘    └─────────────┘    └─────────────┘             │
│                            │                                        │
│                            ▼                                        │
│                    ┌─────────────┐                                  │
│                    │   ZDROJE    │                                  │
│                    │             │                                  │
│                    │ • gnomAD    │                                  │
│                    │ • ClinVar   │                                  │
│                    │ • PubMed    │                                  │
│                    │ • BayesDel  │                                  │
│                    │ • SpliceAI  │                                  │
│                    └─────────────┘                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Moduly

### Modul 1: ACMG Engine

Automatické vyhodnocení kritérií na základě databází a in silico predikcí.

**Vstup:** Varianta (chr, pos, ref, alt) nebo HGVS notace

**Výstup:** 
```json
{
  "variant": "BRCA1 c.181T>G",
  "criteria": {
    "PM2_Supporting": {
      "met": true,
      "evidence": "gnomAD AC=0, depth=45",
      "points": 1
    },
    "PP3": {
      "met": true,
      "evidence": "BayesDel=0.42, SpliceAI=0.01",
      "points": 1
    }
  },
  "total_points": 2
}
```

### Modul 2: Literatura Search

Vyhledání a extrakce důkazů z vědecké literatury pomocí LLM.

**Vstup:** Varianta + výsledky z Modulu 1

**Výstup:**
```json
{
  "variant": "BRCA1 c.181T>G",
  "literature_evidence": [
    {
      "pmid": "12345678",
      "criterion": "PS3",
      "assay_type": "HDR",
      "cell_line": "HAP1",
      "result": "abnormal",
      "activity_percent": 15,
      "quote": "The C61G variant showed significantly reduced HDR activity..."
    }
  ]
}
```

## Technologie

| Komponenta | Technologie | Poznámka |
|------------|-------------|----------|
| Backend | Python | Hlavní logika |
| LLM | Mistral 7B / Claude API | Extrakce z článků |
| Databáze | SQLite / PostgreSQL | Cache, výsledky |
| UI | Streamlit | MVP |
| Notebooky | Colab | Vývoj, prototypy |
