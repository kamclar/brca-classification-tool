# Požadavky

## Vstup

Prioritizované varianty z NGS pipeline, které prošly filtry laboratoře:

| Pole | Popis | Příklad |
|------|-------|---------|
| chr | Chromozom | 17 |
| start | Počáteční pozice | 43094464 |
| end | Koncová pozice | 43094464 |
| ref | Referenční alela | G |
| alt | Alternativní alela | A |

Pro biologa/lékaře přepis na úrovni **c.** a **p.** dle definovaných transkripčních variant:
- BRCA1: NM_007294.4
- BRCA2: NM_000059.4

## Výstup

Strukturovaný report obsahující:

1. **Automaticky zjištěná kritéria** s odkazy na zdroje
2. **Nalezené důkazy z literatury** (pokud existují)
3. **Navržené ACMG skóre** a klasifikace
4. **Upozornění** na nejistoty nebo chybějící data

## Otevřené otázky

> 🔲 Čeká na odpověď od zadavatele

### 1. Odpovídá dvoustupňový návrh vašim představám?

### 2. Co je hlavní problém, který řešíte?

- [ ] a) Klasifikace úplně nových variant (není v ClinVar, gnomAD, literatuře)?
- [ ] b) Reklasifikace existujících VUS (posunout na Likely Pathogenic/Benign)?
- [ ] c) Časová náročnost klasifikace?
- [ ] d) Potřeba konzistentního postupu?
- [ ] e) Něco jiného?

### 3. Jak často a kolik variant?

- Kolik variant typicky klasifikujete za měsíc/rok?
- Kolik z nich jsou VUS vs. úplně nové?

### 4. Jaké nástroje používáte nyní?

- WARSAM, HerediVar, jiné?
- Co jim chybí?

### 5. Jaký má být výstup?

- Report pro lékaře (Word/PDF)?
- Data do databáze (JSON)?
- Podklad pro ClinVar submission?

### 6. Kdo bude nástroj používat?

- Bioinformatik?
- Genetik/lékař?
- Oba?

---

## Historie dokumentu

| Datum | Změna |
|-------|-------|
| 2026-04-12 | Zaslány odpovědi od J.S. (ENIGMA_ELIXIR_guAIdlines_20260412_JS.docx) |
| 2026-04-08 | Zaslán úvodní dokument od zadavatele |
| 2026-04-03 | Úvodní schůzka |
