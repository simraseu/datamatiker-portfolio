---
title: "Vidensrejse"
weight: 5
description: "Min metodiske udvikling: Fra broad search til systematisk triangulering"
---

# Min Vidensrejse: Fra Kaos til Metode

Dette semester har ikke kun været en rejse ind i to tekniske specialiseringer. Det har været en øvelse i **Studentercentreret Læring**, hvor jeg har skullet lære at tilegne mig ny, kompleks viden struktureret og effektivt.

Min læringsproces var ikke lineær. Den startede med nysgerrighed, blev formet af forvirring (kaos), og endte med en skarp, gentagelig metode.

Her er historien om, hvordan jeg udviklede det framework, som danner den røde tråd i denne portfolio.

---

## Fase 1: Fundamentet (Database & Storage)

Min rejse startede her. Jeg vidste, at projektet krævede en database, men jeg stod over for et hav af buzzwords: *Microservices, Polyglot Persistence, Vector Databases, NoSQL.*

### Trin 1: Den brede scanning (Exploration)
Jeg startede bredt. Jeg læste branche-blogs, så YouTube-talks fra konferencer og kiggede på "Tech Radar" lister.
* **Målet:** At få et overblik over landskabet.
* **Erkendelsen:** Der er for mange holdninger og for lidt evidens. Alle vil sælge deres egen database.
* **Resultatet:** Jeg indså, at jeg ikke kunne vælge ud fra popularitet. Jeg var nødt til at definere mine egne kriterier.

### Trin 2: Strukturering af Læringsmål
Med det generelle overblik på plads, kunne jeg formulere mine **Læringsmål**. Jeg vidste nu, at jeg skulle undersøge:
1.  Performance (JSON vs SQL)
2.  Konsistens (ACID vs BASE)
3.  Vektor-søgning (AI-integration)

### Trin 3: Systematisk Triangulering
Her stødte jeg på muren: Jeg havde ingen brugere til at teste performance med. Hvordan kunne jeg validere mine valg?
Jeg valgte at anvende en **Triangulerings-metode** for at sikre kildekritik:
* Jeg søgte **Vendor Benchmarks** (OnGres)
* Jeg krydstjekkede med **Peer-Reviewed Forskning** (Makris et al.)
* Jeg verificerede med **Technical Documentation** (AWS/Microsoft)

Når tre uafhængige kilder pegede samme vej (mod PostgreSQL), accepterede jeg det som tilstrækkelig arkitektonisk evidens. Dette blev fundamentet for min **Unified Monolith** konklusion.

---

## Fase 2: Ekspansionen (Data Science & Analytics)

Da jeg var færdig med Database-delen, stod jeg med to ting:
1.  En solid teknisk platform (PostgreSQL med JSONB og Vectors).
2.  En **afprøvet metode** til at angribe ukendte problemer.

Jeg gik nu videre til Data Science, men denne gang startede jeg ikke fra nul.

### Trin 1: Genbrug af Metode (Efficiency)
I stedet for at starte med tilfældig googling, applicerede jeg straks min triangulerings-model. Jeg vidste, at jeg skulle lede efter **Convergent Validity** mellem:
* **Jura** (GDPR lovtekster)
* **Teori** (Goodhart's Law, Kleppmann)
* **Best Practice** (Google HEART)

Fordi jeg havde metoden på plads, kunne jeg hurtigere skære igennem støjen og identificere kerneproblemerne: *Privacy og Vanity Metrics.*

### Trin 2: Fra Teknik til Etik
Hvor Database-delen var teknisk tung (latency, disk usage), tvang Data Science mig til at udvide min horisont.
Jeg brugte min database-viden som springbræt:
* *"Hvis jeg gemmer data i JSONB (Database viden), hvordan sikrer jeg så, at det er lovligt (Data Science viden)?"*

Dette ledte til **Privacy-First Logging** (DP2). Jeg byggede videre på fundamentet, men tilføjede et lag af etisk refleksion.

---

## Fase 3: Syntese (Kolbs Læringscirkel)

Gennem processen har jeg destilleret min arbejdsmetode ned til en cyklus på 4 trin, der i praksis afspejler **Kolbs Læringscirkel** (Experiential Learning):

### 1. Hypotese (Konkret Erfaring)
*"Data er guld"* eller *"Polyglot er bedst"*. Jeg starter altid med en antagelse eller et konkret problem, jeg møder.

### 2. Triangulering (Reflekterende Observation)
Jeg tester hypotesen mod tre kilder: **Akademisk teori, Industristandarder og Officiel dokumentation.** Jeg observerer, hvor kilderne er enige eller uenige.

### 3. Design Patterns (Abstrakt Begrebsliggørelse)
Jeg omsætter min viden til normative regler/modeller (f.eks. "DP4: ACID-First"). Her danner jeg de abstrakte begreber, der skal løse problemet.

### 4. Konceptuel Validering (Aktivt Eksperiment)
Jeg tester implementeringen gennem audits (GDPR check, TCO analyse, Architecture review) for at verificere, om teorien holder i praksis.

---

## Refleksion over rejsen

Jeg startede semesteret som en udvikler, der ledte efter "det bedste værktøj".
Jeg afslutter semesteret som en arkitekt, der leder efter "den bedst underbyggede beslutning".

Den vigtigste læring er ikke PostgreSQLs JSON-performance eller SHA-256 hashing. Det er visheden om, at jeg kan navigere i ukendt teknisk og juridisk terræn ved at anvende en struktureret, deduktiv proces.

**Næste:** [Samlet Refleksion →]({{< relref "refleksion/_index.md" >}})