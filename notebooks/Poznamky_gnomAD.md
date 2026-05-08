**Co je gnomAD**

gnomAD (Genome Aggregation Database) je databáze která shromažďuje data ze sekvenování od stovek tisíc lidí:



\~140,000 exomů (kódující oblasti)

\~76,000 genomů (celé genomy)



**K čemu to slouží**

Říká jak častá je varianta v populaci.

Pokud varianta:



Je velmi častá (>0.1% populace) -> pravděpodobně není patogenní, protože tolik lidí by nemělo nemoc

Je vzácná nebo chybí -> může být patogenní (ale nemusí)



**Příklad**

Máme variantu BRCA1 c.181T>G. Zeptáme se gnomAD:



"Kolik lidí z těch 140,000 má tuto variantu?"

Odpověď: 0 lidí -> varianta je absent -> může být patogenní



Jiná varianta BRCA1 c.XXX:



Odpověď: 1500 lidí (1%) -> varianta je běžná -> pravděpodobně benigní



**ENIGMA pravidla pro frekvence**

Co gnomAD říká		Kritérium		Význam

AF > 0.1%		BA1			Benign (hotovo, nic dalšího neřešit)

AF > 0.01%		BS1 			StrongSilný důkaz pro benign

AF > 0.002%		BS1 Supporting		Slabý důkaz pro benign

Absent			PM2			Slabý důkaz pro patogenní (jen pro SNV!)





**Proč "non-cancer"**

ENIGMA vyžaduje subset bez onkologických pacientů. Důvod: Pokud má někdo BRCA mutaci, pravděpodobně měl rakovinu a je v databázi. To by zkreslilo frekvence - patogenní varianty by vypadaly častější než jsou v běžné populaci.



**Proč PM2 ne pro indely**

Indely (inserční/deleční varianty) jsou technicky obtížnější na detekci. gnomAD je může podhodnocovat - varianta může být absent ne proto že je vzácná, ale proto že ji sekvenování nezachytilo. Proto ENIGMA říká: "PM2 pro indely nepoužívat."



&#x20;buňka 10 (gnomAD Lookup) v notebooku *BRCA\_ACMG\_Criteria\_Module1\_v1.1.ipynb*

