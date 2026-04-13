# Datové zdroje

## Přehled

| Zdroj | Typ | API | Použití |
|-------|-----|-----|---------|
| gnomAD | Frekvence | REST | BA1, BS1, PM2 |
| ClinVar | Klasifikace | E-utilities | PS1, PM5, existující klasifikace |
| PubMed | Literatura | E-utilities | PS3, BS3, PP1, PS4 |
| PMC | Full-text | E-utilities | PS3, BS3 |
| BayesDel | In silico | Precomputed DB | PP3, BP4 |
| SpliceAI | In silico | Precomputed DB / API | PP3, BP4 |
| BRCA Exchange | Agregace | REST | Ověření, doplnění |
| LOVD | Variace | REST (?) | Funkční studie |

## gnomAD

### Použití

- **BA1:** FAF > 0.1% → Stand-alone Benign
- **BS1:** FAF > 0.01% → Strong; FAF > 0.002% → Supporting
- **PM2:** AC = 0, depth ≥ 25 → Supporting

### Důležité poznámky

- Použít **FAF** (Filtering Allele Frequency), ne raw AF
- Použít **popmax** bez ASJ (Ashkenazi) a FIN (Finnish) — founder populace
- Kontrolovat **read depth** (min 20 pro BA1, min 25 pro PM2)
- Použít **non-cancer** subset

### API

```
https://gnomad.broadinstitute.org/api
```

GraphQL API, dokumentace: https://gnomad.broadinstitute.org/api-docs

## ClinVar

### Použití

- Existující klasifikace varianty
- PS1: Stejná aminokyselina, jiný nukleotid, pathogenic
- PM5: Stejná pozice, jiná aminokyselina, pathogenic

### API

NCBI E-utilities:
```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/
```

### Příklad

```python
# Hledání varianty
esearch_url = f"https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=clinvar&term=BRCA1[gene]+AND+c.181T>G"

# Fetch detailů
efetch_url = f"https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=clinvar&id={id}&rettype=vcv"
```

## PubMed / PMC

### Použití

- Vyhledání článků o variantě
- Extrakce funkčních studií (PS3/BS3)
- Extrakce segregačních dat (PP1/BS4)

### API

NCBI E-utilities:
```python
# Search
esearch_url = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi"
params = {
    "db": "pubmed",
    "term": "BRCA1 c.181T>G functional",
    "retmax": 50
}

# Fetch abstracts
efetch_url = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi"
params = {
    "db": "pubmed",
    "id": ",".join(pmids),
    "rettype": "abstract"
}

# PMC full-text
pmc_url = f"https://www.ncbi.nlm.nih.gov/research/bionlp/RESTful/pmcoa.cgi/BioC_xml/{pmc_id}/unicode"
```

## BayesDel

### Použití

- **PP3:** BayesDel (no AF) ≥ 0.28
- **BP4:** BayesDel (no AF) ≤ 0.15

### Zdroj

Precomputed scores: https://fenglab.chpc.utah.edu/BayesDel/

~5 GB databáze, potřeba lokální kopie nebo API wrapper

## SpliceAI

### Použití

- **PP3:** SpliceAI ≥ 0.2
- **BP4/BP1:** SpliceAI ≤ 0.1

### Zdroj

- Illumina API: https://spliceailookup.broadinstitute.org/
- Precomputed: https://basespace.illumina.com/s/otSPW8hnhaZR

## BRCA Exchange

### Použití

- Agregovaná data z ClinVar, LOVD, BIC
- Ověření klasifikací

### API

```
https://brcaexchange.org/backend/data/?format=json&search={variant}
```

## LOVD

### Použití

- Kurátorované funkční studie
- Splicing assay výsledky

### URL

- BRCA1: https://databases.lovd.nl/shared/genes/BRCA1
- BRCA2: https://databases.lovd.nl/shared/genes/BRCA2

### API

TODO: Zjistit dostupnost API

## Poznámky k implementaci

### Rate limiting

| Zdroj | Limit | Řešení |
|-------|-------|--------|
| NCBI E-utilities | 3 req/s (bez API key) | API key → 10 req/s |
| gnomAD | Neznámý | Cache |
| BRCA Exchange | Neznámý | Cache |

### Caching

Všechny odpovědi by měly být cachovány:
- gnomAD frekvence se nemění často
- ClinVar klasifikace — cache s TTL 24h
- PubMed výsledky — cache s TTL 7 dní

### Offline fallback

Pro kritické zdroje (gnomAD, BayesDel, SpliceAI) zvážit lokální kopii pro offline použití.
