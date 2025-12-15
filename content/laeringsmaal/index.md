---
title: "Læringsmål"
weight: 3
description: "Mine faglige kompetencemål for Database og Data Science specialiseringerne"
---

# Mine Læringsmål

For at sikre en struktureret læringsproces har jeg defineret klare mål for mine to valgfag. Målene tager udgangspunkt i datamatiker-uddannelsens kompetencemål og afspejler min rejse fra teoretisk forståelse til konkret teknisk implementering.

---

## 1. Specialisering: Database & Storage

### Introduktion
Jeg har valgt at specialisere mig i avanceret database-teknologi for at udfordre standarden om "Polyglot Persistence". Mit fokus har været at undersøge, om en **Unified Monolith** kan reducere systemets samlede kompleksitet.

### Viden (Teorien)
* **Redegøre for** hybride datamodeller og teorien bag at konsolidere relationelle data, dokument-data (JSON) og vektorer i samme system.
* **Analysere** forskellene på ACID og BASE transaktionsmodeller, særligt i forhold til datakonsistens og GDPR-krav i distribuerede systemer.
* **Forklare** principperne bag avanceret indeksering (GIN/HNSW) i moderne databaser, og hvordan binær lagring (JSONB) adskiller sig fra tekstlagring.

### Færdigheder (Værktøjerne)
* **Anvende** container-teknologi til at konfigurere PostgreSQL i Docker og optimere den i forhold til definerede krav.
* **Implementere** semantisk søgning i praksis ved at anvende `pgvector`-udvidelsen til at lagre og søge i AI-embeddings.
* **Konstruere** performante forespørgsler mod ustrukturerede data ved brug af JSONB og GIN-indeksering.

### Kompetencer (Anvendelsen)
* **Vurdere og validere** arkitektoniske valg (Unified Monolith vs. Microservices) baseret på arkitekturanalyse, dokumenterede benchmarks og teoretiske vurderinger.
* **Argumentere fagligt** for valget af en samlet database-platform frem for en distribueret løsning i en AI-kontekst.

---

## 2. Specialisering: Data Science & Analytics

### Introduktion
Jeg har valgt Data Science & Analytics for at adressere spændingsfeltet mellem datadrevet indsigt og **Privacy by Design**. Målet er at undersøge og argumentere for, at værdiskabelse ikke behøver ske på bekostning af brugernes anonymitet.

### Viden (Teorien)
* **Redegøre for** GDPR Artikel 25 (Privacy by Design) og forstå, hvordan juridiske krav om dataminimering omsættes til softwarearkitektur.
* **Forstå og anvende** teorien bag metrikker og bias (herunder *Goodhart’s Law*) til at identificere og undgå misvisende "Vanity Metrics".
* **Redegøre for** arkitektoniske principper i en Lambda Arkitektur til håndtering af balancen mellem real-time data og historisk analyse.

### Færdigheder (Værktøjerne)
* **Udvikle** en Privacy-First pipeline i C#, der anvender kryptografiske metoder (SHA-256) til at anonymisere data før lagring.
* **Implementere** simple Intent Classification metoder til at transformere fritekst til strukturerede kategorier.
* **Opbygge** en Dual-Speed dataarkitektur (Lambda Lite), der teknisk separerer "Hot Path" (in-memory) og "Cold Path" (database) dataflows.

### Kompetencer (Anvendelsen)
* **Reflektere** over etiske dilemmaer i AI-overvågning og designe løsninger, der balancerer forretning og etik.
* **Designe** Context-Aware dashboards, der præsenterer beslutningsgrundlag (Actionable Insights) tilpasset specifikke interessenter.

---

## Evaluering
Opfyldelsen af disse mål er dokumenteret gennem mine analyser og implementeringer, hvor teorien er bearbejdet, vurderet og omsat til praksis:
* [Database & Storage Analyse]({{< relref "database/_index.md" >}})
* [Data Science & Analytics Analyse]({{< relref "data-science/_index.md" >}})