---
title: "4. Semester Portfolio - Simon"
---

# Velkommen: Fra Intuition til Metodisk Valg

## Intro
Denne portefølje dokumenterer mit arbejde på 4. semester med fokus på **Systemudvikling**, **Teknologi** og **Programmering**. 

Målet har været at professionalisere min tilgang: At bevæge mig fra antagelser til beslutninger funderet i systematisk research, arkitektonisk analyse og metodisk refleksion. Porteføljen er struktureret som en **Læringsrejse** – med afsæt i principperne for *Studentercentreret Læring* – hvor jeg dokumenterer processen fra problemformulering til metodisk løsning.

---

## Oversigt: Hvad finder du her?

Portfolioen er struktureret gennem 7 faglige stationer:

### 1. 📋 [Om Projektet]({{< relref "om-projektet/_index.md" >}})
**Konteksten:** Overblik over projektets forretningsmål samt min rolle og specialisering i den samlede arkitektur.

### 2. 🎯 [Læringsmål]({{< relref "laeringsmaal/_index.md" >}})
**Kompasset:** De faglige læringsmål, der har styret min proces fra start til slut.

### 3. 🗄️ [Database & Storage]({{< relref "database/_index.md" >}})
**Analysen:** Fra "Polyglot Persistence" til "Unified Monolith".
Min hypotese var, at systemet krævede tre specialiserede databaser. Gennem **konceptuel validering** og **komparativ analyse** argumenterede jeg for, at en samlet løsning (**PostgreSQL**) reducerer *Accidental Complexity* og *Integration Tax* uden at gå på kompromis med performance.

### 4. 📊 [Data Science & Analytics]({{< relref "data-science/_index.md" >}})
**Analysen:** Fra "Data er Guld" til "Data er Liability".
Med afsæt i **NIST-frameworket** og **GDPR** redefinerede jeg sundhedsdata fra at være et ubetinget aktiv til at være en sikkerhedsrisiko (*Radioaktivt Affald*). Her designer jeg et analytics-system, der balancerer forretningsværdi med *Privacy by Design*.

### 5. 🧭 [Vidensrejse]({{< relref "vidensrejse/_index.md" >}})
**Metoden:** Hvordan lærte jeg det?
En gennemgang af min metode: Fra ad-hoc informationssøgning til systematisk **metodisk triangulering** af kilder.

### 6. 💭 [Samlet Refleksion]({{< relref "refleksion/_index.md" >}})
**Konklusionen:** Hvad tager jeg med videre?
Evaluering af læringsmål, tværgående temaer (Privacy, TCO, Developer Experience) og min professionelle udvikling.

---

## Læsevejledning

Du kan tilgå indholdet på to måder, afhængigt af dit fokus:

### A. Den Processuelle Tilgang (Anbefalet)
Læs siderne kronologisk for at følge argumentationen og den metodiske udvikling fra problem til løsning.
`Om Projektet → Læringsmål → Database → Data Science → Vidensrejse → Refleksion`

### B. Den Resultatorienterede Tilgang
Hop direkte til konklusionerne. Hver specialiserings-side indledes med et **Executive Summary**, der opsummerer fund og beslutninger.

---

## 💡 Fokusområder undervejs

* **Evidens over Intuition:** Jeg argumenterer ud fra **konvergent evidens** (benchmarks, peer-reviewed studier, lovgivning) frem for popularitetstrends.
* **Fejl som Læringskilde:** Jeg dokumenterer eksplicit de antagelser, der blev modbevist gennem validering (jvf. V-modellen).
* **Arkitektonisk Helhed:** Jeg behandler ikke Database og Data Science som siloer, men som to sider af samme **Information Architecture**: Hvordan persisterer og anvender vi data ansvarligt?

---

### Om Mig
Jeg er Simon, 26 år. Min profil kombinerer backend-arkitektur med systematisk analyse. Dette semester har lært mig, at en dygtig udvikler ikke kun skriver kode – han designer løsninger, der kan forsvares teknisk, økonomisk og etisk.

**Start rejsen her:** [Om Projektet]({{< relref "om-projektet/_index.md" >}})