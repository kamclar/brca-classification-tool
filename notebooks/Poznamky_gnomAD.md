# Co je gnomAD

**gnomAD** (*Genome Aggregation Database*) je databáze, která shromažďuje data ze sekvenování od stovek tisíc lidí:

- přibližně **140 000 exomů**  
  *(kódující oblasti genomu)*
- přibližně **76 000 genomů**  
  *(celé genomy)*

---

## K čemu gnomAD slouží

gnomAD říká, **jak častá je konkrétní genetická varianta v populaci**.

Pokud je varianta:

- **velmi častá**  
  například **> 0,1 % populace**,  
  je pravděpodobně **benigní**, protože tolik lidí by pravděpodobně nemělo závažné genetické onemocnění.

- **vzácná nebo v gnomAD úplně chybí**,  
  může být **patogenní**, ale samotná absence nestačí jako důkaz patogenity.

---

## Příklad

Máme variantu:

```text
BRCA1 c.181T>G
```

Zeptáme se gnomAD:

> Kolik lidí z těch přibližně 140 000 má tuto variantu?

Odpověď:

```text
0 lidí
```

Varianta je tedy v gnomAD **absentní**.  
To znamená, že **může být patogenní**, ale není to samo o sobě definitivní důkaz.

---

Jiná varianta:

```text
BRCA1 c.XXX
```

Odpověď v gnomAD:

```text
1500 lidí
```

To odpovídá přibližně:

```text
1 %
```

Varianta je tedy **běžná v populaci** a je pravděpodobně **benigní**.

---

## ENIGMA pravidla pro populační frekvence

| Co říká gnomAD | Kritérium | Význam |
|---|---|---|
| **AF > 0,1 %** | **BA1** | Benigní kritérium. Varianta je považována za benigní a není potřeba ji dál řešit jako patogenní. |
| **AF > 0,01 %** | **BS1 Strong** | Silný důkaz pro benigní interpretaci. |
| **AF > 0,002 %** | **BS1 Supporting** | Slabší podpůrný důkaz pro benigní interpretaci. |
| **Absent** | **PM2** | Slabý důkaz pro patogenitu, ale pouze pro **SNV**. |

---

## Proč se používá „non-cancer“ subset

ENIGMA vyžaduje použití subsetu bez onkologických pacientů, tedy **non-cancer subset**.

Důvod:

Pokud má někdo patogenní variantu v **BRCA1** nebo **BRCA2**, mohl mít vyšší riziko nádorového onemocnění a mohl se proto dostat do databáze právě kvůli onkologické diagnóze.

To by mohlo zkreslit populační frekvenci:

```text
patogenní varianta by vypadala častější,
než skutečně je v běžné populaci
```

Proto se pro hodnocení frekvencí podle ENIGMA používá subset bez onkologických pacientů.

---

## Proč se PM2 nepoužívá pro indely

**Indely** jsou inserční nebo deleční varianty.

Tyto varianty jsou technicky obtížnější na detekci než jednoduché jednonukleotidové varianty (**SNV**).

gnomAD je proto může podhodnocovat:

```text
varianta může být absentní ne proto, že je opravdu vzácná,
ale proto, že ji sekvenování nebo variant calling nezachytily spolehlivě
```

Proto ENIGMA říká:

> PM2 pro indely nepoužívat.

PM2 se tedy v tomto kontextu používá pouze pro **SNV**.

---

## Poznámka k notebooku

Tato část odpovídá:

```text
buňka 10 – gnomAD Lookup
```

v notebooku:

```text
BRCA_ACMG_Criteria_Module1_v1.1.ipynb
```
