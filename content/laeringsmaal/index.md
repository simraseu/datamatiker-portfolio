---
title: "Læringsmål"
weight: 3
description: "Mine faglige mål for Database og Data Science specialiseringerne"
---

# Mine Læringsmål

For at sikre en struktureret læringsproces har jeg defineret klare mål for mine to valgfag. Målene tager udgangspunkt i datamatiker-uddannelsens kompetencemål og afspejler min rejse fra teoretisk forståelse til konkret teknisk implementering.

---

## 1. Specialisering: Database & Storage

### Introduktion
Jeg har valgt at specialisere mig i avanceret database-teknologi, fordi moderne applikationer kræver mere end bare standard SQL-lagring.
Mit overordnede mål har været at undersøge, om man kan samle flere datatyper i én arkitektur for at undgå kompleksiteten ved "Polyglot Persistence".

### Viden (Teorien)
* Jeg vil opnå dybdegående viden om **hybride datamodeller** og forstå teorien bag at blande relationelle data, dokument-data (JSON) og vektorer i samme system.
* Jeg vil forstå de fundamentale forskelle på **ACID og BASE** transaktionsmodeller, særligt i forhold til datakonsistens og GDPR-krav i distribuerede systemer.
* Jeg vil forstå principperne bag **avanceret indeksering** i moderne databaser, og hvordan binær lagring (som JSONB) adskiller sig fra ren tekstlagring.

### Færdigheder (Værktøjerne)
* Jeg skal kunne opsætte og konfigurere **PostgreSQL i Docker** og optimere den til produktionslignende workloads.
* Jeg vil kunne implementere semantisk søgning i praksis ved at anvende **pgvector-udvidelsen** til at lagre og søge i AI-embeddings.
* Jeg vil kunne skrive performante forespørgsler mod ustrukturerede data ved at udnytte PostgreSQLs **JSONB-format** og GIN-indekser.

### Kompetencer (Anvendelsen)
* Jeg vil være i stand til selvstændigt at **validere arkitektoniske valg** (Unified Monolith vs. Microservices) baseret på konkrete målinger af latency og driftsomkostninger.
* Jeg vil kunne argumentere fagligt for valget af en samlet database-platform frem for en distribueret løsning i en AI-kontekst.

---

## 2. Specialisering: Data Science & Analytics

### Introduktion
Jeg har valgt Data Science & Analytics for at løse dilemmaet mellem behovet for indsigt og retten til privatliv.
Mit mål er at bevise, at man kan skabe værdi gennem dataanalyse uden at gå på kompromis med brugernes anonymitet eller lovgivningen.

### Viden (Teorien)
* Jeg vil have indgående kendskab til **GDPR Artikel 25 (Privacy by Design)** og forstå, hvordan juridiske krav om dataminimering kan oversættes til softwarearkitektur.
* Jeg vil forstå teorien bag **metrikker og bias**, herunder *Goodhart’s Law*, for at kunne identificere og undgå misvisende "Vanity Metrics".
* Jeg vil forstå de arkitektoniske principper i en **Lambda Arkitektur** for at kunne håndtere balancen mellem real-time data og historisk analyse.

### Færdigheder (Værktøjerne)
* Jeg skal kunne udvikle en **Privacy-First pipeline i C#**, der anvender kryptografiske metoder (som **SHA-256**) til at anonymisere data før lagring.
* Jeg vil kunne implementere **Intent Classification** algoritmer til at transformere fritekst til strukturerede kategorier.
* Jeg vil kunne opbygge en **Dual-Speed dataarkitektur** (Lambda Lite), der teknisk separerer "Hot Path" (in-memory) og "Cold Path" (database) dataflows.

### Kompetencer (Anvendelsen)
* Jeg vil kunne reflektere over de **etiske dilemmaer** i AI-overvågning og designe løsninger, der balancerer forretning og etik.
* Jeg vil være i stand til at designe **Context-Aware dashboards**, der præsenterer beslutningsgrundlag (Actionable Insights) tilpasset specifikke interessenter i et projekt.

---

## Evaluering
Opfyldelsen af disse mål er dokumenteret gennem mine analyser og implementeringer, hvor teorien er omsat til praksis:
* [Database & Storage Analyse]({{< relref "database/_index.md" >}})
* [Data Science & Analytics Analyse]({{< relref "data-science/_index.md" >}})