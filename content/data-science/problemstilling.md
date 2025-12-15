---
title: "Problemstilling"
draft: false
weight: 1
description: "Fra 'log alt' til privacy-first analytics: Dilemmaet mellem indsigt og etik"
---

## Dataguld eller radioaktivt affald?[^6]

Jeg startede Data Science-delen af projektet med en klassisk antagelse: **"Data er guld – vi logger alt."**

Udgangspunktet var en traditionel 'Collect-All' strategi, hvor al data centraliseres. Dette udfordrede jeg gennem Risk Management tænkning (jf. NIST-frameworket), hvor data klassificeres som et Asset, der uden styring bliver til en Liability

Men i et system, der håndterer sundhedsrelaterede spørgsmål, indså jeg hurtigt, at rå data er mindre som guld og mere som **radioaktivt affald: værdifuldt, men farligt.**

Raw message text fra *"Jeg har ondt i hovedet"* er "særlige kategorier af personoplysninger" (helbredsoplysninger) under GDPR Artikel 9[^1]. Det kræver eksplicit samtykke. Uden samtykke er det ulovligt. Med en samtykke-prompt risikerer jeg op mod 70% user drop-off.

Jeg stod med et fundamentalt spørgsmål:
**Hvordan bygger jeg analytics der skaber værdi for produktet – uden at overvåge brugerne eller overtræde loven?**

For at finde svaret måtte jeg gennem tre dybe dilemmaer.

---

## Dilemma 1: Vanity metrics vs. virkelige indsigter

Et simpelt dashboard kan vise **"20 beskeder per session"**. Men det tal fortæller mig intet om kvaliteten. Overvej to scenarier med identiske tal:

**Scenarie A – Succes:**
* **User:** "Hvad er coherence-terapi?"
* **Bot:** *[Giver detaljeret, korrekt svar]*
* **User:** *[Stiller 19 opfølgende spørgsmål fordi emnet er interessant]*
* **Resultat:** 20 beskeder = dyb læring, høj værdi.

**Scenarie B – Fiasko:**
* **User:** "Hvad er coherence-terapi?"
* **Bot:** *[Misforstår, giver irrelevant svar]*
* **User:** *[Omformulerer 19 gange i desperation]*
* **Resultat:** 20 beskeder = total frustration, nul værdi.

To identiske metrics. To modsatte oplevelser.

### Goodhart's Law i praksis
Her ramte jeg **Goodhart's Law**[^2]: *"Når en metric bliver et mål, ophører den med at være en god metric."*

Hvis jeg optimerer Guest Chatbot mod "højt engagement" (flere beskeder), risikerer jeg at belønne en bot, der **misforstår** brugerne. Jo mere forvirret brugeren er, jo flere beskeder skriver de, og jo "bedre" ser mine tal ud.

Jeg havde brug for KPI'er der måler **værdi**, ikke bare **aktivitet**.

---

## Dilemma 2: Privacy-paradokset (GDPR Art. 25)

Product Owneren har et legitimt behov: *"Hvad spørger folk om? Hvor fejler modellen?"* For at forbedre systemet, er vi nødt til at forstå indholdet.

Men loven siger:
* Beskeder om helbred er **følsomme data**.
* **GDPR Artikel 25** kræver "Databekyttelse gennem design"[^3].

Jeg stod i et paradoks: **Vi har brug for indsigt – men må ikke læse beskederne.**

Det tvang mig til at designe analytics med et klart princip: **Systemet skal forstå problemet, men glemme personen.**

Løsningen blev et *Privacy-First* mønster: `Intent Category + SessionHash` i stedet for `Raw Text + UserID`. Vi logger *"Bruger spurgte om Stress"*, men vi logger aldrig *"Jeg er så stresset over min chef"*.

---

## Dilemma 3: Real-time alarmer vs. historiske trends

Systemet har to modstridende behov til arkitekturen:

1.  **Drift (Hot Path):** Hvis modellen fejler nu, skal Owner se det inden for sekunder.
2.  **Forretning (Cold Path):** Fastholdelse (retention) og langsigtede mønstre kræver tunge batch-beregninger over uger af data.

En ren real-time løsning mister historik. En ren batch-løsning reagerer for langsomt.

Jeg indså, at en "one-size-fits-all" pipeline ville bryde sammen. Vi havde brug for en **Lambda Architecture** tilgang[^4], der kan håndtere både alarmklokker og langsigtede trends uden at kvæle databasen.

---

## Tre chatbots, tre succeskriterier

Fordi systemet har tre forskellige brugertyper, gav det ingen mening at blande tallene sammen. Jeg definerede specifikke succeskriterier for hver kontekst:

| Chatbot Type | Primært Mål | Korrekt KPI (Actionable) | Vanity KPI (Undgå) | Konsekvens af fejl |
| :--- | :--- | :--- | :--- | :--- |
| **Guest** | Hurtig afklaring | Avg time-to-answer | Total messages | Bot belønnes for misforståelser |
| **Authenticated** | Task completion | Semantic search success | Session duration | Kan ikke skelne læring fra frustration |
| **Owner** | System stabilitet | Error rate trends | Total users | Problemer opdages for sent |

---

## Min metodiske tilgang

Uden tusindvis af rigtige brugere kan jeg ikke eksperimentere mig frem med A/B tests eller real-world målinger. Jeg er nødt til at designe analytics-kernen korrekt fra start.

For at gøre det valgte jeg en deduktiv, evidensbaseret tilgang:

1. **Systematisk Research** → Hvordan måler industrien brugeroplevelse, privacy og systemstabilitet?
2. **Design Patterns** → Fire arkitektoniske regler der erstatter hypoteser i et miljø uden live data.
3. **Simuleret Validering** → Mock-data, GDPR-audit og arkitektur-tests for at bevise at mønstrene faktisk kan fungere.

Målet er ikke at lave dashboards, men at designe et målesystem der skaber **forretningsindsigt uden at kompromittere etik**.

---

## Næste skridt: Fra dilemma til teori

Jeg ved nu *hvilke* problemer analytics skal løse.  
Nu skal jeg forstå *hvordan* andre systemer i industrien løser dem.

- Hvordan måler Google store brugerflows?  
- Hvordan implementerer man privacy-by-design i praksis?  
- Hvordan kombinerer man real-time fejlalarmer med tunge historiske beregninger?  
- Hvordan undgår man vanity metrics og Goodhart’s Law-fælder?

Det teoretiske fundament skal nu lægges.

**Næste:** [Research →]({{< relref "data-science/research.md" >}})

[^1]: EU General Data Protection Regulation (GDPR), Article 9: Processing of special categories of personal data.
[^2]: Strathern, M. (1997). "Improving ratings: audit in the British University system". European Review.
[^3]: EU General Data Protection Regulation (GDPR), Article 25: Data protection by design and by default.
[^4]: Kleppmann, M. (2017). "Designing Data-Intensive Applications". O'Reilly Media. Chapter 11.
[^5]: Rodden, K., Hutchinson, H., & Fu, X. (2010). "Measuring the User Experience on a Large Scale: HEART". Google Research.
[^6]: Schneier, B. (2016). "Data Is a Toxic Asset: So Why Not Throw It Out?". Schneier on Security.