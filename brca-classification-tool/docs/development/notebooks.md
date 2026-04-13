# Notebooky

## Přehled

| Notebook | Účel | Status |
|----------|------|--------|
| Literature Search Prototype | Modul 2 — PubMed + LLM | 🔄 V vývoji |
| ACMG Engine | Modul 1 — automatická kritéria | 🔲 Plánováno |
| Data Exploration | Průzkum BRCA Exchange, ClinVar | ✅ Hotovo |

## Literatura Search Prototype

**Umístění:** `notebooks/BRCA_Literature_Search_Prototype.ipynb`

**Co dělá:**
1. Vyhledá články v PubMed pro zadanou variantu
2. Stáhne abstrakty a PMC full-text
3. Použije Mistral 7B pro extrakci strukturovaných dat
4. Vrátí JSON s informacemi o funkčních studiích

**Jak spustit:**
1. Otevři v Google Colab
2. Připoj Google Drive
3. Nastav GPU runtime (T4)
4. Spusť všechny buňky

**Testovací varianta:** `BRCA1 c.181T>G (p.Cys61Gly)`

## Známé problémy a řešení

### PMC ID extrakce

**Problém:** PMC ID se někdy nenačte správně z PubMed XML

**Řešení:** Hledat ve dvou místech:
```python
# Cesta 1
article.findall('.//ArticleIdList/ArticleId[@IdType="pmc"]')
# Cesta 2
article.findall('.//PubmedData/ArticleIdList/ArticleId[@IdType="pmc"]')
```

### LLM klasifikace

**Problém:** LLM označovalo haplotypové analýzy jako funkční studie

**Řešení:** Explicitní seznam v promptu co NENÍ funkční assay:
```
- NOT functional assays: 
  - haplotype, population genetics, case-control, genotyping, sequencing
  - expression studies on tumor samples
  - clinical outcome studies
```

## Colab links

> TODO: Doplnit linky na sdílené notebooky
