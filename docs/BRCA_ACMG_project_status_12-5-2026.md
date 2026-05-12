# BRCA1/2 ACMG Automatic Classification — stav projektu

Datum: květen 2026  
Referenční specifikace: ENIGMA VCEP v1.2  
Testovací sada: 13 variant z EQA cvičení

---

## Notebooky

| Notebook | Účel |
|---|---|
| `BRCA_ACMG_Criteria_Module1_v1_5_6.ipynb` | Hlavní klasifikační pipeline |
| `BRCA_ACMG_Prepare_gnomAD_BRCA_Region_Cache_v1_2.ipynb` | Příprava lokální gnomAD cache (jednorázově) |
| `BRCA_ACMG_Prepare_gnomAD_Coverage_BRCA_v1_0.ipynb` | Příprava gnomAD coverage dat (jednorázově) |
| `BRCA_ACMG_Prepare_SpliceAI_BRCA_Subset_v1_4_3.ipynb` | Příprava lokálního SpliceAI subsetu (jednorázově) |

---

## Implementovaná kritéria

### PVS1 - Loss of function variants
- Rozhodovací strom podle ENIGMA Table 4
- Frameshift a nonsense: NMD predikce pomocí ověřených exonových hranic (CDS souřadnice z NM_007294.4 / NM_000059.4, celková délka BRCA1 = 5592 bp, BRCA2 = 10257 bp)
- NMD escape: poslední exon nebo posledních 50 bp předposledního exonu -> PVS1_Moderate (+2)
- Speciální exony: exon 10 v BRCA1 i BRCA2 — downgrade platí pouze pro splice_site varianty kde skip exonu produkuje in-frame izoformu; frameshift/nonsense uvnitř exonu 10 dostane PVS1_Very Strong
- Splice site varianty: canonical ±1,2  -> Very Strong; non-canonical vyžaduje SpliceAI ≥ 0.2  -> Supporting; canonical s nízkým SpliceAI  -> flagováno pro manuální review
- Exon delece: nelze automaticky rozlišit in-frame vs. out-of-frame  -> flagováno

### BP1_Strong - Varianta mimo funkční doménu
- Aplikuje se na: missense, synonymous/silent, inframe_deletion, inframe_insertion, inframe_delins, delins
- Podmínky: varianta mimo klinicky důležitou doménu AND SpliceAI ≤ 0.1 (musí být potvrzeno, ne chybějící)
- Funkční domény BRCA1: RING (2–101), coiled-coil (1391–1424), BRCT (1650–1857)
- Funkční domény BRCA2: PALB2 binding (10–40), DNA binding domain (2481–3186)
  - BRC repeats nejsou zahrnuty — ENIGMA VCEP v1.2 Appendix J je neuvádí pro BP1/PP3/BP4

### PP3 / BP4 — In silico predikce
- BayesDel_noAF s gene-specific prahy (BRCA1: PP3 ≥ 0.28, BP4 ≤ 0.15; BRCA2: PP3 ≥ 0.30, BP4 ≤ 0.18)
- SpliceAI: PP3 ≥ 0.2; BP4 vyžaduje SpliceAI ≤ 0.1 AND BayesDel ≤ threshold
- PP3 ze SpliceAI se neaplikuje na splice_site varianty (ty jdou přes PVS1), nonsense/PTC, frameshift, ani exon delece
- PP3 a BP4 jsou oba Supporting (+1 / -1); PP3 ze SpliceAI a BayesDel se nestackují
- Rozsah mezi prahy BP4 a PP3 je záměrně neinformativní

### BP7 - Synonymous varianta bez splice efektu
- Aplikuje se pouze na silent/synonymous varianty UVNITŘ funkční domény a pouze pokud bylo splněno BP4
- Silent varianty mimo doménu dostávají BP1_Strong (ne BP7)
- Vyžaduje potvrzené SpliceAI ≤ 0.1 (None = nekritizovat)

### BA1 / BS1 / PM2 - Populační frekvence
- Zdroj: lokální BRCA1/2 gnomAD cache (GRCh37, gnomAD v2.1 non-cancer)
- BA1: max_AF > 0.1%, BS1_Strong: > 0.01%, BS1_Supporting: > 0.002%
- PM2_Supporting: varianta absent z gnomAD (status = "absent") AND pokrytí ≥ 25×
- PM2 se **neaplikuje** pro: indely (frameshift, inframe_deletion/insertion/delins, exon_deletion); API error; chybějící koordináty; nevyplněnou cache
- Stav gnomAD lookupového výsledku je explicitně sledován: `found` / `absent` / `api_error` / `not_queried` / `no_coordinates` / `cache_missing`

---

## Datové zdroje a infrastruktura

### CoordinateResolver
- Centrální první krok pro všechny lookups
- Pořadí: VariantValidator  -> Mutalyzer → hardcoded fallback zatim takhle (pouze pro 13 testovacích variant, `USE_COORDINATE_FALLBACK = True`)
- Vrací GRCh37 a GRCh38 koordináty v jednom objektu `ResolvedVariant`
- gnomAD/myvariant.info používá GRCh37, SpliceAI používá GRCh38

### gnomAD — lokální cache
- Přípravný notebook `Prepare_gnomAD_BRCA_Region_Cache_v1_2` stáhne BRCA1/2 region z gnomAD VCF přes tabix (remote nebo lokálně) a uloží jako JSON cache na Google Drive
- Přípravný notebook `Prepare_gnomAD_Coverage_BRCA_v1_0` doplní reálná coverage data (mean depth) ke každé pozici
- Hlavní notebook načte cache ze souboru, bez live API volání za běhu klasifikace
- Aktuální stav v testovací sadě: cache nenačtena (`cache_missing`) — soubory na Drive chybí nebo nejsou připojeny

### BayesDel_noAF
- Zdroj: myvariant.info REST API, endpoint `/v1/variant/{hgvs}`, HGVS ve formátu GRCh37
- Správný field path: `dbnsfp.bayesdel.no_af.score` (ne `dbnsfp.bayesdel_noaf_score` — to v aktuálním schématu neexistuje)
- Cache v paměti na dobu session

### SpliceAI
- Lokální BRCA1/2 subset VCF (`spliceai_brca_hg38.vcf.gz`) připravený přípravným notebookem
- Přípravný notebook `Prepare_SpliceAI_BRCA_Subset_v1_4_3` extrahuje BRCA1/2 regiony z Ensembl precomputed VCF (remote tabix nebo lokální stažení), zapíše bgzipped + tabix-indexed subset
- Hlavní notebook načítá subset do paměti při startu; 497 487 SNV záznamů
- Indely v aktuálním subsetu nejsou (Ensembl soubor je SNV-only)

---

## Co chybí (prioritizováno)

### Infrastruktura (blokující)
- **gnomAD cache** musí být nahrána na Drive a připojena — bez ní BA1/BS1/PM2 nefunguje pro žádnou variantu
- **Coordinate fallback** je hardcoded jen pro 13 testovacích variant; pro produkci potřeba spolehlivý VariantValidator/Mutalyzer

### Kritéria (implementační mezery)
- **FAF místo raw AF** pro BA1/BS1 — ENIGMA specifikuje FAF95 z non-founder populací
- **gnomAD v3.1** (GRCh38) zatím nekontrolováno — PM2 by měl vyžadovat absenci v obou
- **PS1** — ClinVar lookup pro stejnou aminokyselinovou změnu jako známý P/LP variant (infrastruktura je, funkce chybí)
- **PM5_PTC** — exon-specific váhy pro předčasné stop kodony
- **Exon deletion frame detection** — in-frame vs. out-of-frame automaticky
- **SpliceAI pro indely** — aktuální subset je SNV-only

### Moduly (budoucí práce)
- **PS3/BS3** — MAVE lookup tabulka (Findlay, Cheng data pro BRCA1/2)
- **PP1/BS4** — segregační data (nelze snadno automatizovat)
- **Lokální BayesDel tabulka** pro BRCA1/2 (eliminovat závislost na myvariant.info)
- **Modul 2** — literární vyhledávání pro PS3, PP1, PS4

